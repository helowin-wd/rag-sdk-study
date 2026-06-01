# 评估

## 概述

评估包（`@rag-sdk/eval`）提供用于衡量 RAG 流水线质量的指标和测试框架。评估分为两个主要类别：**检索质量**和**生成质量**。

## 检索指标

衡量系统检索相关文档的效果。

| 指标 | 描述 | 范围 |
|--------|-------------|-------|
| 命中率（Hit Rate） | 检索结果中包含相关文档的查询比例 | [0, 1] |
| MRR（平均倒数排名） | 按排名加权的命中率 | [0, 1] |
| NDCG@K（归一化折损累计增益） | 位置感知的排序质量 | [0, 1] |
| Precision@K | 前 K 个结果中相关结果的比例 | [0, 1] |
| Recall@K | 检索到的相关文档比例 | [0, 1] |

## 生成指标

衡量 LLM 生成的答案质量。

| 指标 | 描述 | 方法 |
|--------|-------------|--------|
| 忠实度（Faithfulness） | 答案受到上下文支持 | LLM 作为评判 |
| 答案相关性（Answer Relevance） | 答案回应了查询 | LLM 作为评判 |
| 上下文精确度（Context Precision） | 检索到的上下文是相关的 | LLM 作为评判 |
| 上下文召回率（Context Recall） | 所有需要的上下文都被检索到 | LLM 作为评判 |
| 答案正确性（Answer Correctness） | 答案与标准答案一致 | 对比评估 |

## 评估测试框架

```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│  测试数据集  │───▶│  RAG         │───▶│  指标计算器  │
│             │    │  流水线      │    │             │
└─────────────┘    └──────────────┘    └──────┬──────┘
                                              │
                                     ┌────────▼──────┐
                                     │  报告生成器    │
                                     └───────────────┘
```

### 测试数据集格式

```typescript
interface EvalExample {
  id: string;
  query: string;
  expectedAnswer: string;
  relevantDocIds: string[];
  metadata?: Record<string, unknown>;
}

interface EvalDataset {
  name: string;
  examples: EvalExample[];
}
```

### 运行评估

```typescript
import { EvalHarness } from '@rag-sdk/eval';
import { pipeline } from './my-pipeline';

const harness = new EvalHarness({
  pipeline,
  metrics: ['hit_rate', 'mrr', 'faithfulness', 'answer_relevance'],
});

const results = await harness.run(dataset);
console.log(results.report());
```

## 未来增强

- 不同流水线配置间的 A/B 测试
- 回归检测（与基线对比）
- 自定义指标插件
- 合成测试数据集生成
- 在线评估（用户反馈集成）
