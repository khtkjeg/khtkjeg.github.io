---
layout: post
title: nodejs的几种部署方式
categories: nodejs
description: 每天记录一点点，快乐工作一辈子
keywords: pm2 forever supervisor
---

> 开发nodejs的人都知道它是单进程的模式，每次执行的时候都要执行`node XXX.js`，`CTRL+C`后服务就停止了，如果他们能都在后台运行，并且能够实时监控到这些状态信息，就解决了部署的很多麻烦，这里介绍三个常用的后台开发部署工具：`supervisor`、`pm2`、`forever`;不同的工具应用场景也不同，下面详细介绍一下：

# supervisor

>`supervisor`适合应用在开发调试阶段，它的优点就是实时可以监控文件的改变，然后重新启动服务，避免了开发人员修改文件后频繁的重启服务，其特点类似PHP脚本语言的部署。

## npm全局安装

```shell
npm install -g supervisor
```
## 详细命令说明

```shell
Node Supervisor is used to restart programs when they crash.
It can also be used to restart programs when a *.js file changes.

Usage:
  supervisor [options] <program>
  supervisor [options] -- <program> [args ...]

Required:
  <program>
    The program to run.

Options:
  -w|--watch <watchItems>
    A comma-delimited list of folders or js files to watch for changes.
    When a change to a js file occurs, reload the program
    Default is '.'

  -i|--ignore <ignoreItems>
    A comma-delimited list of folders to ignore for changes.
    No default

  --ignore-symlinks
    Enable symbolic links ignoring when looking for files to watch.

  -p|--poll-interval <milliseconds>
    How often to poll watched files for changes.
    Defaults to Node default.

  -e|--extensions <extensions>
    Specific file extensions to watch in addition to defaults.
    Used when --watch option includes folders
    Default is 'node,js'

  -x|--exec <executable>
    The executable that runs the specified program.
    Default is 'node'

  --debug[=port]
    Start node with --debug flag. 

  --debug-brk[=port]
    Start node with --debug-brk[=port] flag.

  --harmony
    Start node with --harmony flag.
  --inspect
    Start node with --inspect flag.

  --harmony_default_parameters
    Start node with --harmony_default_parameters flag.

  -n|--no-restart-on error|exit
    Don't automatically restart the supervised program if it ends.
    Supervisor will wait for a change in the source files.
    If "error", an exit code of 0 will still restart.
    If "exit", no restart regardless of exit code.
    If "success", no restart only if exit code is 0.

  -t|--non-interactive
    Disable interactive capacity.
    With this option, supervisor won't listen to stdin.

  -k|--instant-kill
    use SIGKILL (-9) to terminate child instead of the more gentle SIGTERM.

  --force-watch
    Use fs.watch instead of fs.watchFile.
    This may be useful if you see a high cpu load on a windows machine.

  -s|--timestamp
    Log timestamp after each run.
    Make it easy to tell when the task last ran.

  -h|--help|-?
    Display these usage instructions.

  -q|--quiet
    Suppress DEBUG messages

  -V|--verbose
    Show extra DEBUG messages

Options available after start:
rs - restart process.
     Useful for restarting supervisor eaven if no file has changed.

Examples:
  supervisor myapp.js
  supervisor myapp.coffee
  supervisor -w scripts -e myext -x myrunner myapp
  supervisor -- server.js -h host -p port
```

## 应用

> 常用快捷方式命令

```shell
supervisor  app.js
```

> 在express框架中执行入口在`./bin/www`在执行时应该在`./`目录执行命令

```shell
supervisor bin/www
```

# pm2

> pm2是目前处理大并发后台程序最好的工具，能够详细的监控程序的运行状态，遇到异常能够自动重启，同时还提供高级功能设置如支持配置文件启动，配置文件中可以添加内存限制、cpu核数、输入输出日志等高级设置，同时pm2提供对外的API接口，可以根据API开发自己的监控页面。

## npm全局安装

```shell
npm install -g pm2
```

## 应用

