# SDK 概述

## 设计理念

RAG SDK 遵循以下核心设计原则：

1. **类型安全优先** — 全 TypeScript 支持，严格的类型贯穿始终
2. **可组合性** — 小而专注的包，能够良好组合
3. **提供商无关** — 更换提供商无需修改应用代码
4. **默认可观测** — 所有流水线步骤内置链路追踪和指标

## 包职责

### @rag-sdk/core

基础包，提供：
- 核心类型定义（Document、Chunk、Embedding 等）
- 基础抽象和接口
- 跨包使用的值对象

*依赖：* 无
*被依赖：* 所有其他包

### @rag-sdk/adapters

提供商集成层：
- `LLMProvider` 接口 + 实现（OpenAI、Anthropic 等）
- `EmbeddingProvider` 接口 + 实现
- `VectorStore` 接口 + 实现（Pinecone、Weaviate、pgvector 等）
- 提供工厂和配置管理

*依赖：* `@rag-sdk/core`

### @rag-sdk/indexing

文档处理流水线：
- 文档加载器（PDF、HTML、Markdown、纯文本）
- 文本分割器/分块器（字符级、递归、语义）
- Embedding 编排
- 向量存储加载
- 元数据提取

*依赖：* `@rag-sdk/core`、`@rag-sdk/adapters`

### @rag-sdk/runtime

流水线编排：
- 流水线 DAG 定义
- 步骤执行引擎
- 上下文传播
- 错误处理和重试
- 中间件支持

*依赖：* `@rag-sdk/core`、`@rag-sdk/indexing`、`@rag-sdk/adapters`

### @rag-sdk/eval

RAG 评估：
- 检索指标（命中率、MRR、NDCG）
- 生成指标（忠实度、相关性、答案正确性）
- 端到端评估测试框架
- 测试数据集管理

*依赖：* `@rag-sdk/core`、`@rag-sdk/adapters`

### @rag-sdk/observability

横切可观测性：
- 分布式链路追踪（兼容 OpenTelemetry）
- 结构化日志
- 性能指标
- 流水线步骤仪表化

*依赖：* `@rag-sdk/core`

### @rag-sdk/utils

共享工具函数：
- 异步辅助（节流、批处理、重试）
- 文本处理（计数 tokens、字符串工具）
- 配置辅助
- 类型守卫和断言

*依赖：* 无

## 开发工作流

### 环境搭建

```bash
# 克隆并安装
pnpm install

# 验证工作区
pnpm list --depth -1
```

### 包开发

每个包遵循相同的结构：

```
packages/<name>/
├── src/
│   ├── index.ts        # 公共 API 导出
│   ├── types.ts        # 类型定义（如果特定于此包）
│   └── ...             # 实现模块
├── package.json
└── tsconfig.json
```

### 依赖管理策略

在 monorepo 中，依赖管理遵循 **"谁使用，谁声明"** 的原则。

#### 哪些包安装在根目录？

**开发工具类（devDependencies）** 统一安装在根目录，保证全项目使用同一个版本：

| 包 | 理由 |
|------|--------|
| `typescript` | 统一种编译版本，避免跨包类型冲突 |
| `eslint` / `prettier` | 代码风格统一 |
| `vitest` | 统一测试运行器 |
| `tsup` / `esbuild` | 统一构建工具 |

安装命令：
```bash
pnpm add typescript -w -D
```

> `-w` 代表 workspace-root，`-D` 代表 devDependencies。

#### 哪些包安装在子包中？

**运行时依赖（dependencies）** 必须安装在具体使用它的子包中，即使多个子包都用到同一个库：

| 包 | 安装位置 | 理由 |
|------|--------------|--------|
| `zod` | 使用它的每个子包 | 子包发布后，用户安装时自动下载 |
| `openai` | `@rag-sdk/adapters` | 仅在适配器中使用 |
| `lodash-es` | 使用它的每个子包 | 显式声明，不依赖隐式继承 |

安装命令：
```bash
pnpm --filter @rag-sdk/core add zod
```

#### 为什么这样做？

1. **显式依赖**：每个子包的 `package.json` 必须能独立描述其运行依赖。发布到 npm 后，用户安装 `@rag-sdk/core` 时才能自动下载 `zod`
2. **pnpm 天然去重**：即使 5 个子包都装了 `zod`，pnpm 在磁盘上也只存一份物理文件（通过硬链接），不会浪费空间
3. **避免幽灵依赖**：子包不应该能访问父包或其他子包未声明的依赖

#### 特殊场景：Workspace 内部包引用

子包之间的依赖通过 workspace 协议引入：

```bash
pnpm --filter @rag-sdk/runtime add @rag-sdk/core@workspace:*
```

这会在 `@rag-sdk/runtime/package.json` 中生成：
```json
{
  "dependencies": {
    "@rag-sdk/core": "workspace:*"
  }
}
```

### 跨包开发

在跨包开发时：

1. 在 `@rag-sdk/core` 中定义接口
2. 在对应包中实现
3. 在 `@rag-sdk/runtime` 中组装
4. 在 `@rag-sdk/eval` 中添加评估
5. 确保可观测性钩子已就位

### 添加新提供商

1. 在 `@rag-sdk/core` 中定义接口（如果尚未定义）
2. 在 `@rag-sdk/adapters` 中实现适配器
3. 添加提供商工厂注册
4. 使用模拟提供商编写测试
5. 在流水线测试中验证集成

## 编码规范

- **仅使用 ESM 模块** — 使用 `import`/`export`，不使用 `require`
- **严格 TypeScript** — tsconfig 中 `strict: true`
- **从包根目录导出** — 所有公共 API 从包根目录导出
- **测试与源码共存** — `src/__tests__/` 或 `.test.ts` 放在源码旁
- **公共 API 添加 JSDoc** — 为接口和关键函数添加文档
