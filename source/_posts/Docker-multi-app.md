---
title: Docker多容器应用
toc: true
categories:
  - Study Share
tags:
  - Docker
date: 2021-07-15 11:02:20
---



## 前言

:wind_face: *本篇文章大部分是docker的官方英文文档内的内容，自己对这些内容进行了尝试，然后加了一些自己的解释和实践。这是比较简单的多容器应用，未来会再尝试构建更复杂的多容器应用的。*

我们一直在使用单容器应用程序。但是，我们现在想要将 MySQL 添加到应用程序中。经常会出现下面的问题——“MySQL会在哪里运行？安装在同一个容器中还是单独运行？” 

**Tips**：如果想了解单容器应用或者更多基础内容，可以看 [Our-application](https://github.com/docker/getting-started/blob/master/docs/tutorial/our-application/index.md)、[Docker-learning-notes](https://wydgits.github.io/Docker-learning-notes/)
<!-- more -->

一般来说，**每个容器都应该做一件事，并且做好。** 因此，我们将更新我们的应用程序，使其工作如下：

![](https://cdn.wydgit.top/blog/docker-multi-app/multi-app-architecture.png)

## 容器网络

要记住，默认情况下，容器是独立运行的，并且对同一台机器上的其他进程或容器一无所知。那么，我们如何让一个容器与另一个容器通信呢？答案是**网络**。

> 如果两个容器在同一个网络上，它们可以相互通信。如果他们不在，他们就不能相互通信。

## 启动 MySQL

有两种方法可以将容器放在网络上：

1) 在开始时分配它

2) 连接现有的容器

现在，我们将首先创建网络并在启动时附加 MySQL 容器。

1. 创建网络。

   ```bash
   docker network create todo-app
   ```

