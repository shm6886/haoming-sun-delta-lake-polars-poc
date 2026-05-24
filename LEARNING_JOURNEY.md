# 学习历程 - 从零到精通数据湖

## 为什么是 Polars + Delta Lake + S3？

在 OpenAI、Anthropic 等公司，所有非业务核心的数据都存在 S3：
- 模型训练数据
- 用户行为日志
- 分析数据
- 外部数据源

这就是**数据湖**。

## Polars 的威力

Polars 是 Rust 写的，比 Pandas 快 10-100 倍。关键 API：

```python
# 列操作
df.with_columns(new_col=expr)
df.select(['col1', 'col2'])
df.filter(condition)

# 聚合
df.group_by('key').agg(pl.sum('value'))

# Join
df1.join(df2, on='id', how='inner')
```

## Delta Lake 的三种写模式

1. **overwrite** - 危险！删除所有旧数据
2. **append** - 安全但允许重复
3. **error** - 最安全，表存在则失败

## Merge/Upsert - 数据湖的灵魂

```python
DeltaTable(uri).merge(
    source_df,
    predicate="target.id = source.id"
).when_matched_update_all() \
 .when_not_matched_insert_all() \
 .execute()
```

这个操作：
- 更新匹配的行
- 插入新行
- 保留未匹配的行
- 原子执行（ACID）

## Bronze-Silver 架构

**Bronze** (原始层)：
- 用 append 模式，保留所有数据
- PII 暴露
- 是源数据的精确副本

**Silver** (清洁层)：
- 用 merge 模式，允许更新
- PII 掩码（SHA-256 hash）
- 去重、过滤、转换
- 生产级别的数据

## 生产管道

```
日志/API/数据库
    ↓
Bronze (append-only)
    ↓
ETL 清洁（Polars）
    ↓
Silver (merge updates)
    ↓
分析 / ML / 应用
```

## 为什么不用 Spark？

Spark 有两个问题：
1. 需要 JVM 和集群管理的开销
2. 对小数据集过于复杂

Polars + Delta Lake：
- 纯 Python，无 JVM
- 轻量级（可在 notebook 里跑）
- 生产就绪（ACID、时间旅行）
