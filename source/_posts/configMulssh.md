---
title: 如何配置多个ssh的id_rsa
date: 2021-05-02 10:58:56
toc: true
categories: 
  - Config Share
tags:
 - SSH
 - Linux
---



相信大多数开发人员用`ssh`免密登录会遇到这样的情况：

1. 用`ssh`本地登录自己的远程服务器
2. 用`ssh`本地配置自己的GitHub账号

但是，默认生成的`id_rsa`文件是首选，要怎么才能够让各个不同的应用场景对应各个不同的`id_rsa`呢？

<!-- more -->

> 由于以前胡乱配置过一些相关信息，可能你们有的问题我没出现，但是问题不大，把报错内容查一下可以解决的

## 生成ssh公钥和私钥

```shell
cd ~
# 查看是否有.ssh文件，它属于隐藏文件
# Windows下如果想在文件管理器下查看，需要设置可见隐藏文件
ll -a
# 如果有直接进入
cd .ssh
# 如果没有，会自动生成
# 但是执行这个，相当于指定认证了本机，一般适用于登录自己的服务器
ssh-keygen -t rsa
# 认证绑定GitHub用户邮箱，一般适用于GitHub
ssh-keygen -t rsa -C "example@example.com"
```

```shell
ssh-keygen -t rsa
# 它会提示以下内容，如果是第一次生成，三次enter就可以了，默认名字为id_rsa
# 如果有能力和兴趣，可以了解搜下它分别提示的内容
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa):
```

```shell
# 但是如果你又要生成GitHub的，它可能会覆盖原有的id_rsa，所以要自己设置下名称github_id_rsa
ssh-keygen -t rsa -C "example@example.com"
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa): github_id_rsa
```

之后会生成两组密钥，四个文件：`id_rsa`、 `id_rsa.pub`、 `github_id_rsa`、  `github_id_rsa.pub`  

`.pub`文件是公钥，是要发给服务器的，另一个是私钥，是要自己保存的

### Git配置时出现Could not open a connection to your authentication agent

先执行如下命令

```shell
ssh-add ~/.ssh/github_id_rsa
```

如果依然报错，则执行下面的命令

```shell
eval `ssh-agent`  #（是～键上的那个` 反引号） 
ssh-add ~/.ssh/github_id_Rsa
ssh-add -l
```

或者

```bash
ssh-agent bash 
ssh-add ~/.ssh/github_id_Rsa
ssh-add -l
```



## 配置服务器

### GitHub

【点击头像】=>【Settings】=>【SSH and GPG keys】=>【New SSH key】

`Title`随便命名一个自己能够区别的名称

`Key`内容是相应的公钥内容，也就是`github_id_rsa.pub`内容

### Linux服务器

```shell
cd .ssh
# 如果没有
ssh localhost
# 查看是否又authorized_keys文件
ls
# 如果没有
touch authorized_keys
# 然后将本地的 id_rsa.pub 复制到里面
```

## 设置本地.ssh文件

```shell
touch config
```

填写转发内容，其中 `IdentityFile` 需要是绝对路径，相对路径会报错

```
Host 39.108.179.50 # 别名，不重要
HostName 39.108.179.50 # 对应网站的url，重要！
User root
IdentityFile ~/.ssh/id_rsa

Host github.com # 别名，不重要
HostName github.com # 对应网站的url，重要！
User WYDgits
IdentityFile ~/.ssh/github_id_rsa
```

## 验证

```shell
ssh -T git@github.com
# ssh -Tv git@github.com 来debug
# 提示信息
# Hi <yourname>! You've successfully authenticated, but GitHub does not provide shell access.
ssh root@39.108.179.50
# 成功的话能够直接登录进去服务器的bash
```

## 参考

[是否必须每次添加ssh-add](https://segmentfault.com/q/1010000000835302)

[Git配置时出现Could not open a connection to your authentication agent](https://blog.csdn.net/qq_19393857/article/details/81629431)

