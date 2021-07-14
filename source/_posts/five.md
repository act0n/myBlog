---
title: 解决Hexo引入图片时的路径错误
date: 2020-02-28 23:32:21
toc: true
excerpt: (^_^)
categories:
  - Config Share
tags:
  - Hexo
---



# 问题
>hexo在文章中引入图片有两种方法
>>前提：修改<code>_config.yml</code>配置文件中的<code>post_asset_folder</code>项为true。
>*是为了在创建博客的时候，同时创建一个同名文件夹*
>
> - 不使用插件：
>
	{% asset_img 这是一个新的博客的图片.jpg 这是一个新的博客的图片的说明 %}

> - 使用[插件](https://github.com/xcodebuild/hexo-asset-image)：
>
		npm install hexo-asset-image --save

**但是！插件好像有bug，我的图片路径一直时错误的，而显示不出来！**
# 解决
>打开<code>/node_modules/hexo-asset-image/index.js</code>，将内容更换为下面的代码


	'use strict';
	var cheerio = require('cheerio');
	// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
	function getPosition(str, m, i) {
	  return str.split(m, i).join(m).length;
	}
	var version = String(hexo.version).split('.');
	hexo.extend.filter.register('after_post_render', function(data){
	  var config = hexo.config;
	  if(config.post_asset_folder){
	    	var link = data.permalink;
		if(version.length > 0 && Number(version[0]) == 3)
		   var beginPos = getPosition(link, '/', 1) + 1;
		else
		   var beginPos = getPosition(link, '/', 3) + 1;
		// In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
		var endPos = link.lastIndexOf('/') + 1;
	    link = link.substring(beginPos, endPos);
	    var toprocess = ['excerpt', 'more', 'content'];
	    for(var i = 0; i < toprocess.length; i++){
	      var key = toprocess[i];
	      var $ = cheerio.load(data[key], {
	        ignoreWhitespace: false,
	        xmlMode: false,
	        lowerCaseTags: false,
	        decodeEntities: false
	      });
	      $('img').each(function(){
			if ($(this).attr('src')){
				// For windows style path, we replace '\' to '/'.
				var src = $(this).attr('src').replace('\\', '/');
				if(!/http[s]*.*|\/\/.*/.test(src) &&
				   !/^\s*\//.test(src)) {
				  // For "about" page, the first part of "src" can't be removed.
				  // In addition, to support multi-level local directory.
				  var linkArray = link.split('/').filter(function(elem){
					return elem != '';
				  });
				  var srcArray = src.split('/').filter(function(elem){
					return elem != '' && elem != '.';
				  });
				  if(srcArray.length > 1)
					srcArray.shift();
				  src = srcArray.join('/');
				  $(this).attr('src', config.root + link + src);
				  console.info&&console.info("update link as:-->"+config.root + link + src);
				}
			}else{
				console.info&&console.info("no src attr, skipped...");
				console.info&&console.info($(this));
			}
	      });
	      data[key] = $.html();
	    }
	  }
	});
>最好重启下，因为我执行<code>hexo g</code>之后没有立即改变，之后就可以正常显示图片了
# 参考资料
>[Hexo发布博客引用自带图片的方法](https://zhuanyongxigua.github.io/2017/05/19/Hexo%E5%8F%91%E5%B8%83%E5%8D%9A%E5%AE%A2%E5%BC%95%E7%94%A8%E8%87%AA%E5%B8%A6%E5%9B%BE%E7%89%87%E7%9A%84%E6%96%B9%E6%B3%95/)
>[hexo引用本地图片无法显示](https://blog.csdn.net/xjm850552586/article/details/84101345)