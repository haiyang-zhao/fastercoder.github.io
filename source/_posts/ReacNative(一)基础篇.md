title: react native基础(一)
date: 2017-07-12 16:47:55
categories: react native
tags:  react native
---


#react native

##使用StyleSheet创建样式
```
	 <View style={styles.container}>
     </View>
     
     
    const styles = StyleSheet.create({
    container: {
        backgroundColor: '#ff0000',
        flex: 1
    }
});
```
##常用属性

```
const styles = StyleSheet.create({
    container: {
        backgroundColor: '#ff0000',//背景
        flex: 1,//填满屏幕
        margin: 30,//外边距
        borderWidth: 10,//边框宽度
        borderColor: '#00ff00',//边框颜色
        borderRadius: 16,//圆角半径
        shadowColor: '#0000ff',//阴影颜色
        shadowOpacity: 0.6,//阴影透明度
        shadowRadius: 5,//阴影角度
        shadowOffset: { //x,y偏移
            height: 1,
            width: 0
        }

    }
});
```
![常用属性](http://7xppgb.com1.z0.glb.clouddn.com/style_common_properties.png)


##文字样式

```
  <View style={styles.container}>
    <Text style={styles.text}>
      forevercoder.com
    </Text>
  </View>
  
   text: {
            fontSize: 26,
            color: '#6435c9',
            textAlign: 'center',
            fontStyle: 'italic',
            letterSpacing: 2,//字间距
            lineHeight: 33,//行间距
            fontFamily: 'Helvetica Neue',//字体
            fontWeight: 'bold',//粗细
            textDecorationLine: 'underline',//下划线 
            textDecorationStyle: 'dashed',//  虚线
            textDecorationColor: '#00ffff'
        }
```




