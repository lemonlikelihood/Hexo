---
title: Markdown flowchart.js画图
date: 2019-05-28 20:22:19
tags:
---

## Markdown笔记：如何画流程图
> Flowchart.js 仅需几行代码即可在 Web 上完成流程图的构建。可以从文字表述中画出简单的 SVG 流程图，也可以画出彩色的图表。 

### 先来看一段入门案例
流程图代码在 Markdown 编辑中应该是下面这样的
```flow
st=>start: Start
e=>end: End
op1=>operation: My Operation
sub1=>subroutine: My Subroutine
cond=>condition: Yes or No?
io=>inputoutput: catch something...
st->op1->cond
cond(yes)->io->e
cond(no)->sub1(right)->op1
```