```
(gdb) bt
#0  tty_read (file=0x63008600, buf=0x7fbff59859 <error: Cannot access memory at address 0x7fbff59859>, count=1, ppos=0x63277e58)
    at drivers/tty/tty_io.c:839
#1  0x00000000600d89d3 in __vfs_read (file=0x63008600, buf=<optimized out>, count=<optimized out>, pos=0x63277e58)
    at fs/read_write.c:411
#2  0x00000000600d8bcf in vfs_read (file=0x63008600, buf=0x7fbff59859 <error: Cannot access memory at address 0x7fbff59859>, 
    count=<optimized out>, pos=0x63277e58) at fs/read_write.c:447
#3  0x00000000600d9273 in SYSC_read (count=<optimized out>, buf=<optimized out>, fd=<optimized out>) at fs/read_write.c:573
#4  SyS_read (fd=<optimized out>, buf=548681390169, count=1) at fs/read_write.c:566
#5  0x000000006001c298 in handle_syscall (r=0x630d3b90) at arch/um/kernel/skas/syscall.c:32
#6  0x000000006002bedc in handle_trap (local_using_sysemu=<optimized out>, regs=<optimized out>, pid=<optimized out>)
    at arch/um/os-Linux/skas/process.c:172
#7  userspace (regs=0x630d3b90, aux_fp_regs=<optimized out>) at arch/um/os-Linux/skas/process.c:416
#8  0x0000000060018534 in fork_handler () at arch/um/kernel/process.c:153
#9  0x0000000000000000 in ?? ()
```