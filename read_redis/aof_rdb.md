Redis 协议
```
https://redis.io/topics/protocol
```
协议翻译
```
http://redisdoc.com/topic/protocol.html
```

rdb 和 aof 介绍翻译
```
http://www.redis.cn/topics/persistence.html
```
## RDB的优点
RDB是一个非常紧凑的文件,它保存了某个时间点得数据集,非常适用于数据集的备份,比如你可以在每个小时报保存一下过去24小时内的数据,同时每天保存过去30天的数据,这样即使出了问题你也可以根据需求恢复到不同版本的数据集.
RDB是一个紧凑的单一文件,很方便传送到另一个远端数据中心或者亚马逊的S3（可能加密），非常适用于灾难恢复.
RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以RDB持久化方式可以最大化redis的性能.
与AOF相比,在恢复大的数据集的时候，RDB方式会更快一些.
## RDB的缺点
如果你希望在redis意外停止工作（例如电源中断）的情况下丢失的数据最少的话，那么RDB不适合你.虽然你可以配置不同的save时间点(例如每隔5分钟并且对数据集有100个写的操作),是Redis要完整的保存整个数据集是一个比较繁重的工作,你通常会每隔5分钟或者更久做一次完整的保存,万一在Redis意外宕机,你可能会丢失几分钟的数据.
RDB 需要经常fork子进程来保存数据集到硬盘上,当数据集比较大的时候,fork的过程是非常耗时的,可能会导致Redis在一些毫秒级内不能响应客户端的请求.如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续1秒,AOF也需要fork,但是你可以调节重写日志文件的频率来提高数据集的耐久度.
## AOF 优点
使用AOF 会让你的Redis更加耐久: 你可以使用不同的fsync策略：无fsync,每秒fsync,每次写的时候fsync.使用默认的每秒fsync策略,Redis的性能依然很好(fsync是由后台线程进行处理的,主线程会尽力处理客户端请求),一旦出现故障，你最多丢失1秒的数据.
AOF文件是一个只进行追加的日志文件,所以不需要写入seek,即使由于某些原因(磁盘空间已满，写的过程中宕机等等)未执行完整的写入命令,你也也可使用redis-check-aof工具修复这些问题.
Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子， 如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态。
## AOF 缺点
对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。 在一般情况下， 每秒 fsync 的性能依然非常高， 而关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。
## 执行流程
get 命令的主要执行步骤
![aaa](https://raw.githubusercontent.com/769344359/read_redis/master/read_redis/redis_get.svg?sanitize=true)
> 通过hash(key) 判断是不是在桶内
```
(gdb) bt
#0  dictFind (d=0x7fe373818360, key=0x7fe37392fefb) at dict.c:484
#1  0x00000000004478f1 in lookupKey (db=0x7fe3738f8000, key=0x7fe37392fee8, flags=0) at db.c:54
#2  0x0000000000447a10 in lookupKeyReadWithFlags (db=0x7fe3738f8000, key=0x7fe37392fee8, flags=0) at db.c:127
#3  0x0000000000447a6f in lookupKeyRead (db=0x7fe3738f8000, key=0x7fe37392fee8) at db.c:138
#4  0x0000000000447ad9 in lookupKeyReadOrReply (c=0x7fe37391e140, key=0x7fe37392fee8, reply=0x7fe373817940) at db.c:152
#5  0x0000000000456f83 in getGenericCommand (c=0x7fe37391e140) at t_string.c:160
#6  0x0000000000456ff2 in getCommand (c=0x7fe37391e140) at t_string.c:173
#7  0x000000000042f630 in call (c=0x7fe37391e140, flags=15) at server.c:2229
#8  0x00000000004301b5 in processCommand (c=0x7fe37391e140) at server.c:2510
#9  0x000000000044075e in processInputBuffer (c=0x7fe37391e140) at networking.c:1354
#10 0x0000000000440b31 in readQueryFromClient (el=0x7fe37383a0f0, fd=8, privdata=0x7fe37391e140, mask=1) at networking.c:1444
#11 0x0000000000426d88 in aeProcessEvents (eventLoop=0x7fe37383a0f0, flags=11) at ae.c:443
#12 0x0000000000426fa2 in aeMain (eventLoop=0x7fe37383a0f0) at ae.c:501
#13 0x0000000000433dfc in main (argc=1, argv=0x7fffc866ac88) at server.c:3894

```

函数是 `dictSdsKeyCompare`

```
p d->type->keyCompare 
$18 = (int (*)(void *, const void *, const void *)) 0x42b81c <dictSdsKeyCompare>
```

> 返回
堆栈
```
(gdb) bt
#0  addReply (c=0x7fe37391e140, obj=0x7fe37386fd40) at networking.c:311
#1  0x000000000043e837 in addReplyBulkLen (c=0x7fe37391e140, obj=0x7fe373817cf0) at networking.c:549
#2  0x000000000043e877 in addReplyBulk (c=0x7fe37391e140, obj=0x7fe373817cf0) at networking.c:556
#3  0x0000000000456fd3 in getGenericCommand (c=0x7fe37391e140) at t_string.c:167
#4  0x0000000000456ff2 in getCommand (c=0x7fe37391e140) at t_string.c:173
#5  0x000000000042f630 in call (c=0x7fe37391e140, flags=15) at server.c:2229
#6  0x00000000004301b5 in processCommand (c=0x7fe37391e140) at server.c:2510
#7  0x000000000044075e in processInputBuffer (c=0x7fe37391e140) at networking.c:1354
#8  0x0000000000440b31 in readQueryFromClient (el=0x7fe37383a0f0, fd=8, privdata=0x7fe37391e140, mask=1) at networking.c:1444
#9  0x0000000000426d88 in aeProcessEvents (eventLoop=0x7fe37383a0f0, flags=11) at ae.c:443
#10 0x0000000000426fa2 in aeMain (eventLoop=0x7fe37383a0f0) at ae.c:501
#11 0x0000000000433dfc in main (argc=1, argv=0x7fffc866ac88) at server.c:3894
```
对内容进行格式化
```
void addReplyBulk(client *c, robj *obj) {
    addReplyBulkLen(c,obj);
    addReply(c,obj);
    addReply(c,shared.crlf);
}
```

> 写的时候 
添加到缓冲区
```
(gdb) bt
#0  dictAdd (d=0x7ff35c218360, key=0x7ff35c216779, val=0x7ff35c32fe58) at dict.c:267
#1  0x0000000000447b8e in dbAdd (db=0x7ff35c2ff000, key=0x7ff35c32fe88, val=0x7ff35c32fe58) at db.c:169
#2  0x0000000000447d31 in setKey (db=0x7ff35c2ff000, key=0x7ff35c32fe88, val=0x7ff35c32fe58) at db.c:208
#3  0x0000000000456a44 in setGenericCommand (c=0x7ff35c31eec0, flags=0, key=0x7ff35c32fe88, val=0x7ff35c32fe58, expire=0x0, unit=0, ok_reply=0x0, abort_reply=0x0) at t_string.c:86
#4  0x0000000000456dad in setCommand (c=0x7ff35c31eec0) at t_string.c:139
#5  0x000000000042f630 in call (c=0x7ff35c31eec0, flags=15) at server.c:2229
#6  0x00000000004301b5 in processCommand (c=0x7ff35c31eec0) at server.c:2510
#7  0x000000000044075e in processInputBuffer (c=0x7ff35c31eec0) at networking.c:1354
#8  0x0000000000440b31 in readQueryFromClient (el=0x7ff35c2410a0, fd=8, privdata=0x7ff35c31eec0, mask=1) at networking.c:1444
#9  0x0000000000426d88 in aeProcessEvents (eventLoop=0x7ff35c2410a0, flags=11) at ae.c:443
#10 0x0000000000426fa2 in aeMain (eventLoop=0x7ff35c2410a0) at ae.c:501
#11 0x0000000000433dfc in main (argc=1, argv=0x7ffddbfa41a8) at server.c:3894

```
真正可写的时候 ，调用write 写到socket
```
(gdb) bt
#0  write () at ../sysdeps/unix/syscall-template.S:84
#1  0x000000000043f545 in writeToClient (fd=8, c=0x7ff35c31eec0, handler_installed=0) at networking.c:905
#2  0x000000000043f8fe in handleClientsWithPendingWrites () at networking.c:1012
#3  0x000000000042cf00 in beforeSleep (eventLoop=0x7ff35c2410a0) at server.c:1232
#4  0x0000000000426f91 in aeMain (eventLoop=0x7ff35c2410a0) at ae.c:500
#5  0x0000000000433dfc in main (argc=1, argv=0x7ffddbfa41a8) at server.c:3894

```



<del>写的时候的aof堆栈</del>
```
(gdb) bt
#0  expireIfNeeded (db=0x7ff35c2ff000, key=0x7ff35c32fe88) at db.c:1117
#1  0x0000000000447a94 in lookupKeyWrite (db=0x7ff35c2ff000, key=0x7ff35c32fe88) at db.c:147
#2  0x0000000000447d15 in setKey (db=0x7ff35c2ff000, key=0x7ff35c32fe88, val=0x7ff35c32fea0) at db.c:207
#3  0x0000000000456a44 in setGenericCommand (c=0x7ff35c31eec0, flags=0, key=0x7ff35c32fe88, val=0x7ff35c32fea0, expire=0x0, unit=0, ok_reply=0x0, abort_reply=0x0) at t_string.c:86
#4  0x0000000000456dad in setCommand (c=0x7ff35c31eec0) at t_string.c:139
#5  0x000000000042f630 in call (c=0x7ff35c31eec0, flags=15) at server.c:2229
#6  0x00000000004301b5 in processCommand (c=0x7ff35c31eec0) at server.c:2510
#7  0x000000000044075e in processInputBuffer (c=0x7ff35c31eec0) at networking.c:1354
#8  0x0000000000440b31 in readQueryFromClient (el=0x7ff35c2410a0, fd=8, privdata=0x7ff35c31eec0, mask=1) at networking.c:1444
#9  0x0000000000426d88 in aeProcessEvents (eventLoop=0x7ff35c2410a0, flags=11) at ae.c:443
#10 0x0000000000426fa2 in aeMain (eventLoop=0x7ff35c2410a0) at ae.c:501
#11 0x0000000000433dfc in main (argc=1, argv=0x7ffddbfa41a8) at server.c:3894

```


aof　添加到buf
```
(gdb) bt
#0  feedAppendOnlyFile (cmd=0x93b7c0 <redisCommandTable+160>, dictid=0, 
    argv=0x7f035b421dd8, argc=3) at aof.c:555
#1  0x000000000042f39e in propagate (cmd=0x93b7c0 <redisCommandTable+160>, 
    dbid=0, argv=0x7f035b421dd8, argc=3, flags=3) at server.c:2112
#2  0x000000000042f8fe in call (c=0x7f035b51e700, flags=15) at server.c:2291
#3  0x00000000004301b5 in processCommand (c=0x7f035b51e700) at server.c:2510
#4  0x000000000044075e in processInputBuffer (c=0x7f035b51e700)
    at networking.c:1354
#5  0x0000000000440b31 in readQueryFromClient (el=0x7f035b43a140, fd=9, 
    privdata=0x7f035b51e700, mask=1) at networking.c:1444
#6  0x0000000000426d88 in aeProcessEvents (eventLoop=0x7f035b43a140, flags=11)
    at ae.c:443
#7  0x0000000000426fa2 in aeMain (eventLoop=0x7f035b43a140) at ae.c:501
#8  0x0000000000433dfc in main (argc=2, argv=0x7fff24014948) at server.c:3894
```

fsync 同步,set　的主线程发送“信号”
```
(gdb) bt
#0  bioCreateBackgroundJob (type=1, arg1=0x8, arg2=0x0, arg3=0x0) at bio.c:132
#1  0x0000000000473089 in aof_background_fsync (fd=8) at aof.c:203
#2  0x0000000000473911 in flushAppendOnlyFile (force=0) at aof.c:488
#3  0x000000000042cefb in beforeSleep (eventLoop=0x7f035b43a140) at server.c:1229
#4  0x0000000000426f91 in aeMain (eventLoop=0x7f035b43a140) at ae.c:500
#5  0x0000000000433dfc in main (argc=2, argv=0x7fff24014948) at server.c:3894
```

作为同步的线程
```
(gdb) bt
#0  fdatasync () at ../sysdeps/unix/syscall-template.S:84
#1  0x000000000049392a in bioProcessBackgroundJobs (arg=0x1) at bio.c:190
#2  0x00007f035c2346ba in start_thread (arg=0x7f035abfe700) at pthread_create.c:333
#3  0x00007f035bf6a41d in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:109
```

加载aof文件
```
(gdb) bt
#0  loadAppendOnlyFile (filename=0x7ffff6817010 "appendonly.aof") at aof.c:673
#1  0x000000000043305c in loadDataFromDisk () at server.c:3554
#2  0x0000000000433d0d in main (argc=2, argv=0x7fffffffe518) at server.c:3870
```




--- 
分割线

> rdb 加载
```
(gdb) bt
#0  rdbLoad (filename=0x7ffff6817000 "dump.rdb", rsi=0x7fffffffe370) at rdb.c:1677
#1  0x000000000043310f in loadDataFromDisk () at server.c:3558
#2  0x0000000000433d0d in main (argc=1, argv=0x7fffffffe528) at server.c:3870
```

> 手动调用命令`save`
```
(gdb) bt
#0  rdbSave (filename=0x7ffff6817000 "dump.rdb", rsi=0x0) at rdb.c:1003
#1  0x000000000045655e in saveCommand (c=0x7ffff691ed00) at rdb.c:2004
#2  0x000000000042f630 in call (c=0x7ffff691ed00, flags=15) at server.c:2229
#3  0x00000000004301b5 in processCommand (c=0x7ffff691ed00) at server.c:2510
#4  0x000000000044075e in processInputBuffer (c=0x7ffff691ed00) at networking.c:1354
#5  0x0000000000440b31 in readQueryFromClient (el=0x7ffff683c0a0, fd=8, privdata=0x7ffff691ed00, mask=1) at networking.c:1444
#6  0x0000000000426d88 in aeProcessEvents (eventLoop=0x7ffff683c0a0, flags=11) at ae.c:443
#7  0x0000000000426fa2 in aeMain (eventLoop=0x7ffff683c0a0) at ae.c:501
#8  0x0000000000433dfc in main (argc=1, argv=0x7fffffffe528) at server.c:3894
```
