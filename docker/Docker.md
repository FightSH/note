## Docker名词概念

    镜像images：


​    
​    
### 底层原理
---

#####     Docker是怎么工作的？
    Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上。通过Socket从客户端访问。
    DockerServer接收到Docker-Client的指令，就会执行这个命令


## Docker常用命令
### 帮助命令
    docker version #显示docker版本信息
    docker info  #显示docker系统信息
    docker --help # 万能帮助命令
    docker 文档：docs.docker.com/reference

### 镜像命令
####    docker images

    [root@shenhao /]# docker images
    REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
    hello-world   latest    d1165f221234   2 weeks ago   13.3kB
    redis         latest    f877e80bb9ef   3 weeks ago   105MB
    mysql         latest    8457e9155715   3 weeks ago   546MB
    
    # docker images可选项
    docker -a,--all   # 列出所有镜像
    docker -q,--quiet # 只显示镜像id

####    docker search
    [root@shenhao /]# docker search mysql
    NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
    mysql                             MySQL is a widely used, open-source relation…   10665     [OK]       
    mariadb                           MariaDB Server is a high performing open sou…   3997      [OK]   
    
    # 可选项
    --filter=STARS=3000  # 搜索出的镜像的stars大于3000

#### docker pull
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

#### docker rmi 
    [root@shenhao /]# docker rmi --help
    
    Usage:  docker rmi [OPTIONS] IMAGE [IMAGE...]
    
    Remove one or more images
    
    Options:
      -f, --force      Force removal of the image
          --no-prune   Do not delete untagged parents


​    
​     docker rmi -f $(docker images -aq) # 强制删除所有镜像
​     
​     docker rmi -f 容器id 容器id

### 容器命令

**说明：有了镜像才可以创建容器**
    
    docker pull centos

#### docker run
    docker run [可选参数] image
    # 参数说明
    -- name-"Name"   指定容器名字 ，以区分容器
    -d      后台方式运行
    -it     使用交互方式运行，进入容器查看内容
    -p      指定容器的端口   端口暴露概念
        -p ip:端口  
        -p 主机端口:容器端口  常用
        -p 容器端口
    -P      随机指定端口


​    
​    #启动并进入容器，内部容器id即为镜像id
​    [root@shenhao /]# docker run -it centos /bin/bash
​    [root@15813679c0c3 /]# ls
​    bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

#### docker ps
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

#### 退出
    exit    # 直接停止容器并推出
    ctrl + P + Q     # 容器不停止退出

#### 删除
    docker rm 容器id                  # 删除指定容器，不能删除正在运行中的容器 
    docker rm -f $(docker ps -ap)     # 删除所用容器 
    docker ps -a -q|xargs docker rm   # 删除所有容器

#### 启动和停止容器
    docker start    # 启动
    docker restart  # 重启容器
    docker stop     #停止
    docker kill     # 强制停止

### 常用其他命令
#### 后台启动容器
    [root@shenhao ~]# docker run -d centos
    7edf4e37e20d47e474ba6c46de05574d42e53c457a18b65bd72fdc0d2fe5a49e
    [root@shenhao ~]# docker ps
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES


​    
​    # 常见的坑，docker容器使用后台运行，就必须要有一个前台进程，前台没有应用，就会自动停止

#### 查看日志
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

#### 显示容器内部进程信息
    docker top
    [root@shenhao ~]# docker top eac419b56757
    UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
    root                6413                6375                0                   17:02               ?                   00:00:00            /bin/sh -c while true;do echo 666;sleep 1;done
    root                6889                6413                0                   17:07               ?                   00:00:00            /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1

#### 查看镜像元数据
    [root@shenhao ~]# docker inspect --help
    
    Usage:  docker inspect [OPTIONS] NAME|ID [NAME|ID...]
    
    Return low-level information on Docker objects
    
    Options:
      -f, --format string   Format the output using the given Go template
      -s, --size            Display total file sizes if the type is container
          --type string     Return JSON for specified type


