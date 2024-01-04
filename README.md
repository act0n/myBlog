# myBlog
> 🎸日常笔记和总结

查看博客效果👉[demo](https://waydez.github.io/)



### 下载

```shell
git clone https://github.com/waydez/myBlog.git
```

### 安装⚠️

```shell
## 安装依赖
npm install

# 安装应用主题，如果不安装应用主题，没办法正常使用和生成文章
# 也可以不加 -b <version number> 来获取最新版的主题版本
git clone https://github.com/ppoffice/hexo-theme-icarus.git themes/icarus [-b <version number>] --depth 1
```

### 使用

```shell
# 发布文章，浏览器目录path显示的就是后者，而我们文章标题为前者
hexo new post "我的中文标题" -s "my-url-title"

# 将md文件生成html文件资源
hexo g
# 开启本地服务
hexo s
# 部署到远程仓库，这里前提需要在 _config.yml 配置好相应的部署地址
hexo d
```

### 了解更多

[icarus用户指南](https://ppoffice.github.io/hexo-theme-icarus/tags/Icarus%E7%94%A8%E6%88%B7%E6%8C%87%E5%8D%97/)
