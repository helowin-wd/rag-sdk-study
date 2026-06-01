# 索引流水线

## 概述

索引包（`@rag-sdk/indexing`）处理完整的文档加载流水线：加载原始文档、分割为分块、提取元数据、生成 embedding 并存储到向量索引中。

## 流水线阶段

### 1. 文档加载

```
┌──────────┐   ┌──────────┐   ┌──────────┐
│  PDF     │   │  Markdown│   │  HTML    │   ...
│  加载器  │   │  加载器   │   │  加载器  │
└────┬─────┘   └────┬─────┘   └────┬─────┘
     │              │              │
     └──────────────┴──────────────┘
                        │
                 ┌──────▼──────┐
                 │  Document   │
                 │ （标准化）   │
                 └─────────────┘
```

每个加载器实现 `DocumentLoader` 接口，返回标准化的 `Document` 对象。

### 2. 文本分割 / 分块

| 策略 | 描述 | 使用场景 |
|----------|-------------|----------|
| 固定大小 | 按字符数分割，带重叠 | 简单、均匀的分块 |
| 递归分割 | 按段落 → 句子 → 分隔符分割 | 通用场景 |
| 语义分块 | 在 embedding 相似度边界处分割 | 高质量分块 |

### 3. 元数据提取

自动提取以下内容：
- 文档标题和标题层级（从结构中提取）
- 创建/修改日期
- 来源 URL/文件路径
- 文档类型和格式
- 通过可扩展的提取器自定义元数据

### 4. Embedding 与存储

由 `@rag-sdk/indexing` 协调，由 `@rag-sdk/adapters` 执行：
1. 使用配置的 embedding 提供商为每个分块生成 embedding
2. 批量 upsert 到配置的向量存储

## 关键接口（规划中）

```typescript
interface DocumentLoader {
  load(input: LoaderInput): Promise<Document[]>;
}

interface TextSplitter {
  splitDocuments(documents: Document[]): Promise<Chunk[]>;
}

interface MetadataExtractor {
  extract(document: Document): Promise<DocumentMetadata>;
}

interface IndexingPipeline {
  run(input: IndexingInput): Promise<IndexingResult>;
}
```

## 未来增强

- 增量索引（仅更新变更的文档）
- 图片提取和 OCR
- 表格提取和结构化数据处理
- 文档去重
- 多模态文档支持
