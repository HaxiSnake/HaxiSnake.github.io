---
title: TIDB源码学习-DDL-CreateColumn时不同状态对Insert语句的影响
date: 2020-12-01 17:53:53
tags:
- TIDB
- Golang
categories:
- 学习笔记
---

Add Column会schema会经历none, delete-only, write-only, write-reorganization, public五个状态

当运行insert语句时，只有处于public状态的节点才会执行成功，其他状态的节点都会无法找到该列。

原因如下：

insert语句经过parse, 会由ExecuetStmt()->Compile()->Optimize()->optimize()->build()中的planBuilder.buildInsert()生成查询计划。

对于INSERT ... VALUES ... 会进入buildValuesListOfInsert()中：

```go
func (b *PlanBuilder) buildInsert(ctx context.Context, insert *ast.InsertStmt) (Plan, error) {
...

if len(insert.Setlist) > 0 {
		// Branch for `INSERT ... SET ...`.
		err := b.buildSetValuesOfInsert(ctx, insert, insertPlan, mockTablePlan, checkRefColumn)
		if err != nil {
			return nil, err
		}
	} else if len(insert.Lists) > 0 {
		// Branch for `INSERT ... VALUES ...`.
		err := b.buildValuesListOfInsert(ctx, insert, insertPlan, mockTablePlan, checkRefColumn)
		if err != nil {
			return nil, err
		}
	} else {
		// Branch for `INSERT ... SELECT ...`.
		err := b.buildSelectPlanOfInsert(ctx, insert, insertPlan)
		if err != nil {
			return nil, err
		}
	}

...
}
```


在buildValuesListOfInsert()中会由PlanBuilder.getAffectCols()确定实际要插入的列。

```go
func (b *PlanBuilder) buildValuesListOfInsert(ctx context.Context, insert *ast.InsertStmt, insertPlan *Insert, mockTablePlan *LogicalTableDual, checkRefColumn func(n ast.Node) ast.Node) error {
	affectedValuesCols, err := b.getAffectCols(insert, insertPlan)

...
}
```

``` go planbuilder.go:2415:getAffectCols():
func (b *PlanBuilder) getAffectCols(insertStmt *ast.InsertStmt, insertPlan *Insert) (affectedValuesCols []*table.Column, err error) {
	if len(insertStmt.Columns) > 0 {
		// This branch is for the following scenarios:
		// 1. `INSERT INTO tbl_name (col_name [, col_name] ...) {VALUES | VALUE} (value_list) [, (value_list)] ...`,
		// 2. `INSERT INTO tbl_name (col_name [, col_name] ...) SELECT ...`.
		colName := make([]string, 0, len(insertStmt.Columns))
		for _, col := range insertStmt.Columns {
			colName = append(colName, col.Name.O)
		}
		var missingColName string
		affectedValuesCols, missingColName = table.FindCols(insertPlan.Table.VisibleCols(), colName, insertPlan.Table.Meta().PKIsHandle)
		if missingColName != "" {
			return nil, ErrUnknownColumn.GenWithStackByArgs(missingColName, clauseMsg[fieldList])
		}
	} else if len(insertStmt.Setlist) == 0 {
		// This branch is for the following scenarios:
		// 1. `INSERT INTO tbl_name {VALUES | VALUE} (value_list) [, (value_list)] ...`,
		// 2. `INSERT INTO tbl_name SELECT ...`.
		affectedValuesCols = insertPlan.Table.VisibleCols()
	}
	return affectedValuesCols, nil
}
```

在确立AffectCols时是依据table的VisibleCols来确定的：

``` go
// VisibleCols implements table.Table VisibleCols interface.
func (t *TableCommon) VisibleCols() []*table.Column {
	if len(t.VisibleColumns) > 0 {
		return t.VisibleColumns
	}
	return t.getCols(visible)
}

func (t *TableCommon) getCols(mode getColsMode) []*table.Column {
	columns := make([]*table.Column, 0, len(t.Columns))
	for _, col := range t.Columns {
		if col.State != model.StatePublic {
			continue
		}
		if (mode == visible && col.Hidden) || (mode == hidden && !col.Hidden) {
			continue
		}
		columns = append(columns, col)
	}
	return columns
}
```

根据代码中的判断条件，如果列的状态不为StatePublic，那么就不会被归结为VisibleCols(), 也就不会被getAffectCols()所感知，Inser这一列的操作自然会报错。

