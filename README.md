# AI视频生成智能体系统 (VideoGen)

> 从剧本到视频的一站式AI智能体系统，集成LLM剧本解析、好莱坞导演分镜、静态/动态提示词生成、AI图片/视频生成全流程。

---

## 项目简介

VideoGen 是一个AI视频生成智能体系统，通过以下流程将文字剧本转化为视频片段：

```
剧本输入 → 角色/场景提示词解析 → 好莱坞导演分镜表 → 静态提示词表 → 动态提示词表 → 视频片段生成
```

### 核心能力

1. **剧本解析** - 从剧本自动提取角色和场景，生成角色一致性提示词和场景环境提示词
2. **分镜生成** - 结合好莱坞导演风格与视听语言，生成分镜表（分镜号、角度、景别、运动、时长、画面内容、台词、音效）
3. **静态提示词** - 根据分镜表去除运动描述，生成优化的静态图片提示词
4. **动态提示词** - 基于静态提示词+分镜运动信息，生成视频片段提示词
5. **视频生成** - 调用ComfyUI或API生成静态图片和视频片段，支持单独/一键下载
6. **全流程编辑** - 每个环节均支持手动编辑和重新生成

### 技术特点

- **模型灵活接入** - LLM支持OpenAI API / Claude / 通义千问 / Ollama本地部署
- **生成引擎灵活** - 图片/视频支持ComfyUI本地部署或第三方API
- **角色一致性** - 通过IPAdapter+一致性标签确保角色在不同镜头中外观统一
- **异步架构** - Celery任务队列 + WebSocket实时进度推送
- **全流程可编辑** - 每个环节支持手动编辑，编辑后可触发下游重生成

---

## 技术栈

| 层级 | 技术选型 |
|------|----------|
| 前端 | React 18 + TypeScript + Vite + Ant Design + Zustand |
| 后端 | Python 3.11 + FastAPI + SQLAlchemy 2.0 + Pydantic v2 |
| AI框架 | LangGraph (Agent编排) |
| 任务队列 | Celery + RabbitMQ |
| 数据库 | PostgreSQL 16 |
| 缓存 | Redis 7 |
| 对象存储 | MinIO |
| 容器化 | Docker + Docker Compose |
| 反向代理 | Nginx |

---

## 项目结构

```
videogen/
├── backend/                 # 后端服务
│   ├── app/
│   │   ├── api/v1/          # API路由
│   │   ├── core/            # 核心配置（数据库、安全、中间件）
│   │   ├── models/          # SQLAlchemy ORM模型
│   │   ├── schemas/         # Pydantic数据模型
│   │   ├── services/        # 业务逻辑层
│   │   ├── agent/           # LangGraph Agent
│   │   │   ├── nodes/       # Agent节点
│   │   │   └── prompts/     # Prompt模板
│   │   ├── engines/         # 生成引擎
│   │   │   ├── image/       # 图片引擎（ComfyUI/API）
│   │   │   └── video/       # 视频引擎（ComfyUI/API）
│   │   ├── llm/             # LLM适配层（OpenAI/Ollama/Claude/Qwen）
│   │   ├── tasks/           # Celery异步任务
│   │   └── storage/         # MinIO存储管理
│   ├── alembic/             # 数据库迁移
│   └── requirements.txt
├── frontend/                # 前端应用
│   ├── src/
│   │   ├── pages/           # 页面组件
│   │   ├── components/      # 通用组件
│   │   ├── api/             # API请求封装
│   │   ├── stores/          # Zustand状态管理
│   │   └── types/           # TypeScript类型定义
│   └── package.json
├── docker/                  # Docker配置
│   ├── docker-compose.yml
│   └── nginx/
├── docs/                    # 项目文档
│   ├── 01-需求文档.md
│   ├── 02-系统架构技术文档.md
│   ├── 03-任务清单文档.md
│   ├── 04-API设计文档.md
│   └── 05-数据库设计文档.md
└── README.md
```

---

## 快速启动

### 环境要求

- Python 3.11+
- Node.js 20+
- Docker & Docker Compose
- PostgreSQL 16（或使用Docker）
- Redis 7（或使用Docker）
- GPU（可选，用于本地ComfyUI）

### 1. 克隆项目

```bash
git clone <repository-url>
cd videogen
```

### 2. 启动基础设施

```bash
docker compose -f docker/docker-compose.yml up -d
```

### 3. 后端启动

```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
alembic upgrade head
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### 4. 前端启动

```bash
cd frontend
npm install
npm run dev
```

### 5. 访问系统

- 前端页面：http://localhost:5173
- API文档：http://localhost:8000/docs
- MinIO控制台：http://localhost:9001

---

## 配置说明

### 环境变量

复制 `.env.example` 为 `.env` 并填写配置：

```env
# 数据库
DATABASE_URL=postgresql+asyncpg://videogen:password@localhost:5432/videogen

# Redis
REDIS_URL=redis://localhost:6379/0

# MinIO
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin

# JWT
JWT_SECRET_KEY=your-secret-key
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=1440

# LLM（可通过前端设置页配置）
LLM_PROVIDER=openai
OPENAI_API_KEY=sk-xxx
# 或
OLLAMA_BASE_URL=http://localhost:11434

# ComfyUI
COMFYUI_SERVER_URL=http://localhost:8188
```

---

## 使用流程

```
1. 输入剧本    →  在对话框中输入或粘贴剧本
2. 剧本解析    →  系统自动提取角色和场景，生成提示词
3. 编辑角色/场景 →  手动调整角色和场景提示词
4. 生成分镜    →  系统按好莱坞导演风格生成分镜表
5. 编辑分镜    →  调整分镜号、角度、景别、运动等
6. 生成静态提示词 →  系统去除运动描述，生成静态图片提示词
7. 生成静态图片 →  调用ComfyUI/API生成分镜图片
8. 生成动态提示词 →  基于静态提示词+运动信息生成动态提示词
9. 生成视频片段 →  调用ComfyUI/API生成视频
10. 下载视频    →  单独下载或一键打包下载
```

---

## 开发文档

| 文档 | 说明 |
|------|------|
| [需求文档](docs/01-需求文档.md) | 产品需求、功能模块、用户流程 |
| [系统架构技术文档](docs/02-系统架构技术文档.md) | 架构设计、技术选型、目录结构 |
| [任务清单文档](docs/03-任务清单文档.md) | WBS分解、任务详情、验收标准 |
| [API设计文档](docs/04-API设计文档.md) | REST API及WebSocket接口规范 |
| [数据库设计文档](docs/05-数据库设计文档.md) | ER图、表结构、索引、迁移 |

---

## 版本

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-07-14 | 初始文档版本 |

---

## License

MIT