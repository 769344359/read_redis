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