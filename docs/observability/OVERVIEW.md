# 可观测性

## 概述

可观测性包（`@rag-sdk/observability`）为 RAG SDK 提供横切的链路追踪、日志记录和指标收集。它设计为在**应用层可选启用**，但在**流水线层默认接入**。

## 设计

可观测性分为三个信号：

### 1. 链路追踪

跨流水线步骤的分布式追踪：

```
用户请求
  │
  ├─ pipeline.run（span）
  │  ├─ chunking（span）
  │  ├─ embedding（span）
  │  │  └─ http.request（span - 调用提供商 API）
  │  ├─ vector_search（span）
  │  │  └─ db.query（span - 调用向量数据库）
  │  └─ llm.generate（span）
  │     └─ http.request（span - 调用 LLM API）
  └─ response（span）
```

每个 span 捕获：
- 耗时
- 输入/输出大小（tokens、分块数等）
- 错误状态
- 提供商元数据

### 2. 结构化日志

每个流水线步骤都会发出结构化日志条目：

```typescript
// 日志条目的数据结构
{
  timestamp: '2026-06-01T10:00:00.000Z',
  level: 'info',
  step: 'embedding',
  pipeline: 'qa-pipeline',
  duration: 342,
  inputChunks: 15,
  outputEmbeddings: 15,
  provider: 'openai',
  model: 'text-embedding-3-small',
}
```

### 3. 指标

聚合性能指标：

| 指标 | 类型 | 描述 |
|--------|------|-------------|
| pipeline_duration_seconds | Histogram | 流水线总执行时间 |
| step_duration_seconds | Histogram | 每步执行时间 |
| chunks_processed_total | Counter | 处理的总分块数 |
| tokens_used_total | Counter | LLM 消耗的总 Token 数 |
| retrieval_latency_ms | Histogram | 向量搜索延迟 |
| error_total | Counter | 按步骤统计的错误数 |

## 集成

可观测性在 `@rag-sdk/runtime` 层面集成。每个流水线步骤自动创建 span 并发出日志。

```typescript
import { Observability } from '@rag-sdk/observability';

// 配置一次
const obs = new Observability({
  tracing: { exporter: 'console' },
  logging: { level: 'info' },
  metrics: { prefix: 'rag_sdk' },
});

// 绑定到流水线
const pipeline = new Pipeline({
  steps: [...],
  observability: obs,
});
```

特定于提供商的仪表化（调用 LLM/Embedding API 的 HTTP 请求）在 `@rag-sdk/adapters` 中处理，并通过可观测性系统连接。

## 未来增强

- OpenTelemetry 导出器集成（Jaeger、Zipkin、Datadog）
- 指标面板推荐（Prometheus + Grafana）
- 每次流水线运行的成本追踪
- 流水线性能退化告警规则
- 用于调试流水线运行的会话回放
