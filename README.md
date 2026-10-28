# llm_rag

基于 LangGraph 的多模态企业知识库 RAG 系统，分为**知识库导入**与**智能问答**两条 pipeline。

## 架构

```
导入 pipeline (port 8000)                     查询 pipeline (port 8001)
┌──────────────────────────────┐              ┌──────────────────────────────┐
│ PDF ──► MinerU转MD ──► 图片提取│              │ 问题 ──► 改写/HyDE ──► 多路召回 │
│   ──► 文档分块 ──► 项目名识别   │              │  (向量+KG+Web+Rerank+RRF融合)  │
│   ──► BGE向量化 ──► 入Milvus   │              │   ──► 答案生成 ──► SSE 流式输出 │
└──────────────────────────────┘              └──────────────────────────────┘
```

- **导入**：LangGraph 状态图编排 7 个节点（PDF→MD→图片→分块→识别→向量化→入库）
- **查询**：多路召回（向量 / HyDE / 知识图谱 Neo4j / Web 搜索 MCP）+ BGE Reranker + RRF 融合
- **服务**：FastAPI，查询侧 SSE 流式输出，MinIO 存图片，Mongo 存会话历史

## 技术栈

| 组件 | 用途 |
|---|---|
| LangGraph | 工作流编排 |
| Milvus | 向量库 |
| Neo4j | 知识图谱 |
| MinIO | 对象存储（图片） |
| MongoDB | 会话历史 |
| BGE-M3 / BGE-Reranker | 向量化 / 重排 |
| MinerU | PDF 解析 |
| DashScope (qwen) | LLM / VL 模型 |

## 快速开始

```bash
# 1. 安装依赖（uv）
uv sync

# 2. 配置环境变量
cp .env.example .env
# 编辑 .env，填入自己的 API key 与服务地址

# 3. 启动导入服务（端口 8000）
python -m app.import_process.api.file_import_service

# 4. 启动查询服务（端口 8001）
python -m app.query_process.api.query_service
```

## 目录结构

```
app/
├── clients/          # Milvus / MinIO / Mongo / Neo4j 客户端
├── conf/             # embedding / LLM / MCP 配置
├── core/             # 日志、prompt 加载
├── import_process/   # 导入 pipeline（FastAPI + LangGraph）
│   ├── api/          #   文件导入服务 :8000
│   └── agent/        #   导入工作流（7 节点）
├── query_process/    # 查询 pipeline（FastAPI + LangGraph）
│   ├── api/          #   查询服务 :8001
│   └── agent/        #   查询工作流（8 节点）
├── lm/               # 模型调用封装
├── tool/             # 工具集
└── utils/            # 通用工具
prompts/              # 提示词模板（.prompt）
test/                 # 测试脚本
```
