title: es6新特性
date: 2017-07-13 11:52:19
categories: javascript
tags: javascript
---

## let-块级作用域

以前在js中只有全局作用域和函数作用域

```javascript
if (true) {
    var fruit = 'apple';
}

console.log(fruit); //apple

```

var声明了一个全局变量fruit,全局范围都有效。若只在用的到代码块中有效则使用let

```javascript
if (true) {
    let fruit = "zhy";
    console.log(fruit);//zhy
}
console.log(fruit);//ReferenceError: fruit is not defined

```
在let代码块外面访问时报错


<!-- more -->
## const 声明常量

```javascript
//声明一个常量，声明后不能重新赋值
const fruit = 'apple';
console.log(fruit);

//const只是限制赋值动作，并不能决定对象的值是什么
const fruits = [];
fruits.push('apple')
console.log(fruits)
```

## Destructuring 解构
解构赋值允许你使用类似数组或对象字面量的语法将数组和对象的属性赋给各种变量,这种赋值语法极度简洁，同时还比传统的属性访问方法更为清晰

### Array Destructuring 数组解构

```javascript
//解构语法
function breakfast() {
    return ['apple', 'orange', 'banana'];
}
let [breakfast1, breakfast2, breakfast3] = breakfast();
console.log(breakfast1, breakfast2, breakfast3) //apple orange banana


//你可以在对应位留空来跳过被解构数组中的某些元素
let [, , third] = [1, 2, 3];
console.log(third) //3
//你还可以通过“不定参数”模式捕获数组中的所有尾随元素
let [head, ...tail] = [1, 2, 3, 4];
console.log(tail); //[ 2, 3, 4 ]

//也可以赋初始值
let [x = 5, y] = [];
console.log(x, y); //5 undefined
```

### Object Destructuring 对象解构

通过解构对象，你可以把它的每个属性与不同的变量绑定，首先指定被绑定的属性，然后紧跟一个要解构的变量

```javascript
var personA = {name: "张三"};
var personB = {name: "李四"};

let {name: nameA} = personA;
let {name: nameB} = personB;
console.log(nameA, nameB);//张三 李四


//当属性名与变量名一致时, 可以简写为
let {name, age} = {name: '张三', age: 18};
console.log(name, age); //张三 18

//可以嵌套赋值
var personC = {
    name1: '王五',
    favorite: ['football', {work: 'program'}]
}

let {name1, favorite: [first, {work}]} = personC;
console.log(name1, first, work);

```

### Template Strings 模板字符串



```javascript
//模板字符串 `${表达式或变量}`
let dessert = 'dessert', drink = 'drink';
let breakfast = '今天的早餐是' + dessert + '和' + drink;
console.log(breakfast)

//使用字符模板表示
let myBreakfast = `今天的早餐是${dessert}和${drink}`;
console.log(myBreakfast);

//可以调用函数
function sayHello() {
    return "hello"
}
console.log(`i 'am say ${sayHello()}`)
```

### Tagged template 标签模板


```javascript
//标签模板
'use strict'

var a = 5, b = 6;
function tag(strings, ...values) {
    console.log(strings); //[ 'hello ', ' world', '' ]
    console.log(values); //[ 11, 30 ]
    return 'Hello every body';

}

//标签模板函数第一个参数是字符串模板的常量数组，后面的每一个参数为表达式的计算结果，函数名称可以任意指定
let result = tag`hello ${a + b} world${a * b}`;
console.log(result);
```

### ...操作符

```javascript
//...展开操作符
'use strict'

let fruits = ['apple', 'grapes', 'pear'];
console.log(fruits);//[ 'apple', 'grapes', 'pear' ]

//使用...展开数组
console.log(...fruits);//apple grapes pear

//也可以使用...作为数组元素
let foods = ['cake', ...fruits];
console.log(foods);

//...此外还可以作为Rest操作符作为函数的参数表示剩余的参数

function myFamily(person1, person2, ...person) {
    console.log(person1, person2, person);
}
myFamily('张三', '李四', '赵五', '王麻子');//张三 李四 [ '赵五', '王麻子' ]
```

### Arrow Function 箭头函数(lambda 表达式)

```javascript
//与lambda表达式类似

'use strict'

//在es6之前，通常这样定义和使用函数
var say = function (name, word) {
    return name + ' say: ' + word;
}
console.log(say('张三', '你好'));//张三 say: 你好

//使用箭头函数可以简化
var say = (name, word) => {
    return name + ' say: ' + word;
};

console.log(say('张三', '你好'));//张三 say: 你好
//还可以简化为

var say = (name, word) => name + ' say: ' + word;
console.log(say('张三', '你好'));//张三 say: 你好
//如果没有参数可简化为
var say = () => 'hello world!';
console.log(say())


//遍历数组大学转为小写
let words = ['HELLO', 'WORLD'];
console.log(words.map(word => word.toLowerCase()));//[ 'hello', 'world' ]

//数组排序
let array = [1, 6.4, 3];
array.sort((a, b) => a - b);
console.log(array);//[ 1, 3, 6.4 ]

//this的作用域
function Course() {
    this.name = "";
    this.description = "";
    this.author = "";
    this.getSummary = function () {
        return this.name + ", " + this.description;
    };
    this.getDetails = function()
    {
        window.setTimeout(() => {
            console.log(this.getSummary() + " " + this.author)
        }, 1000);
    }
}

var course = new Course();
course.getDetails();
//以上，this的作用域指的是Course，而不是window。
//也就是说，lambda表达式中的this的作用域的指向取决于在哪里定义，而不是取决于在哪里使用。

```
