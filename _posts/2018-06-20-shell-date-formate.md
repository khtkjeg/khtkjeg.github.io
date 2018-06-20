---
layout: post
title: shell 日期处理
categories: shell
description: 每天记录一点点，快乐工作一辈子
keywords: shell
---

> 通过给定日期，格式化成我们想要的参数。


### 例子1

通过date格式化，执行`sh test.sh '2018-06-08 12:00:00'`,

```shell
#!/bin/bash
day=`date -d "$1" +'%d' |sed -r 's/0+([1-9])/\1/g'`
if [ "$day" -gt 10 ]
then echo "yes"
else echo "no"
fi
```

### 例子2

字符串格式化

```shell
#!/bin/bash
var=$1
ss=${var:0:2}
if [ "$ss" -gt 10 ]
then echo "yes"
else echo "no"
fi

```


> ok。