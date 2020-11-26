---
title: TIDB源码学习笔记-range的生成
date: 2020-11-18 15:10:19
tags:
- TIDB
- Golang
categories:
- 学习笔记
---

## point
``` go
// Point is the end point of range interval.
type point struct {
    value types.Datum
    excl  bool // exclude
    start bool
}
```

## ranger
``` go
// RangeType is alias for int.
type RangeType int

// RangeType constants.
const (
    IntRangeType RangeType = iota
    ColumnRangeType
    IndexRangeType
)

// Range represents a range generated in physical plan building phase.
type Range struct {
    LowVal  []types.Datum
    HighVal []types.Datum

    LowExclude  bool // Low value is exclusive.
    HighExclude bool // High value is exclusive.
}

// fullRange is (-∞, +∞).
var fullRange = []point{
    {start: true},
    {value: types.MaxValueDatum()},
}

FullIntRange unsigned [0, MaxUint64] signed [MinInt64, MaxInt64]
FullRange [,MaxValueDatum) [null, +∞)
FullNotNullRange (MinNotNullDatum, MaxValueDatum) (-∞, +∞)
NullRange [,] [null, null]
```

## 单列range生成过程

* 通过builder将expression转化为[]point.主要的转化逻辑如下表所示：

| method | return | 备注 |
| --- | --- | --- |
| buildFromColumn | [-inf, 0) (0, +inf] | 列名表达式等价于列名为真。|
| buildFromConstant | nil 或者 fullRange | 当常量Eval转化出错，或者常量为Null或者常量为0时，返回nil，否则返回fullRange |
| buildFromScalarFunc | --- |  |
| buildFromBinOp | a==NULL [,] a==1 [1, 1] <br> a!=1 [-inf, 1) (1, +inf]) <br> a>1 (1, +inf] |
| ast.LogicAnd |  | 取交集 
| ast.LogicOr |  | 取并集 取交集和取并集通过merge实现，通过flag区分。merge函数的假设：<br> * a, b 两个区间均已经做过去重<br> * 单个区间序列内部不会有重叠的部分
| buildFromIsTrue | [-inf, 0) (0, +inf] | 还有包含is not和with null的处理
| buildFromIsFalse| [0, 0] | 还有包含is not的处理
| buildFromIn | a in (1, 2, 3) <br> {[1, 1], [2, 2], [3, 3]} | 对每个点生成一个rangePoint，然后排序去重。|
| buildFromPatternLike | a LIKE 'abc%' [abc, abd) <br> a LIKE 'abc_' (abc, abd) | 找前缀然后再计算起始点和终止点。|
| isNull | [,] |
| buildFromNot | | 对于Not前缀的处理

* 通过points2TableRanges和points2Ranges将point转化为range.

    对于TableRange, 会将points中的null移除，同时将MinNotNull转化为MinInt64或者MinUint64, MaxValue转化为MaxInt64或者MaxUint64

    再将point转化为range时，会将point转化为对应的FieldType，并将转化后的数据与原数据作比较，以此来修正point的excl属性，这部分逻辑在convertPoint中。举例 “a > 1.9” 在1.9转化为整形时会变为“a>=2”.

    不同数据的比较逻辑放在datum.go中的CompareDatum中，比较特殊的几个类型的大小关系如下所示：

    * MinNotNullDatum用来表示最小值即-inf
    * MaxValueDatum用来表示最大值即+inf
    * Null < MinNotNull < 其他类型的具体值 < MaxValue

    如果column无Null值，那么会移除[null, null]这样的range.

* 得到range之后，对于Column，会尝试进行range的裁剪和range的合并

    * range裁剪的应用场景猜测：

        对于string列和bytes列，其有一定长度，比如col1 VarChar(6), 如果range中的端点长度超过这个值，即15，比如col1 > "12345678",那么这个端点就会被裁剪为 col1 >= "123456"
    
    * range 合并场景：

        [a, b] [c, d] 如果a <= c 并且b >= c，那么这两个区间就可以合并为[a, max(b, d)]