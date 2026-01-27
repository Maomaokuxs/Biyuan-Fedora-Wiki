
# 说明

目的是使桌面环境中使用中文，而在 tty 中不会因为不支持中文而带方块字，不能够识别。

- 更改文件`/etc/locale.conf`

```bash
sudo vim /etc/locale.conf
```

- 以下仅为例子按需更改

```text
LANG="en_US.UTF-8"
LANGUAGE="zh_CN:cn"
LC_ALL="en_US.UTF-8"
```
