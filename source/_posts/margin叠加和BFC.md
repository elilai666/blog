---
title: margin叠加和BFC
date: 2019-01-20 17:37:47
tags: css
categories: css
---
## 引子
自己最初根本不知道 BFC 和 margin 叠加的概念，但是在写 web 简历页面的时候发现个奇怪的现象，才了解到原来 CSS 还有这么个神奇的东西。先简单说说当初遇到的现象吧，就是下面这样：

```html
<body>
    <div class="first-block"></div>
    <div class="second-block">
	    <h2>title</h2>
    </div>
</body>
```

css:
```css
.first-block
{
	background: #F44336;
	width: 200px;
	height: 200px;
}
.second-block
{
	background: #00BCD4;
	width: 200px;
	height: 200px;
}
```

很简单是吧，可是看到浏览器中的效果时我还是有点懵逼，是这样的：

![](http://ww1.sinaimg.cn/large/6a06ba4bgw1f6th3d7wnpj206n0ccdfu.jpg)

为什么 first-block 和 second-block 之间会有这么宽的间距？我并没有给他们设置 margin 啊！

## 原因
### 外边距折叠
那么为什么会这样呢，先自己动手找下原因吧，F12审查下元素，很快找到原因了：

![](http://ww2.sinaimg.cn/large/6a06ba4bgw1f6thmbiuegj2071059jra.jpg)

原来这个间距是h2的上外边距引起的，可是 h2 不是 second-block 的子元素么，为什么它的 margin 可以穿透出去顶到 first-block ？

不懂就google一下吧，没费多大劲就找到原因了：

>在CSS当中，相邻的两个盒子（可能是兄弟关系也可能是祖先关系）的外边距可以结合成一个单独的外边距。这种合并外边距的方式被称为折叠，并且因而所结合成的外边距称为折叠外边距。

原来是外边距折叠造成的，以前也在head first上看到过这个概念，但是当时书上只提到说两个相邻的块元素的上下外边框直接会产生折叠现象，当时想当然地以为所谓的相邻的块元素是指两个兄弟关系的块元素，没想到也包括祖先关系。

### 折叠的条件
两个块元素要产生折叠现象，必须满足一个必备条件，那就是这两个元素的 margin 必须是**相邻**的，那么如果定义相邻呢，w3c 规范，两个 margin 是邻接的必须满足以下条件：
* 必须是处于常规文档流（非float和绝对定位）的块级盒子,并且处于同一个 BFC 当中。
* 没有inline盒子，没有空隙，没有 padding 和 border 将他们分隔开。
* 都属于垂直方向上相邻的外边距，可以是下面任意一种情况：
    * 元素的 margin-top 与其第一个常规文档流的子元素的 margin-top。
    * 元素的 margin-bottom 与其下一个常规文档流的兄弟元素的 margin-top。
    * height 为 auto 的元素的 margin-bottom 与其最后一个常规文档流的子元素的 margin-bottom。
    * 高度为0并且最小高度也为0，不包含常规文档流的子元素，并且自身没有建立新的BFC的元素的 margin-top 和 margin-bottom。

### BFC
好不容易才理解了折叠，怎么又跑出来个BFC？继续google吧：
>BFC(Block formatting context)直译为"块级格式化上下文"。它是一个独立的渲染区域，只有 Block-level box 参与， 它规定了内部的 Block-level Box 如何布局，并且与这个区域外部毫不相干。

简单来说，CSS 里的布局和渲染都是以 Box 为单位的，不同的 Box 有不同的布局规则，而 BFC 是其中的一种，相似的还有 IFC 和 CSS3 中增加的 GFC 和 FFC，这里就不深入介绍了。

#### BFC布局规则
BFC 的布局遵从如下规则：
* 内部的 Box 会在垂直方向，一个接一个地放置。
* Box 垂直方向的距离由 margin 决定。属于同一个 BFC 的两个相邻 Box 的 margin 会发生重叠
* 每个元素的 margin box 的左边， 与包含块 border box 的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。
* BFC 的区域不会与 float box 重叠。
* BFC 就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。
* 计算BFC的高度时，浮动元素也参与计算。

#### 哪些元素会生成BFC
* 根元素。
* float 属性不为 none。
* position 为 absolute 或 fixed。
* display 为 inline-block, table-cell, table-caption, flex, inline-flex。
* overflow 不为 visible(hidden, scroll, auto)。

结合 BFC 布局规则一看，以前学过的好多 CSS 规则方法的原理都在 BFC 这啊，比如最常用的清除浮动什么的。

## 解决方案
知道了问题产生的原因，那就可以对症下药了。
### 方法一

我们注意到折叠条件的第二条：***没有inline盒子，没有空隙，没有 padding 和 border 将他们分隔开。*** 自然而然地就可以想到我们可以把 second-block 加上一个边框来让折叠失效：
    
修改css：
```css
.second-block
{
    background: #00BCD4;
    width: 200px;
    height: 200px;
    border: 1px solid rgba(0,0,0,0); /*添加一个透明边框*/
}
```


效果：

![](http://ww4.sinaimg.cn/large/6a06ba4bgw1f6tk7zh813j206h0bs3yh.jpg)

嗯，折叠问题解决了，但是由于有1px的边框，second-block 看起来会比 first-block 宽一点，没关系，添加`box-sizing: border-box`属性可以解决这个问题：

修改css：

```css
.second-block
{
    background: #00BCD4;
    width: 200px;
    height: 200px;
    border: 1px solid rgba(0,0,0,0); /*添加一个透明边框*/
    box-sizing: border-box;
}
```

效果：
![](http://ww1.sinaimg.cn/large/6a06ba4bgw1f6tkfcirmoj206b0ca0so.jpg)

### 方法二
 
我们知道，这里之所以会产生折叠，是因为两个 block 处于同一个 BFC（根元素生成的BFC）中，那我们可以让 second-block 单独生成一个 BFC，就可以防止出现折叠了。

修改css：
```css
.second-block
{
    background: #00BCD4;
    width: 200px;
    height: 200px;
    overflow: hidden; /*触发BFC*/
}
```

效果：

![](http://ww1.sinaimg.cn/large/6a06ba4bgw1f6tkvmtjuoj206n0c10sp.jpg)

嗯，完美~！

    
参考文章：

* http://www.w3cplus.com/css/understanding-bfc-and-margin-collapse.html
* http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html
