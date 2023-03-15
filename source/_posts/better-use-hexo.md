---
title: 如何更好地使用hexo发布博客
toc: true
date: 2021-07-08 15:38:08
categories: 
  - Config Share
tags:
  - Hexo
  - Markdown
---

为了方便在`Hexo`自己撰写并且发布博客，将自己的一些技巧和配置分享一下
<!-- more -->

## 设置模板

一般我们总是会被写一篇文章还要配置`hexo`文章的头部（包括时间、标签、分类等）烦的要死，而且有的`hexo`主题没有给文章`查看更多`的总配置，更加增加了撰写的复杂度。

所以我们要用到`scaffolds`下的`post.md`文件，它是我们发布文章的时候默认使用的模板，以下是我配置内容。

```markdown
---
title: {{ title }}
date: {{ date }}
toc: true
categories:
excerpt: 显示在文章缩略图的 文字
tags:

---
这里填写 显示在文章缩略图的 文字
<!-- more -->
这里填写 更多文章内容
```

## 巧用 命令

一般我们发布文章：

```shell
hexo new post "file-name"
```

我们会为了文章的`url`中不包含汉字，会用我们贫瘠的英语绞尽脑汁地想出几个英文名称，然后引来”嘲笑“，我们自己也会觉得难看。

但是，**福音来了**！，只需要一个简单的参数`-s`就可以解决，如下：

```shell
hexo new post "我的中文标题" -s "my-url-title"
```

这样配置之后，浏览器目录`path`显示的就是后者了。

## 巧用GitHub

最初，编写`Markdown`发布博客文章都是将`Hexo`部署到云服务器上，然后开启Web网络服务来进行使用。但是随着大学毕业，“学生机”已经无法继续续费使用了，因此，就想到了GitHub还有个博客功能。

发布思路：

```
本地机器 ---------->  Hexo  -------->  远程GitHub.io
   |                  |                    |
   |                  |                    |
   |                  |                    |
撰写文章           配置部署功能         浏览器浏览访问
```



### 配置Hexo

找到`Hexo`的主配置文件`_config.yml`的`deploy`字段，比如我的配置文件：

```yaml
deploy:
  type: git
  repo: < 自己的git的博客地址 >
  branch: master
```

如果没有Git博客，需要先去`GitHub`上申请一个

还要配置一下`post_asset_folder`参数，保证我们能把图片资源上传到git

```yaml
post_asset_folder: true
```

发布部署

```shell
// 生成html等资源
hexo g  # hexo generate
// 部署到远程
hexo d  # hexo deploy
```

也许会让你输入git账号密码，可能是因为我之前有git化这个文件，所以没有输入账号密码。

之后就能够在相应的Git上查看自己的博客了！我的博客为 => [这里](https://adyoow.github.io/)
