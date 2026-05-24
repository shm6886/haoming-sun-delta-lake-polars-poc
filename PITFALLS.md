# 常见坑和解决方案

## 坑 1：Append 产生重复

**问题：**
```python
df.write_delta(uri, mode="append")  # 不检查重复
```
连续 append 相同的数据会产生重复行。

**解决：**
用 merge 代替：
```python
DeltaTable(uri).merge(
    source_df,
    predicate="target.id = source.id"
).when_matched_update_all() \
 .when_not_matched_insert_all() \
 .execute()
```

## 坑 2：Overwrite 删除所有数据

**问题：**
```python
df.write_delta(uri, mode="overwrite")  # 删除全部！
```
误用 overwrite 会清空整个表。

**解决：**
- 默认用 merge
- 仅在需要完全重建时用 overwrite
- 用时间旅行恢复

## 坑 3：Schema 不匹配

**问题：**
```python
df_new.write_delta(uri)  # schema 和表不一样
```
Polars 对类型很严格。

**解决：**
```python
df_new.write_delta(uri, schema_mode="merge")
```

## 坑 4：并发写冲突

**问题：**
多个进程同时写同一张表。

**解决：**
- 用 merge 处理并发（带锁）
- 或用分区避免冲突
- 或用队列串行化

## 坑 5：忘记 PII 掩码

**问题：**
敏感数据（卡号、邮箱）暴露在 Silver 层。

**解决：**
```python
df.with_columns(
    card_id_hash=pl.col("card_id").map_elements(
        lambda x: hashlib.sha256(x.encode()).hexdigest()[:16]
    )
).drop("card_id")
```

## 坑 6：Vacuum 删除太多

**问题：**
```python
DeltaTable(uri).vacuum(retention_hours=0)  # 删除所有文件
```
可能删除并发读还在用的文件。

**解决：**
```python
DeltaTable(uri).vacuum(retention_hours=168)  # 至少 1 周
```

## 坑 7：没有幂等性

**问题：**
脚本重跑会产生不同结果（重复数据、卡住等）。

**解决：**
```python
# 开始前清理
s3dir.delete()
# 然后重新生成
# 这样即使脚本中途失败，重跑也是安全的
```
