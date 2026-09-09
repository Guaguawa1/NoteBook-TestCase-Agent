# Notebook Test Case Agent

面向 Camera、TouchPad、Active Pen 等笔记本测试模块设计的 RAG + AI Agent 项目，首期完成 Camera 端到端验证。系统支持自然语言检索测试 Case、生成带来源引用的 Case Draft、重复检测、评审、多轮修订，以及受控运行过程的 Memory 与 Trace。

> 本仓库不包含公司 Case、规范、测试记录或模型运行数据。示例数据均为 synthetic data，仅用于展示数据结构和系统流程。

## Architecture

```text
Notebook Test Case Agent

Architecture
├── Web
├── Agent Runtime
│   ├── Semantic Router
│   ├── Agent Loop
│   ├── Evaluator
│   ├── Replan
│   └── Repair
├── Tool Gateway
├── RAG
│   ├── Dense
│   ├── Sparse
│   └── Hybrid
├── Memory
└── Trace
```

## Technical Highlights

- **Controlled Agent Loop**：模型基于最新 Observation 选择白名单 Tool 与下一步 Action，确定性 Evaluator 判断完成状态，并在预算约束下执行 Replan 或 Draft Repair。
- **Hybrid RAG**：基于 multilingual-e5-large Dense Retrieval、BM25 Sparse Retrieval 与 Qdrant RRF 融合，并回读 PostgreSQL 权威父记录。
- **Traceable Draft**：Draft 使用严格 Schema 和字段级 Citation；Runtime 负责权限、来源身份、内容哈希、校验及重复门禁。
- **Memory & Observability**：支持分层会话记忆、可恢复 Summary、不可变版本、CAS 并发控制与结构化脱敏 Trace。
- **Security Boundary**：正式 Case 只读；Agent 不开放任意 Shell、SQL 或本地文件 Tool，模型输出不能直接成为系统事实。

## Evaluation

### Measured Camera Baseline

| Evaluation | Result |
|---|---:|
| Hybrid Recall@5 | 100% |
| Hybrid Recall@10 | 100% |
| Hybrid MRR@10 | 0.929 |
| Citation Recall@5 / @10 | 100% / 100% |
| Controlled DeepSeek workflows | 4 / 4 |
| Automatic duplicate decision precision | 100% |
| Automatic duplicate decision coverage | 92% |
| Agent security scenario categories | 12 passed |

Retrieval results come from 14 source-grounded queries covering 12 parent Case categories. Duplicate detection uses 25 positive and negative samples, of which 23 receive an automatic decision. The controlled DeepSeek acceptance covers search, Draft generation, revision, and review.

### Multi-module Acceptance Gates

| Evaluation | Gate |
|---|---:|
| Recall@5 | ≥ 90% |
| Agent task completion rate | ≥ 85% |
| Field citation accuracy | ≥ 95% |
| Draft Schema validity | ≥ 99% |
| Replan success rate | ≥ 80% |

## Synthetic Data

[`examples/synthetic-camera-cases.json`](examples/synthetic-camera-cases.json) contains fictional Camera Case records. IDs, steps, versions, sources, and project names are fabricated and are not derived from company data.

## Tech Stack

Vue 3 · TypeScript · Node.js · Pi Agent SDK · FastAPI · PostgreSQL · Qdrant · DeepSeek · Docker Compose
