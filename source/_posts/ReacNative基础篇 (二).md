title: ReacNative基础篇(二)
date: 2017-07-16 17:42:07
categories: react native
tags: react native
---

# flexBox 布局

与css中的flexBox布局类似，如下所示(转载网路)
<table class="table table-bordered table-striped table-condensed"><tr><td><img src='http://7xppgb.com1.z0.glb.clouddn.com/flex-direction.jpg'></td><td><img src='http://7xppgb.com1.z0.glb.clouddn.com/flex-wrap.jpg'></td></tr><tr><td><img src='http://7xppgb.com1.z0.glb.clouddn.com/justify-content.jpg'></td><td><img src='http://7xppgb.com1.z0.glb.clouddn.com/align-items.jpg'></td></tr><tr><td><img src='http://7xppgb.com1.z0.glb.clouddn.com/align-content.jpg'></td></table>


<!-- more -->
## 弹性（Flex）宽高

在组件样式中使用flex可以使其在可利用的空间中动态地扩张或收缩。一般而言我们会使用flex:1来指定某个组件扩张以撑满所有剩余的空间。如果有多个并列的子组件使用了flex:1，则这些子组件会平分父容器中剩余的空间。如果这些并列的子组件的flex值不一样，则谁的值更大，谁占据剩余空间的比例就更大（即占据剩余空间的比等于并列组件间flex值的比）。

* 指定flex:1占满整个屏幕

```javascript
container: {
            backgroundColor: '#eae7ff',//背景
            marginTop: 20,
            flex: 1
        },

```

![指定flex:1](http://7xppgb.com1.z0.glb.clouddn.com/flex_1.png)

* 不指定flex

```javascript
container: {
        backgroundColor: '#eae7ff',//背景
        marginTop: 20
        }

```
![指定flex:1](http://7xppgb.com1.z0.glb.clouddn.com/no_flex.png)

## Flex Direction

在组件的style中指定flexDirection可以决定布局的主轴。子元素是应该沿着水平轴(row)方向排列，还是沿着竖直轴(column)方向排列呢？默认值是竖直轴(column)方向。有以下值：'row', 'row-reverse', 'column', 'column-reverse'

```javascript
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */

import React, {Component} from 'react';
import {
    AppRegistry,
    StyleSheet,
    Text,
    View
} from 'react-native';

export default class MovieTalk extends Component {
    render() {
        return (
            <View style={styles.container}>
                <Text style={styles.item1}>
                    1
                </Text>
                <Text style={styles.item2}>
                    2
                </Text>
                <Text style={styles.item3}>
                    3
                </Text>
            </View>
        );
    }
}

const styles = StyleSheet.create({
        container: {
            backgroundColor: '#eae7ff',//背景
            marginTop: 20,
            flex: 1,
            flexDirection: 'row'
        },
        item1: {
            backgroundColor: 'powderblue',
            width: 50,
            height: 50,
            margin: 5,
            fontSize: 30,
            color: 'red'

        },
        item2: {
            backgroundColor: 'skyblue',
            width: 50,
            height: 50,
            margin: 5,
            fontSize: 30,
            color: 'green'

        },
        item3: {
            backgroundColor: 'steelblue',
            width: 50,
            height: 50,
            margin: 5,
            fontSize: 30,
            color: 'blue'

        },


    })
;

AppRegistry.registerComponent('MovieTalk', () => MovieTalk);

```
<table class="table table-bordered table-striped table-condensed"><tr><th>flexDirection: 'row'</th><th>flexDirection: 'row-reverse'</th><th>flexDirection: 'column'</th><th>flexDirection: 'column-reverse'</th></tr><tr><td><img src='http://7xppgb.com1.z0.glb.clouddn.com/flex_derection_row.png'/></td>
<td><img src='http://7xppgb.com1.z0.glb.clouddn.com/row-reverse.png'/</td><td><img src='http://7xppgb.com1.z0.glb.clouddn.com/flex_direction_column.png'/></td><td><img src='http://7xppgb.com1.z0.glb.clouddn.com/column-reverse.png'/</td></tr></table>

## Justify Content
在组件的style中指定justifyContent可以决定其子元素沿着主轴的排列方式。子元素是应该靠近主轴的起始端还是末尾段分布呢？亦或应该均匀分布？对应的这些可选项有：flex-start、center、flex-end、space-around以及space-between。

* flex-start
弹性项目向行头紧挨着填充。这个是默认值。第一个弹性项的main-start外边距边线被放置在该行的main-start边线，而后续弹性项依次平齐摆放。

* flex-end
弹性项目向行尾紧挨着填充。第一个弹性项的main-end外边距边线被放置在该行的main-end边线，而后续弹性项依次平齐摆放。

* center
弹性项目居中紧挨着填充。（如果剩余的自由空间是负的，则弹性项目将在两个方向上同时溢出）。

* space-between
弹性项目平均分布在该行上。如果剩余空间为负或者只有一个弹性项，则该值等同于flex-start。否则，第1个弹性项的外边距和行的main-start边线对齐，而最后1个弹性项的外边距和行的main-end边线对齐，然后剩余的弹性项分布在该行上，相邻项目的间隔相等。
* space-around
弹性项目平均分布在该行上，两边留有一半的间隔空间。如果剩余空间为负或者只有一个弹性项，则该值等同于center。否则，弹性项目沿该行分布，且彼此间隔相等（比如是20px），同时首尾两边和弹性容器之间留有一半的间隔（1/2*20px=10px）。

```javascript
container: {
            backgroundColor: '#eae7ff',//背景
            marginTop: 20,
            flex: 1,
            flexDirection: 'row',
            justifyContent:'flex-start'

        },
```

| flex-start    | center        | flex-end  | space-around | space-between |
| ------------- |:-------------:| -----:|:--------:|:--------:|:--------:|
| ![](http://7xppgb.com1.z0.glb.clouddn.com/justify_cotent_flex_start.png)      | ![](http://7xppgb.com1.z0.glb.clouddn.com/justify_cotent_center.png) | ![](http://7xppgb.com1.z0.glb.clouddn.com/justify_cotent_flex-end.png)  | ![](http://7xppgb.com1.z0.glb.clouddn.com/justify_cotent_space-around.png)  | ![](http://7xppgb.com1.z0.glb.clouddn.com/justify_cotent_space-between.png)

## Align Items
在组件的style中指定alignItems可以决定其子元素沿着次轴（与主轴垂直的轴，比如若主轴方向为row，则次轴方向为column）的排列方式。子元素是应该靠近次轴的起始端还是末尾段分布呢？亦或应该均匀分布？对应的这些可选项有：flex-start、center、flex-end以及stretch。

```javascript
  container: {
            backgroundColor: '#eae7ff',//背景
            marginTop: 20,
            flex: 1,
            flexDirection: 'column',
            justifyContent:'center',
            alignItems:'flex-start'

        },
```

| flex-start    | center        | flex-end  | stretch |
| ------------- |:-------------:| -----:|:--------:|:--------:|
|![](http://7xppgb.com1.z0.glb.clouddn.com/alignItems_flex-start.png)|![](http://7xppgb.com1.z0.glb.clouddn.com/alignItems_center.png)|![](http://7xppgb.com1.z0.glb.clouddn.com/alignItems_flex-end.png)|![](http://7xppgb.com1.z0.glb.clouddn.com/alignItems_stretch.png)|

* 注意
 要使stretch选项生效的话，子元素在次轴方向上不能有固定的尺寸。以下面的代码为例：只有将子元素样式中的 width: 50去掉之后，alignItems: 'stretch'才能生效。
