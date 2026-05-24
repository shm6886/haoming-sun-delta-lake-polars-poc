# 核心概念详解

## ACID 事务
Delta Lake 保证：
- **Atomicity** - 要么全成功，要么全失败
- **Consistency** - 数据总是一致的
- **Isolation** - 并发读不会互相影响
- **Durability** - 提交后永久保存

## 时间旅行（Time Travel）
读取历史版本：
```python
df = pl.read_delta(uri, version=N)
```

每个写操作创建一个新版本，可以随时回溯。用于：
- 调试 "这个数据什么时候变的"
- 恢复误操作
- A/B 测试

## Schema Evolution
添加新列到现有表：
```python
df.write_delta(uri, schema_mode="merge")
```
旧行自动在新列处填 null。

## Idempotent（幂等性）
脚本可以安全重跑，不产生副作用。

所有示例都遵循：
```python
s3dir.delete()  # 清理旧数据
# 重新生成
```

## PII 掩码
用 SHA-256 hash 保护敏感数据：
```python
df.with_columns(
    card_id_hash=pl.col("card_id").map_elements(
        lambda x: hashlib.sha256(x.encode()).hexdigest()[:16]
    )
)
```

相同的卡号产生相同的 hash，但无法反推原值。