2. 启动一个 MySQL 容器并将其连接到网络。我们还将定义一些数据库将用于初始化数据库的环境变量（请参阅[MySQL Docker Hub 列表中](https://hub.docker.com/_/mysql/)的“环境变量”部分）。

   ```bash
   docker run -d \
       --network todo-app --network-alias mysql \
       -v todo-mysql-data:/var/lib/mysql \
       -e MYSQL_ROOT_PASSWORD=secret \
       -e MYSQL_DATABASE=todos \
       mysql:5.7
   ```

   如果你使用的是 PowerShell，请使用此命令。

   ```bash
   docker run -d `
       --network todo-app --network-alias mysql `
       -v todo-mysql-data:/var/lib/mysql `
       -e MYSQL_ROOT_PASSWORD=secret `
       -e MYSQL_DATABASE=todos `
       mysql:5.7
   ```

   - -d：`daemon`简写，表示后台运行
   - --network todo-app：选择之前创建的网络名称
   - --network-alias mysql：设置别名
   - -v todo-mysql-data:/var/lib/mysql：设置数据卷，前者为宿主机文件，后者为容器内文件
   - -e MYSQL_ROOT_PASSWORD=secret ：设置数据库root用户的密码
   - -e MYSQL_DATABASE=todos： 创建一个名为todos的数据库
   - mysql:5.7：选择`mysql:5.7`的镜像image

   我们指定了`--network-alias`标志。稍后我们会回到这个话题。

   > 提示
   >
   > 注意到我们使用了一个名为`todo-mysql-data`here的卷并将其安装在`/var/lib/mysql`，这是 MySQL 存储其数据的地方。但是，我们从未运行过`docker volume create`命令。Docker 识别出我们想要使用命名卷并自动为我们创建一个。

3. 要确认我们已启动并运行数据库，请连接到数据库并验证它已连接。

   ```bash
   docker exec -it <mysql-container-id> mysql -p
   ```

   当密码提示出现时，输入**secret**。在 MySQL shell 中，列出数据库并验证是否看到了`todos`数据库。

   ```mysql
   mysql> SHOW DATABASES;
   ```

   会看到如下所示的输出：

   ```mysql
   +--------------------+
   | Database           |
   +--------------------+
   | information_schema |
   | mysql              |
   | performance_schema |
   | sys                |
   | todos              |
   +--------------------+
   5 rows in set (0.00 sec)
   ```

   欧耶！我们有我们的`todos`数据库，可以使用了！

## 连接到 MySQL

现在我们知道 MySQL 已启动并运行，让我们使用它！但是，问题是……如何？如果我们在同一网络上运行另一个容器，我们如何找到该容器（记住每个容器都有自己的 IP 地址）？

为了解决这个问题，我们将使用[nicolaka/netshoot](https://github.com/nicolaka/netshoot)容器，它附带了许多可用于故障排除或调试网络问题的工具。

*感觉这一步来创建这个新容器有点多余，其实你们可以不用弄，这里是为了解释这个主机名的由来*

1. 使用 nicolaka/netshoot 镜像启动一个新容器。确保将其连接到同一网络。

   ```bash
   docker run -it --network todo-app nicolaka/netshoot
   ```

2. 在容器内部，我们将使用`dig`命令，这是一个有用的 DNS 工具。我们将查找主机名的 IP 地址`mysql`。

   ```bash
   dig mysql
   ```

   你会得到这样的输出......

   实例的输出：
   
   ```bash
   ; <<>> DiG 9.14.1 <<>> mysql
   ;; global options: +cmd
   ;; Got answer:
   ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32162
   ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
   
   ;; QUESTION SECTION:
   ;mysql.             IN  A
   
   ;; ANSWER SECTION:
   mysql.          600 IN  A   172.23.0.2
   
   ;; Query time: 0 msec
   ;; SERVER: 127.0.0.11#53(127.0.0.11)
   ;; WHEN: Tue Oct 01 23:47:24 UTC 2019
   ;; MSG SIZE  rcvd: 44
   ```

   我的输出：
   
   ```bash
   ; <<>> DiG 9.16.16 <<>> mysql
   ;; global options: +cmd
   ;; Got answer:
   ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3033
   ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
   
   ;; QUESTION SECTION:
   ;mysql.                         IN      A
   
   ;; ANSWER SECTION:
   mysql.                  600     IN      A       172.18.0.2
   
   ;; Query time: 0 msec
   ;; SERVER: 127.0.0.11#53(127.0.0.11)
   ;; WHEN: Tue Jul 20 04:42:44 UTC 2021
   ;; MSG SIZE  rcvd: 44
   
   ```
   
   在“**ANSWER SECTION**”中，你将看到解析为的`A`记录（你的 IP 地址很可能具有不同的值）。虽然通常不是有效的主机名，但`Docker`能够将其解析为具有该网络别名的容器的 IP 地址（还记得我们之前使用的标志吗？）。
   
   这意味着……我们的应用程序只需要连接到一个名为的主机`mysql`，它就会与数据库对话！

## 使用 MySQL 运行我们的应用程序

todo 应用程序支持设置一些环境变量来指定 MySQL 连接设置。他们是：

- `MYSQL_HOST` - 正在运行的 MySQL 服务器的主机名
- `MYSQL_USER` - 用于连接的用户名
- `MYSQL_PASSWORD` - 用于连接的密码
- `MYSQL_DB` - 连接后使用的数据库

> 警告
>
> 虽然使用 env vars 来设置连接设置对于开发来说通常是可以的，但在生产中运行应用程序时，这是**非常不推荐的**。Docker 前安全主管 Diogo Monica[写了一篇博客文章](https://diogomonica.com/2017/03/27/why-you-shouldnt-use-env-variables-for-secret-data/)解释了原因。
>
> 更安全的机制是使用你的容器编排框架提供的秘密支持。在大多数情况下，这些机密作为文件安装在正在运行的容器中。你会看到许多应用程序（包括 MySQL 映像和 todo 应用程序）也支持 env vars，`_FILE`后缀指向包含该变量的文件。
>
> 例如，设置`MYSQL_PASSWORD_FILE`var 将导致应用程序使用引用文件的内容作为连接密码。Docker 不做任何事情来支持这些环境变量。你的应用程序需要知道查找变量并获取文件内容。

解释完所有这些之后，就可以启动我们开发就绪的容器！

**需要注意：**

> 这里我们使用了`$(pwd)`这个参数，它是只当前本目录，所以我们应该进入到**本地目录下的app/**目录下，这里我之前没有进入正确的目录下，导致容器一直启动失败。

1. 我们将指定上面的每个环境变量，并将容器连接到我们的应用程序网络。

   ```bash
   docker run -dp 8888:3000 \
     -w /app -v "$(pwd):/app" \
     --network todo-app \
     -e MYSQL_HOST=mysql \
     -e MYSQL_USER=root \
     -e MYSQL_PASSWORD=secret \
     -e MYSQL_DB=todos \
     node:12-alpine \
     sh -c "yarn install && yarn run dev"
   ```

   *我这里使用 bash 报错了，所以我后面使用 PowerShell 后就成功了*

   如果你使用的是 PowerShell，请使用此命令。

   ```bash
   docker run -dp 8888:3000 `
     -w /app -v "$(pwd):/app" `
     --network todo-app `
     -e MYSQL_HOST=mysql `
     -e MYSQL_USER=root `
     -e MYSQL_PASSWORD=secret `
     -e MYSQL_DB=todos `
     node:12-alpine `
     sh -c "yarn install && yarn run dev"
   ```

2. 启动之后，会返回一个container ID，我们来检测下情况

   ```bash
   // 查看容器状态是否在运行
   docker ps -a
   // 我的结果
   CONTAINER ID    IMAGE            CREATED          STATUS           PORTS         
   e3dd1fe96684    node:12-alpine   3 minutes ago    Up 3 minutes     0.0.0.0:8888->3000/tcp, :::8888->3000/tcp   
   ```
   
   
   
3. 如果我们查看容器 ( `docker logs <container-id>`)的日志，我们应该会看到一条消息，表明它正在使用 mysql 数据库。

   ```bash
   # Previous log messages omitted
   $ nodemon src/index.js
   [nodemon] 1.19.2
   [nodemon] to restart at any time, enter `rs`
   [nodemon] watching dir(s): *.*
   [nodemon] starting `node src/index.js`
   Connected to mysql db at host mysql
   Listening on port 8888
   ```

4. 在浏览器中输入：`http://localhost:8888/`打开该应用程序，并输入一些字符串添加到你的待办事项列表中。

   需要注意的是，要用`英文`、`数字`，不能使用`中文`字符。（如果要使用，可以研究下数据库代码来进行配置哦 :wink: ）

5. 连接mysql数据库，证明项目正在写入数据库。请记住，密码是**secret**。

   ```bash
   docker exec -it <mysql-container-id> mysql -p todos
   ```

   在 mysql shell 中，运行以下命令：

   ```mysql
   mysql> select * from todo_items;
   +--------------------------------------+---------+-----------+
   | id                                   | name    | completed |
   +--------------------------------------+---------+-----------+
   | 444454fc-709f-4088-8fb6-d6a965ef4cef | ffff    |         0 |
   | ca7cb357-9c5c-4c8c-8020-1c6e1c56fb4c | check   |         0 |
   | 3ced0027-dd54-4c27-845a-3ad3e238d86e | home    |         0 |
   | 4775a3aa-86fa-470e-a719-9a583802ff4e | go work |         0 |
   | cbb728b5-f23a-4af0-8d8d-f26271383685 | go home |         0 |
   +--------------------------------------+---------+-----------+
   ```

   可能每个人的数据表看起来会有所不同。

如果你快速查看 Docker 仪表板，你会看到我们有两个应用程序容器正在运行。但是，没有真正的迹象表明它们被组合在一个应用程序中。

![Docker 仪表板显示两个未分组的应用程序容器](https://cdn.wydgit.top/blog/docker-multi-app/dashboard-multi-container-app.png)

## Docker Compose

### 安装

**Windows**

如果已经安装了Docker Desktop，那么就不用再安装了

**Linux**

1. 运行此命令以下载 Docker Compose 的当前稳定版本：

   ```
   sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

   > 要安装不同版本的 Compose，请替换`1.29.2` 为你要使用的 Compose 版本。

   如果你在安装时遇到问题`curl`，请参阅 上面的[替代安装选项选项](https://docs.docker.com/compose/install/#alternative-install-options)卡。

2. 对二进制文件应用可执行权限：

   ```
   sudo chmod +x /usr/local/bin/docker-compose
   ```

> **注意**：如果`docker-compose`安装后命令失败，请检查你的路径。你还可以`/usr/bin`在路径中创建指向或任何其他目录的符号链接。

### 配置、使用

请记住，这是我们用来定义应用程序容器的命令。

```
docker run -dp 8888:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

如果你使用的是 PowerShell，请使用此命令。

```
docker run -dp 8888:3000 `
  -w /app -v "$(pwd):/app" `
  --network todo-app `
  -e MYSQL_HOST=mysql `
  -e MYSQL_USER=root `
  -e MYSQL_PASSWORD=secret `
  -e MYSQL_DB=todos `
  node:12-alpine `
  sh -c "yarn install && yarn run dev"
```

1. 首先，让我们为容器定义服务条目和图像。我们可以为服务选择任何名称。该名称将自动成为网络别名，这在定义我们的 MySQL 服务时会很有用。

   ```
   version: "3.8"
   
   services:
     app:
       image: node:12-alpine
   ```

2. 通常，你会在`image`定义附近看到命令，尽管对排序没有要求。所以，让我们继续把它移到我们的文件中。

   ```
   version: "3.8"
   
   services:
     app:
       image: node:12-alpine
       command: sh -c "yarn install && yarn run dev"
   ```

3. 让我们`-p 8888:3000`通过`ports`为服务定义来迁移命令的一部分。我们将在这里使用[短语法](https://docs.docker.com/compose/compose-file/#short-syntax-1)，但也有更详细的[长语法](https://docs.docker.com/compose/compose-file/#long-syntax-1)可用。

   ```
   version: "3.8"
   
   services:
     app:
       image: node:12-alpine
       command: sh -c "yarn install && yarn run dev"
       ports:
         - 8888:3000
   ```

4. 接下来，我们将使用和定义迁移工作目录 ( `-w /app`) 和卷映射 ( `-v "$(pwd):/app"`) 。Volumes 也有[短句](https://docs.docker.com/compose/compose-file/#short-syntax-3)和[长](https://docs.docker.com/compose/compose-file/#long-syntax-3)句。`working_dir``volumes`

   Docker Compose 卷定义的优点之一是我们可以使用当前目录的相对路径。

   ```
   version: "3.8"
   
   services:
     app:
       image: node:12-alpine
       command: sh -c "yarn install && yarn run dev"
       ports:
         - 8888:3000
       working_dir: /app
       volumes:
         - ./:/app
   ```

5. 最后，我们需要使用`environment`密钥迁移环境变量定义。

   ```
   version: "3.8"
   
   services:
     app:
       image: node:12-alpine
       command: sh -c "yarn install && yarn run dev"
       ports:
         - 8888:3000
       working_dir: /app
       volumes:
         - ./:/app
       environment:
         MYSQL_HOST: mysql
         MYSQL_USER: root
         MYSQL_PASSWORD: secret
         MYSQL_DB: todos
   ```

### 定义 MySQL 服务

现在，是时候定义 MySQL 服务了。我们用于该容器的命令如下：

```
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```

如果你使用的是 PowerShell，请使用此命令。

```
docker run -d `
  --network todo-app --network-alias mysql `
  -v todo-mysql-data:/var/lib/mysql `
  -e MYSQL_ROOT_PASSWORD=secret `
  -e MYSQL_DATABASE=todos `
  mysql:5.7
```

我们将首先定义新服务并为其命名，`mysql`以便它自动获取网络别名。我们将继续并指定要使用的图像。

```
version: "3.8"

services:
  app:
    # The app service definition
  mysql:
    image: mysql:5.7
```

1. 接下来，我们将定义卷映射。当我们使用 运行容器时`docker run`，会自动创建命名卷。但是，使用 Compose 运行时不会发生这种情况。我们需要在顶级`volumes:`部分定义卷，然后在服务配置中指定挂载点。通过仅提供卷名，使用默认选项。不过，还有[更多可用选项](https://docs.docker.com/compose/compose-file/#volume-configuration-reference)。

   ```
   version: "3.8"
   
   services:
     app:
       # The app service definition
     mysql:
       image: mysql:5.7
       volumes:
         - todo-mysql-data:/var/lib/mysql
   
   volumes:
     todo-mysql-data:
   ```

2. 最后，我们只需要指定环境变量。

   ```
   version: "3.8"
   
   services:
     app:
       # The app service definition
     mysql:
       image: mysql:5.7
       volumes:
         - todo-mysql-data:/var/lib/mysql
       environment: 
         MYSQL_ROOT_PASSWORD: secret
         MYSQL_DATABASE: todos
   
   volumes:
     todo-mysql-data:
   ```

此时，我们的完整`docker-compose.yml`应该是这样的：

```
version: "3.8"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 8888:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

### 运行我们的应用程序堆栈

现在我们有了我们的`docker-compose.yml`文件，我们可以启动它了！

1. 确保没有其他 app/db 副本首先运行（`docker ps`和`docker rm -f <ids>`）。

2. 使用`docker-compose up`命令启动应用程序堆栈。我们将添加`-d`标志以在后台运行所有内容。

   ```
   docker-compose up -d
   ```

   当我们运行它时，我们应该看到这样的输出：

   ```
   Creating network "app_default" with the default driver
   Creating volume "app_todo-mysql-data" with default driver
   Creating app_app_1   ... done
   Creating app_mysql_1 ... done
   ```

   你会注意到创建了卷以及网络！

   *默认情况下，Docker Compose 会自动为应用程序堆栈创建一个网络（这就是我们没有在 compose 文件中定义网络的原因）。*

3. 让我们使用`docker-compose logs -f`命令查看日志。你将看到来自每个服务的日志交错到一个流中。当你想观察与时序相关的问题时，这非常有用。该`-f`标志“跟随”的日志，因为它产生这样会给你现场输出。

   如果你还没有，你会看到看起来像这样的输出......

   ```
   mysql_1  | 2019-10-03T03:07:16.083639Z 0 [Note] mysqld: ready for connections.
   mysql_1  | Version: '5.7.27'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
   app_1    | Connected to mysql db at host mysql
   app_1    | Listening on port 8
   ```

   服务名称显示在行首（通常是彩色的）以帮助区分消息。如果要查看特定服务的日志，可以在logs 命令的末尾添加服务名称（例如，`docker-compose logs -f app`）。

   > 专业提示 - 在启动应用程序之前等待数据库
   >
   > 当应用程序启动时，它实际上会等待 MySQL 启动并准备好，然后再尝试连接到它。Docker 没有任何内置支持在启动另一个容器之前等待另一个容器完全启动、运行和准备就绪。对于基于 Node 的项目，你可以使用[等待端口](https://github.com/dwmkerr/wait-port)依赖项。其他语言/框架也存在类似的项目。

4. 此时，你应该能够打开你的应用程序并看到它正在运行。嘿！我们只需要一个命令！

### 拆除我们的应用堆栈

```bash
docker-compose down
```

然后就会看到

```bash
$ docker-compose down
Stopping app_app_1   ...
Stopping app_mysql_1 ...
Stopping app_app_1   ... done
Stopping app_mysql_1 ... done
Removing app_app_1   ...
Removing app_mysql_1 ...
Removing app_mysql_1 ... done
Removing app_app_1   ... done
Removing network app_default
```

容器将停止，网络将被移除。默认情况下，运行`docker-compose down`. 如果要删除卷，则需要添加`--volumes`标志。

拆除后，你可以切换到另一个项目，运行`docker-compose up`开始新的项目。

### 在 Docker 仪表板中查看我们的应用程序堆栈

如果我们查看 Docker 仪表板，我们会看到有一个名为**app**的组。这是来自 Docker Compose 的“项目名称”，用于将容器组合在一起。默认情况下，项目名称只是所在目录的名称`docker-compose.yml`。

如果你向下旋转应用程序，你将看到我们在撰写文件中定义的两个容器。这些名称也更具描述性，因为它们遵循`<project-name>_<service-name>_<replica-number>`. 因此，很容易快速查看哪个容器是我们的应用程序，哪个容器是 mysql 数据库。

![带有应用程序项目的 Docker 仪表板](https://cdn.wydgit.top/blog/docker-multi-app/dashboard-app-project-expanded.png)

## 参考资料

[Docker-getting-started-tutorial](https://github.com/docker/getting-started/tree/master/docs/tutorial)

