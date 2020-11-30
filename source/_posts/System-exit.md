---
layout: tweet
title: System.exit()
date: 2020-10-28 23:00:01
tags: 
  - java
  - 学习笔记
---

System.exit(int  status)这个方法是用来结束当前正在运行中的java虚拟机。

status是非零参数，那么表示是非正常退出。

System.exit(0)是正常退出程序，而System.exit(1)或者说非0表示非正常退出程序。

System.exit(status)不管status为何值都会退出程序。和return 相比有以下不同点：return是回到上一层，而System.exit(status)是回到最上层

## 示例

在一个if-else判断中，如果我们程序是按照我们预想的执行，到最后我们需要停止程序，那么我们使用System.exit(0)，而System.exit(1)一般放在catch块中，当捕获到异常，需要停止程序，我们使用System.exit(1)。这个status=1是用来表示这个程序是非正常退出。
