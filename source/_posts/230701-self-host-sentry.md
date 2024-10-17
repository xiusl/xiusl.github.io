---
title: 部署自己的 sentry 服务
date: 2023-07-01 09:29:32
categories: technology
tags:
  - sentry
---



自建 [sentry](https://sentry.io) 服务，作为错误信息监控。采用的是旧的 9.1.2 版本，如需最新版本，参考官方文档。

<!-- more -->

推荐在 Debain 系统上进行部署，之前使用了 REHL 系列的系统，`self-hosted-912-web`  服务会分配和占用很大的内存，具体什么原因还不清楚。



主机上需要安装好 `docker` 服务



在 `github` 上下载官方提供的部署脚本（附：[下载地址](https://github.com/getsentry/self-hosted/archive/refs/tags/9.1.2.zip)），解压文件。



由于目前安装的 `docker compose` 是最新的版本，需要修改 `install.sh` 中的部分代码，主要是替换 `docker-compose` 为 `docker compose`，完成代码参考 [install.sh](https://gist.github.com/xiusl/ea846430ab10561ddb5b569409186568)

```shell
# line 24: get docker compose version
COMPOSE_VERSION=$(docker compose version --short || echo '')

# line 68: docker-compose to docker compose
SECRET_KEY=$(docker compose run --rm web config generate-secret-key 2> /dev/null | tail -n1 | sed -e 's/[\/&]/\\&/g')

# line 75, 83: docker-compose to docker compose
# ...
```



执行修改后的 `install.sh` 脚本

```shell
./install.sh
```

脚本执行过程中会提示是否创建 sentry 账号，可以选择在这个地方创建



脚本执行完成后，启动服务

```sh
docker compose up -d
```



查询服务状态，正常会有以下 7 个服务运行

```
CONTAINER ID   IMAGE                    COMMAND                  CREATED        STATUS        PORTS                                       NAMES
9b1e803d7dcf   self-hosted-912-worker   "/entrypoint.sh run …"   15 hours ago   Up 15 hours   9000/tcp                                    self-hosted-912-worker-1
6b4977a45949   self-hosted-912-web      "/entrypoint.sh run …"   15 hours ago   Up 15 hours   0.0.0.0:9000->9000/tcp, :::9000->9000/tcp   self-hosted-912-web-1
199f1760fc26   self-hosted-912-cron     "/entrypoint.sh run …"   15 hours ago   Up 15 hours   9000/tcp                                    self-hosted-912-cron-1
1d2cf8c084ce   postgres:9.5             "docker-entrypoint.s…"   15 hours ago   Up 15 hours   5432/tcp                                    self-hosted-912-postgres-1
b0d87c431e33   memcached:1.5-alpine     "docker-entrypoint.s…"   15 hours ago   Up 15 hours   11211/tcp                                   self-hosted-912-memcached-1
0b0077bfe895   redis:3.2-alpine         "docker-entrypoint.s…"   15 hours ago   Up 15 hours   6379/tcp                                    self-hosted-912-redis-1
0e90ca779b36   tianon/exim4             "docker-entrypoint.s…"   15 hours ago   Up 15 hours   25/tcp                                      self-hosted-912-smtp-1
```



通过网页打开 `http://your-host-ip:9000` 登录创建好的账号就可以使用了

![WX20230701-095613@2x](//img-uss.likehub.top/uPic/2023-07-01/TANnbo-WX20230701-095613@2x.png)



可以在前面加一层 `nginx` 代理，实现自定义域名和 `https` 访问

```nginx
server{
	listen 80;
	listen 443 ssl;
	server_name sentry.you-domain.com;

	ssl_certificate      cert/you-domain.crt;
	ssl_certificate_key  cert/you-domain.key;

  ssl_session_cache    shared:SSL:1m;
  ssl_session_timeout  5m;
  ssl_ciphers  HIGH:!aNULL:!MD5;
  ssl_prefer_server_ciphers  on;

	location / {
		proxy_set_header X-Forwarded-Host $host;
		proxy_set_header X-Forwarded-Server $host;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header Host $host;
		proxy_pass http://127.0.0.1:9000;
	}
}
```



linux 增加 swap 空间

```sh
# 分配空间
fallocate -l 8G /swapspace
# 增加权限
chmod 600 /swapspace
# 格式化文件
mkswap /swapspace
# 启用交换空间
swapon /swapspace
# 开启自动启动
echo "/swapspace swap swap defaults 0 0" | tee -a /etc/fstab

# 检查是否生效
swapon --show
free -h
```



