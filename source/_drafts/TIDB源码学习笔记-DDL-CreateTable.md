---
title: TIDB源码学习笔记-DDL-CreateTable
tags:
- TIDB
- Golang
categories:
- 学习笔记
---

DDL的CreateTable入口在 ddl_api.go 中的CreateTable函数，其主要流程如下所示：

* check

    database是否存在？即Schema是否存在。

    如果有LIKE关键词，则检查参考的表是否存在。

* buildTableInfo:

    * with LIKE:

        检查参考表是一个物理表。

        复制参考表的列信息，只保留public状态的列信息，并调整列的offset. 这里的调整只是看到将列信息按照offset存起来后按最大offset截断。

        调整表的索引信息，只保留public状态的索引信息。

        赋值新的表名，自增ID设为0， 外键设为空。

        对TiFlashReplica和Partition信息的复制。

    delete only 时insert会怎样

    queue放到kv哪个节点


    