> 启动一个简单的服务命令

```shell
[root@localhost www]# pm2 start helloworld.js --name 'helloworld'
[PM2] Spawning PM2 daemon
[PM2] PM2 Successfully daemonized
[PM2] Starting helloworld.js in fork_mode (1 instance)
[PM2] Done.
┌────────────┬────┬──────┬──────┬────────┬─────────┬────────┬─────────────┬──────────┐
│ App name   │ id │ mode │ pid  │ status │ restart │ uptime │ memory      │ watching │
├────────────┼────┼──────┼──────┼────────┼─────────┼────────┼─────────────┼──────────┤
│ helloworld │ 0  │ fork │ 2251 │ online │ 0       │ 0s     │ 14.715 MB   │ disabled │
└────────────┴────┴──────┴──────┴────────┴─────────┴────────┴─────────────┴──────────┘
 Use `pm2 show <id|name>` to get more details about an app
```

> 启动一个express框架服务

```shell
pm2 start ./bin/www --name 'draw'
```

## 常用命令说明

1. `pm2 list`  查看全部运行任务列表
![](/images/posts/node/pm2-list.png)
```shell
[root@localhost draw]# pm2 list
┌──────────┬────┬──────┬───────┬────────┬─────────┬────────┬─────┬───────────┬──────┬──────────┐
│ App name │ id │ mode │ pid   │ status │ restart │ uptime │ cpu │ mem       │ user │ watching │
├──────────┼────┼──────┼───────┼────────┼─────────┼────────┼─────┼───────────┼──────┼──────────┤
│ draw     │ 0  │ fork │ 28303 │ online │ 0       │ 3h     │ 0%  │ 38.4 MB   │ draw │ disabled │
└──────────┴────┴──────┴───────┴────────┴─────────┴────────┴─────┴───────────┴──────┴──────────┘
 Use `pm2 show <id|name>` to get more details about an app
```
2. `pm2 stop  <app_name|id|all>`  停止

3. `pm2 delete <app_name|id|all>` 删除

4. `pm2 restart <app_name|id|all>` 重启

5. `pm2 reload <app_name|id|all>` 重载

6. `pm2 monit`  监控
![](/images/posts/node/pm2-monit.png)

7. `pm2 start app.js --max_memory_restart 1024M`  当内存超过1024M时自动重启

## 高级用法

> `pm2 draw` 生成 draw.json配置文件

```json
{
    /**
    * Application configuration section
    * http://pm2.keymetrics.io/docs/usage/application-declaration/
    * 多个服务，依次放到apps对应的数组里
    */
    apps : [
    // First application
        {
            name      : "nova",
            max_memory_restart: "300M",
            script    : "/root/nova/app.js",
            out_file : "/logs/nova_out.log",
            error_file : "/logs/nova_error.log",
            instances  : 4,
            exec_mode  : "cluster",
            env: {
                NODE_ENV: "production"
            }
        }
    ]
 }
```
> `pm2 start draw.json`

# forever

> forever相比于pm2，同样能够实现后台异常重启服务，能够后台服务运行，能够监控文件变化自动重启服务，能够输出日志，缺点是功能单一不能实现实时监控，不能应用到大并发应用部署，不能知道资源部署，如果我们只是本地跑个简单的应用使用还是很方便的。

## npm全局安装

```shell
npm install -g forever 
```

## 常用命令

1. `forever start app.js `  启动
2. `forever stop app.js`    停止
3. `forever start -l forever.log app.js `  指定forever信息输出文件，当然，默认它会放到~/.forever/forever.log
4. `forever start -o out.log -e err.log app.js ` app.js中的日志信息和错误日志输出, -o 就是console.log输出的信息，-e 就是console.error输出
5. `forever start -l forever.log -a app.js `  -a 追加日志
6. `forever start -w app.js `  监听文件夹是否改动
7. `forever list  `  显示所有任务


# 总结

> 综合比较supervisor非常适合开发阶段使用，pm2相比forever丰富高级很多，部署环境首选pm2

