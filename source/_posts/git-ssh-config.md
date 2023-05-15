---
title: git ssh 的配置
date: 2019-04-29 11:52:43
tags: [git, ssh]
categories: [技术]
---

git ssh 的配置，多个 git 仓库
<!-- more -->
### ssh key 配置

#####  生成 SSH key
```
ssh-keygen -t rsa -C "xiushilin@hotmail.com"
```

##### 多个 git 服务器

- 新建 ~/.ssh/config

```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa

Host git.dev.tencent.com
    HostName git.dev.tencent.com
    User git
    IdentityFile ~/.ssh/tencent_rsa

Host git2.dev.tencent.com
    HostName git.dev.tencent.com
    User git
    IdentityFile ~/.ssh/tencent2_rsa
```


- Host  别名，clone时替换为这个
- HostName git仓库的地址
- User 一般为git
- IdentityField 公钥文件


- 测试
```
ssh -T git@github.com
ssh -T git@git.dev.tencent.com
ssh -T git@git2.dev.tencent.com
```

