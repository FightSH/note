# Docker名词概念

    镜像images：


​    

# 底层原理
---

#####     Docker是怎么工作的？
    Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上。通过Socket从客户端访问。
    DockerServer接收到Docker-Client的指令，就会执行这个命令


# Docker常用命令
### 帮助命令

```shell
docker version #显示docker版本信息
docker info  #显示docker系统信息
docker --help # 万能帮助命令
docker 文档：docs.docker.com/reference

```

### 镜像命令
####    docker images

```SH
[root@shenhao /]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
hello-world   latest    d1165f221234   2 weeks ago   13.3kB
redis         latest    f877e80bb9ef   3 weeks ago   105MB
mysql         latest    8457e9155715   3 weeks ago   546MB

# docker images可选项
docker -a,--all   # 列出所有镜像
docker -q,--quiet # 只显示镜像id
```

####    docker search
```shell
[root@shenhao /]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10665     [OK]       
mariadb                           MariaDB Server is a high performing open sou…   3997      [OK]   

# 可选项
--filter=STARS=3000  # 搜索出的镜像的stars大于3000
```

#### docker pull
```shell
[root@shenhao /]# docker pull --help

Usage:  docker pull [OPTIONS] NAME[:TAG|@DIGEST]

Pull an image or a repository from a registry

Options:
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
      --platform string         Set platform if server is multi-platform capable
  -q, --quiet                   Suppress verbose output
  
不指定版本，默认使用latest
docker使用分层下载，可以极大节省内存，共用层就不需要下载
```

#### docker rmi 
```shell
[root@shenhao /]# docker rmi --help

Usage:  docker rmi [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Options:
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents
      
      
docker rmi -f $(docker images -aq) # 强制删除所有镜像
   
docker rmi -f 容器id 容器id    
```


​    

### 容器命令

**说明：有了镜像才可以创建容器**
    
    docker pull centos

#### docker run
```shell
docker run [可选参数] image
# 参数说明
-- name-"Name"   指定容器名字 ，以区分容器
-d      后台方式运行
-it     使用交互方式运行，进入容器查看内容
-p      指定容器的端口   端口暴露概念
-p 		ip:端口  
-p		主机端口:容器端口  常用
-p 		容器端口
-P      随机指定端口

# 启动并进入容器，内部容器id即为镜像id
 [root@shenhao /]# docker run -it centos /bin/bash
 [root@15813679c0c3 /]# ls
 bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

```




#### docker ps
```shell
列出正在运行的命令
[root@shenhao /]# docker ps --help

Usage:  docker ps [OPTIONS]

List containers

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display container IDs
  -s, --size            Display total file sizes

-a  #列出当前正在运行的命令+带出历史运行过的容器
-n=?  #显示最近创建的容器
-q      #只显示容器编号
```

#### 退出
```shell
exit    # 直接停止容器并推出
ctrl + P + Q     # 容器不停止退出
```

#### 删除
```shell
docker rm 容器id                  # 删除指定容器，不能删除正在运行中的容器 
docker rm -f $(docker ps -ap)     # 删除所用容器 
docker ps -a -q|xargs docker rm   # 删除所有容器
```

#### 启动和停止容器
```shell
docker start    # 启动
docker restart  # 重启容器
docker stop     #停止
docker kill     # 强制停止
```

### 常用其他命令
#### 后台启动容器
```shell
[root@shenhao ~]# docker run -d centos
7edf4e37e20d47e474ba6c46de05574d42e53c457a18b65bd72fdc0d2fe5a49e
[root@shenhao ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```


​    
​    # 常见的坑，docker容器使用后台运行，就必须要有一个前台进程，前台没有应用，就会自动停止

#### 查看日志
```shell
[root@shenhao ~]# docker logs --help
Usage:  docker logs [OPTIONS] CONTAINER
Fetch the logs of a container
Options:
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
      --since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)
  -n, --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
      --until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)
      
docker logs -f -t --tail 容器id
# 显示日志
-t          # 显示时间戳
-f          # 输出
--tail num  # 显示的日志条数
```

#### 显示容器内部进程信息
```SHELL
docker top
[root@shenhao ~]# docker top eac419b56757
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                6413                6375                0                   17:02               ?                   00:00:00            /bin/sh -c while true;do echo 666;sleep 1;done
root                6889                6413                0                   17:07               ?                   00:00:00            /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
```

