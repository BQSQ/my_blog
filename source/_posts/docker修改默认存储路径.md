---
title: docker修改默认存储路径
date: 2018-01-27 20:01:58
tags: [docker,默认路径]
categories: docker
keywords: [docker,默认路径,修改默认路径]
---


服务器环境：Linux app-test 3.10.0-693.11.6.el7.x86_64 #1 SMP Thu Jan 4 01:06:37 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux   
安装命令：$ yum install -y docker   
安装docker后镜像存储默认地址：/var/lib/docker，这样会让/挂载点空间占用越来越大，同时我们也需要将文件放在我们自己定义的位置，并且空间足够大。   

<!-- more -->

* 停止 Docker 服务
```
$ service docker stop
```

* 将原来默认的/var/lib/docker备份一下，然后复制到别的位置并建立一个软链接
```
$ cd /var/lib
$ cp -rf docker docker.bak
$ mv -f docker /<my_new_location>/
$ ln -s /<my_new_location>/docker docker
```

* 启动 Docker 服务
```
$ service docker start
```

* 最后使用 docker info 查看更新结果
```
[root@app-test ~]# docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 1.12.6
Storage Driver: devicemapper
 Pool Name: docker-8:6-40586-pool
 Pool Blocksize: 65.54 kB
 Base Device Size: 10.74 GB
 Backing Filesystem: xfs
 Data file: /dev/loop0
 Metadata file: /dev/loop1
 Data Space Used: 11.8 MB
 Data Space Total: 107.4 GB
 Data Space Available: 107.4 GB
 Metadata Space Used: 581.6 kB
 Metadata Space Total: 2.147 GB
 Metadata Space Available: 2.147 GB
 Thin Pool Minimum Free Space: 10.74 GB
 Udev Sync Supported: true
 Deferred Removal Enabled: true
 Deferred Deletion Enabled: true
 Deferred Deleted Device Count: 0
 Data loop file: /app/docker/docker/devicemapper/devicemapper/data
 WARNING: Usage of loopback devices is strongly discouraged for production use.                                                               Use `--storage-opt dm.thinpooldev` to specify a custom block storage device.
 Metadata loop file: /app/docker/docker/devicemapper/devicemapper/metadata
 Library Version: 1.02.140-RHEL7 (2017-05-03)
Logging Driver: journald
Cgroup Driver: systemd
Plugins:
 Volume: local
 Network: host null bridge overlay
Swarm: inactive
Runtimes: docker-runc runc
Default Runtime: docker-runc
Security Options: seccomp selinux
Kernel Version: 3.10.0-693.11.6.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
Number of Docker Hooks: 3
CPUs: 12
Total Memory: 15.47 GiB
Name: app-test
ID: W67H:APQX:PC6P:45BF:KASR:422V:5I4K:SZFD:AXR6:U4RF:FYKA:OEQ7
Docker Root Dir: /app/docker/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Insecure Registries:
 127.0.0.0/8
Registries: docker.io (secure)
```

