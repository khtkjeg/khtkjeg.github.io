---
layout: post
title: 谷歌最新推出Parcel打包工具(一)
categories: parcel
description: 每天记录一点点，快乐工作一辈子
keywords: web,parcel
---

> Parcel 是一个Web应用程序打包器(bundler) ，与以往的开发人员使用的打包器有所不同。它利用多核处理提供极快的性能，并且你不需要进行任何配置。

### 安装

npm安装命令

```shell
npm install -g parcel-bundler
```

使用以下命令在你的项目目录中创建一个 package.json 文件：

```shell
npm init -y
```

### 示例

创建一个 index.html 和 index.js 文件

```html
<html>
    <body>
      <script src='./index.js'></script>
    </body>
</html>
```

```javascript
console.log("hello world");
```

Parcel 内置了一个开发服务器，这会在你更改文件时自动重建你的应用程序，并支持 模块热替换 ，以便你快速开发。你只需指定 入口文件 即可：

```shell
parcel index.html
```

现在在你浏览器中打开 http://localhost:1234/ 。 您也可以使用 -p <port number> 选项覆盖默认端口。

如果您没有自己的服务器，或者你的应用完全是客户端渲染的，那么请使用开发服务器。如果你有自己的服务器，您可以在 watch 模式下运行 Parcel 。这样在文件更改时，Parcel 仍然会自动重建文件，并支持模块热替换，但不启动 Web 服务器。

```shell
parcel watch index.html
```

当您准备为生产构建时，build 模式会关闭监视，并且只会构建一次。 参见 Production 部分了解更多细节。

### 总结
parcel 很好的解决了webpack配置复杂和打包性能的问题，未来上升趋势值得关注