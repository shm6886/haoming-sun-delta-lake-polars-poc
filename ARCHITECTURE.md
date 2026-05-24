# 数据湖架构设计

## 整体架构

```
数据源 (日志/API/数据库)
    ↓
Bronze 层 (原始数据)
    ├─ append-only
    ├─ 保留 PII
    └─ 是源数据的精确副本
    ↓
ETL 清洁 (Polars 转换)
    ├─ 去重
    ├─ PII 掩码
    ├─ 过滤无效行
    └─ 添加元数据
    ↓
Silver 层 (清洁数据)
    ├─ merge 更新
    ├─ PII 已掩码
    └─ 可用于分析/ML
    ↓
应用层 (分析/ML/报表)
```

## 关键设计决策

### 为什么用 Bronze-Silver？

| 方面 | Bronze | Silver |
|------|--------|--------|
| 写模式 | append | merge |
| 数据质量 | 原始 | 清洁 |
| PII | 暴露 | 掩码 |
| 用途 | 审计 | 分析 |
| 保留期 | 长期 | 按需 |

### 为什么用 Merge？

```python
# ❌ 错误方式：append（产生重复）
df.write_delta(uri, mode="append")

# ✅ 正确方式：merge（精准更新）
DeltaTable(uri).merge(
    source_df,
    predicate="target.id = source.id"
).when_matched_update_all() \
 .when_not_matched_insert_all() \
 .execute()
```

Merge 的威力：
- 原子性（ACID）
- 避免重复
- 高效（只处理变化部分）
- 可追踪（版本历史）

## 生产最佳实践

1. **幂等性** - 脚本可重跑，不产生副作用
2. **监控** - 记录 ETL 的行数变化
3. **回退** - 用时间旅行恢复误操作
4. **分区** - 按日期分区以加快查询
5. **schema 演进** - 新字段用 merge mode 添加
