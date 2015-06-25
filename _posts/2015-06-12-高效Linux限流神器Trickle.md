---
layout: post
title:  "高效的Liunx限流神器Trickle"
keywords: "liunx,trickle"
description: "Trickle 是个非常小巧实用的 Linux 命令限流工具，Trickle 可以限制 Linux 命令行工具的上传和下载流量。在跨地域文件传输或者备份时非常有用，因为外网带宽往往会比较贵。
或者你想备份进程或者下载进程不对同机器的其他服务产生影响，也需要 Trickle 这样的限流工具。
再或者你在办公室想下载大文件，不希望影响其他网络用户或者应用，Trickle 就是为此设计。"
category: linux shell
tags: [linux]
---
#### 什么是Trickle，它能帮助我们干什么
Trickle 是个非常小巧实用的Linux命令限流工具，Trickle 可以限制 Linux 命令行工具的上传和下载流量。在跨地域文件传输或者备份时非常有用，因为外网带宽往往会比较贵。
或者你想备份进程或者下载进程不对同机器的其他服务产生影响，也需要 Trickle 这样的限流工具。
再或者你在办公室想下载大文件，不希望影响其他网络用户或者应用，Trickle 就是为此设计。
#### Trickle的使用非常简单
只需要把 trickle 和相关参数附加在其他 Linux 命令行工具命令之前即可

比如限制 Wget 下载文件的速度为 20KB/S

```
trickle -s -d 20 wget -c http://blog.jack.com/
```

限制文件备份到 AWS S3 的上传速度为 1MB/S

```
trickle -s -u 1024 s3cmd sync /mnt/data/ s3://my-backu
```
当然，你也可以把trickle加在现有的服务器自动化脚本中完成限流功能

#### 其他功能

trickle -L 设置延迟为多少 ms

trickle -w 设置窗口长度为多少 KB