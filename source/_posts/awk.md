---
layout: tweet
title: awk
date: 2020-10-28 21:59:21
tags: 
  - linux
  - 学习笔记
---

AWK 是一种处理文本文件的语言

```·
awk [选项参数] 'script' var=value file(s)
或
awk [选项参数] -f scriptfile var=value file(s)
```
```
# 每行按空格或TAB分割，输出文本中的1、4项
awk  '${print $1,$4}' log.txt 
# 格式化输出, %-8s指输出字符串长度为8，向左对齐,%-6s指输出字符串长度为6，向右对齐
$ awk '{printf "%-8s %6s\n",$1,$4}' log.txt
```

