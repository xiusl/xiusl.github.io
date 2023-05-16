---
title: nginx 配置 gzip
date: 2019-09-26 10:57:36
categories:
- technology
tags:
- nginx
- gzip
---

nginx 配置开启 gzip 压缩
<!-- more -->


```
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_comp_level 2;
    gzip_types text/plain application/json application/javascript application/x-javascript text/css application/xml text/javascript image/jpeg image/gif image/png;
    gzip_vary off;
```
