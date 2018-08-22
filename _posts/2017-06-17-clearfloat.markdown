---
layout:     post
title:      "清除浮动的七种方法"
date:       2017-6-17
author:     "leeyixuan"
header-img: "img/background/post-bg-clearfloat.jpg"
tags:
    - css
---
---


## 清除浮动是什么？

清除浮动是为了解决子元素浮动导致父容器高度坍塌。具体来说，就是当子元素全部设为float（全部脱离文档流，父元素内部没有容纳任何元素），而且父元素没有设置宽高，此时父元素高度为0，没有出现在页面上。
清除浮动总结起来一共有7种，其实按照原理，这几种方法可以归纳为两种：**clear:both法**和**构造BFC法**。
## 方案1——clear:both法
>使用clear属性时要注意的是，clear是仅作用于当前元素，例如元素A是浮动元素，靠左显示。元素B是block元素紧跟在A后面。此时要清除浮动，是在B上设clear:left。你在A上设clear:right是没有用的，因为A的右边没有浮动元素。


在父级盒子的最后添加一个盒子，并为该盒子设置**clear：both**，该盒子要设置内容隐藏：    

方法1. 可以直接添加一个空的`<div>`     
方法2. 可以直接添加一个换行符`<br>`    
方法3. 使用**after**伪元素    

对于前两种方法而言具有以下优缺点：
优点：简单，代码少，浏览器兼容性好。
缺点：需要添加大量无语义的html元素，代码不够优雅，后期不容易维护。

第3种方法和前两种方法相比具有以下优缺点：
优点：不需要添加大量无语义的html元素，代码不够优雅。
缺点：代码较多，浏览器兼容性差。


## 方案2——构造BFC法

>BFC（块级格式化上下文），是在CSS2.1中提出的一个概念。BFC是一个独立的渲染区域，计算BFC的高度时，浮动元素也参与计算。所以我们可以通过触发父div的BFC属性，使下面的子div都处在父div的同一个BFC区域之内。

方法4. 父元素设置 overflow:hidden    
方法5. 父元素设置 overflow:auto     
方法6. 父元素也设成float       
方法7. 父元素设置display:table      


以上方法或多或少都会影响当前页面的布局，并且在IE浏览器中没有BFC的概念，浏览器兼容性差，始终不是完美的方案。

## IE兼容方案
在IE6，7下没有BFC这个概念，但是有一个与BFC性质相似的概念：layout。

有些元素例如table，input本身就has layout，我们可以通过以下途径触发元素的layout属性：
1. position:absolute；
2. float不为none；
3. display:inline-block；
4. height：除auto外任意值；
5. width：除auto外任意值；
6. zoom：除normal外任意值；

所以在IE6-7下，清除浮动除了可以使用clear:both外（::after伪元素无法使用），另一种方法就是让父元素has layout。

## 最佳方案：
 
```
.fix { 
	*zoom: 1; 	
	}
/* 触发 hasLayout ，兼容IE6和IE7*/ 
.fix:after { 
	content: ''; 
	display: table; 
	clear: both; 
	}
```

