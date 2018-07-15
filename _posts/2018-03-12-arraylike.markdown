---
layout:     post
title:      "类数组对象"
subtitle:   "arguments|nodelist|HTML collection"
date:       2018-3-12
author:     "leeyixuan"
header-img: "img/background/post-bg-arraylike.jpg"
tags:
    - js
---

## 类数组对象
JavaScript 类数组对象的定义：**可以通过索引访问元素，并且拥有 length 属性。**

```
var foo = {
    0: 'Java',
    1: 'Python',
    2: 'Scala',
    length: 3
}
```

类数组对象在某些方面和数组非常相似，看上去可以直接使用从 Array.prototype 上继承的方法。然而，除了forEach方法， 类数组对象没有这些类似数组的方法。

## 类数组对象的分类
类数组一共有三种，分别是arguments、HTML collection和Nodelist。
### 1. arguments
<img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1525664392591.jpg" />

函数的命名参数只是提供便利，并不是必须的，函数体内部通过`arguments`对象访问传进函数的参数。arguments不是Array的实例。
- arguments对象是所有（非箭头）函数中都可用的局部变量。
- arguments对象和命名参数的内存空间是独立的，会随时进行同步。
- arguments对象的长度是由传入的参数个数决定的，没有传递值的命名参数将被自动赋予undefined。（如果命名参数的个数是2，只传入了1个参数，但是在函数体内部定义了`arguments[1]=10`，此时第二个命名参数和`arguments[2]`并不会同步，第二个命名参数是undefined，而`arguments[1]=10`实际上变成了一个键为1，值为10的属性）
- arguments弥补了JavaScript没有**函数重载**的缺憾。
- 可以向函数传递任意数量的参数（用不用是你的事）
- 具有一个特殊的属性**callee**。该属性是一个指针，指向拥有这个arguments对象的函数。

> **javascript没有函数签名**：参数类型，参数个数，参数位置，出入参数，js统统不关心，它所有的值都被放到arguments中了。需要返回值的话直接return，不用声明。

### 2. Nodelist
NodeList是一个Node集合。在DOM中，Node的类型总共有12种，包括element、comment、text等等。
Node.childNodes、document.getElementsByName()、document.querySelectAll()方法返回的就是Nodelist对象。

**注意**：querySelectorAll()返回的nodeList对象比较特殊，它是个静态（static）的对象。其他返回的类数组对象都是节点的引用，是动态的。
### 3. HTML collection
HTMLCollection是element集合。它具有一个nameItem()方法，可以返回集合中name属性和id属性值得元素。
HTMLDocument 接口的许多属性都是 HTMLCollection 对象，document.images和document.forms是HTMLCollection对象。
Node.children和document.getElementsByXXX()方法返回的就是HTML collection对象。

### NodeList v.s. HTML collection
1. 包含节点的类型不同
- NodeList：一个节点的集合，既可以包含元素和其他节点(注释节点、文本节点等)。
- HTMLCollection：元素集合, 只有Element
2. 使用方法
相同点：
- 它们都有length属性
- 都有元素的getter，叫做item，可以传入索引值取得元素。
- 都是类数组
不同点：
HTMLCollection还有一个nameItem()方法，可以返回集合中name属性和id属性值的元素。

## 类数组转换成数组

```
var arrayLike = {0: 'name', 1: 'age', 2: 'sex', length: 3 }
// 1. slice
Array.prototype.slice.call(arrayLike); // ["name", "age", "sex"] 
// 2. splice
Array.prototype.splice.call(arrayLike, 0); // ["name", "age", "sex"] 
// 3. ES6 Array.from
Array.from(arrayLike); // ["name", "age", "sex"] 
// 4. apply
Array.prototype.concat.apply([], arrayLike)
```

## 类数组直接借用数组的方法
类数组不转成数组，利用call，直接借用array的方法。


```
var arrayLike = {0: 'name', 1: 'age', 2: 'sex', length: 3 }

Array.prototype.join.call(arrayLike, '&'); // name&age&sex

Array.prototype.slice.call(arrayLike, 0); // ["name", "age", "sex"] 
// slice可以做到类数组转数组

Array.prototype.map.call(arrayLike, function(item){
    return item.toUpperCase();
}); 
// ["NAME", "AGE", "SEX"]
```
## 为类数组对象添加方法
Array.prototype 上的方法添加到 NodeList.prototype 上。
```
var arrayMethods = Object.getOwnPropertyNames( Array.prototype );


function attachArrayMethodsToNodeList(methodName)
{
  if(methodName !== "length") {
    NodeList.prototype[methodName] = Array.prototype[methodName];
  }
};

arrayMethods.forEach( attachArrayMethodsToNodeList );
 
var divs = document.getElementsByTagName( 'div' );
var firstDiv = divs[ 0 ];

firstDiv.childNodes.forEach(function( divChild ){
  divChild.parentNode.style.color = '#0F0';
});
```
## ES6的 ... 运算符
将arguments转成数组
```
function func(...arguments) {
    console.log(arguments); // [1, 2, 3]
}

func(1, 2, 3);
```

## Reference
1.  [NodeList- MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/NodeList)
2.  [Array.prototype.slice()-MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)  
