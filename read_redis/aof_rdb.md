Redis 协议
```
https://redis.io/topics/protocol
```

## 执行流程
get 命令的主要执行步骤

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
