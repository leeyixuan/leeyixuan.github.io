---
layout:     post
title:      "实现滚动无限加载效果"
subtitle:   "sroll爬坑之旅"
date:       2017-5-3
author:     "leeyixuan"
header-img: "img/background/post-bg-git.jpg"
tags:
    - js
---


scrollInfiniti是百度IFE的一个任务，任务要求实现滚动无限加载效果。demo地址：https://leeyixuan.github.io/IFE/scrollInfinite/index.html

## 设计思路
### 主体思路  
step1. 初始化获取数据并渲染页面；    
step2. 监听scroll事件，判断当前是否已经滚动到页面底部。如果页面滚动到底部，则获取新的数据并渲染页面。

### 实现细节
1. 返回promise对象，模拟数据。
```
function getdata(num, curret) {
        return new Promise(function (resolve, reject) {
            setTimeout(function () {
                var data = [];
                for (var i = current; i < current + num; i++) {
                    data.push('item ' + (i + 1));
                }
                resolve(data);
            }, 1000);
        });
    }
```
2. 获取数据并渲染页面
```
getdata(num, current).then(function (data) {
                var block = document.createDocumentFragment();
                data.forEach(function (item) {
                    var li = document.createElement('li');
                    li.innerText = item;
                    block.appendChild(li);
                });
                list.appendChild(block);

                current += num;
                isAddItem=false;
            }).catch(function (err) {
                console.log(err);
            });
```
这部分主要操作包括：异步获取数据，将新数据包装成DOM节点添加到页面。    
这里使用`document.createDocumentFragment()`创建一个文档片段。因为文档片段存在于内存中，并不在DOM树中，所以将子元素插入到文档片段时不会引起页面回流(reflow)，会起到优化性能的作用。避免了因createElement多次添加到document.body引起的效率问题。
3. 判断页面滚动到页面底部
如果元素滚动到底，下面等式返回true，没有则返回false。
`element.scrollHeight - element.scrollTop === element.clientHeight`

## scroll相关属性和方法总结

首先先介绍几个Element对象的属性：
- clientHeight/clientWidth：只读。包括padding，但不包括滚动条、border和margin。   
`element.clientHeight = element.height + element.padding `

<img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1531222115532.png" />

- offsetHeight/ offsetWidth：只读。这两个属性返回的是元素的高度或宽度，包括元素的边框、内边距和滚动条。   
`element.offsetHeight = element.clientHeight + 滚动条 + element.border`

<img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1531222115537.png" />

- scrollHeight,/scrollWidth：只读。返回元素内容的整体尺寸，包括元素看不见的部分（需要滚动才能看见的）。返回值包括padding，但不包括margin和border。   
`element.scrollHeight = element.clientHeight + 溢出的部分` 

<img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1531222115538.png" />

- scrollTop/scrollLeft：**可读可写**。获取或设置一个元素的内容垂直滚动的像素数。
设置scrollTop的值小于0，scrollTop 被设为0；如果设置了超出这个容器可滚动的值, scrollTop 会被设为最大值。

<img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1531222115540.png" />


再介绍几个window对象的属性：
- window.innerHeight/window.innerWidth：只读。返回网页在当前窗口中可见部分的高度和宽度，即“视口”（viewport）的大小（单位像素）。
- window.outerHeight，window.outerWidth：只读。返回浏览器窗口的高度和宽度，包括浏览器菜单和边框（单位像素）。

<img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1531222115657.png" />

- window.scrollX/window.scrollY：只读。属性返回页面的水平（垂直）滚动距离，单位都为像素。
- window.pageXOffset/window.pageYOffset：window.scrollX和window.scrollY别名。


再再介绍几个window对象滚动的方法：
- window.scrollTo()：将文档滚动到指定**位置**（单位px）。
- window.scroll()：window.scrollTo()方法的别名。
- window.scrollBy()：方法用于将网页滚动指定**距离**（单位px）。

最后介绍几个element对象滚动的属性和方法：
- Element.scrollTop/Element.scrollLeft：该属性可读可写，将元素滚动指定距离。
- Element.scrollIntoView()



**易混淆点**：document.documentElement.offsetWidth，document.body.offsetWidth，window.innerWidth，$(window).width()
>document.documentElement属性返回当前文档的根节点（root）。HTML网页的该属性，一般是`<html>`节点。
>document.body属性指向`<body>`节点。


- document.documentElement.offsetWidth：`document.documentElement.offsetWidth = html.width + html.padding + 滚动条 + html.border`
- document.body.offsetWidth：
`document.body.offsetWidth = body.width + body.padding + 滚动条 + body.border`     
`document.body.offsetWidth = document.documentElement.scrollWidth`


以下的规则只是在不另外为html元素和body元素指定width和height属性时生效：
- window.innerWidth：
`window.innerWidth = html.width + html.padding + 滚动条 + html.border`
`window.innerWidth = document.documentElement.offsetWidth `

- $(window).width()：
`$(window).width() = html.width + html.padding`
`$(window).width() = document.documentElement.clientWidth `



 
 
 
**总结**：
1. window的outerHeight/outerWidth/innerHeight/innerWidth属性和浏览器的视窗（viewport）有关，一直都是固定的。另外，调节网页大小的时候innerHeight/innerWidth属性会随之变化。
2. document.documentElement.offsetWidth/offsetHeigth是html元素的大小；
document.body.offsetWidth/offsetHeigth是body元素的大小；
window.innerWidth正在代表视窗viewport；
window.outerWidth在视窗的基础上还加上了一些工具栏的大小。

3. body节点的offsetWidth/offsetWidth就是实际页面的大小，不过被viewport限制了。从`document.body.offsetWidth = document.documentElement.scrollWidth`就可以看出来。


4. 获取屏幕的可视区域的宽度：`document.documentElement.clientWidth`或者`$(window).width`。

5. 一般情况下，不会为html元素和body元素指定width和height属性，所以body和html的width和height都是100%（width和height的默认值是auto，而此时的padding、margin、border都没有设置，默认值是0，最后浏览器计算除auto的值是100%）
再次基础上，可以得到以下结论：
- window.innerWidth实际上也是HTML元素大小，等于document.documentElement.offsetWidth；
- 视窗大小限制了document.documentElement.offsetWidth的大小；
- 滑动条是来自HTML元素，实际滑动的是HTML盒子；
- window的scrollY属性和documentElement.scrollTop属性都能获取页面滚动过的距离。

**兼容性问题**：
1.  IE8以及以下不支持window.innerWidth/window.innerHeight。
2. IE，火狐，谷歌，360浏览器，Safari都支持document.documentElement.clientWidth/ document.documentElement.clientHeight。
3. 兼容性代码如下:
```
//获取滚轮滚动的距离,适配所有的浏览器
function getScrollY(){
    return window.pageYOffset || document.documentElement.scrollTop;
}
//设置垂直方向滚轮滚动的距离,适配所有的浏览器，num为滚动距离
function setScrollY(num){
    document.body.scrollTop = document.documentElement.scrollTop = num;
}
```







## reference
1. [Element.scrollTop-MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/scrollTop)
2. [window 对象](https://wangdoc.com/javascript/bom/window.html#windowscrollto%EF%BC%8Cwindowscroll%EF%BC%8Cwindowscrollby)
3. [一张图彻底掌握scrollTop, offsetTop, scrollLeft, offsetLeft](https://github.com/pramper/Blog/issues/10)
4. [获取屏幕宽高width(),outerWidth,innerWidth,clientWidth的区别](https://segmentfault.com/a/1190000010746091)
5. [javascript中与Scroll有关的知识点](https://github.com/hehongwei44/my-blog/issues/207)
