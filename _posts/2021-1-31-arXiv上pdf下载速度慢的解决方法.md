---
title: arXiv上pdf下载速度慢的解决方法
tags: Tutorial
mathjax: true
mathjax_autoNumber: true
comment: true
key: arXiv上pdf下载速度慢的解决方法
---

1. 命令行直接下载：
 arxiv 上的论文使用wget下载时需要加参数--user-agent=Lynx，速度才能较快：
```wget --user-agent=Lynx https://arxiv.org/pdf/1608.00375```
  上述命令需要在Linux或者WSL的命令行中执行。

2. 修改网址：（不稳定）
将https://arxiv.org改成 http://xxx.itp.ac.cn，网址其他部分不变。
或者将https://arxiv.org改成http://cn.arxiv.org，网址其他部分不变。