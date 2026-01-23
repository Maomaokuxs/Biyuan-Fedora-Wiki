# 1.整体思路

在$变量中添加命令所在的文件夹。

# 2.查看$PATH变量

```bash
echo $PATH
```

# 3.临时添加命令所在目录

```bash
PATH = $PATH:/usr/sbin
```
重启后将失效
# 4.永久添加命令所在目录
在$HOME/.bashrc文件中添加

```bash
export PATH=$PATH:/usr/sbin
```
