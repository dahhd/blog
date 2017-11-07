---

title: 一个简单的python小脚本
date: 2016-12-05 10:51:51
tags:
categories: 混口饭吃

---

##一个简单的python小程序-脚本

由于自己开发用的是macBook，所有很多时候都是用终端命令行，在进行一些日常操作，前几日正好需要输入终端的某一条命令，反复进行下去，实现循环执行功能。

用git下载文件到本地文件夹，想实时监测到下载的进度-和感受网络情况是否还不错，就需要循环监测下载到本地的文件大小是多少。

很简短的一段代码：


```
#usr/bin/python
#coding:utf-8

import os    #系统函数模块
import time  #时间函数模块

x = 1
while x<= 100000:
    currentMB = os.system("du sh *")
    print "已经下载%rM了哦！"%currentMB
    print "哎哟不错哦！第%d次循环..."%x
    print "\n"
    time.sleep(10)  #每间隔10秒，执行一次，文件大小变化效果看的较明显。
    x += 1
```

说明：”du sh” 命令是macOS自带的系统命令，用来查看当前文件夹下每个文件的存储大小； 保存python文件，以.py结尾，拷贝此文件到所需要的文件夹目录，执行 python xxxx.py 即可。