---
title: openshift-deploy-flask
date: 2017-09-10 22:31:22
tags: 
- openshift 
- flask
- mongodb
categories: 
- technology
- Python
---

> 新版 Openshift 部署 Flask+mongodb 应用

<!-- more -->
### Openshift

- [Openshift](https://openshift.com) 是红帽的云开发平台即服务 (Pass)。之前的旧版本已经停止使用，新的 Openshift V3 已经全面的开放使用。
- 新版本的免费空间提供 1 Project (1G Memory, 1G Storage)，对于我们部署一个简单的应用足够了。
- 想要开始需要先注册 Openshift 的账号，具体注册流程不做介绍，可以参考[免费资源部落](https://www.freehao123.com/openshift-1g-php/)的第一步。

### Project

- 登录账号后就就可以创建 Project
![img](http://image.sleen.top/hexo/create-2.png)

- 点击 Create 后，会要求我们选择想要语言、技术等，因为要使用 MongoDB，所有先选择 Data Stores
![img](http://image.sleen.top/hexo/data.png)

### MongoDB
- 接下来在选择 MongoDB(Persisent)
![img](http://image.sleen.top/hexo/mongo.png)

- 之后的页面需要我们对 MongoDB 设置一些参数
![img](http://image.sleen.top/hexo/mongo-config.png)

- 等待一段时间就会创建成功，创建成功后我们可以通过 面板的 Applications-Service 
![img](http://image.sleen.top/hexo/mongo-service.png)

- 来查看 MongoDB 的主机地址
![img](http://image.sleen.top/hexo/mongo-host.png)

### Flask
- 接下来添加 Flask 应用，点击顶部的 Add to Project - Browse Catalog，接下来的页面选择 Python，还需要选择 Python 版本 (我选择的是 2.7)。
- 设置 Flask 应用的一些信息，设置 git 仓库 (可以参考我的代码https://github.com/SleenXiu/openshift-flask.git)
![img](http://image.sleen.top/hexo/careate-flask.png)

- 点击 advanced options 填写一些额外的配置 
![img](http://image.sleen.top/hexo/deploy-config.png)

- 设置内存限制
![img](http://image.sleen.top/hexo/deploy-limit.png)

- 设置git push 自动部署
![img](http://image.sleen.top/hexo/git-buid.png)
![img](http://image.sleen.top/hexo/git-hook.png)

- 数据库配置
   ```
   // 填写上面 MongoDB 中填写的信息
   MONGODB_SETTINGS = {
       'db': 'xiudb',
       'host': '172.30.224.230:27017', 
       'username': 'xiu',
       'password': 'xiu'
   }
   ```
  
### 最后

- 利用业余时间研究了很久，终于完成了新版 Openshift 的部署，[小站](http://xiu.insword.cn)，算是跑通了流程吧，有时间再去深入研究下 Docker
- 第一次使用 Docker，还有很多不了解的地方
- 前期也就先这样吧，特别感谢[RedHat](https://www.redhat.com)，一直是使用它的免费云平台来学习。