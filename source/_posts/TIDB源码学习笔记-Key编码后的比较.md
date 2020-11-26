---
title: TIDB源码学习笔记-Key编码后的比较
date: 2020-11-26 15:05:34
tags:
- TIDB
- Golang
categories:
- 学习笔记
---

key经过编码之后是字节数组，两个字节数组的比较方案如下所示：

判断两个字节数组的地址是否一样，如果一样则说明在同一段内存存储，直接判断长度，长度小的key编码较小。

如果首地址不一样，则按照两个数组的较小长度逐个元素比较，如有不同直接按照元素大小返回。

当在较小长度内所有元素都相同，则较大长度字节数组大于较小长度数组。如果两者长度相同且所有元素相同，则数组大小相同。

## example

例如有如下两个表：

table1, tableId为1, 第三列value含有索引，indexID为1
```go
rowid, name,        role, value
1,   "TiDB", "SQL Layer", 10
2,   "TiKV", "KV Engine", 20
3,     "PD",   "Manager", 30
```

table2, tableId为2, 第二列到第四列都含有索引，indexID从1到4
``` go
rowid,   name, school, age, graduate date, personID(unique)
1, "XiaoMing", "A",     19,    2020-06-20, 210283
2, "XiaoHong", "C",     19,    2019-03-20, 270447
3, "XiaoWang", "B",     18,    2018-07-01, 159668
```

则其在Tikv中排布情况如下所示：
``` go
t1_i1_intFlag{10}_1 --> null
t1_i1_intFlag{20}_2 --> null
t1_i1_intFlag{30}_3 --> null
t1_r1 --> ["TiDB", "SQL Layer", 10]
t1_r2 --> ["TiKV", "KV Engine", 20] 
t1_r3 --> ["PD", "Manager", 30]
t2_i1_bytesFlag{A}_1 --> null
t2_i1_bytesFlag{B}_3 --> null
t2_i1_bytesFlag{C}_2 --> null
t2_i2_uintFlag{18}_3 --> null
t2_i2_uintFlag{19}_1 --> null
t2_i2_uintFlag{19}_2 --> null
t2_i3_uintFlag{2018-07-01}_3 --> null
t2_i3_uintFlag{2019-03-20}_2 --> null
t2_i3_uintFlag{2020-06-20}_1 --> null
t2_i4_uintFlag{159668} --> 3
t2_i4_uintFlag{210283} --> 1
t2_i4_uintFlag{270447} --> 2
t2_r1 --> ["XiaoMing", "A", 19, 2020-06-20]
t2_r2 --> ["XiaoHong", "B", 19, 2019-03-20]
t2_r3 --> ["XiaoWang", "C", 18, 2018-07-01]
```
tikv是一个全局有序的分布式Key-Value引擎，因此，通过以上的设计可以让同一个表下同一个indexID的索引按照ColumnsValue分布到一段连续的区间上.这个columnsValue的编码方式主要在codec模块中。

columnsValue在进行实际的编码时，会在编码开始前添加一个codeFlag，如下所示：

``` go
const (
    NilFlag          byte = 0
    bytesFlag        byte = 1
    compactBytesFlag byte = 2
    intFlag          byte = 3
    uintFlag         byte = 4
    floatFlag        byte = 5
    decimalFlag      byte = 6
    durationFlag     byte = 7
    varintFlag       byte = 8
    uvarintFlag      byte = 9
    jsonFlag         byte = 10
    maxFlag          byte = 250
)
```

按照本人目前的理解，主要有两个作用：
    
* 一是用来识别不同数据类型的编码方案。比如int64 compareble就会用intFlag标记，string compareable就会用bytesFlag来标记。

* 二是方便统一不同类型对于边界值处理。在tidb中任何类型都有三个较为特殊的值，Null, MinNotNull, MaxValue.它主要用来表示一些筛选条件的range边界。这三个值的codeFlag是：

``` go
Null       --> NilFlag byte = 0
MinNotNull --> bytesFlag byte = 1
MaxValue   --> maxFlag byte = 250
```

比如 `select * from table1 where value < 15` 这样一个查询， value的range就是[MinNotNull, 15). 对于这三个特殊值，并不依据具体的类型做编码，而是通过codeFlag这个字节来标识。结合key的编码比较方案，就可以实现对range边界的确定。

例如

``` sql
select from * from table2 where school <= "B" AND school is Not Null
```

这样一个查询, 生成的range就是:
`[MaxNotNull, "B"]`
对应到tikv中的key编码就是:
`[t2_i1_bytesFlag, t2_i1_bytesFlag{B}]`
而包含Null的查询:
``` sql 
select from * from table2 where school <= "B"
```
这样一个查询, 生成的range就是:
`[Null, "B"]`
对应到tikv中的key编码就是:
`[t2_i1_NilFlag, t2_i1_bytesFlag{B}]`