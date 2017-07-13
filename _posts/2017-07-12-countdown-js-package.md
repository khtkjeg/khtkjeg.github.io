---
layout: post
title: 简单实现多个倒计时的页面显示
categories: javascript
description: 封装一个倒计时对象和方法，基于定时器和当前时间比较
keywords: javascript
---

通过定时器与当前时间做差得到时间戳，再除以 1000 转成秒,再通过秒转成天、小时、分钟、秒格式；为了实现给页面多个id绑定倒计时，可以通过创建一个倒计时的对象，通过 prototype 属性添加自定义的属性的方式实现两个方法，创建倒计时和删除倒计时。

## 说明

1、定义一个倒计时对象 `Ticts`
{% highlight ruby %}
var Ticts=function Ticts() {
    this.ticts = {}; 
};
{% endhighlight %}
2、为"Ticts"对象添加自定义方法 `createTicts` 和 `deleteTicts`
{% highlight ruby %}
Ticts.prototype.createTicts=function(id,endtime){
})
Ticts.prototype.deleteTicts = function(id) {
})
{% endhighlight %}
3、通过id创建对应的定时器对象,倒计时结束后删掉定时器对象
{% highlight ruby %}
Ticts.prototype.createTicts=function(id,endtime){
    var ticts=this;
    var now = new Date();
    var endDate = new Date(endtime);
    var time=endDate.getTime()-now.getTime();
    var _ticts=this.ticts[id] = {
        endtime:endtime,
        id:id,
        time:time,
        interval:setInterval(function(){
            var t = null;
            var d = null;
            var h = null;
            var m = null;
            var s = null;
            t=_ticts.time/1000;
            d = Math.floor(t / (24 * 3600));
            h = Math.floor((t - 24 * 3600 * d) / 3600);
            m = Math.floor((t - 24 * 3600 * d - h * 3600) / 60);
            s = Math.floor((t - 24 * 3600 * d - h * 3600 - m * 60));
            document.getElementById(id).innerHTML = d + '天' + h + '小时' + m + '分钟' + s + '秒';
            _ticts.time -= 1000;
            if (_ticts.time < 0)
				ticts.deleteTicts(id);                               
        },1000)
    }       
}
{% endhighlight %}
4、删除倒计时对象中的定时器对象
{% highlight ruby %}
Ticts.prototype.deleteTicts = function(id) {
    clearInterval(this.ticts[id].interval);
    delete this.ticts[id];
};
{% endhighlight %}
5、最后创建一个倒计时对象，并添加到 window 方法中
{% highlight ruby %}
window.Ticts=new Ticts();
{% endhighlight %}
## 引用
{% highlight ruby %}
<script src="tick.js"></script>
{% endhighlight %}
## 调用
{% highlight ruby %}
Ticts.createTicts("daojishi1","2017-07-12 21:20:20");
Ticts.createTicts("daojishi2","2017-07-12 21:30:12");
{% endhighlight %}
## 总结
通过简单的实例熟悉了 `javascript` 中自定义属性的创建，`this`指针指向关系等知识点。
## 源码地址
*　[tick]('https://github.com/khtkjeg/Tick')

