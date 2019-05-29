---
title: ORB-SLAM2源码分析二
date: 2019-05-28 15:43:30
tags: 
- SLAM
- Tracking
categories: ORB-SLAM2
---

## **Tracking 线程分析**
```flow 
st=>start: 开始
e=>end: 结束
op=>operation: 我的操作
cond=>condition: 确认？

st->op->cond
cond(yes)->e
cond(no)->op
```
