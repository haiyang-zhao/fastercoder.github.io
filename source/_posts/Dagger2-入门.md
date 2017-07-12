title: Dagger2 初体验
date: 2017-02-27 17:36:58
categories: Dagger
tags: Dagger2
---

## 什么是Dagger2

  A fast dependency injector for Android and Java.<br>
  Dagger2起源于Dagger，是一款基于Java注解来实现的完全在编译阶段完成依赖注入的开源库，主要用于模块间解耦、提高代码的健壮性和可维护性。Dagger2在编译阶段通过apt利用Java注解自动生成Java代码，然后结合手写的代码来自动帮我们完成依赖注入的工作。
  
##  初步体验

### 添加依赖
  在app module的build.gradle中增加如下依赖

    dependencies {
            ......

            compile 'com.google.dagger:dagger:2.x'
            annotationProcessor 'com.google.dagger:dagger-compiler:2.x'
   }

  