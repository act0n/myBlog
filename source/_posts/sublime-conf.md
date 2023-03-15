---
title: Sublime Text3 配置C、C++环境，并且能调用cmd
date: 2021-03-21 11:24:58
toc: true
categories:
  - Config Share
tags:
	- C++
	- C
	- Sublime
---

# 准备工作

## 安装Sublime Text3

[下载链接1](https://sublimetextcn.com/3/)

[下载链接2](https://www.onlinedown.net/soft/68602.htm)

安装过程不在这里啰嗦，汉化等功能自行百度

## 下载MinGW

[下载链接](https://sourceforge.net/projects/mingw-w64/)

下载安装完成之后，要记住安装路径，配置的时候要用到

<!-- more -->

# 配置工作

## 配置环境变量

1. 【**此电脑】->鼠标右键【属性】->【高级系统设置】->【环境变量】->【系统变量】**

2. 找到`Path`变量，添加上你刚才下载的MinGW的安装路径

   ![1](http://cdn.wydgit.top/sub-1.png) 

3. 桌面打开【cmd】，输入`g++ -v`或者`gcc -v`,出现如下内容即配置成功

   ![2](http://cdn.wydgit.top/sub-2.png) 

## 配置Sublime Text3 

默认Sublime Text3 中的编译系统是**不会调用cmd**的，因此我们需要新建满足我们需求的编译系统

**【打开sublime】->【工具】->【编译系统】->【新建编译系统】**，会出现如下内容

![3](http://cdn.wydgit.top/sub-3.png) 

### 新建C++编译系统

将内容替换为如下内容

```json
{  
    "cmd": ["g++", "${file}", "-fexec-charset=gbk","-o", "${file_path}/${file_base_name}","-Wall" ,"&&","start","cb_console_runner.exe","${file_path}/${file_base_name}"],  
    "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",  
    "working_dir": "${file_path}",  
    "selector": "source.c, source.c++",  
    "shell": true,  
    "encoding":"cp936" 
}  
```

保存命名为`C++.sublime-build`，当然也可以命名为其他你自己可以记住的

### 新建C 编译系统

将内容替换为如下内容

```json
{
	"cmd": ["gcc", "${file}", "-fexec-charset=gbk","-o", "${file_path}/${file_base_name}", "&", "start", "cmd", "/c", "${file_base_name} & echo. & pause"],
	"file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
	"working_dir": "${file_path}",
	"selector": "source.c, source.c++",
	"shell": true,
	"encoding":"cp936"
} 
```

保存命名为`C.sublime-build`，当然也可以命名为其他你自己可以记住的

## 测试

### C++

源码：

```c++
#include <iostream>

using namespace std;
int main(int argc, char const *argv[])
{
	int n;
	cin >> n;
	cout << "芜湖，起飞：" << n << endl;
	return 0;
}
```

运行结果：

![4](http://cdn.wydgit.top/sub-4.png) 

### C

源码：

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
	int n;
	scanf("%d",&n);
	printf("芜湖，起飞：%d\n",n);
	return 0;
}
```

运行结果：

![5](http://cdn.wydgit.top/sub-5.png) 



# 参考资料

[Sublime Text 配置C++运行，带黑窗口，支持中文[windows]](https://blog.csdn.net/qq_31281327/article/details/78107987)