#### 查看镜像元数据
```shell
[root@shenhao ~]# docker inspect --help

Usage:  docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Return low-level information on Docker objects

Options:
  -f, --format string   Format the output using the given Go template
  -s, --size            Display total file sizes if the type is container
      --type string     Return JSON for specified type
      
  [root@shenhao ~]# docker inspect eac419b56757
    [
        {
            "Id": "eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c",
            "Created": "2021-03-27T09:02:48.336698733Z",
            "Path": "/bin/sh",
            "Args": [
                "-c",
                "while true;do echo 666;sleep 1;done"
            ],
            "State": {
                "Status": "running",
                "Running": true,
                "Paused": false,
                "Restarting": false,
                "OOMKilled": false,
                "Dead": false,
                "Pid": 6413,
                "ExitCode": 0,
                "Error": "",
                "StartedAt": "2021-03-27T09:02:48.755176558Z",
                "FinishedAt": "0001-01-01T00:00:00Z"
            },
            "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
            "ResolvConfPath": "/var/lib/docker/containers/eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c/resolv.conf",
            "HostnamePath": "/var/lib/docker/containers/eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c/hostname",
            "HostsPath": "/var/lib/docker/containers/eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c/hosts",
            "LogPath": "/var/lib/docker/containers/eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c/eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c-json.log",
            "Name": "/dazzling_payne",
            "RestartCount": 0,
            "Driver": "overlay2",
            "Platform": "linux",
            "MountLabel": "",
            "ProcessLabel": "",
            "AppArmorProfile": "",
            "ExecIDs": null,
            "HostConfig": {
                "Binds": null,
                "ContainerIDFile": "",
                "LogConfig": {
                    "Type": "json-file",
                    "Config": {}
                },
                "NetworkMode": "default",
                "PortBindings": {},
                "RestartPolicy": {
                    "Name": "no",
                    "MaximumRetryCount": 0
                },
                "AutoRemove": false,
                "VolumeDriver": "",
                "VolumesFrom": null,
                "CapAdd": null,
                "CapDrop": null,
                "CgroupnsMode": "host",
                "Dns": [],
                "DnsOptions": [],
                "DnsSearch": [],
                "ExtraHosts": null,
                "GroupAdd": null,
                "IpcMode": "private",
                "Cgroup": "",
                "Links": null,
                "OomScoreAdj": 0,
                "PidMode": "",
                "Privileged": false,
                "PublishAllPorts": false,
                "ReadonlyRootfs": false,
                "SecurityOpt": null,
                "UTSMode": "",
                "UsernsMode": "",
                "ShmSize": 67108864,
                "Runtime": "runc",
                "ConsoleSize": [
                    0,
                    0
                ],
                "Isolation": "",
                "CpuShares": 0,
                "Memory": 0,
                "NanoCpus": 0,
                "CgroupParent": "",
                "BlkioWeight": 0,
                "BlkioWeightDevice": [],
                "BlkioDeviceReadBps": null,
                "BlkioDeviceWriteBps": null,
                "BlkioDeviceReadIOps": null,
                "BlkioDeviceWriteIOps": null,
                "CpuPeriod": 0,
                "CpuQuota": 0,
                "CpuRealtimePeriod": 0,
                "CpuRealtimeRuntime": 0,
                "CpusetCpus": "",
                "CpusetMems": "",
                "Devices": [],
                "DeviceCgroupRules": null,
                "DeviceRequests": null,
                "KernelMemory": 0,
                "KernelMemoryTCP": 0,
                "MemoryReservation": 0,
                "MemorySwap": 0,
                "MemorySwappiness": null,
                "OomKillDisable": false,
                "PidsLimit": null,
                "Ulimits": null,
                "CpuCount": 0,
                "CpuPercent": 0,
                "IOMaximumIOps": 0,
                "IOMaximumBandwidth": 0,
                "MaskedPaths": [
                    "/proc/asound",
                    "/proc/acpi",
                    "/proc/kcore",
                    "/proc/keys",
                    "/proc/latency_stats",
                    "/proc/timer_list",
                    "/proc/timer_stats",
                    "/proc/sched_debug",
                    "/proc/scsi",
                    "/sys/firmware"
                ],
                "ReadonlyPaths": [
                    "/proc/bus",
                    "/proc/fs",
                    "/proc/irq",
                    "/proc/sys",
                    "/proc/sysrq-trigger"
                ]
            },
            "GraphDriver": {
                "Data": {
                    "LowerDir": "/var/lib/docker/overlay2/c7182b72ac77ebfe51a7cb598d4887771dd2e932eca2866836cb0bf9cda47d9f-init/diff:/var/lib/docker/overlay2/b0ffefc3829d7dda67b0610f1a77d46483cd519c9f42f83a030cbb282e1d0a9d/diff",
                    "MergedDir": "/var/lib/docker/overlay2/c7182b72ac77ebfe51a7cb598d4887771dd2e932eca2866836cb0bf9cda47d9f/merged",
                    "UpperDir": "/var/lib/docker/overlay2/c7182b72ac77ebfe51a7cb598d4887771dd2e932eca2866836cb0bf9cda47d9f/diff",
                    "WorkDir": "/var/lib/docker/overlay2/c7182b72ac77ebfe51a7cb598d4887771dd2e932eca2866836cb0bf9cda47d9f/work"
                },
                "Name": "overlay2"
            },
            "Mounts": [],
            "Config": {
                "Hostname": "eac419b56757",
                "Domainname": "",
                "User": "",
                "AttachStdin": false,
                "AttachStdout": false,
                "AttachStderr": false,
                "Tty": false,
                "OpenStdin": false,
                "StdinOnce": false,
                "Env": [
                    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
                ],
                "Cmd": [
                    "/bin/sh",
                    "-c",
                    "while true;do echo 666;sleep 1;done"
                ],
                "Image": "centos",
                "Volumes": null,
                "WorkingDir": "",
                "Entrypoint": null,
                "OnBuild": null,
                "Labels": {
                    "org.label-schema.build-date": "20201204",
                    "org.label-schema.license": "GPLv2",
                    "org.label-schema.name": "CentOS Base Image",
                    "org.label-schema.schema-version": "1.0",
                    "org.label-schema.vendor": "CentOS"
                }
            },
            "NetworkSettings": {
                "Bridge": "",
                "SandboxID": "654e484c55dccc369715c34312a2f5c6a9024e81b37b78a5fb5fb421d4d59d20",
                "HairpinMode": false,
                "LinkLocalIPv6Address": "",
                "LinkLocalIPv6PrefixLen": 0,
                "Ports": {},
                "SandboxKey": "/var/run/docker/netns/654e484c55dc",
                "SecondaryIPAddresses": null,
                "SecondaryIPv6Addresses": null,
                "EndpointID": "c8f8f3382385d2cb5db0d73e8949aea6bc62da6ef7ab065a4a5dc532ce06700d",
                "Gateway": "172.18.0.1",
                "GlobalIPv6Address": "",
                "GlobalIPv6PrefixLen": 0,
                "IPAddress": "172.18.0.3",
                "IPPrefixLen": 16,
                "IPv6Gateway": "",
                "MacAddress": "02:42:ac:12:00:03",
                "Networks": {
                    "bridge": {
                        "IPAMConfig": null,
                        "Links": null,
                        "Aliases": null,
                        "NetworkID": "cd4dd4c9c9f016ee4074d228681e21ecb604ec0b58496553d80527104a6c4f69",
                        "EndpointID": "c8f8f3382385d2cb5db0d73e8949aea6bc62da6ef7ab065a4a5dc532ce06700d",
                        "Gateway": "172.18.0.1",
                        "IPAddress": "172.18.0.3",
                        "IPPrefixLen": 16,
                        "IPv6Gateway": "",
                        "GlobalIPv6Address": "",
                        "GlobalIPv6PrefixLen": 0,
                        "MacAddress": "02:42:ac:12:00:03",
                        "DriverOpts": null
                    }
                }
            }
        }
    ]
```



#### 进入当前正在运行的容器

>   docker exec  进入容器后开启一个新的终端，可以在里面操作（常用）
>   docker attach 进入容器正在执行的终端，不会启动新的终端

```shell
# 通常容器都是后台运行，往往需要进入容器，修改一些配置
**方式一**
[root@shenhao ~]# docker exec --help
Usage:  docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
Run a command in a running container
Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables
      --env-file list        Read in a file of environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])
  -w, --workdir string       Working directory inside the container
  
docker exec -it 容器id /bin/bash

**方式二**
docker attach 容器id
```




#### 从容器内拷贝文件到容器外

```shell
docker cp 容器id:容器内路径 目的主机路径

[root@shenhao ~]# docker cp --help
Usage:  docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
        docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
Copy files/folders between a container and the local filesystem
Use '-' as the source to read a tar archive from stdin
and extract it to a directory destination in a container.
Use '-' as the destination to stream a tar archive of a
container source to stdout.
Options:
  -a, --archive       Archive mode (copy all uid/gid information)
  -L, --follow-link   Always follow symbol link in SRC_PATH
  
   # 拷贝是一个手动过程，未来可以使用 -v 卷的技术，可以实现自动同步 /home /home
```


​    ![image-20210328164036720](\img\image-20210328164036720.png)



### 作业练习

#### nginx

