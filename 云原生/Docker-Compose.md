- [docker-compse 概念](#docker-compse-概念)
- [compose模版文件](#compose模版文件)
- [命令](#命令)

# docker-compse 概念
Compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。

Compose 使用的三个步骤：

- 使用 Dockerfile 定义应用程序的环境。
- 使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
- 最后，执行 docker-compose up 命令来启动并运行整个应用程序。

# compose模版文件
```dockerfile
#指定compose配置文件呢版本
version: ''

#服务配置
services:
  服务1: web
    #服务配置
    image  #镜像 如果镜像不存在，Compose会自动拉取镜像，除非指定了build，这种情况下会使用指定选项构建镜像并给镜像打上指定标签。
    build  #构建应用的配置项。一般直接指定Dockerfile所在 文件夹路径，可以是绝对路径，或者相对于Compose配置文件的路径
    network
    ...
  服务2: redis
    #服务配置
    images
    build
    network
    container_name:
    ports:
    environment:
    command:
    ...
 ...
 
 #其他配置
 volumes:
 networks:
 configs:
```

# 命令

 - -f: 指定文件
 - up: 启动
 - -d: 后台启动
 - --build: 重建image


```bash
docker-compose -f misc/compose/docker-compose.wlx.yml up -d --build scene_editor
```
