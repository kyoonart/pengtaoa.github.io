---
title: ncm  nvm安装的时候遇到的一些坑
date: 2020-06-01 16:40:59
tags: 其他
---

### nvm安装的时候遇到的一些坑

1：nvm是切换node版本的一个工具

2：高版本的nvm切换的时候会丢失npm包这个需要自己去安装 一般安装的时候不会出现环境变量的问题 如果出现了那就有点倒霉了

<!-- more -->

需要自己下载node包https://npm.taobao.org/mirrors/node?spm=a2c6h.14029880.0.0.3ba675d7vE7Dhj

然后会发现新装的v-8xxxx版本里面的node_modules是空的这个时候就需要到刚刚下载的镜像里面把npm包拷贝过来然后就可以用npm了，否则无法使用

![](https://s1.ax1x.com/2020/06/04/t0uBa4.png)

![](https://s1.ax1x.com/2020/06/04/t0uUMV.png)

3：nvm安装的时候最好放在c盘的根目录别放在二级目录，我装了好几次都不行，其次中文路径一定不能出现！