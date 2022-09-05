---
title: nodemon
date: 2022-07-18 08:12:46
cover: https://6973-isundae-3gbm0y3523031b2a-1258186980.tcb.qcloud.la/blog/202209041448201.png
tags: nodemon
---

![](https://6973-isundae-3gbm0y3523031b2a-1258186980.tcb.qcloud.la/blog/202209041448201.png)



    Swap [nodemon](https://nodemon.io/) instead of **node** to run your code, and now your process will automatically restart when your code changes. To install, get [Node.js](https://nodejs.org/), then from your terminal run:



# 安装

`npm install -g nodemon`

# 配置

有两种配置方式

## 1. 创建nodemon.json

```json
{
 "verbose": true,
 "ignore": ["*.test.js", "fixtures/*"],
 "execMap": {
 "rb": "ruby",
 "pde": "processing --sketch={{pwd}} --run"
 }
}
```

## 2. 在package.json中

```json
{
 "name": "nodemon",
 "homepage": "http://nodemon.io",
 "...": "... other standard package.json values",
 "nodemonConfig": {
 "ignore": ["test/*", "docs/*"],
 "delay": "2500"
 }
}
```

# 使用

`nodemon [your app.js]`

# 参数

## -h或-help:

查看帮助菜单

`nodemon -h`

## --exec

运行非js程序

```bash
● nodemon --exec ts-node src/index.ts 
// 通过ts-node运行src目录下的index.ts

● nodemon --exec "python -v" ./app.py 
// 通过verbose模式的python运行app.py
```

如果编译的时候带参数，则需要加""但如果没有参数则不需要""

## --ignore

热更新时忽略某些文件/目录/文件模式

`nodemon --ignore lib/     忽略lib内部文件更改`

## --watch

热更新时监视更多的文件，若这些被监视的文件更新，则项目也会进行热更新

`nodemon --watch index.js --watch ./dist/ceshi.js`

一般用于不构成依赖关系，但想要监视的文件。

## -e

可以使用-e（或--ext）开关指定自己的列表

默认情况下，nodemon查找与文件.js，.mjs，.coffee，.litcoffee，和.json扩展。

`nodemon -e js，pug   // 现在pug文件更新时，也会导致项目热更新了`
