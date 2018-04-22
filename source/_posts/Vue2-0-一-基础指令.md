title: Vue2.0(一)基础指令
date: 2018-04-22 10:59:42
categories: 前端开发
tags: Vue
---


## Setup

###  安装
- 官方 https://cn.vuejs.org
- 下载vue.js
    - 开发环境：包含完整的警告和调试模式
    - 生产环境：删除了警告，30.90KB min+gzip
- 引入项目
    - CDN  
    
    ```
    <script src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
    ```
    - 直接在script中引用
    
    ```
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    ```
    
<!-- more -->
### 创建项目
- 使用WebStorm创建项目
- 结构如下
- 
```
├── assets
│   ├── css
│   └── js
│       └── vue.js
├── example
│   └── 01_01_hello_world.html
├── index.html
└── package.json 
```

![image](http://p7ggkwz0d.bkt.clouddn.com/vue_01_01_hello_world_project_tree.png)
### 安装live-server
    
- npm install live-server -g
- 启动  live-server
### 编写第一个HelloWorld代码：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello World Vue</title>
    <script type="text/javascript" src="../assets/js/vue.js"></script>
</head>
<body>
<h1>Vue2.0 Hello World</h1>
<hr/>
<div id="app">
    {{message}}

</div>
<script type="text/javascript">
    var vue = new Vue({
        el:'#app',
        data:{
            message:'hello world'
        }
    })
</script>
```

## v-if v-else v-show
  
* v-if用来判断是否加载html的DOM,v-else与之对应
```
<div v-if="isLogin">你好，Vue！</div>
<div v-else="isLogin">你好，请登录 v-else</div>
```
* v-show 调整css中display属性，DOM已经加载，只是CSS控制没有显示出来。
```
<div v-show="isLogin">你好，请登录 v-show</div>
```
* v-if 和v-show的区别：
    * v-if： 判断是否加载，可以减轻服务器的压力，在需要时加载。
    * v-show：调整css dispaly属性，可以使客户端操作更加流畅。
## v-for
* 基本用法
```
<li v-for="item in items">
        {{item}}
</li>
```
* 带索引的用法
```
  <li v-for="(item,index) in items">
            {{index}}-{{item}}
  </li>
```
* 排序

我们已经顺利的输出了我们定义的数组，但是我需要在输出之前给数组排个序，那我们就用到了Vue的computed:属性。
    
    var vue = new Vue({
        el: '#app',
        data: {
            items: [12, 24, 7, 33]
        },
        computed: {
            sortedItems: function () {
                return this.items.sort();
            }
        }
    })
    
    在computed中从新声明sortedItems，这样为了防止污染数据源.
    我们要做sort()中传入排序方法，否则按ascII码排序。
```
var vue = new Vue({
        el: '#app',
        data: {
            items: [12, 24, 7, 33]
        },
        computed: {
            sortedItems: function () {
                return this.items.sort(sortNumber);
            }
        }
    });

    function sortNumber(a, b) {
        return a - b
    }
```
* 对象循环输出
```    
var vue = new Vue({
        el: '#app',
        data: {
            items: [12, 24, 7, 33],
            students: [
                {name: 'jspang', age: 32},
                {name: 'Panda', age: 30},
                {name: 'PanPaN', age: 21},
                {name: 'King', age: 45}
            ]
        },
        computed: {
            sortedItems: function () {
                return this.items.sort(sortNumber);
            },

            sortedStudents: function () {
                return sortByKey(this.students,'age');
            }
        }
    });

    function sortNumber(a, b) {
        return a - b
    }

    //数组对象方法排序:
    function sortByKey(array, key) {
        return array.sort(function (a, b) {
            var x = a[key];
            var y = b[key];
            return ((x < y) ? -1 : ((x > y) ? 1 : 0));
        });
    }

```
## v-text v-html
* v-text

使用{{xxx}}时存在弊端，比如当网速慢或者js出错时会暴露{{xxx}},此时用v-text可以防止次问题
```
<span>{{ message }}</span>=<span v-text="message"></span><br/>
```
* v-html

如果在javascript中写有html标签，用v-text是输出不出来的，这时候我们就需要用v-html标签了。
```
<span v-html="htmlMessage"></span>
```

```
<div id="app">
    {{message}}
    <br>
    <span v-text="message"></span>
    <br>
    <span v-html="htmlMessage"></span>

</div>
    var vue = new Vue({
        el:'#app',
        data:{
            message:'hello world',
            htmlMessage: '<h2 style="color: red">hello Vue!</h2>'
        }
    })

```
## v-on
* 绑定事件   v-on:click可以简写成@click
```
<button v-on:click="plus">加分</button>
<button @click="minus">加分</button>
```
* 也可以绑定其他事件
```
<input type="text" v-on:keyup.enter="onEnter" v-model="secondCount">
```

![image](http://7xjyw1.com1.z0.glb.clouddn.com/20170227001137.jpg)

## v-model
* 把数据源绑定到指定的元素上，可实现双向数据绑定
* 修饰符
    - .lazy :取代 input 监听 change 事件
    - .number: 
    - .trim : 
* 单选按钮绑定

```
    <input type="radio" id="man" value="男" v-model="sex"/>
    <label for="man">男</label>

    <input type="radio" id="woman" value="女" v-model="sex"/>
    <label for="man">女</label>
    <p v-text="sex"></p>
```
* 多选按钮绑定一个值

```
<input type="checkbox" id="isChecked" v-model="isChecked"/>
<label for="isChecked">{{isChecked}}</label>
```
* 多选按钮绑定多个值

```
<input type="checkbox" id="1" value="Java" v-model="courses"/>
    <label for="1">Java</label>

    <input type="checkbox" id="2" value="C" v-model="courses"/>
    <label for="2">C</label>

    <input type="checkbox" id="3" value="C++" v-model="courses"/>
    <label for="3">C++</label>
    <p v-text="courses"></p>
```

## v-bind
	一般拥有绑定标签属性，可以简写为':''
* 绑定src
 
```
<img v-bind:src="imageSrc" width="200px">
```
* 绑定href

```
<p><a :href="webUrl" target="_blank">百度</a></p>
```
* 绑定css

```
    <div :class="className">1、绑定ClassA</div>
    <div :class="{classA:isOk}">2、绑定Class中的判断</div>
    <div>
        <input type="checkbox" v-model="isOk" id="isOk">
        <label for="isOk">使用ClassA样式</label>
    </div>
    <div :class="[classA,classB]">3、绑定ClassA,ClassB</div>
    <div :class="isOk?classA:classB">4、三元运算符</div>
    <div :style="{color:styleColor,fontFamily: styleFontFamily}">5、绑定Style</div>
    <div :style="styleObject">6、绑定StyleObject</div>
 
    var vue = new Vue({
        el: '#app',
        data: {
            webUrl: 'http://www.baidu.com',
            imageSrc: 'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1524302878433&di=3d1a79fed4b310bc4ff6e8f7f03f79ec&imgtype=0&src=http%3A%2F%2Fstatic.jysq.net%2Fdata%2Fattachment%2Falbum%2F201803%2F18%2F093928w9z2a363831h2p1a.jpg',
            className: 'classA',
            isOk: true,
            classA: 'classA',
            classB: 'classB',
            styleColor: 'green',
            styleFontFamily: '.SF NS Display',
            styleObject: {
                color: 'blue',
                fontFamily: '.SF NS Display'
            }
        }
    })
    
    
    
    .classA {
        color: red;
    }

    .classB {
        font-size: 3em;
    }
```
