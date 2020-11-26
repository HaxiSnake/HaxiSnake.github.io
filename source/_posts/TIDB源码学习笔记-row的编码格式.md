---
title: TIDB源码学习笔记-row的编码格式
date: 2020-11-12 15:12:50
tags:
- TIDB
- Golang
categories:
- 学习笔记
---

# 方案一 实现在util/rowcodec 

会记录当前行的空置和非空值数量

如果总列数大于255，会用uint32存储column id,否则使用byte存储column id.

会将同一行非空列排布在前面，空值列排布在后面，分别按照column id进行排序。 每一个非空列column id对应一个value.
空值列的column id对应的value位置没有value重新赋值的操作。

会有一个notNullIdx来区分非空列和空列的界限。

最后转化为bytes的结构

* CodecVer: 1 byte
* flag: 1 byte : large 1 small 0 
* numNotNullCols: 2 bytes
* numNullCols: 2 bytes
* colIDs32: uint32[] 转 byte[], 如果是small则为colIDs byte[]
* offsets32:  uint32[] 转 byte[],  如果是small则为sffsets u16[] 转 byte[]
* data: byte[]
# 方案二 

这套方案比较简单，申请row长度2倍的datum数组，前面用来存储数值，后面用来存储column id 一一对应。