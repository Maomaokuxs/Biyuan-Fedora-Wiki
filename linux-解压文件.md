
- 基本解压命令
	- 解压到当前目录

```bash
tar -xzf filename.tar.gz
```

- 解压到指定目录

```bash
tar -xzf filename.tar.gz -C /path/to/directory
```

- 查看压缩包内容（不解压）

```bash
tar -tzf filename.tar.gz
```

- 详细显示解压过程

```bash
tar -xvzf filename.tar.gz
```

- 参数说明：

    -x：提取文件

    -z：通过gzip过滤归档

    -v：显示详细过程

    -f：指定归档文件名

    -C：指定解压目录

    -t：列出归档内容