~~~shell
 -d 后台运行
 -p 端口映射
 --name 容器命名
[root@shenhao ~]# docker run -d --name nagin01 -p 8080:80 nginx
14a19e5b96be0568c9c89a8a48e4d7f7dfab05461a6b5bec6843508dbadfbc7e
[root@shenhao ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
14a19e5b96be   nginx     "/docker-entrypoint.…"   7 seconds ago   Up 6 seconds   0.0.0.0:8080->80/tcp   nagin01


[root@shenhao ~]# docker exec -it nagin01 /bin/bash
root@14a19e5b96be:/# ls
bin   dev                  docker-entrypoint.sh  home  lib64  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint.d  etc                   lib   media  opt  root  sbin  sys  usr
root@14a19e5b96be:/# ll
bash: ll: command not found
root@14a19e5b96be:/# ls
bin   dev                  docker-entrypoint.sh  home  lib64  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint.d  etc                   lib   media  opt  root  sbin  sys  usr
root@14a19e5b96be:/# cd /etc/nginx

root@14a19e5b96be:/etc/nginx# ls
conf.d  fastcgi_params  koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params  uwsgi_params  win-utf
root@14a19e5b96be:/etc/nginx# 



~~~

此时每次改动nginx配置文件，都需要进入容器修改。因此需要为在容器外部进行映射，确保在外部修改配置文件后，即可成功映射

#### Tomcat

~~~shell
# 官方使用
docker run -it --rm tomcat:9.0

# 官方的这个方法，是用完即删除


[root@shenhao ~]# docker run -d --name tomcat01 -p 8080:8080 tomcat
e1ecd29e64b32b365ce98e04e25c40a0ba120d7b19888070a7a92a13e6b6b947
[root@shenhao ~]# docker exec -it tomcat01 /bin/bash
root@e1ecd29e64b3:/usr/local/tomcat# ls
BUILDING.txt     LICENSE  README.md      RUNNING.txt  conf  logs            temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin          lib   native-jni-lib  webapps  work
root@e1ecd29e64b3:/usr/local/tomcat# 

root@e1ecd29e64b3:/usr/local/tomcat# cd webapps
root@e1ecd29e64b3:/usr/local/tomcat/webapps# ls
root@e1ecd29e64b3:/usr/local/tomcat/webapps# 

root@e1ecd29e64b3:/usr/local/tomcat# ls
BUILDING.txt     LICENSE  README.md      RUNNING.txt  conf  logs            temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin          lib   native-jni-lib  webapps  work
root@e1ecd29e64b3:/usr/local/tomcat# cp -r webapps.dist/* webapps
root@e1ecd29e64b3:/usr/local/tomcat# cd webapps

# docker下载的tomcat是阉割版本的
	1.linux命令少了
	2.没有webapps
# 原因在于阿里云镜像默认是最小的镜像，所以不必要的都剔除掉。通过将webapps.dist的内容复制到webapps后，即可访问8080端口

依然需要在容器外提供映射路径，达到便携发布

~~~



#### es+kibana

~~~shell
# 1.es暴漏的端口较多
# 2.es十分耗内存
# 3.es的数据一般需要挂载到安全目录

# --net somenetwork  网络配置
 docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.11.2
 
# 启动后会发现linux会变卡 docker stats 查看cpu等状态

# 因此需要修改配置文件（-e ），加入内存的限制
 docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS-"-Xms64m -Xmx512m" elasticsearch:7.6.2
 
 如何使用kibana连接es
# 此时需要启动kibana的话，需要将两个dokcer的网络连接起来（要了解docker网络）
~~~



# Docker可视化工具

- portainer

- Rancher(CI/CD时使用)



#### 什么是portainer

> ​	Docker图形化界面管理工具，提供一个后台面板供我们使用



~~~shell
docker run -d -p 8080:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
~~~





# Docker镜像讲解

### 镜像是什么

镜像是一种轻量级，可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件。他包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件

所有的应用，直接打包docker镜像，就可以直接运行起来

如何得到镜像

- 远程仓库下载
- 别人拷贝
- 自己制作镜像

### Docker镜像加载原理

> UnionFS（联合文件系统）

UnionFs：Union文件系统是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时将不同目录挂载到同一个虚拟文件系统下（unite serveral directories into a single virtual filesystem）。Union文件系统时Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的的应用镜像。

特性：一次同时加载多个文件系统，但从外面看来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录



> Docker镜像加载原理

ocker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。

![img](\img\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhbnlmMjAxNg==,size_16,color_FFFFFF,t_70)

 

平时我们安装进虚拟机的CentOS都是好几个G，为什么docker这里才200M？？

对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs。


### 分层理解

> 分层的镜像

下载镜像时，可以发现是一层一层下载的

![img](\img\watermark,tyasdadpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhbnlmMjAxNg==,size_16,color_FFFFFF,t_70)



所有的Docker 镜像都是起始于一个基础的镜像，当进行修改或者增加新的内容的时候，就会创建新的镜像成。 举一个简单的例子，加入CentOS 创建了一个新的镜像，这就是新镜像的第一层；如果在该镜像中添加Python 包，就会在基础镜像上创建第二个镜像层；如果继续添加一个安全补丁，就会去常见第三层镜像

![在这里插入图片描述](\img\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5MzgxNzc1,size_16,color_FFFFFF,t_70)

在添加额外的镜像层的同时，镜像始终保持是当前所有镜像的组合，理解这一点非常重要，下图举例了一个简单的例子，每个镜像层包含3个文件，而镜像包含了来自两个镜像层的6个文件

![在这里插入图片描述](\img\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1bGVpMjkyMTYyNTk1Nw==,size_16,color_FFFFFF,t_70)

上图中的镜像层跟之前图中的略有区别，主要目的是便于展示文件。
下图中展示了一个稍微复杂的三层镜像，在外部看来整个镜像只有6个文件，这是因为最上层中的文件7是文件5的一个更新版本。

![在这里插入图片描述](\img\watermark,type_ZmFuZ3poZW5nasdqweow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1bGVpMjkyMTYyNTk1Nw==,size_16,color_FFFFFF,t_70)

这种情况下，上层镜像层中的文件覆盖了底层镜像层中的文件。这样就使得文件的更新版本作为一个新镜像层添加到镜像当中。

Docker通过存储引擎（新版本采用快照机制）的方式来实现镜像层堆栈，并保证多镜像层对外展示为统一的文件系统。

Linux上可用的存储引擎有AUFS、Overlay2、Device Mapper、Btrfs以及ZFS。顾名思义，每种存储引擎都基于Linux中对应的文件系统或者块设备技术，并且每种存储引擎都有其独有的性能特点。

Docker在Windows上仅支持windowsfiter一种存储引擎，该引擎基于NTFS文件系统之上实现了分层和CoW[1]。

下图展示了与系统显示相同的三层镜像。所有镜像层堆叠并合并，对外提供统一的视图。
![在这里插入图片描述](\img\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZtryNTk1Nw==,size_16,color_FFFFFF,t_70)



> 特点

**Docker镜像都只是可读的，当容器启动时，一个新的可写层被加载到镜像的顶部！**

**这一层就是我们通常所说的容器层，容器之下的都叫镜像层**

![在这里插入图片描述](\img\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1bGVpoiTk1Nw==,size_16,color_FFFFFF,t_70)

### commit镜像

~~~shell
docker commit 提交容器为一个新的副本

# 命令和git原理类似
docker commit -m="提交的信息" -a="作者" 容器ID 目标镜像名[:tag]
~~~

测试

~~~shell
[root@wulei wulei]# docker run -d -p 6379:8080 --name tomcat01 tomcat
05c036135c85d57526a1b637c25d2423d52b2fd4e366acbecdd734c8818cfd71
[root@wulei wulei]# docker ps
CONTAINER ID   IMAGE     COMMAND             CREATED         STATUS         PORTS                    NAMES
100be89194d5   tomcat    "catalina.sh run"   4 seconds ago   Up 3 seconds   0.0.0.0:3366->8080/tcp   tomcat01
# 进入Tomcat容器
[root@wulei wulei]# docker exec -it 100be89194d5 /bin/bash
# 删除 webapps 空文件夹
root@05c036135c85:/usr/local/tomcat# rmdir webapps
# 重命名webapps.dist 文件夹为 webapps
root@05c036135c85:/usr/local/tomcat# mv webapps.dist webapps
root@05c036135c85:/usr/local/tomcat# cd webapps/
root@05c036135c85:/usr/local/tomcat/webapps# ls
ROOT  docs  examples  host-manager  manager

# 以上的步骤做完之后，我以后不想这么麻烦，直接用改好的容器。
[root@wulei ~]# docker ps
CONTAINER ID   IMAGE     COMMAND             CREATED        STATUS         PORTS                    NAMES
100be89194d5   tomcat    "catalina.sh run"   15 hours ago   Up 5 minutes   0.0.0.0:6379->8080/tcp   tomcat01
# 将我们修改过的容器通过commit提交成一个新的镜像，我们以后就可以使用自己的镜像即可
[root@wulei ~]# docker commit -a="shen" -m="add webapps app" 100be89194d5 tomcat02:1.0
sha256:6ad64d57c483c7a9f73e6c6cfa183aa70436a475ce119cc1270ebbc18df6d203
[root@shenhao ~]# docker images
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
tomcat02              1.0       8e7615fc6ed6   13 seconds ago   654MB
# 现在只是把它提交到本地，后面会说到提交到仓库

~~~

如果想保存当前容器的状态，就可以通过commit来提交，获得一个镜像。类似虚拟机的快照



# 容器数据卷

### 什么是容器数据卷

#### docker理念

将应用和环境打包成镜像直接启动！

但是数据如果在容器中，将容器删除，数据就会丢失。因此我们需要将数据持久化

需要容器之间可以有一个数据共享的技术！Docker容器中产生的数据，同步到本地，这就是卷技术（目录的挂载，将容器内的目录，挂载到Linux上面）

![image-20210330224115603](\img\image-20210330224115603.png)



**容器的持久化和同步操作，容器间也是可以数据共享的**



### 使用数据卷

> 方式一：直接使用命令来挂载 -v

~~~shell
docker run -it -v 主机目录:容器目录

# 测试
[root@shenhao ~]# cd /home/
[root@shenhao home]# ls
admin  beiluo
[root@shenhao home]# docker run -it -v /home/ceshi:/home centos /bin/bash

[root@shenhao ~]# cd /home/
[root@shenhao home]# ls
admin  beiluo  ceshi

[root@212532162a87 home]# touch test.java
[root@shenhao ceshi]# ls
test.java

~~~

![image-20210330224827799](\img\image-20210330224827799.png)

这是一个双向过程，无论在容器内外修改文件，两边都可看到其变化。

不过会占用双倍的存储空间

### 安装MySql

MySql数据在/data下。

~~~shell
# 官方命令 首次运行MySql要配置密码
-e # 表示配置环境
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag



-d 后台运行
-p 端口映射
-v 数据卷挂载
-e 环境配置
--name 容器名字
[root@shenhao ~]# docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=admin123 --name mysql01 mysql:latest


# 注意，如果使用mysql8.0版，注意密码的形式，可使用如下命令修改
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';

mysql> FLUSH PRIVILEGES;
~~~



### 具名和匿名挂载

~~~shell
# 匿名挂载
-v 容器内路径

# docker run -d -P --name nginx01 -v /etc/nginx nginx
 
 //端口映射-p(小写)、-P(大写)区别
# -p指定要映射的端口，一个指定端口上只可以绑定一个容器
# P将容器内部开放的网络端口随机映射到宿主机的一个端口上


[root@shenhao data]# docker volume --help
Usage:  docker volume COMMAND
Manage volumes
Commands:
  create      Create a volume
  inspect     Display detailed information on one or more volumes
  ls          List volumes
  prune       Remove all unused local volumes
  rm          Remove one or more volumes
Run 'docker volume COMMAND --help' for more information on a command.


[root@shenhao data]# docker volume ls
DRIVER    VOLUME NAME
local     352d7c96eabbce17534d96861ff35dd1f6e39471f5a63d8d7068cbc94d887ddc
local     29068fa75992e185aa4721c6fa663209a702d5528374d16a8b875ac7f2be3762
local     ad44582f0653466e0f8244456b4d263bb333b7750abe73263d7312887d9caaba

# 这种就是匿名挂载，在-v的时候只写了容器内的路径，没有写容器外路径


# docker run -d -P --name nginx02 -v Bertram:/etc/nginx nginx
# docker volume ls
DRIVER              VOLUME NAME
.....
local               Bertram


~~~

所有的docker容器内的卷，没有指定目录的情况下都是在 '/var/lib/docker/volumes/xxxx/_data'目录下

这里的默认存储路径修改过的，参考文章**[默认docker默认存储路径](https://blog.csdn.net/chj_1224365967/article/details/109053895)**进行修改

我们通过具名挂载可以方便的找到我们的一个卷，大多数使用的都是具名挂载。

#### 如何确定是具名挂载还是匿名挂载还是指定路径挂载

~~~shell
-v 容器内路径 # 匿名挂载
-v 卷名:容器内路径 # 具名挂载
-v /宿主机路径:容器内路径 # 指定路径挂载

docker run -d -P --name nginx02 -v /etc/nginx nginx                    # 匿名挂载
docker run -d -P --name nginx02 -v nginxconfig:/etc/nginx nginx        # 卷名
docker run -d -P --name nginx02 -v /nginxconfig:/etc/nginx nginx       # 目录名
~~~

拓展

~~~shell
# 通过 -v 容器内路径，ro rw改变读写权限
ro readonly  #只读
rw readwrite #可读可写
 
# 一旦这个设置了容器权限，容器对我们挂载出来的内容就有了限定
docker run -d -P --name nginx02 -v Bertram:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v Bertram:/etc/nginx:rw nginx
# 只要看到ro就说明这个路径只能通过宿主机来操作，容器内无法操作。

~~~





初始Dockerfile

Dockerfile就是用来构建docker镜像的构造文件，就是一段命令脚本

通过这个脚本可以生成镜像，镜像是一层一层的。脚本也是一个个的命令，是一层一层的

~~~shell
# 创建一个dockerfile文件，名字可随意
# 文件中的内容 指令（大写） 参数


FROM centos

VOLUME ["volume01","volume02"]

CMD echo "---end---"

CMD /bin/bash

# 每个命令，就是镜像的一层
~~~

![image-20210406212955579](\img\image-20210406212955579.png)

![image-20210406213319448](\img\image-20210406213319448.png)

其中的两个目录，就是我们自己挂载的，和外部一定有一个同步的目录（匿名挂载方式）



![image-20210406213916114](\img\image-20210406213916114.png)

### 数据卷容器

两个mysql同步数据

![image-20210406214114873](\img\image-20210406214114873.png)



~~~shell
# 启动多个容器
docker run -it --name docker01 beiluo/centos:1.0
docker run -it --name docker02 --volumes-from docker01 beiluo/centos:1.0

[root@ef959e1392ac /]# cd volume01
[root@ef959e1392ac volume01]# touch docker01
[root@ef959e1392ac volume01]# ls
docker01

[root@f0469b0cb9c8 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  volume01  volume02
[root@f0469b0cb9c8 /]# cd volume01/
[root@f0469b0cb9c8 volume01]# ls
docker01
[root@f0469b0cb9c8 volume01]# 

# docker01就是数据卷容器
# 通过--volumes-from 就可以达成容器间的数据挂载
# 数据共享机制，本质还是挂载在宿主机上，如果将宿主机文件删除，则容器中数据也会消失

# 容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有容器使用为止
~~~



# DockerFile

### dockerfile介绍

dockerfile是用来构建docker镜像的文件！命令参数脚本！

构建步骤：

1.便携dockerfile文件

2.docker build 构建成为一个镜像

3.docker run运行镜像

4.docker push 发布镜像



### dockerfile构建过程

**基础知识：**

1.每个保留关键字（指令）都必须是大写字母

2.执行顺序从上至下

3.# 表示注释

4.每一个指令都会创建提交一个新的镜像层，并提交

![image-20210406222125080](\img\image-20210406222125080.png)

dockerfile是面向开发的，以后发布项目，做镜像，就需要编写dockerfile文件

DockerFile：构建文件，定义了一切步骤，源代码

DockerImages：通过DockerFile构建生产的镜像，最终发布和运行的产品

Docker容器：容器就是镜像运行起来提供服务的



### DockerFile指令

~~~shell
FROM		# 基础镜像，一切从此构建
MAINTAINER  # 镜像是谁写的，姓名+邮箱
RUN			# 镜像构建时需要执行的命令
ADD			# 步骤
WORKDIR 	# 镜像工作目录
VOLUME		# 挂载目录
EXPOSE		# 指令暴露端口
CMD			# 指定这个容器启动时执行的命令,只有最后一个会生效，可被替换
ENTRYPOINT  # 指定这个容器启动时执行的命令,可以追加命令
ONBUILD		# 当构建一个被继承 DockerFile 这个时候会运行ONBUILD的命令
COPY		# 类似ADD，将我们文件拷贝到镜像中
ENV			# 构建时设置环境命令
~~~



![img](\img\e84ae621fe610f9aadb4bdf218f.JPG)



### 实战测试（centos）

dockerhun中 99%的镜像都是从这个基础镜像 FROM scratch 构建出来的

![image-20210406223326028](\img\image-20210406223326028.png)





>  创建一个centos

~~~shell
# 1.编写dockerfile
FROM centos
MAINTAINER beiluo<shenhao189@foxmail.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80
CMD echo $MYPATH
CMD echo "-----end-----"
CMD /bin/bash

# 2.通过文件构建镜像
# 命令 docker build -f dockerfile文件路径 -t 镜像名:[tag]

 [root@shenhao home]# docker build -f mydockerfile-centos -t mycentos:0.1 .
Sending build context to Docker daemon  190.8MB
Step 1/10 : FROM centos
 ---> 300e315adb2f
Step 2/10 : MAINTAINER beiluo<shenhao189@foxmail.com>
 ---> Running in 70bb3e6dd7d2
Removing intermediate container 70bb3e6dd7d2
 ---> 519e27983630
Step 3/10 : ENV MYPATH /usr/local
 ---> Running in cd63c67b4f62
Removing intermediate container cd63c67b4f62
 ---> fc46be05beb4
Step 4/10 : WORKDIR $MYPATH
 ---> Running in d1fb605a6469
Removing intermediate container d1fb605a6469
 ---> 9570b3e051fd
Step 5/10 : RUN yum -y install vim
 ---> Running in 2f95fde7e2ab
CentOS Linux 8 - AppStream                      3.3 MB/s | 6.3 MB     00:01    
CentOS Linux 8 - BaseOS                         1.9 MB/s | 2.3 MB     00:01    
CentOS Linux 8 - Extras                         4.9 kB/s | 9.6 kB     00:01    
Dependencies resolved.
================================================================================
 Package             Arch        Version                   Repository      Size
================================================================================
Installing:
 vim-enhanced        x86_64      2:8.0.1763-15.el8         appstream      1.4 M
Installing dependencies:
 gpm-libs            x86_64      1.20.7-15.el8             appstream       39 k
 vim-common          x86_64      2:8.0.1763-15.el8         appstream      6.3 M
 vim-filesystem      noarch      2:8.0.1763-15.el8         appstream       48 k
 which               x86_64      2.21-12.el8               baseos          49 k

Transaction Summary
================================================================================
Install  5 Packages

Total download size: 7.8 M
Installed size: 30 M
Downloading Packages:
(1/5): gpm-libs-1.20.7-15.el8.x86_64.rpm        793 kB/s |  39 kB     00:00    
(2/5): vim-filesystem-8.0.1763-15.el8.noarch.rp 1.6 MB/s |  48 kB     00:00    
(3/5): vim-common-8.0.1763-15.el8.x86_64.rpm     42 MB/s | 6.3 MB     00:00    
(4/5): vim-enhanced-8.0.1763-15.el8.x86_64.rpm  8.5 MB/s | 1.4 MB     00:00    
(5/5): which-2.21-12.el8.x86_64.rpm             542 kB/s |  49 kB     00:00    
--------------------------------------------------------------------------------
Total                                           1.8 MB/s | 7.8 MB     00:04     
warning: /var/cache/dnf/appstream-02e86d1c976ab532/packages/gpm-libs-1.20.7-15.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: NOKEY
CentOS Linux 8 - AppStream                      1.6 MB/s | 1.6 kB     00:00    
Importing GPG key 0x8483C65D:
 Userid     : "CentOS (CentOS Official Signing Key) <security@centos.org>"
 Fingerprint: 99DB 70FA E1D7 CE22 7FB6 4882 05B5 55B3 8483 C65D
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : which-2.21-12.el8.x86_64                               1/5 
  Installing       : vim-filesystem-2:8.0.1763-15.el8.noarch                2/5 
  Installing       : vim-common-2:8.0.1763-15.el8.x86_64                    3/5 
  Installing       : gpm-libs-1.20.7-15.el8.x86_64                          4/5 
  Running scriptlet: gpm-libs-1.20.7-15.el8.x86_64                          4/5 
  Installing       : vim-enhanced-2:8.0.1763-15.el8.x86_64                  5/5 
  Running scriptlet: vim-enhanced-2:8.0.1763-15.el8.x86_64                  5/5 
  Running scriptlet: vim-common-2:8.0.1763-15.el8.x86_64                    5/5 
  Verifying        : gpm-libs-1.20.7-15.el8.x86_64                          1/5 
  Verifying        : vim-common-2:8.0.1763-15.el8.x86_64                    2/5 
  Verifying        : vim-enhanced-2:8.0.1763-15.el8.x86_64                  3/5 
  Verifying        : vim-filesystem-2:8.0.1763-15.el8.noarch                4/5 
  Verifying        : which-2.21-12.el8.x86_64                               5/5 

Installed:
  gpm-libs-1.20.7-15.el8.x86_64         vim-common-2:8.0.1763-15.el8.x86_64    
  vim-enhanced-2:8.0.1763-15.el8.x86_64 vim-filesystem-2:8.0.1763-15.el8.noarch
  which-2.21-12.el8.x86_64             

Complete!
Removing intermediate container 2f95fde7e2ab
 ---> 4985e545c676
Step 6/10 : RUN yum -y install net-tools
 ---> Running in 81e2413b3554
Last metadata expiration check: 0:00:12 ago on Tue Apr  6 14:45:00 2021.
Dependencies resolved.
================================================================================
 Package         Architecture Version                        Repository    Size
================================================================================
Installing:
 net-tools       x86_64       2.0-0.52.20160912git.el8       baseos       322 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 322 k
Installed size: 942 k
Downloading Packages:
net-tools-2.0-0.52.20160912git.el8.x86_64.rpm   2.4 MB/s | 322 kB     00:00    
--------------------------------------------------------------------------------
Total                                           412 kB/s | 322 kB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : net-tools-2.0-0.52.20160912git.el8.x86_64              1/1 
  Running scriptlet: net-tools-2.0-0.52.20160912git.el8.x86_64              1/1 
  Verifying        : net-tools-2.0-0.52.20160912git.el8.x86_64              1/1 

Installed:
  net-tools-2.0-0.52.20160912git.el8.x86_64                                     

Complete!
Removing intermediate container 81e2413b3554
 ---> 81c9b051c6e0
Step 7/10 : EXPOSE 80
 ---> Running in da8f59fc4d56
Removing intermediate container da8f59fc4d56
 ---> 21a1d1953c20
Step 8/10 : CMD echo $MYPATH
 ---> Running in d598e9931e59
Removing intermediate container d598e9931e59
 ---> 2041c6b84d2c
Step 9/10 : CMD echo "-----end-----"
 ---> Running in d529a52f3c78
Removing intermediate container d529a52f3c78
 ---> 430c4bd2d931
Step 10/10 : CMD /bin/bash
 ---> Running in 83a36cd740d8
Removing intermediate container 83a36cd740d8
 ---> 597019bfbe8c
Successfully built 597019bfbe8c
Successfully tagged mycentos:0.1
[root@shenhao home]# 

# 3.测试运行会发现vim命令和ifconfig命令都可以使用了
~~~



~~~shell
# 可以通过docker history 命令观察镜像如何构建

[root@shenhao home]# docker history 8457e9155715
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
8457e9155715   5 weeks ago   /bin/sh -c #(nop)  CMD ["mysqld"]               0B        
<missing>      5 weeks ago   /bin/sh -c #(nop)  EXPOSE 3306 33060            0B        
<missing>      5 weeks ago   /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B        
<missing>      5 weeks ago   /bin/sh -c ln -s usr/local/bin/docker-entryp…   34B       
<missing>      5 weeks ago   /bin/sh -c #(nop) COPY file:74c3ab1ae4fa929c…   13.4kB    
<missing>      8 weeks ago   /bin/sh -c #(nop) COPY dir:2e040acc386ebd23b…   1.12kB    
<missing>      8 weeks ago   /bin/sh -c #(nop)  VOLUME [/var/lib/mysql]      0B        
<missing>      8 weeks ago   /bin/sh -c {   echo mysql-community-server m…   411MB     
<missing>      8 weeks ago   /bin/sh -c echo 'deb http://repo.mysql.com/a…   55B       
<missing>      8 weeks ago   /bin/sh -c #(nop)  ENV MYSQL_VERSION=8.0.23-…   0B        
<missing>      8 weeks ago   /bin/sh -c #(nop)  ENV MYSQL_MAJOR=8.0          0B        
<missing>      8 weeks ago   /bin/sh -c set -ex;  key='A4A9406876FCBD3C45…   2.61kB    
<missing>      8 weeks ago   /bin/sh -c apt-get update && apt-get install…   52.2MB    
<missing>      8 weeks ago   /bin/sh -c mkdir /docker-entrypoint-initdb.d    0B        
<missing>      8 weeks ago   /bin/sh -c set -eux;  savedAptMark="$(apt-ma…   4.17MB    
<missing>      8 weeks ago   /bin/sh -c #(nop)  ENV GOSU_VERSION=1.12        0B        
<missing>      8 weeks ago   /bin/sh -c apt-get update && apt-get install…   9.34MB    
<missing>      8 weeks ago   /bin/sh -c groupadd -r mysql && useradd -r -…   329kB     
<missing>      8 weeks ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      8 weeks ago   /bin/sh -c #(nop) ADD file:d5c41bfaf15180481…   69.2MB    
[root@shenhao home]# 


~~~



#### CMD和ENTRYPOINT的区别

> CMD：指定这个容器启动的时候需要执行的命令，但是只有最后一个CMD会生效
> ENTRYPOINT：指定这个容器启动的时候需要执行的命令，且可以追加命令



### 实战测试（Tomcat）

1.准备镜像文件tomcat压缩包，jdk压缩包

2.编写Dockerfile文件

3.构建镜像

~~~shell
FROM centos   # 基本镜像
MAINTAINER Aut<123@123.com>    # 作者信息
 
COPY readme.txt /usr/local/readme.txt    # 复制readme到容器内部的这个路径
 
ADD /home/dockerfile/jdk-8u11-linux-x64.tar.gz /usr/local/    # ADD 命令会自动解压 将tar包解压到这个路径
ADD /home/dockerfile/apache-tomcat-9.0.22.tar.gz /usr/local/
 
RUN yum -y install vim    # 安装一些基本命令
 
ENV MYPATH /usr/local     # 设置默认的工作目录
WORKDIR $MYPATH
 
ENV JAVA_HOME /usr/local/jdk1.8.0_11    # java 环境变量
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/toos.jar
 
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22    # tomcat 环境变量
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.22
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
 
EXPOSE 8080  # 暴露 tomcat 端口
 
CMD /usr/local/apache-tomcat-9.0.22/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.22/bin/logs/catalina.out  # 启动 tomcat 可以通过 && 拼接一些参数  tail -F 显示文件新追加的内容



docker run -d -p 9090:8080 --name testtomcat -v /home/tomcat/test:/usr/local/apache-tomcat-9.0.22/webapps diytomcat
~~~



### 发布自己的镜像

> DockerHub

1.注册DockerHub账号

2.本地linux进行登录

3.服务器上提交自己的镜像

4.docker push 

​	提交前可以给自己的镜像增加tag    docker tag   id      作者们/镜像名:版本号



### 小结

![image-20210407223150165](\img\image-20210407223150165.png)

# Docker网络

### 理解Docker0网络

~~~shell
[root@shenhao ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:16:3e:12:08:39 brd ff:ff:ff:ff:ff:ff
    inet 172.17.7.217/18 brd 172.17.63.255 scope global dynamic eth0
       valid_lft 312663611sec preferred_lft 312663611sec
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:7e:f5:dc:28 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global docker0
       valid_lft forever preferred_lft forever
[root@shenhao ~]# 


# 分别是本机回环地址，阿里云内网地址，docker0地址
# 三个网络

# docker是如何处理容器网络访问？


[root@shenhao ~]# docker run -d -P --name tomcat01 tomcat

# 查看容器内部网络地址 ip addr
# 容器启动时会得到一个eth0@if53的IP地址，是docker分配的

[root@shenhao ~]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
52: eth0@if53: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# linux 能否ping通容器内部？
[root@shenhao ~]# ping 172.18.0.2
PING 172.18.0.2 (172.18.0.2) 56(84) bytes of data.
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.077 ms
64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.070 ms
64 bytes from 172.18.0.2: icmp_seq=3 ttl=64 time=0.056 ms
64 bytes from 172.18.0.2: icmp_seq=4 ttl=64 time=0.071 ms
# linux 可以 ping 通 docker 容器内部
~~~

> 原理

1. 我们每启动一个 docker 容器，docker就会给docker容器分配一个ip。只有安装了docker，就会有一个docker0网卡，使用了桥接模式，使用的技术是evth-pair技术

再次测试ip addr ，发现多了一个网卡。

~~~ shell
[root@shenhao ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:16:3e:12:08:39 brd ff:ff:ff:ff:ff:ff
    inet 172.17.7.217/18 brd 172.17.63.255 scope global dynamic eth0
       valid_lft 312662924sec preferred_lft 312662924sec
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:7e:f5:dc:28 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global docker0
       valid_lft forever preferred_lft forever
53: veth30e91af@if52: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP 
    link/ether e6:44:6c:03:a8:fc brd ff:ff:ff:ff:ff:ff link-netnsid 0
~~~



​    2.再启动一个容器，会发现又多了一个网卡，并且是对应的

~~~shell
# evth-pair 就是一堆的虚拟设备接口，他们都是成对出现的，一段连着协议，一段彼此相连
# 正因为有这个特性，evth-pair 充当一个桥梁，连接各种虚拟网络设备
# OpenStac，Docker容器之间的连接, OVS的连接,都是使用 evth-pair 技术.
~~~



​    3.容器之间是可以 ping 通的

~~~shell
[root@shenhao ~]# docker exec -it tomcat02 ping 172.18.0.2
PING 172.18.0.2 (172.18.0.2) 56(84) bytes of data.
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.089 ms
64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.098 ms
64 bytes from 172.18.0.2: icmp_seq=3 ttl=64 time=0.084 ms

~~~

![image-20210408223735399](\img\image-20210408223735399.png)



所有的容器不指定网络的情况下,都是docker0路由的,docker会给容器分配一个默认的可用ip.

Docker 使用的是Linux的桥接,宿主机中是一个Docker容器的网桥,docker0

Docker中的所有网络接口都是虚拟的,虚拟的转发效率高.

只要容器删除,对应的网桥就没了.





### ——Link

~~~shell
# 通过--link既可以解决网络连通问题

[root@shenhao ~]# docker run -d -P --name tomcat03 --link tomcat02 tomcat
3f6d29cfc8d92677efa15a02d9ff173af94bcd519f2de0048835ca4194951e3c
[root@shenhao ~]# docker exec tomcat03 ping tomcat02
PING tomcat02 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.17.0.3): icmp_seq=1 ttl=64 time=0.106 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=2 ttl=64 time=0.045 ms

# tomcat03虽然配置了tomcat02,但是tomcat02没有配置tomcat03所有无法连通

# --link 就是在hosts配置中增加了一个容器ip地址的映射.

# 现在docker已经不建议使用--link，自定义网络,不使用docker0.docker0不支持容器名连接访问.
~~~



### 自定义网络

> 查看所有的Docker网络
>
> [root@shenhao ~]# docker network ls
> NETWORK ID     NAME      DRIVER    SCOPE
> cd4dd4c9c9f0   bridge    bridge    local
> d4069aed9d55   host      host      local
> b5e6d5925898   none      null      local

> **网络模式**
>
> bridge：桥接模式 docker默认
>
> none：不配置网络
>
> host：和宿主机共享网络
>
> container：容器网络连通（用得少,局限大）

~~~shell
#直接启动的命令 --net bridge 这个就是默认的docker0
# docker run -d -P --name tomcat01 tomcat
# docker run -d -P --name tomcat01 --net bridge tomcat
 
# docker0特点:默认,域名不能访问, --link可以打通连接.


# 自定义网络
#--driver bridge
#--subnet 192.168.0.0/16
#--gateway 192.168.0.1
[root@shenhao ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
906fe6dddb1bd8d073dd0eb31ddf85d62f106b65086933a072a6c02f5cd93ec5
[root@shenhao ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
cd4dd4c9c9f0   bridge    bridge    local
d4069aed9d55   host      host      local
906fe6dddb1b   mynet     bridge    local
b5e6d5925898   none      null      local
[root@shenhao ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "906fe6dddb1bd8d073dd0eb31ddf85d62f106b65086933a072a6c02f5cd93ec5",
        "Created": "2021-04-09T23:10:32.120496555+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]

[root@shenhao ~]# docker run -d -P --name tomcat-net01 --net mynet tomcat
9e6bce620227c1064bededfb9498a0c71a397b04eb6e3c45381b5d3ca9334d99
[root@shenhao ~]# docker run -d -P --name tomcat-net02 --net mynet tomcat
ab92a845a1effda1e2c215f8e726f03360cde3d13ea470eb186101c453625ebe
[root@shenhao ~]# docker exec -it tomcat-net01 ping tomcat-net02
PING tomcat-net02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.069 ms
64 bytes from tomcat-net02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.069 ms
64 bytes from tomcat-net02.mynet (192.168.0.3): icmp_seq=3 ttl=64 time=0.070 ms
64 bytes from tomcat-net02.mynet (192.168.0.3): icmp_seq=4 ttl=64 time=0.067 ms

# 现在不使用 --link 也可以通过ping容器名字连接网络.

# 好处：不同的集群使用不同的网络,保证集群是安全和健康的
~~~



### 网络连通

~~~shell
[root@shenhao ~]# docker network connet --help

Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.



# docker network connect mynet[网络名] tomcat01[容器名]
# 连通之后就是将容器放到网络下,一个容器两个ip地址.
# 即对容器增加了一个虚拟网卡，放在指定的网络下

[root@shenhao ~]# docker run -d -P --name tomcat01 tomcat
123b24933feb4cf9a6db90c2c8a4b7ed3b5621ef7ff7f015bdf55643eacc5d69
[root@shenhao ~]# docker exec -it tomcat01 ping tomcat-net01
ping: tomcat-net01: Name or service not known
[root@shenhao ~]# docker network connect mynet tomcat01
[root@shenhao ~]# docker exec -it tomcat01 ping tomcat-net01
PING tomcat-net01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat-net01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.072 ms
64 bytes from tomcat-net01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.082 ms
64 bytes from tomcat-net01.mynet (192.168.0.2): icmp_seq=3 ttl=64 time=0.073 ms
64 bytes from tomcat-net01.mynet (192.168.0.2): icmp_seq=4 ttl=64 time=0.072 ms

~~~



### 实战：部署redis集群



![image-20210413222550607](\img\image-20210413222550607.png)



> shell脚本

~~~shell
# 创建网卡
docker network create redis --subnet 172.38.0.0/16 

# 通过脚本创建6个redis设置
for port in $(seq 1 6); \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379 
bind 0.0.0.0
cluster-enabled yes 
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done

# 启动容器
# 节点1
docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
    -v /mydata/redis/node-1/data:/data \
    -v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
    -d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

# 节点2 
docker run -p 6372:6379 -p 16372:16379 --name redis-2 \
 -v /mydata/redis/node-2/data:/data \
 -v /mydata/redis/node-2/conf/redis.conf:/etc/redis/redis.conf \
 -d --net redis --ip 172.38.0.12 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

...

# 节点6
docker run -p 6376:6379 -p 16376:16379 --name redis-6 \
 -v /mydata/redis/node-6/data:/data \
 -v /mydata/redis/node-6/conf/redis.conf:/etc/redis/redis.conf \
 -d --net redis --ip 172.38.0.16 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf



# 创建集群
[root@shenhao conf]# docker exec -it redis-1 /bin/sh
/data # ls
appendonly.aof  nodes.conf
/data # redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.38.0.15:6379 to 172.38.0.11:6379
Adding replica 172.38.0.16:6379 to 172.38.0.12:6379
Adding replica 172.38.0.14:6379 to 172.38.0.13:6379
M: 27fdd5f113fadc12448032c0caa83bc541fd9763 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
M: 5f060e4b39a988fcdc18a198da87bb5501e3a32a 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
M: f6c642cd597d65629cc4be1d2289666b19606196 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
S: 2289c2d3e99ac8c792a63f35ad5e525ddffc248f 172.38.0.14:6379
   replicates f6c642cd597d65629cc4be1d2289666b19606196
S: 967af8fed19bb4f5301712dc210a4a988921b062 172.38.0.15:6379
   replicates 27fdd5f113fadc12448032c0caa83bc541fd9763
S: 075ef90d86f1fed19662e638f0e81d3f00d1132d 172.38.0.16:6379
   replicates 5f060e4b39a988fcdc18a198da87bb5501e3a32a
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.....
>>> Performing Cluster Check (using node 172.38.0.11:6379)
M: 27fdd5f113fadc12448032c0caa83bc541fd9763 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 2289c2d3e99ac8c792a63f35ad5e525ddffc248f 172.38.0.14:6379
   slots: (0 slots) slave
   replicates f6c642cd597d65629cc4be1d2289666b19606196
M: f6c642cd597d65629cc4be1d2289666b19606196 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 075ef90d86f1fed19662e638f0e81d3f00d1132d 172.38.0.16:6379
   slots: (0 slots) slave
   replicates 5f060e4b39a988fcdc18a198da87bb5501e3a32a
S: 967af8fed19bb4f5301712dc210a4a988921b062 172.38.0.15:6379
   slots: (0 slots) slave
   replicates 27fdd5f113fadc12448032c0caa83bc541fd9763
M: 5f060e4b39a988fcdc18a198da87bb5501e3a32a 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

/data # redis-cli -c
127.0.0.1:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:156
cluster_stats_messages_pong_sent:156
cluster_stats_messages_sent:312
cluster_stats_messages_ping_received:151
cluster_stats_messages_pong_received:156
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:312
127.0.0.1:6379> cluster nodes
27fdd5f113fadc12448032c0caa83bc541fd9763 172.38.0.11:6379@16379 myself,master - 0 1618325812000 1 connected 0-5460
2289c2d3e99ac8c792a63f35ad5e525ddffc248f 172.38.0.14:6379@16379 slave f6c642cd597d65629cc4be1d2289666b19606196 0 1618325813000 4 connected
f6c642cd597d65629cc4be1d2289666b19606196 172.38.0.13:6379@16379 master - 0 1618325813579 3 connected 10923-16383
075ef90d86f1fed19662e638f0e81d3f00d1132d 172.38.0.16:6379@16379 slave 5f060e4b39a988fcdc18a198da87bb5501e3a32a 0 1618325813579 6 connected
967af8fed19bb4f5301712dc210a4a988921b062 172.38.0.15:6379@16379 slave 27fdd5f113fadc12448032c0caa83bc541fd9763 0 1618325813078 5 connected
5f060e4b39a988fcdc18a198da87bb5501e3a32a 172.38.0.12:6379@16379 master - 0 1618325812000 2 connected 5461-10922
127.0.0.1:6379> 
~~~

### SpringBoot打包镜像

- 架构SpringBoot项目
- 打包应用
- 编写DockerFile
- 构建镜像
- 发布运行