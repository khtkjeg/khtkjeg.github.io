---
layout: post
title: CSS 技巧总结
categories: CSS
description: 每天记录一点点，快乐工作一辈子
keywords: web css
---

>都是前人css技巧的提炼，特再此总结

## @font-face

>一种用其他服务器上的字体的好方法。很明显，如果你不能在托管服务器上找到你需要的字体，你可以在样式中使用这个方法来引入你需要的字体.

```css
@font-face {
    font-family: Blackout;
    src:url("assests/blackout.ttf")format("truetype");
}
```

## .clearfix
>如果你没法清除元素的浮动，这是清除浮动一种方法。你可以对任何HTML元素单独使用这种方法.

```css
.clearfix:after {
    content:".";
    display:block;
    clear:both;
    visibility:hidden;
    line-height:0;
    height:0;
}
```

## @media
>设置你当前响应网站的媒介，它能帮助你根据用户的显示器调整网站的宽度.

```css
@mediascreenand (max-width:960px){}
```

## transform: rotate(30deg)
>结合这些转换属性和CSS转场效果来创造有意思的动态效果

```css
.title {transform: rotate(40deg);}
```

## background-size
>这条规则帮助你在网站中适应全屏幕背景。这不像之前的CSS版本必须写很笨重的代码

```css
body {
    background:url(image.jpg)no-repeat;
    background-size:100%;
}
```

## input[type="text"]
>这个input[type="text"]选择器和其他高级选择器把你从一般水平带到高级水平非常有帮助。使用属性选择器来控制输入元素的提交版本或为一个外链增加一个图标

```css
input[type="text"] {width:250px;}
```

## margin: 0 auto
>大家都知道的，这个方法可以使块级元素在父元素中居中，前提是父级设置宽度

```css
.container {margin:0 auto;}
```

## a {outline: none;}
>移除点一个链接的虚线框

```css
a {outline:none;}
```

## overflow: hidden
>这是最好的清除还没加载到你CSS里面的元素浮动的方法。使用它使网站的响应速度更快

```css
.container {overflow:hidden;}
```

## color: rgba()
>PNG图片因为它的透明性使它在网页设计中很流行并广泛使用，但是现在你可以使用下面这种方法同样实现透明。它使用红、绿、蓝三通道并设置其不透明度值来实现透明，像0.5对应%50的不透明度

```css
.btn {color: rgba(0,0,0,0.5);}
```

## font
>多种属性设置可以一行表达

```css
a{
    font-size: 1em; 
    line-height: 1.5em; 
    font-weight: bold; 
    font-style: italic; 
    font-variant: small-caps; 
    font-family: verdana,serif 
}
a{
    font: 1em/1.5em bold italic small-caps verdana,serif;
}
```

## !important
>优先级会比和它同名属性的优先级高

```css
.body{
    margin-top: 3.5em !important;
}
```

## 垂直调整
>大多数情况下可以用vertical-align，如tabel中vertical-align: middle，
但是高度限制后可以用line-height属性
