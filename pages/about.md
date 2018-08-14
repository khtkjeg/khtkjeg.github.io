---
layout: page
title: About
description: 打码改变世界
keywords: Ming Lu, 路明
comments: true
menu: 关于
permalink: /about/
---

我是路明，突破极致，只为追求那个不平凡的自己。

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## 专业技能

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
