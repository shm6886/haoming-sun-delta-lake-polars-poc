# Delta Lake & Polars POC — 学习成果展示

用 **Polars + Delta Lake + S3** 从零开始学习现代数据湖。这个项目记录了完整的学习历程——从基础配置到生产级 ETL 管道。

## 🎯 核心学习点

**为什么这个组合？**
- OpenAI、Anthropic 等公司都用 S3 存储非业务核心数据（模型训练数据、日志等）
- **Polars + Delta Lake + S3** = 强大 + 轻量 + 生产就绪

**掌握这个你就能说：我能干数据湖的活了**
- ✅ 不难 — API 简洁
- ✅ 投入小 — 无需 Spark 集群  
- ✅ 产出高 — 生产级数据基础设施

## 📚 学习内容（按优先级）

### 1️⃣ Polars ETL 核心
- 列操作：`with_columns()`, `drop()`, `cast()`, `filter()`, `select()`
- 去重和聚合：`unique()`, `group_by()`, `agg()`
- 连接：`inner join`, `left join`, `full outer join`
- 高级转换：`when/then/otherwise`, `map_elements()`

### 2️⃣ Delta Lake 三种写模式
| 模式 | 用途 | 风险 |
|------|------|------|
| `overwrite` | 完全替换表 | ⚠️ 删除所有旧数据 |
| `append` | 追加行（允许重复） | ⚠️ 可能产生重复 |
| `error` | 表存在则失败 | ✅ 最安全 |

### 3️⃣ Merge/Upsert（关键！）
```python
DeltaTable(uri).merge(
    source_df,
    predicate="target.id = source.id"
).when_matched_update_all() \
 .when_not_matched_insert_all() \
 .execute()
```
- 更新匹配的行
- 插入新行  
- 保留未匹配的行
- **这是数据湖的灵魂操作**

### 4️⃣ Bronze-Silver 分层架构
```
原始数据 → Bronze (append-only, PII exposed)
         ↓
       ETL 清洁
         ↓
      Silver (clean, merged, auditable)
         ↓
    分析 / ML / 应用
```

## 🚀 快速开始

```bash
# 1. 安装依赖
pip install -r requirements.txt

# 2. 配置 AWS
cp .env.example .env
# 编辑 .env，设置 AWS_PROFILE

# 3. 运行示例
python 01-polars-basics.py
python 07-merge-basic.py
python 10-silver-etl.py
```

## 📁 项目结构

```
.
├── README.md                      # 这个文件
├── LEARNING_JOURNEY.md            # 完整学习路径
├── CONCEPTS.md                    # 核心概念详解
├── requirements.txt               # Python 依赖
├── .env.example                   # AWS 配置模板
│
├── 01-polars-basics.py           # Polars 列操作
├── 02-polars-dedup.py            # 去重和聚合
├── 03-polars-join.py             # Join 操作
│
├── 04-delta-overwrite.py         # Delta 覆盖模式
├── 05-delta-append.py            # Delta 追加模式
├── 06-delta-error.py             # Delta 错误模式
│
├── 07-merge-basic.py             # 基础 merge
├── 08-merge-production.py        # 生产规模 merge
│
├── 09-bronze-write.py            # Bronze 原始层
└── 10-silver-etl.py              # Silver 清洁层
```

## 💡 关键收获

**技术层面：**
- ✅ Polars DataFrame 操作（快速、Rust 引擎）
- ✅ Delta Lake 特性（ACID、时间旅行、schema evolution）
- ✅ Merge/Upsert 增量更新模式
- ✅ PII 掩码和数据安全

**架构层面：**
- ✅ Bronze-Silver 分层设计
- ✅ 幂等操作（可安全重跑）
- ✅ 增量更新 vs 全量重算的取舍

**生产应用：**
- ✅ 从日志 → Bronze → Silver 的完整管道
- ✅ 处理重复、清洁、转换数据
- ✅ 版本控制和审计追踪

## 📖 学习路径

**Day 1：基础（3小时）**
1. 运行 `01-polars-basics.py` — 理解 Polars API
2. 运行 `02-polars-dedup.py` — 掌握去重和聚合
3. 运行 `03-polars-join.py` — 理解 join 操作

**Day 2：Delta Lake（3小时）**
4. 运行 `04-05-06` — 理解三种写模式的区别
5. 修改代码，尝试自己的数据

**Day 3：高级（2小时）**
6. 运行 `07-merge-basic.py` — 理解 merge 逻辑
7. 运行 `08-merge-production.py` — 看生产规模
8. 运行 `09-10` — 看完整 ETL 管道

## ⚠️ 常见坑

1. **Append 不去重** — 如果需要避免重复，用 merge 而不是 append
2. **Overwrite 很危险** — 会删除所有旧数据，生产环境要谨慎
3. **Schema 必须匹配** — Polars 对类型很严格

## 🎓 后续建议

1. **看源项目** — https://github.com/easyscale-academy/learn_delta_lake_and_polars_basic-project
   - 有 16 个完整的示例和详细文档

2. **自己做一个项目** — 用这个模式处理你自己的数据

3. **了解替代方案** — Apache Iceberg、Apache Hudi

## 📚 参考资源

- [Delta Lake 官方文档](https://docs.delta.io/)
- [Polars 官方文档](https://docs.pola.rs/)
- [AWS S3 + Polars](https://docs.pola.rs/api/python/stable/reference/io/polars.read_parquet.html)

---

**这个项目展示了：** 如何在没有 Spark 的情况下，用纯 Python 构建生产级的数据湖。

**关键洞察：** Polars + Delta Lake + S3 = 强大、轻量、可靠的现代数据基础设施。
