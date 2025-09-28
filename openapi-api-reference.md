# OpenAI API 参考文档 (OpenAI API Reference)

## 目录 (Table of Contents)

- [介绍 (Introduction)](#介绍-introduction)
- [认证 (Authentication)](#认证-authentication)
- [调试请求 (Debugging Requests)](#调试请求-debugging-requests)
- [向后兼容性 (Backward Compatibility)](#向后兼容性-backward-compatibility)
- [Responses API](#responses-api)
- [Conversations API](#conversations-api)
- [音频 API (Audio APIs)](#音频-api-audio-apis)
- [图像 API (Images API)](#图像-api-images-api)
- [嵌入 API (Embeddings API)](#嵌入-api-embeddings-api)
- [评估 API (Evals API)](#评估-api-evals-api)
- [微调 API (Fine-tuning API)](#微调-api-fine-tuning-api)
- [批处理 API (Batch API)](#批处理-api-batch-api)
- [文件 API (Files API)](#文件-api-files-api)
- [模型 API (Models API)](#模型-api-models-api)
- [内容审核 API (Moderations API)](#内容审核-api-moderations-api)
- [向量存储 API (Vector Stores API)](#向量存储-api-vector-stores-api)

## 介绍 (Introduction)

这个 API 参考文档描述了用于与 OpenAI 平台交互的 RESTful、流式和实时 API。REST API 可以在任何支持 HTTP 请求的环境中使用。

## 认证 (Authentication)

OpenAI API 使用 API 密钥进行认证。请在组织设置中创建、管理和了解有关 API 密钥的更多信息。

**重要提醒：** 您的 API 密钥是机密！不要与他人分享或在客户端代码中暴露它。API 密钥应安全地从环境变量或密钥管理系统中的服务器加载。

API 密钥应通过 HTTP Bearer 认证提供：

```
Authorization: Bearer OPENAI_API_KEY
```

如果您属于多个组织或通过旧版用户 API 密钥访问项目，请传递一个标头来指定要用于 API 请求的组织和项目：

```bash
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "OpenAI-Organization: org-tO6OFWPE2FfgMVvGXqYyKzsS" \
  -H "OpenAI-Project: $PROJECT_ID"
```

## 调试请求 (Debugging Requests)

除了从 API 响应返回的错误代码外，您可以检查包含特定 API 请求的唯一 ID 或应用于请求的速率限制信息的 HTTP 响应标头。

### API 元信息 (API Meta Information)
- `openai-organization`: 与请求关联的组织
- `openai-processing-ms`: 处理 API 请求所花费的时间
- `openai-version`: 用于此请求的 REST API 版本（当前为 2020-10-01）
- `x-request-id`: 此 API 请求的唯一标识符（用于故障排除）

### 速率限制信息 (Rate Limiting Information)
- `x-ratelimit-limit-requests`
- `x-ratelimit-limit-tokens`
- `x-ratelimit-remaining-requests`
- `x-ratelimit-remaining-tokens`
- `x-ratelimit-reset-requests`
- `x-ratelimit-reset-tokens`

## 向后兼容性 (Backward Compatibility)

OpenAI 致力于通过在主要 API 版本中尽可能避免破坏性变更来为 API 用户提供稳定性。这包括：

- REST API（当前 v1）
- 我们的第一方 SDK（已发布的 SDK 遵循语义版本控制）
- 模型系列（如 gpt-4o 或 o4-mini）

## Responses API

OpenAI 最先进的接口，用于生成模型响应。支持文本和图像输入，以及文本输出。通过输出作为输入使用先前响应的输出与模型创建有状态交互。使用内置工具扩展模型能力，如文件搜索、网络搜索、计算机使用等。使用函数调用允许模型访问外部系统和数据。

### 创建模型响应 (Create Model Response)

**POST** `https://api.openai.com/v1/responses`

创建模型响应。提供文本或图像输入以生成文本或 JSON 输出。让模型调用自己的自定义代码或使用内置工具如网络搜索或文件搜索来使用您自己的数据作为模型响应的输入。

#### 请求参数 (Request Parameters)

- `background` (boolean): 是否在后台运行模型响应
- `conversation` (string|object): 此响应所属的对话
- `include` (array): 指定要包含在模型响应中的附加输出数据
- `input` (string|array): 模型的文本、图像或文件输入
- `instructions` (string): 插入模型上下文的系统（或开发者）消息
- `max_output_tokens` (integer): 可以为响应生成的令牌上限
- `max_tool_calls` (integer): 可以在响应中处理的内置工具调用总数最大值
- `metadata` (map): 可以附加到对象的 16 个键值对集合
- `model` (string): 用于生成响应的模型 ID
- `parallel_tool_calls` (boolean): 是否允许模型并行运行工具调用
- `previous_response_id` (string): 先前响应的唯一 ID
- `prompt` (object): 提示模板及其变量的引用
- `reasoning` (object): 推理模型的配置选项（仅限 gpt-5 和 o 系列模型）
- `store` (boolean): 是否存储生成的模型响应
- `stream` (boolean): 是否流式传输模型响应数据
- `temperature` (number): 采样温度，介于 0 和 2 之间
- `text` (object): 模型文本响应的配置选项
- `tool_choice` (string|object): 模型应如何选择使用哪个工具
- `tools` (array): 模型在生成响应时可能调用的工具数组
- `top_p` (number): 核采样的替代方案
- `truncation` (string): 用于模型响应的截断策略

## Conversations API

创建和管理对话以在 Response API 调用中存储和检索对话状态。

### 创建对话 (Create Conversation)

**POST** `https://api.openai.com/v1/conversations`

创建对话。

#### 请求参数
- `items` (array): 要包含在对话上下文中的初始项目
- `metadata` (object): 可以附加到对象的元数据

## 音频 API (Audio APIs)

学习如何将音频转换为文本或将文本转换为音频。

### 语音创建 (Create Speech)

**POST** `https://api.openai.com/v1/audio/speech`

从输入文本生成音频。

#### 请求参数
- `input` (string): 要生成音频的文本
- `model` (string): TTS 模型之一
- `voice` (string): 生成音频时使用的声音
- `response_format` (string): 音频格式
- `speed` (number): 生成音频的速度

### 创建转录 (Create Transcription)

**POST** `https://api.openai.com/v1/audio/transcriptions`

将音频转录为输入语言。

#### 请求参数
- `file` (file): 要转录的音频文件
- `model` (string): 要使用的模型 ID
- `language` (string): 输入音频的语言
- `response_format` (string): 输出的格式
- `temperature` (number): 采样温度

## 图像 API (Images API)

给定提示和/或输入图像，模型将生成新图像。

### 创建图像 (Create Image)

**POST** `https://api.openai.com/v1/images/generations`

根据提示创建图像。

#### 请求参数
- `prompt` (string): 所需图像的文本描述
- `model` (string): 用于图像生成的模型
- `n` (integer): 要生成的图像数量
- `size` (string): 生成图像的大小
- `response_format` (string): 返回生成图像的格式

## 嵌入 API (Embeddings API)

获取给定输入的向量表示，可以轻松被机器学习模型和算法使用。

### 创建嵌入 (Create Embeddings)

**POST** `https://api.openai.com/v1/embeddings`

创建表示输入文本的嵌入向量。

#### 请求参数
- `input` (string|array): 要嵌入的输入文本
- `model` (string): 要使用的模型 ID
- `encoding_format` (string): 返回嵌入的格式
- `dimensions` (integer): 结果输出嵌入应具有的维度数

## 评估 API (Evals API)

在 OpenAI 平台中创建、管理和运行评估。

### 创建评估 (Create Eval)

**POST** `https://api.openai.com/v1/evals`

创建评估的结构，可用于测试模型性能。

#### 请求参数
- `data_source_config` (object): 用于评估运行的数据源配置
- `testing_criteria` (array): 评估组中所有评估运行的评分器列表
- `name` (string): 评估的名称
- `metadata` (map): 可以附加到对象的元数据

## 微调 API (Fine-tuning API)

管理微调作业以根据您的特定训练数据定制模型。

### 创建微调作业 (Create Fine-tuning Job)

**POST** `https://api.openai.com/v1/fine_tuning/jobs`

创建微调作业，开始从给定数据集创建新模型的过程。

#### 请求参数
- `model` (string): 要微调的模型名称
- `training_file` (string): 包含训练数据的已上传文件的 ID
- `hyperparameters` (object): 用于微调作业的超参数
- `suffix` (string): 将添加到微调模型名称的字符串

## 批处理 API (Batch API)

为异步处理创建大量 API 请求的批处理。Batch API 在 24 小时内返回完成，折扣 50%。

### 创建批处理 (Create Batch)

**POST** `https://api.openai.com/v1/batches`

从已上传的请求文件中创建并执行批处理。

#### 请求参数
- `input_file_id` (string): 包含新批处理请求的已上传文件的 ID
- `endpoint` (string): 批处理中所有请求使用的端点
- `completion_window` (string): 批处理应在其中处理的时间框架

## 文件 API (Files API)

文件用于上传可与 Assistants、Fine-tuning 和 Batch API 等功能一起使用的文档。

### 上传文件 (Upload File)

**POST** `https://api.openai.com/v1/files`

上传可用于各个端点的文件。

#### 请求参数
- `file` (file): 要上传的文件对象
- `purpose` (string): 上传文件的预期用途

## 模型 API (Models API)

列出并描述 API 中可用的各种模型。

### 列出模型 (List Models)

**GET** `https://api.openai.com/v1/models`

列出当前可用的模型，并提供每个模型的基本信息。

## 内容审核 API (Moderations API)

根据文本和/或图像输入，分类这些输入是否可能有害。

### 创建内容审核 (Create Moderation)

**POST** `https://api.openai.com/v1/moderations`

分类文本和/或图像输入是否可能有害。

#### 请求参数
- `input` (string|array): 要分类的输入
- `model` (string): 要使用的内容审核模型

## 向量存储 API (Vector Stores API)

向量存储为 Retrieval API 和 Responses 及 Assistants API 中的 file_search 工具提供语义搜索功能。

### 创建向量存储 (Create Vector Store)

**POST** `https://api.openai.com/v1/vector_stores`

创建向量存储。

#### 请求参数
- `file_ids` (array): 向量存储应使用的文件 ID 列表
- `name` (string): 向量存储的名称
- `metadata` (map): 可以附加到对象的元数据

---

*本文档由 OpenAI API 参考内容生成，提供了全面的 API 使用指南。*