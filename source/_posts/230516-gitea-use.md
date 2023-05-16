---
title: gitea 的使用
date: 2023-05-15 16:20:00
categories: 
- technology
- instrument 
tags: git
---

搭建自己的 `git` 仓库，使用 [gitea](https://gitea.io)

<!-- more -->


### 使用 docker

`docker-compose.yml` 文件

```yaml
version: "3"

networks:
  gitea:
    external: false

services:
  server:
    image: gitea/gitea:1.19.3
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=mysql
      - GITEA__database__HOST=192.168.0.36:3306
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea110
      - DOMAIN=192.168.0.36:8030
      - SSH_DOMAIN=192.168.0.36:222
    restart: always
    networks:
      - gitea
    volumes:
      - ./giteadata:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "8030:3000"
      - "222:22"
```

注意⚠️：一定要把数据挂在到主机上，不然会导致数据丢失

```yaml
    volumes:
      - ./giteadata:/data
```

### `gitea 1.19.3`  

支持作为包管理仓库 [gitea packages](https://docs.gitea.io/en-us/usage/packages/overview/)

支持类似于 `github` 的 `action`  [gitea actions](https://docs.gitea.io/en-us/usage/usage/actions/overview/)

### 简单的应用

下载对应的 `act_runner`，地址 [act_runner download](https://dl.gitea.com/act_runner/)

注册并启动

```shell
./act_runner register --no-interactive --instance <instance> --token <token>

./act_runner daemon
```

创建一个 Spring demo 项目，在项目中新建 `gitea/workflows/deploy.yml`

```yaml
name: Gitea Maven Demo
run-name: ${{ gitea.actor }}  🚀
on: [push]

jobs:
  Explore-Gitea-Actions:
    runs-on: ubuntu-latest
    container:
      # 带有 docker 环境的容器
      image: catthehacker/ubuntu:act-latest
      # 持久化工具目录
      volumes:
        - ubuntu_hostedtoolcache:/opt/hostedtoolcache
    steps:
      - uses: actions/checkout@v3

      # 下载 java JDK 8
      - run: |
          download_url="http://192.168.0.35:5244/d/mac/OpenJDK8U-jdk_x64_linux_hotspot_8u372b07.tar.gz"
          wget -O $RUNNER_TEMP/java_package.tar.gz $download_url
      # 设置 java 环境
      - name: Set up JDK 8
        uses: actions/setup-java@v3
        with:
          java-version: '8'
          distribution: 'jdkfile'
          jdkFile: ${{ runner.temp }}/java_package.tar.gz
          cache: 'maven'
      # 配置 maven
      - name: Set up Maven
        uses: http://192.168.0.36:8030/actions/setup-maven@v4.7
        with:
          maven-version: 3.8.2
      - name: maven-settings-xml-action
        uses: http://192.168.0.36:8030/actions/maven-settings-xml-action@v20
        with:
          repositories: |
            [
              {
                "id": "aliyunpublic",
                "name": "aliyun Public",
                "url": "https://maven.aliyun.com/repository/public",
                "releases": {
                  "enabled": "true"
                },
                "snapshots": {
                  "enabled": "true"
                }
              }
            ]
          mirrors: >
            [
              {
                "id": "aliyunpublic",
                "mirrorOf": "*",
                "url": "https://maven.aliyun.com/repository/public"
              }
            ]
      # mvn build
      - name: Build with Maven
        run: mvn clean package

      # gitea docker registry login
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: 192.168.0.36:8030
          username: xiusl
          password:  ${{ security.REGISTRY_PWD }}
      
      # docker build and push
      - name: Build docker
        run: |
          docker build -t 192.168.0.36:8030/xiusl/demo:latest .
          docker push 192.168.0.36:8030/xiusl/demo:latest
```

⚠️ 本地的 `docker registry` 没有 `https`，因此在 `docker` 的 `daemon.json` 中需要增加信任
```json
{
"insecure-registries": [
    "192.168.0.36:8030"
  ]
}
```