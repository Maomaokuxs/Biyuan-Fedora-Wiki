# 说明

因为我在看论坛或者评论区的时候有一些问题是因为设置时区有漏洞，导致不能够安装系统，那么就可以不先设置时区，在安装完之后再重新更改。

不只是可以在 fedora 上使用，使用了systemd管理系统服务，一般都能适用。

## 1.查看当前设置

```bash
timedatectl
```

## 2.列出所有可用的时区

```bash
timedatectl list-timezones

# 使用grep 筛选
timedatectl list-timezones | grep Shanghai
```

## 3.设置新时区 使用以下命令修改（需要管理员权限）：

```bash
sudo timedatectl set-timezone Asia/Shanghai
```

将 Asia/Shanghai 替换为你需要的时区即可。
