---
layout: post
title: nodejs的几种部署方式
categories: nodejs
description: 每天记录一点点，快乐工作一辈子
keywords: pm2 forever supervisor
---

> 开发nodejs的人都知道它是单进程的模式，每次执行的时候都要执行`node XXX.js`，`CTRL+C`后服务就停止了，如果他们能都在后台运行，并且能够实时监控到这些状态信息，就解决了部署的很多麻烦，这里介绍三个常用的后台开发部署工具：`supervisor`、`pm2`、`forever`;不同的工具应用场景也不同，下面详细介绍一下：


