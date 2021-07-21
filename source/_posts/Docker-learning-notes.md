---
title: Docker笔记总结
date: 2020-03-15 14:11:39
toc: true
excerpt: (^_^)
categories: 
  - Study Share
tags:
  - Docker
  - Linux
---



# Docker安装部署

## CentOS

- yum 包更新到最新（花的时间比较多）
	
	```bash
	yum update
	```
- 安装需要的软件包，yum-util 提供的yum-config-manager功能，另外两个是devicemapper驱动依赖的
	
	```bash
	yum install -y yum-utils device-mapper-persistent-data lvm2
	```
- 设置yum源
	
	```bash
	yum-confi-manager \
	--add-repo https://download.docker.com/linux/centos/docker-ce.repo
	```
- 安装docker，出现输入的页面都按 y
	
	```bash
	yum install -y docker-ce
	```
- 查看docker版本，验证是否安装成功

	```bash
	docker -v
	```

### 其他安装方式(推荐)

教程链接：[Here!](https://www.jianshu.com/p/1e5c86accacb)
# Docker命令

## Docker服务相关命令
- 启动docker服务

	```bash
	systemctl start docker
	systemctl start docker
	```
- 停止docker服务

	```bash
	systemctl stop docker
	```
- 重启docker服务

	```bash
	systemctl restart docker
	```
- 查看docker服务状态

	```bash
	systemctl status docker
	```
- 设置开机启动docker服务

	```bash
	systemstl enable docker
	```

## Docker镜像相关命令
- 查看镜像：查看本地所有的镜像

	```bash
	docker images
	docker images -q # 查看所有镜像id
	```
- 搜索镜像：从网络中查找需要的镜像

	```bash
	docker search 镜像名称
	```
- 拉取镜像：从docker仓库下载镜像到本地，镜像名称格式为 名称:版本号，如果版本号不指定则是**最新版本**。如果不知道镜像版本，可以去[docker hub](http://hub.docker.com) 搜索对应镜像查看

	```bash
	docker pull 镜像名称
	```
- 删除镜像

	```bash
	docker rmi 镜像id
	docker rmi `docker images -q` # 删除所有本地镜像
	```

## Docker容器相关的命令
- 查看容器

	```bash
	docker ps # 查看正在运行的容器
	docker ps -a # 查看所有容器
	```
	
- 创建并启动容器

  ```bash
  docker run 参数
  docker run \ 
  --name apache \
  -v /home/myBlog/public/:/usr/local/apache2/htdocs/ \
  -p 80:80 \
  -d httpd
  ```

  - 参数说明：
    - --name：为创建的容器命名。
    - -v：设置数据卷。前者是宿主机的目录，后者是容器内的目录。
    - -i：保持容器运行。通常与`-t`同时使用。加入it这两个参数后，容器创建后自动进入容器中，退出容器后，容器自动关闭。
    - -t：为容器重新分配一个伪输入终端，通常与-i同时使用。
    - -d：以守护（后台）模式运行容器。创建一个容器在后台运行，需要使用`docker exec`进入容器。退出时，容器不会关闭。
    - -it：创建的容器一般称为**交互式容器**。
    - -id：创建的容器一般称为**守护式容器**。
    - -p：端口映射。前者是宿主机端口，后者是容器端口。
    - httpd：指用`httpd`镜像为基础启动容器。

  ```bash
  docker run -it --rm ubuntu:16.04 bash
  ```

  - 参数说明
    - --rm：这个参数是说容器退出后随之将其删除
    - ubuntu:16.04：这是指用ubuntu:16.04镜像为基础来启动容器。
    - bash：放在镜像名后的是命令，这里我们希望有个交互式shell,因此用的是bash。

- 进入容器

	```bash
	docker exec 容器名称 操作命令等参数  # 退出容器，容器不会关闭 ，而且
	
	# 例如：我要查看myblog为名称的容器的路径/usr/local/apache2
	docker exec myblog ls /usr/loacl/apache2 
	# 例如：我要进入myblog为名称的容器内用bash操作
	docker exec myblog -it /bin/bash
	```
	
	- 参数说明
	  - 保持容器运行。通常与`-t`同时使用。加入it这两个参数后，容器创建后自动进入容器中，退出容器后，容器自动关闭。
	  - -t：为容器重新分配一个伪输入终端，通常与-i同时使用。
	  - -it：创建的容器一般称为**交互式容器**。
	
- 启动容器

	```bash
	docker start 容器名称
	```
	
	
	
- 停止容器

	```bash
	docker stop 容器名称
	```
	
	
	
- 删除容器：如果容器是运行状态则删除失败，需要停止容器才能删除

	```bash
	docker rm 容器名称
	```
	
	
	
- 查看容器信息

	```bash
	docker inspect 容器名称
	```

- 导入容器

  ```bash
  docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
  # file
  docker import /path/to/exampleimage.tgz
  # -
  cat exampleimage.tgz | docker import - exampleimagelocal:new
  # URL
  docker import http://study.163.com/image.tgz example/imagerepo
  ```

  

- 导出容器

  ```bash
  docker export 容器名称 > 导出文件名.tar
  ```

  

# Docker容器的数据卷

## 数据卷概念和作用

### 思考
- Docker容器删除后，在容器中产生的数据也会随之销毁吗？
	- 会。
- Docker容器和外部机器可以直接交换文件吗？
	- 不可以。
- 容器之间想要进行数据交互？
	- 不可以。

>*那咋办嘛？这就要用到数据卷了*

### 数据卷
- 数据卷是宿主机中的一个**目录或文件**
- 当容器目录和数据卷目录绑定后，对方的修改会立即同步
- 一个数据卷可以被**多个容器同时挂载**
- 一个容器也可以被挂载**多个数据卷**

### 数据卷的作用
- 容器数据持久化
- 外部机器和容器间接通信
- 容器之间数据交换

## 配置数据卷
- 创建启动容器时，使用`-v`参数设置数据卷

		docker run ...-v 宿主机目录(文件):容器内目录(文件)...
- 注意事项：
	1. 目录必须是**绝对路径**
	2. 如果目录不存在，会**自动创建**
	3. 可以挂载**多个**数据卷

## 数据卷容器
### 配置数据卷容器
- 创建启动c3数据卷容器，使用`-v`参数设置数据卷

		docker run -it --name=c3 -v /volume centos:7 /bin/bash
- 创建启动c1 c2数据卷容器，使用`-volumes-from`参数设置数据卷

		docker run -it --name=c1 -volumes-from c3 centos:7 /bin/bash
		docker run -it --name=c2 -volumes-from c3 centos:7 /bin/bash

# Docker应用部署
## MySQL部署
- 搜索MySQL镜像

		docker search mysql
- 拉取MySQL镜像

		docker pull mysql:5.6
- 创建容器，设置端口映射、目录映射

		# 在/root目录下创建mysql目录用于存储mysql数据信息
		mkdir ~/mysql
		cd ~/mysql

	---
		docker run -id \
		  --name=c_mysql \
		  -p 3307:3306 \
		  -v $PWD/conf:/etc/mysql/confi.d \
		  -v $PWD/logs:/logs \
		  -v $PWD/data:/var/lib/mysql \
		  -e MYSQL_ROOT_PASSWORD=123456 \
		  mysql:5.6
	- 参数说明：
		- -p 3307:3306：将容器的3306端口映射到宿主机的3307端口。
		- -v $PWD/conf:/etc/mysql/confi.d：将主机当前目录下的conf/my.cnf挂载到容器的/etc/mysql/my.cnf。配置目录
		- -v $PWD/logs:/logs：将主机当前目录下的logs目录挂载到容器的/logs。日志目录
		- -v $PWD/data:/var/lib/mysql：将主机当前目录下的data目录挂载到容器的/var/lib/mysql。数据目录
		- -e MYSQL_ROOT_PASSWORD=123456：初始化root用户的密码。

## Tomcat部署
- 搜索Tomcat镜像

		docker search tomcat
- 拉取Tomcat镜像

		docker pull tomcat
- 创建容器，设置端口映射、目录映射

		# 在/root目录下创建tomcat目录用于存储tomcat数据信息
		mkdir ~/tomcat
		cd ~/tomcat

	---
		docker run -id \
		  --name=c_tomcat \
		  -p 8080:8080 \
		  -v $PWD:/usr/local/tomcat/webapps \
		  tomcat
	- 参数说明：
		- -p 8000:8080：将容器的8080端口映射到宿主机的8000端口。
		- -v $PWD:/usr/local/tomcat/webapps：将主机当前目录挂载到容器的/usr/local/tomcat/webapps。

## Nginx部署
- 搜索Nginx镜像

		docker search nginx
- 拉取Nginx镜像

		docker pull nginx
- 创建容器，设置端口映射、目录映射

	```bash
	# 在/root目录下创建nginx目录用于存储nginx数据信息
	mkdir ~/nginx
	cd ~/nginx
	mkdir conf
	cd conf
	# 在~/nginx/conf/下创建nginx.conf文件，粘贴下面内容
	vim nginx.conf
	```
	
	---
	```bash
	user  nginx;
	worker_processes  1;
	
	error_log  /var/log/nginx/error.log warn;
	pid        /var/run/nginx.pid;
	
	events {
	    worker_connections  1024;
	}
	
	http {
	    include       /etc/nginx/mime.types;
	    default_type  application/octet-stream;
	
	    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	                      '$status $body_bytes_sent "$http_referer" '
	                      '"$http_user_agent" "$http_x_forwarded_for"';
	
	    access_log  /var/log/nginx/access.log  main;
	
	    sendfile        on;
	    #tcp_nopush     on;
	
	    keepalive_timeout  65;
	
	    #gzip  on;
	
	    include /etc/nginx/conf.d/*.conf;
	}
	```
	
	---
	```bash
	docker run -id \
	  --name=c_nginx \
	  -p 81:80 \
	  -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
	  -v $PWD/logs:/var/log/nginx \
	  -v $PWD/html:/usr/share/nginx/html \
	  nginx
	```
	
	- 参数说明：
		- -p 81:80：将容器的80端口映射到宿主机的81端口。
		- -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf：将主机当前目录下的/conf/nginx.conf挂载到容器的/etc/nginx/nginx.conf。配置目录
		- -v $PWD/logs:/var/log/nginx：将主机当前目录下的logs目录挂载到容器的/var/log/nginx。日志目录
		- -v $PWD/html:/usr/share/nginx/html：将主机当前目录下的/html挂载到容器的/usr/share/nginx/html。

## Redis部署
- 搜索Redis镜像

	```bash
	docker search redis
	```
- 拉取Redis镜像

	```bash
	docker pull redis:5.0
	```
- 创建容器，设置端口映射、目录映射

	```bash
	docker run -id --name=c_redis -p 6379:6379 redis:5.0
	```
	- 参数说明：
		- -p 6379:6379：将容器的6379端口映射到宿主机的6379端口。
- 使用外部机器连接redis
		
	
	```bash
	./redis-cli.exe -h <your ipAddress> -p 6379	
	```

# Dockerfile
## Docker镜像原理
### 思考
- Docker镜像本质是什么？
	- 是一个分层的文件系统
- Docker中一个centos镜像为什么只有200MB，而一个centos操作系统的`iso`文件要几个GB？
	- Centos的`iso`镜像文件包含`bootfs`和`rootfs`，而docker的centos镜像**复用**操作系统的`bootfs`，只包含`rootfs`和其他镜像层
- Docker中一个tomcat镜像为什么有500MB，而一个tomcat安装包只有70多MB？
	- 由于docker中镜像是分层的，tomcat虽然只有70多MB，但它需要依赖于**父镜像**和**子镜像**，所有整个对外暴露的tomcat镜像大小有500多MB

### Linux文件系统
- bootfs：包含`bootloader`（引导加载系统）和`kernel`（内核）
- rootfs：root文件系统，包含的就是典型的Linux系统中的`/dev`，`/proc`，`/bin`，`/etc`等标准目录和文件
- 不用的Linux发行版，`bootfs`基本一样，而`rootfs`不同，如Ubuntu，centos等

![](layer1.png)

### Docker镜像
- Docker镜像是由特殊的文件系统叠加而成
- 最底端是`bootfs`,并使用宿主机的`bootfs`
- 第二层是root文件系统`rootfs`,称为`base image`
- 然后再往上可以叠加其他的镜像文件
- ***统一文件系统(Union File System)***技术能够将不同的层整合成一个文件系统,为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来,只存在一个文件系统。
- 一个镜像可以放在另一个镜像的上面。位于下面的镜像称为***父镜像***，最底部的镜像成为***基础镜像***。
- 当从一个镜像启动容器时，Docker会从最顶层加载一个**读写文件系统**作为容器

![](layer2.png)
## 镜像制作
- 容器转为镜像

	```bash
	docker commit 容器id 镜像名称:版本号  # 将容器转换为镜像文件
	docker save -o 压缩文件名称 镜像名称:版本号  # 将镜像文件打包成压缩文件，之后就能对压缩文件传送了
	docker load -i 压缩文件名称  # 将压缩文件解压称为镜像文件
	```
- Dockerfile
	
	- *看下面内容*

## Dockerfile概念及作用
### 概念
- Dockerfile是一个文本文件
- 包含了一条条的指令
- 每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像
- 对于开发人员：可以为开发团队提供一个完全一致的开发环境
- 对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了
- 对于运维人员：在部署时，可以实现应用的无缝移植

![](dockerfile1.png)
## Dockerfile关键字
*列举一些常用的*

- `FROM <image name>`: 指定构建使用的基础镜像
  - 例: `FROM ubuntu:14.04`
- `MAINTAINER <author name>`: 创建者信息
  - 例: `MAINTAINER adyo "xxxx@qq.com"`
- `ENV`: 设置环境变量
  - 例: `ENV REFRESHED _AT 2017-03-16`
- `RUN <command>`: 在shell或者exec的环境下执行一条命令.`RUN`指令会在新创建的镜像上添加新的层面，接下来提交的结果可以用在Dockerfile的下一条指令中
  - 例: `RUN apt-get -yqq update`
- `ADD <source> <destination>`: 从当前目录复制文件到容器, source可以是URL或者是启动配置上下文中的一个文件, destination是容器内的路径. 会自动处理目录, 压缩包等情况
  - 例: ``
- `COPY <source> <destination>`: 从当前目录复制文件到容器. 只是单纯地复制文件.
  - 例：`COPY index.html /var/www/html`
- `VOLUME [ "/data" ]`: 声明一个数据卷, 可用于挂载, `[]`里面是路径
  - 例: `VOLUME [ "/var/lib/redis", "/var/log/redis" ]`
- `USER <uid>`: 镜像正在运行时设置的一个UID,`RUN`命令执行时的用户
- `WORKDIR`: 指定`RUN`、`CMD`与`ENTRYPOINT`命令的工作目录
  - 例: `WORKDIR /opt/nodeapp`
- `ONBUILD`: 前缀命令, 放在上面这些命令前面, 表示生成的镜像再次作为"基础镜像"被用于构建时要执行的命令
- `ENTRYPOINT`: 配置给容器一个可执行的命令,这意味着在每次使用镜像创建容器时一个特定的应用程序可以被设置为默认程序.同时也意味着该镜像每次被调用时仅能运行指定的应用.类似于`CMD`,`Docker`只允许一个`ENTRYPOINT`,多个`ENTRYPOINT`会只执行最后的`ENTRYPOINT`指令
  - 例: `ENTRYPOINT [ "nodejs", "server.js" ]`
- `CMD`: 提供了容器默认的执行命令,Dockerfile只允许使用一次CMD指令. 使用多个CMD只有最后一个指令生效
  - 例: `CMD [ "/bin/true" ]`
- `EXPOSE <port>`: 指定容器在运行时监听的端口
  - 例: `EXPOSE 3000`

**附加：**

如果对与`ADD` 与 `COPY`有疑惑或者不太会区分的话，可以参考此篇文章，这篇文章讲得很细节了：

[Docker ADD vs. COPY: What are the Differences?](https://phoenixnap.com/kb/docker-add-vs-copy)

## 制作自定义centos镜像

### 自定义需求
- 默认登录路径为`/usr`
- 可以使用vim
### 操作
- 创建编辑dockerfile文件

	```bash
	mkdir /root/dockerfile
	cd dockerfil
	vim centos_dockerfile
	```
	- 定义父镜像：`FROM centos:7`
	- 定义作者信息：`MAINTAINER adongyo <adongyo@it.cn>`
	- 执行安装vim命令：`RUN yum install -y vim`
	- 定义默认的工作目录：`WORKDIR /usr`
	- 定义容器启动执行的命令：`CMD /bin/bash`
- 执行命令
		
	```bash
	docker build -f ./centos_dockerfile -t myCentos:1 .
	```
	- 参数说明：	
		- -f： 指定dockerfile文件
		- -t： 设置生成的新的镜像的名称
		- .： 别漏了后面还有一个'.'



# 参考资料

- [b站转载黑马程序员](https://www.bilibili.com/video/BV1CJ411T7BK)