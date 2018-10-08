```
https://yq.aliyun.com/articles/617950
https://www.ibm.com/developerworks/cn/linux/l-cn-ftrace/index.html
http://oenhan.com/linux-kernel-write
https://www.ibm.com/developerworks/cn/linux/l-linux-filesystem/index.html
```
> 主要的函数指针
```
const struct file_operations ext4_file_operations = {
	.llseek		= ext4_llseek,
	.read_iter	= ext4_file_read_iter,
	.write_iter	= ext4_file_write_iter,
	.unlocked_ioctl = ext4_ioctl,
#ifdef CONFIG_COMPAT
	.compat_ioctl	= ext4_compat_ioctl,
#endif
	.mmap		= ext4_file_mmap,
	.mmap_supported_flags = MAP_SYNC,
	.open		= ext4_file_open,
	.release	= ext4_release_file,
	.fsync		= ext4_sync_file,
	.get_unmapped_area = thp_get_unmapped_area,
	.splice_read	= generic_file_splice_read,
	.splice_write	= iter_file_splice_write,
	.fallocate	= ext4_fallocate,
};

static const struct address_space_operations ext4_aops = {
	.readpage		= ext4_readpage,
	.readpages		= ext4_readpages,
	.writepage		= ext4_writepage,
	.writepages		= ext4_writepages,
	.write_begin		= ext4_write_begin,
	.write_end		= ext4_write_end,
	.set_page_dirty		= ext4_set_page_dirty,
	.bmap			= ext4_bmap,
	.invalidatepage		= ext4_invalidatepage,
	.releasepage		= ext4_releasepage,
	.direct_IO		= ext4_direct_IO,
	.migratepage		= buffer_migrate_page,
	.is_partially_uptodate  = block_is_partially_uptodate,
	.error_remove_page	= generic_error_remove_page,
};
```


> read   
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

```
// D:\linux-master\fs\read_write.c
ssize_t __vfs_read(struct file *file, char __user *buf, size_t count,
		   loff_t *pos)
{
	if (file->f_op->read)   // 没有read
		return file->f_op->read(file, buf, count, pos);
	else if (file->f_op->read_iter)    // 有read_iter
		return new_sync_read(file, buf, count, pos);   // 调用new_sync_read
	else
		return -EINVAL;
}
```
```
//todo
```

> write 

- 堆栈  
```
(gdb) bt
#0  generic_perform_write (file=0x6268d400, i=0x6268fd90, pos=6) at mm/filemap.c:3099
#1  0x000000006008c710 in __generic_file_write_iter (iocb=0x6268fd60, from=<optimized out>) at mm/filemap.c:3263
#2  0x0000000060163e8a in ext4_file_write_iter (iocb=0x6268fd60, from=0x6268fd90) at fs/ext4/file.c:266
#3  0x00000000600d8ddc in call_write_iter (iter=<optimized out>, kio=<optimized out>, file=<optimized out>) at ./include/linux/fs.h:1781
#4  new_sync_write (ppos=<optimized out>, len=<optimized out>, buf=<optimized out>, filp=<optimized out>) at fs/read_write.c:469
#5  __vfs_write (file=0x6268d400, p=<optimized out>, count=<optimized out>, pos=0x6268fe58) at fs/read_write.c:482
#6  0x00000000600d9061 in vfs_write (file=0x6268d400, buf=0x8872e0 <error: Cannot access memory at address 0x8872e0>, count=3, pos=0x6268fe58) at fs/read_write.c:544
#7  0x00000000600d9343 in SYSC_write (count=<optimized out>, buf=<optimized out>, fd=<optimized out>) at fs/read_write.c:589
#8  SyS_write (fd=<optimized out>, buf=8942304, count=3) at fs/read_write.c:581
#9  0x000000006001c298 in handle_syscall (r=0x624d3b90) at arch/um/kernel/skas/syscall.c:32
#10 0x000000006002bedc in handle_trap (local_using_sysemu=<optimized out>, regs=<optimized out>, pid=<optimized out>) at arch/um/os-Linux/skas/process.c:172
#11 userspace (regs=0x624d3b90, aux_fp_regs=<optimized out>) at arch/um/os-Linux/skas/process.c:416
#12 0x0000000060018534 in fork_handler () at arch/um/kernel/process.c:153
#13 0x0000000000000002 in ?? ()
#14 0x0000000000000000 in ?? ()

```
和read 类似也是经过vfs 之后自己调用write或者write_iter
```
ssize_t __vfs_write(struct file *file, const char __user *p, size_t count,
		    loff_t *pos)
{
	if (file->f_op->write)
		return file->f_op->write(file, p, count, pos);
	else if (file->f_op->write_iter)
		return new_sync_write(file, p, count, pos);
	else
		return -EINVAL;
}
```


调用`new_sync_write` 最后调用的是 `call_write_iter` 也就是`ext4_file_write_iter`
```
static inline ssize_t call_write_iter(struct file *file, struct kiocb *kio,
				      struct iov_iter *iter)
{
	return file->f_op->write_iter(kio, iter);
}
static ssize_t new_sync_write(struct file *filp, const char __user *buf, size_t len, loff_t *ppos)
{
...
	ret = call_write_iter(filp, &kiocb, &iter);
	if (ret > 0)
		*ppos = kiocb.ki_pos;
	return ret;
}
```

下面我们来主要看看`ext4_file_write_iter`



```
ssize_t generic_perform_write(struct file *file,
				struct iov_iter *i, loff_t pos)
{
...
		status = a_ops->write_begin(file, mapping, pos, bytes, flags,
						&page, &fsdata);
		if (unlikely(status < 0))
			break;

		if (mapping_writably_mapped(mapping))
			flush_dcache_page(page);

		copied = iov_iter_copy_from_user_atomic(page, i, offset, bytes);
		flush_dcache_page(page);

		status = a_ops->write_end(file, mapping, pos, bytes, copied,
						page, fsdata);
...
}
```
