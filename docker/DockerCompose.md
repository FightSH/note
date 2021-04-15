# Docker Compose

[官网文档](https://docs.docker.com/compose/)

## 简介

Docker Compose高效的管理容器，定义运行多个容器。批量容器编排

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see [the list of features](https://docs.docker.com/compose/#features).

Compose works in all environments: production, staging, development, testing, as well as CI workflows. You can learn more about each case in [Common Use Cases](https://docs.docker.com/compose/#common-use-cases).

Using Compose is basically a **three-step** process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Run `docker compose up` and the [Docker compose command](https://docs.docker.com/compose/cli-command/) starts and runs your entire app. You can alternatively run `docker-compose up` using the docker-compose binary.



> 核心：多容器，YAML file配置文件（如下） ，命令
>
> Compose是官方的开源项目，需要安装。
>
> - 服务services。单个容器，应用
> - 项目project。一组关联的容器

~~~yaml
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
~~~



## 安装

~~~shell
# 官方地址
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 备用地址（速度快）
curl -L https://get.daocloud.io/docker/compose/releases/download/1.29.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 授权
sudo chmod +x /usr/local/bin/docker-compose
~~~



## Getting Started



# Docker Swarm（集群）







