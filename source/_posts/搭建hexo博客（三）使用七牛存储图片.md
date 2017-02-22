title: 搭建hexo博客（三）使用七牛存储图片
date: 2016-01-09 13:01:41
categories: hexo搭建个人博客
tags: hexo
---

### 1.  为何使用七牛
Hexo文章中的图片，我们可以放到本地，然后一起部署到github中，这样完全没有问题。然而github pages空间毕竟有限（貌似只有300M）,另外图片的管理太混乱了，一些原创的图片可能被盗链。七牛作为国内顶尖的CDN云存储商，选择他有以下几个理由

> * 在国内很稳定，我们公司也是选择七牛来提供云存储的
> * 免费提供10G存储空间，和每月10G下载流量，完全够用
> * hexo有七牛的插件，使用起来也是相当的方便
> 
![](http://7xppgb.com1.z0.glb.clouddn.com/static/images/use-qiniu-store-image-for-hexo/qiniu.png)


### 2.注册和安装七牛工具

首先我们需要申请七牛账号，如果你也需要申请，请访问[**这个链接**](https://portal.qiniu.com/signup?code=3ld4kzjsl1qoi)，这样我可以获得更多的流量（5GB）。然后登录七牛网站，按照官网说明创建空间，比如我创建的空间是为forevercoder-blog 。创建完成后会给你分配个七牛域名比如我的是：
		
		7xppgb.com1.z0.glb.clouddn.com
通过该URL就可以访问你上传的资源了，

```
http://7xppgb.com1.z0.glb.clouddn.com/static/images/use-qiniu-store-image-for-hexo/qiniu.png
```
当然也可以设置自定义域名。

<!--more-->
### 3.安装hexo七牛插件

1. 插件地址：[https://github.com/gyk001/hexo-qiniu-sync](https://github.com/gyk001/hexo-qiniu-sync)

2. 安装

在你的hexo主目录下运行以下命令进行安装：

```
npm install hexo-qiniu-sync --save
```

添加插件配置信息到 ``_config.yml`` 文件中:

```
plugins:
  - hexo-qiniu-sync
```
让后根据官方的README,一步一步完成就OK了

### 4. 使用qiniu插件

配置完成后我们在hexo目录下执行
	 
	 hexo qiniu sync
	 
	
这样就在localDir下生成相对应的文件夹,将图片资源放到images文件夹下，比我的路径是 use-qiniu-store-image-for-hexo/qiniu.png，让后就用下面的标记使用，

```	
    {% qnimg use-qiniu-store-image-for-hexo/qiniu.png title:免费使用七牛 alt:图片说明 'class:class1 class2' %}
```
 
最后在同步上传图片
   
	hexo qiniu sync2
	
然后 hexo g -d 就OK了，非常简单方便
 

