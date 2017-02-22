title: 搭建hexo博客（二）准备工作
date: 2015-12-20 20:03:01
categories:
- hexo搭建个人博客
tags:
- hexo
comments:

---
## 安装homebrew
homebrew是Mac OSX上的软件包管理工具，能在Mac中方便的安装软件或者卸载软件， 只需要一个命令， 非常方便。打开终端输入以下命令即可完成安装：


	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)”
	
## 安装nodejs
hoewbrew安装成功后，我们就可以借助他来安装Nodejs了，输入以下命令：
		
	brew install node
	
## 安装hexo
使用nodejs的npm命令来安装hexo
		
	npm install -g hexo-cli
	
<!--more-->
## 初始化hexo
	
	hexo init Blog
Blog是工程存放的文件夹，名字可以随便取

    cd Blog
    npm install
    
## 安装git插件

	npm install hexo-deployer-git --save

## 注册GitHub
  * 点这里注册 [GitHub](https://github.com/) 
  * 注册完成后 创建你的GitHub Pages 也就是我们博客的主页
  * 如果你的用户名为abc,则需创建名称为abc.github.io的仓库
  


