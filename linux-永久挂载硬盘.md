
- 创建挂载点

```bash
sudo mkdir /disk/
```

- 编辑/etc/fstab文件

```bash
vim /etc/fstab
```

- 添加文本

```
 <file system> <mount point> <type> <optinons> <dump> <pass>
 UUID=   /    btrfs  defaults o  o
```

- 重启

- 特殊

	如果在window启动中开启了快速启动，会导致该磁盘不能够执行写入操作，需要在windows中关闭快速启动并重启进入windows系统一次。
	