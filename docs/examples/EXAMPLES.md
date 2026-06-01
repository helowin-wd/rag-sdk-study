# 使用示例

本文档包含 RAG SDK 的使用示例。这些示例展示了 SDK 的设计意图。具体实现将随着 SDK 的构建而更新。

## 基础 RAG 流水线

```typescript
import { Pipeline } from '@rag-sdk/runtime';
import { OpenAIAdapter, OpenAIEmbedding } from '@rag-sdk/adapters';

const pipeline = new Pipeline({
  name: 'basic-qa',
  steps: [
    {
      name: 'retrieve',
      adapter: new OpenAIEmbedding({ model: 'text-embedding-3-small' }),
      vectorStore: { type: 'pinecone', index: 'my-docs' },
      config: { topK: 5 },
    },
    {
      name: 'generate',
      adapter: new OpenAIAdapter({ model: 'gpt-4o' }),
      config: { temperature: 0.1 },
    },
  ],
});

const answer = await pipeline.run({
  query: '什么是 RAG？',
});

console.log(answer.text);
```

## 自定义文档索引

```typescript
import { DirectoryLoader } from '@rag-sdk/indexing';
import { RecursiveSplitter } from '@rag-sdk/indexing';

const loader = new DirectoryLoader('./documents');
const splitter = new RecursiveSplitter({
  chunkSize: 1000,
  chunkOverlap: 200,
});

const docs = await loader.load();
const chunks = await splitter.splitDocuments(docs);

// 每个分块可以通过适配器进行 embedding 和存储
```

## 评估

```typescript
import { EvalHarness } from '@rag-sdk/eval';
import { pipeline } from './pipeline';

const evalData = [
  {
    query: '什么是 RAG？',
    expectedAnswer: '检索增强生成（RAG）是一种...',
    relevantDocIds: ['doc-1', 'doc-2'],
  },
];

const harness = new EvalHarness({
  pipeline,
  metrics: ['hit_rate', 'faithfulness'],
});

const results = await harness.run(evalData);
console.log(results.summary());
```

## 自定义提供商适配器

```typescript
import { LLMProvider, Message, Completion } from '@rag-sdk/core';

class MyCustomProvider implements LLMProvider {
  name = 'my-custom-llm';

  async complete(messages: Message[]): Promise<Completion> {
    // 实现特定于提供商的逻辑
    return {
      content: '...',
      tokens: { input: 0, output: 0 },
      model: this.name,
    };
  }
}
```