​    
​    [root@shenhao ~]# docker inspect eac419b56757
​    [
​        {
​            "Id": "eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c",
​            "Created": "2021-03-27T09:02:48.336698733Z",
​            "Path": "/bin/sh",
​            "Args": [
​                "-c",
​                "while true;do echo 666;sleep 1;done"
​            ],
​            "State": {
​                "Status": "running",
​                "Running": true,
​                "Paused": false,
​                "Restarting": false,
​                "OOMKilled": false,
​                "Dead": false,
​                "Pid": 6413,
​                "ExitCode": 0,
​                "Error": "",
​                "StartedAt": "2021-03-27T09:02:48.755176558Z",
​                "FinishedAt": "0001-01-01T00:00:00Z"
​            },
​            "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
​            "ResolvConfPath": "/var/lib/docker/containers/eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c/resolv.conf",
​            "HostnamePath": "/var/lib/docker/containers/eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c/hostname",
​            "HostsPath": "/var/lib/docker/containers/eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c/hosts",
​            "LogPath": "/var/lib/docker/containers/eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c/eac419b56757375559ea9437cdf5352567de8439ffc749a12fdd07a4f978724c-json.log",
​            "Name": "/dazzling_payne",
​            "RestartCount": 0,
​            "Driver": "overlay2",
​            "Platform": "linux",
​            "MountLabel": "",
​            "ProcessLabel": "",
​            "AppArmorProfile": "",
​            "ExecIDs": null,
​            "HostConfig": {
​                "Binds": null,
​                "ContainerIDFile": "",
​                "LogConfig": {
​                    "Type": "json-file",
​                    "Config": {}
​                },
​                "NetworkMode": "default",
​                "PortBindings": {},
​                "RestartPolicy": {
​                    "Name": "no",
​                    "MaximumRetryCount": 0
​                },
​                "AutoRemove": false,
​                "VolumeDriver": "",
​                "VolumesFrom": null,
​                "CapAdd": null,
​                "CapDrop": null,
​                "CgroupnsMode": "host",
​                "Dns": [],
​                "DnsOptions": [],
​                "DnsSearch": [],
​                "ExtraHosts": null,
​                "GroupAdd": null,
​                "IpcMode": "private",
​                "Cgroup": "",
​                "Links": null,
​                "OomScoreAdj": 0,
​                "PidMode": "",
​                "Privileged": false,
​                "PublishAllPorts": false,
​                "ReadonlyRootfs": false,
​                "SecurityOpt": null,
​                "UTSMode": "",
​                "UsernsMode": "",
​                "ShmSize": 67108864,
​                "Runtime": "runc",
​                "ConsoleSize": [
​                    0,
​                    0
​                ],
​                "Isolation": "",
​                "CpuShares": 0,
​                "Memory": 0,
​                "NanoCpus": 0,
​                "CgroupParent": "",
​                "BlkioWeight": 0,
​                "BlkioWeightDevice": [],
​                "BlkioDeviceReadBps": null,
​                "BlkioDeviceWriteBps": null,
​                "BlkioDeviceReadIOps": null,
​                "BlkioDeviceWriteIOps": null,
​                "CpuPeriod": 0,
​                "CpuQuota": 0,
​                "CpuRealtimePeriod": 0,
​                "CpuRealtimeRuntime": 0,
​                "CpusetCpus": "",
​                "CpusetMems": "",
​                "Devices": [],
​                "DeviceCgroupRules": null,
​                "DeviceRequests": null,
​                "KernelMemory": 0,
​                "KernelMemoryTCP": 0,
​                "MemoryReservation": 0,
​                "MemorySwap": 0,
​                "MemorySwappiness": null,
​                "OomKillDisable": false,
​                "PidsLimit": null,
​                "Ulimits": null,
​                "CpuCount": 0,
​                "CpuPercent": 0,
​                "IOMaximumIOps": 0,
​                "IOMaximumBandwidth": 0,
​                "MaskedPaths": [
​                    "/proc/asound",
​                    "/proc/acpi",
​                    "/proc/kcore",
​                    "/proc/keys",
​                    "/proc/latency_stats",
​                    "/proc/timer_list",
​                    "/proc/timer_stats",
​                    "/proc/sched_debug",
​                    "/proc/scsi",
​                    "/sys/firmware"
​                ],
​                "ReadonlyPaths": [
​                    "/proc/bus",
​                    "/proc/fs",
​                    "/proc/irq",
​                    "/proc/sys",
​                    "/proc/sysrq-trigger"
​                ]
​            },
​            "GraphDriver": {
​                "Data": {
​                    "LowerDir": "/var/lib/docker/overlay2/c7182b72ac77ebfe51a7cb598d4887771dd2e932eca2866836cb0bf9cda47d9f-init/diff:/var/lib/docker/overlay2/b0ffefc3829d7dda67b0610f1a77d46483cd519c9f42f83a030cbb282e1d0a9d/diff",
​                    "MergedDir": "/var/lib/docker/overlay2/c7182b72ac77ebfe51a7cb598d4887771dd2e932eca2866836cb0bf9cda47d9f/merged",
​                    "UpperDir": "/var/lib/docker/overlay2/c7182b72ac77ebfe51a7cb598d4887771dd2e932eca2866836cb0bf9cda47d9f/diff",
​                    "WorkDir": "/var/lib/docker/overlay2/c7182b72ac77ebfe51a7cb598d4887771dd2e932eca2866836cb0bf9cda47d9f/work"
​                },
​                "Name": "overlay2"
​            },
​            "Mounts": [],
​            "Config": {
​                "Hostname": "eac419b56757",
​                "Domainname": "",
​                "User": "",
​                "AttachStdin": false,
​                "AttachStdout": false,
​                "AttachStderr": false,
​                "Tty": false,
​                "OpenStdin": false,
​                "StdinOnce": false,
​                "Env": [
​                    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
​                ],
​                "Cmd": [
​                    "/bin/sh",
​                    "-c",
​                    "while true;do echo 666;sleep 1;done"
​                ],
​                "Image": "centos",
​                "Volumes": null,
​                "WorkingDir": "",
​                "Entrypoint": null,
​                "OnBuild": null,
​                "Labels": {
​                    "org.label-schema.build-date": "20201204",
​                    "org.label-schema.license": "GPLv2",
​                    "org.label-schema.name": "CentOS Base Image",
​                    "org.label-schema.schema-version": "1.0",
​                    "org.label-schema.vendor": "CentOS"
​                }
​            },
​            "NetworkSettings": {
​                "Bridge": "",
​                "SandboxID": "654e484c55dccc369715c34312a2f5c6a9024e81b37b78a5fb5fb421d4d59d20",
​                "HairpinMode": false,
​                "LinkLocalIPv6Address": "",
​                "LinkLocalIPv6PrefixLen": 0,
​                "Ports": {},
​                "SandboxKey": "/var/run/docker/netns/654e484c55dc",
​                "SecondaryIPAddresses": null,
​                "SecondaryIPv6Addresses": null,
​                "EndpointID": "c8f8f3382385d2cb5db0d73e8949aea6bc62da6ef7ab065a4a5dc532ce06700d",
​                "Gateway": "172.18.0.1",
​                "GlobalIPv6Address": "",
​                "GlobalIPv6PrefixLen": 0,
​                "IPAddress": "172.18.0.3",
​                "IPPrefixLen": 16,
​                "IPv6Gateway": "",
​                "MacAddress": "02:42:ac:12:00:03",
​                "Networks": {
​                    "bridge": {
​                        "IPAMConfig": null,
​                        "Links": null,
​                        "Aliases": null,
​                        "NetworkID": "cd4dd4c9c9f016ee4074d228681e21ecb604ec0b58496553d80527104a6c4f69",
​                        "EndpointID": "c8f8f3382385d2cb5db0d73e8949aea6bc62da6ef7ab065a4a5dc532ce06700d",
​                        "Gateway": "172.18.0.1",
​                        "IPAddress": "172.18.0.3",
​                        "IPPrefixLen": 16,
​                        "IPv6Gateway": "",
​                        "GlobalIPv6Address": "",
​                        "GlobalIPv6PrefixLen": 0,
​                        "MacAddress": "02:42:ac:12:00:03",
​                        "DriverOpts": null
​                    }
​                }
​            }
​        }
​    ]

#### 进入当前正在运行的容器
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


​    
​    # docker exec  进入容器后开启一个新的终端，可以在里面操作（常用）
​    # docker attach 进入容器正在执行的终端，不会启动新的终端


​    
#### 从容器内拷贝文件到容器外

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


​    ![image-20210328164036720](H:\MyCode\note\docker\img\image-20210328164036720.png)



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



## Docker可视化工具

- portainer

- Rancher(CI/CD时使用)



#### 什么是portainer

​	Docker图形化界面管理工具，提供一个后台面板供我们使用

~~~shell
docker run -d -p 8080:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
~~~







