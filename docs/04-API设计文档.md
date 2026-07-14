# AI视频生成智能体系统 - API设计文档

> 版本：v1.0  
> 日期：2026-07-14  
> 状态：草案

---

## 1. 文档概述

本文档定义系统所有REST API及WebSocket接口规范，包含请求/响应格式、认证方式、错误码及示例。所有API基于 `/api/v1` 前缀。

---

## 2. 通用约定

### 2.1 基础信息

| 项目 | 说明 |
|------|------|
| Base URL | `http://localhost:8000/api/v1` |
| 认证方式 | Bearer Token (JWT) |
| 请求格式 | `application/json` |
| 响应格式 | `application/json` |
| 字符编码 | UTF-8 |

### 2.2 统一响应格式

```json
{
  "code": 0,
  "message": "success",
  "data": { ... },
  "timestamp": "2026-07-14T09:00:00Z"
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| code | int | 0=成功，非0=错误码 |
| message | string | 描述信息 |
| data | object/array/null | 业务数据 |
| timestamp | string | ISO 8601时间戳 |

### 2.3 分页响应格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "items": [...],
    "total": 100,
    "page": 1,
    "page_size": 20
  }
}
```

### 2.4 认证

除 `/auth/login` 和 `/auth/register` 外，所有接口需在Header携带：

```
Authorization: Bearer <jwt_token>
```

### 2.5 错误码

| 错误码 | HTTP状态码 | 说明 |
|--------|------------|------|
| 0 | 200 | 成功 |
| 1001 | 400 | 请求参数错误 |
| 1002 | 401 | 未认证 |
| 1003 | 403 | 无权限 |
| 1004 | 404 | 资源不存在 |
| 1005 | 409 | 资源冲突 |
| 2001 | 422 | LLM调用失败 |
| 2002 | 422 | LLM输出解析失败 |
| 3001 | 422 | 图片生成失败 |
| 3002 | 422 | 视频生成失败 |
| 3003 | 503 | 引擎服务不可用 |
| 4001 | 409 | 任务正在执行中 |
| 4002 | 410 | 任务已过期 |
| 5000 | 500 | 服务器内部错误 |

### 2.6 通用查询参数

| 参数 | 类型 | 说明 |
|------|------|------|
| page | int | 页码，默认1 |
| page_size | int | 每页条数，默认20 |
| sort | string | 排序字段，如 `-created_at` |
| keyword | string | 关键词搜索 |

---

## 3. 认证接口

### 3.1 用户注册

```
POST /api/v1/auth/register
```

**请求体：**
```json
{
  "username": "admin",
  "email": "admin@example.com",
  "password": "password123"
}
```

**响应：**
```json
{
  "code": 0,
  "message": "注册成功",
  "data": {
    "id": "u_001",
    "username": "admin",
    "email": "admin@example.com"
  }
}
```

### 3.2 用户登录

```
POST /api/v1/auth/login
```

**请求体：**
```json
{
  "username": "admin",
  "password": "password123"
}
```

**响应：**
```json
{
  "code": 0,
  "message": "登录成功",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_in": 86400,
    "user": {
      "id": "u_001",
      "username": "admin"
    }
  }
}
```

---

## 4. 剧本接口

### 4.1 创建剧本

```
POST /api/v1/scripts
```

**请求体：**
```json
{
  "title": "夜归人",
  "content": "夜幕降临，城市霓虹闪烁...",
  "genre": "悬疑",
  "description": "一个关于深夜归家的悬疑短片"
}
```

**响应：**
```json
{
  "code": 0,
  "message": "剧本创建成功",
  "data": {
    "id": "sc_001",
    "title": "夜归人",
    "content": "夜幕降临...",
    "genre": "悬疑",
    "status": "draft",
    "created_at": "2026-07-14T09:00:00Z",
    "updated_at": "2026-07-14T09:00:00Z"
  }
}
```

### 4.2 获取剧本列表

```
GET /api/v1/scripts?page=1&page_size=20
```

### 4.3 获取剧本详情

```
GET /api/v1/scripts/{script_id}
```

### 4.4 更新剧本

```
PUT /api/v1/scripts/{script_id}
```

### 4.5 删除剧本

```
DELETE /api/v1/scripts/{script_id}
```

### 4.6 解析剧本（生成角色和场景提示词）

```
POST /api/v1/scripts/{script_id}/parse
```

**请求体：**
```json
{
  "async": true,
  "options": {
    "extract_characters": true,
    "extract_scenes": true,
    "generate_prompts": true
  }
}
```

**响应（异步）：**
```json
{
  "code": 0,
  "message": "解析任务已提交",
  "data": {
    "task_id": "task_001",
    "status": "pending",
    "websocket_url": "/api/v1/ws/tasks/task_001"
  }
}
```

**响应（同步）：**
```json
{
  "code": 0,
  "message": "解析完成",
  "data": {
    "characters": [...],
    "scenes": [...],
    "plot_outline": "..."
  }
}
```

---

## 5. 角色提示词接口

### 5.1 获取角色列表

```
GET /api/v1/scripts/{script_id}/characters
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "id": "ch_001",
        "script_id": "sc_001",
        "name": "林晓",
        "role": "主角",
        "appearance": "短发，瘦削，眼神坚定",
        "clothing": "深蓝色风衣，黑色长裤",
        "personality": "冷静、果断",
        "consistency_tag": "lin_xiao_female_28_short_hair",
        "reference_prompt": "a 28-year-old Asian woman with short black hair, wearing a dark blue trench coat, slim build, determined eyes, cinematic portrait, 8k",
        "reference_image_url": null,
        "sort_order": 1,
        "created_at": "2026-07-14T09:00:00Z"
      }
    ],
    "total": 1
  }
}
```

### 5.2 更新角色提示词

```
PUT /api/v1/characters/{character_id}
```

**请求体：**
```json
{
  "name": "林晓",
  "appearance": "短发，瘦削，眼神坚定",
  "clothing": "深蓝色风衣，黑色长裤",
  "personality": "冷静、果断",
  "reference_prompt": "updated prompt text..."
}
```

### 5.3 生成角色参考图

```
POST /api/v1/characters/{character_id}/generate-reference
```

**请求体：**
```json
{
  "async": true,
  "width": 768,
  "height": 1024
}
```

### 5.4 重新生成角色参考图

```
POST /api/v1/characters/{character_id}/regenerate-reference
```

---

## 6. 场景提示词接口

### 6.1 获取场景列表

```
GET /api/v1/scripts/{script_id}/scenes
```

**响应data.items结构：**
```json
{
  "id": "se_001",
  "script_id": "sc_001",
  "name": "城市街道",
  "location": "都市夜景街道",
  "time_of_day": "夜晚",
  "weather": "阴天",
  "environment": "霓虹灯闪烁的都市街道，地面湿润反光",
  "lighting": "冷色调霓虹光，低角度路灯",
  "atmosphere": "紧张、神秘",
  "scene_prompt": "a neon-lit city street at night, wet asphalt with reflections, cold blue tones, cinematic atmosphere, moody lighting",
  "sort_order": 1
}
```

### 6.2 更新场景提示词

```
PUT /api/v1/scenes/{scene_id}
```

---

## 7. 分镜接口

### 7.1 生成分镜表

```
POST /api/v1/scripts/{script_id}/storyboards
```

**请求体：**
```json
{
  "async": true,
  "director_style": "hollywood",
  "shots_per_minute": 10
}
```

### 7.2 获取分镜列表

```
GET /api/v1/scripts/{script_id}/storyboards
```

**响应data.items结构：**
```json
{
  "id": "sb_001",
  "script_id": "sc_001",
  "shot_number": 1,
  "angle": "俯拍",
  "shot_size": "远景",
  "movement": "缓慢推进",
  "duration": 5.0,
  "visual_content": "城市夜景全景，霓虹灯闪烁，街道上空无一人",
  "dialogue": "",
  "sound_effect": "远处传来雷声，低沉的环境音",
  "scene_id": "se_001",
  "characters": ["ch_001"],
  "sort_order": 1
}
```

### 7.3 更新分镜

```
PUT /api/v1/storyboards/{storyboard_id}
```

**请求体（支持单字段更新）：**
```json
{
  "angle": "仰拍",
  "shot_size": "中景",
  "duration": 3.5,
  "visual_content": "更新后的画面内容..."
}
```

### 7.4 新增分镜

```
POST /api/v1/scripts/{script_id}/storyboards
```

**请求体：**
```json
{
  "shot_number": 2,
  "angle": "平视",
  "shot_size": "近景",
  "movement": "固定",
  "duration": 4.0,
  "visual_content": "...",
  "dialogue": "...",
  "sound_effect": "..."
}
```

### 7.5 删除分镜

```
DELETE /api/v1/storyboards/{storyboard_id}
```

### 7.6 调整分镜顺序

```
PUT /api/v1/scripts/{script_id}/storyboards/reorder
```

**请求体：**
```json
{
  "ordered_ids": ["sb_003", "sb_001", "sb_002"]
}
```

### 7.7 重新生成分镜

```
POST /api/v1/scripts/{script_id}/storyboards/regenerate
```

---

## 8. 静态提示词接口

### 8.1 生成静态提示词表

```
POST /api/v1/scripts/{script_id}/static-prompts
```

**请求体：**
```json
{
  "async": true
}
```

### 8.2 获取静态提示词列表

```
GET /api/v1/scripts/{script_id}/static-prompts
```

**响应data.items结构：**
```json
{
  "id": "sp_001",
  "script_id": "sc_001",
  "storyboard_id": "sb_001",
  "shot_number": 1,
  "positive_prompt": "a neon-lit city street at night, wet asphalt with reflections, cold blue tones, cinematic establishing shot, wide angle, 8k, highly detailed",
  "negative_prompt": "blurry, low quality, distorted, watermark, text",
  "character_refs": ["ch_001"],
  "scene_ref": "se_001",
  "width": 1024,
  "height": 576,
  "seed": -1,
  "generated_image_url": null,
  "status": "pending"
}
```

### 8.3 更新静态提示词

```
PUT /api/v1/static-prompts/{prompt_id}
```

**请求体：**
```json
{
  "positive_prompt": "updated prompt...",
  "negative_prompt": "updated negative...",
  "width": 1280,
  "height": 720
}
```

### 8.4 生成静态图片

```
POST /api/v1/static-prompts/{prompt_id}/generate-image
```

**请求体：**
```json
{
  "async": true,
  "seed": 42
}
```

### 8.5 批量生成静态图片

```
POST /api/v1/scripts/{script_id}/static-prompts/batch-generate
```

**请求体：**
```json
{
  "prompt_ids": ["sp_001", "sp_002"],
  "async": true
}
```

### 8.6 重新生成静态提示词

```
POST /api/v1/scripts/{script_id}/static-prompts/regenerate
```

---

## 9. 动态提示词接口

### 9.1 生成动态提示词表

```
POST /api/v1/scripts/{script_id}/dynamic-prompts
```

### 9.2 获取动态提示词列表

```
GET /api/v1/scripts/{script_id}/dynamic-prompts
```

**响应data.items结构：**
```json
{
  "id": "dp_001",
  "script_id": "sc_001",
  "storyboard_id": "sb_001",
  "static_prompt_id": "sp_001",
  "shot_number": 1,
  "positive_prompt": "camera slowly pushes forward through the neon-lit street, rain falling, reflections shimmering on wet asphalt, cinematic motion",
  "negative_prompt": "static, jittery, low fps, distortion",
  "motion_config": {
    "type": "zoom_in",
    "speed": "slow",
    "direction": "forward"
  },
  "duration": 5.0,
  "fps": 24,
  "static_image_url": "http://minio:9000/videogen/images/sp_001.png",
  "generated_video_url": null,
  "status": "pending"
}
```

### 9.3 更新动态提示词

```
PUT /api/v1/dynamic-prompts/{prompt_id}
```

### 9.4 生成视频片段

```
POST /api/v1/dynamic-prompts/{prompt_id}/generate-video
```

**请求体：**
```json
{
  "async": true,
  "use_static_image": true
}
```

### 9.5 批量生成视频片段

```
POST /api/v1/scripts/{script_id}/dynamic-prompts/batch-generate
```

### 9.6 重新生成动态提示词

```
POST /api/v1/scripts/{script_id}/dynamic-prompts/regenerate
```

---

## 10. 视频片段接口

### 10.1 获取视频片段列表

```
GET /api/v1/scripts/{script_id}/videos
```

**响应data.items结构：**
```json
{
  "id": "vc_001",
  "script_id": "sc_001",
  "dynamic_prompt_id": "dp_001",
  "shot_number": 1,
  "video_url": "http://minio:9000/videogen/videos/vc_001.mp4",
  "thumbnail_url": "http://minio:9000/videogen/thumbnails/vc_001.jpg",
  "duration": 5.0,
  "resolution": "1024x576",
  "file_size": 12582912,
  "status": "completed",
  "created_at": "2026-07-14T09:30:00Z"
}
```

### 10.2 获取视频片段详情

```
GET /api/v1/videos/{video_id}
```

### 10.3 单独下载视频

```
GET /api/v1/videos/{video_id}/download
```

**响应：** `application/octet-stream` 文件流

### 10.4 一键下载所有视频（打包ZIP）

```
GET /api/v1/scripts/{script_id}/videos/download-all
```

**查询参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| format | string | `zip` (默认) / `tar` |
| include_images | bool | 是否包含静态图片，默认false |

**响应：** `application/zip` 文件流

### 10.5 合并视频

```
POST /api/v1/scripts/{script_id}/videos/merge
```

**请求体：**
```json
{
  "video_ids": ["vc_001", "vc_002", "vc_003"],
  "output_format": "mp4",
  "add_transitions": true
}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "task_id": "task_merge_001",
    "status": "processing"
  }
}
```

### 10.6 删除视频片段

```
DELETE /api/v1/videos/{video_id}
```

### 10.7 重新生成视频片段

```
POST /api/v1/videos/{video_id}/regenerate
```

---

## 11. 任务接口

### 11.1 获取任务状态

```
GET /api/v1/tasks/{task_id}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "id": "task_001",
    "type": "parse_script",
    "status": "running",
    "progress": 45,
    "current_step": "生成角色提示词",
    "result": null,
    "error": null,
    "created_at": "2026-07-14T09:00:00Z",
    "updated_at": "2026-07-14T09:01:30Z"
  }
}
```

**status枚举值：** `pending` / `running` / `completed` / `failed` / `cancelled`

### 11.2 获取任务列表

```
GET /api/v1/tasks?page=1&page_size=20&status=running
```

### 11.3 取消任务

```
POST /api/v1/tasks/{task_id}/cancel
```

### 11.4 重试任务

```
POST /api/v1/tasks/{task_id}/retry
```

### 11.5 获取任务日志

```
GET /api/v1/tasks/{task_id}/logs?page=1&page_size=50
```

---

## 12. 系统配置接口

### 12.1 获取LLM配置

```
GET /api/v1/config/llm
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "provider": "openai",
    "model": "gpt-4o",
    "api_key": "sk-***masked***",
    "base_url": null,
    "temperature": 0.7,
    "max_tokens": 4096,
    "available_providers": ["openai", "claude", "qwen", "ollama"]
  }
}
```

### 12.2 更新LLM配置

```
PUT /api/v1/config/llm
```

**请求体：**
```json
{
  "provider": "ollama",
  "model": "llama3.1:8b",
  "base_url": "http://localhost:11434",
  "temperature": 0.7,
  "max_tokens": 4096
}
```

### 12.3 测试LLM连接

```
POST /api/v1/config/llm/test
```

**请求体：**
```json
{
  "provider": "ollama",
  "model": "llama3.1:8b",
  "base_url": "http://localhost:11434"
}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "success": true,
    "latency_ms": 350,
    "test_response": "Hello! How can I help you?"
  }
}
```

### 12.4 获取图片引擎配置

```
GET /api/v1/config/image-engine
```

### 12.5 更新图片引擎配置

```
PUT /api/v1/config/image-engine
```

**请求体：**
```json
{
  "type": "comfyui",
  "server_url": "http://localhost:8188",
  "default_width": 1024,
  "default_height": 576
}
```

### 12.6 测试图片引擎连接

```
POST /api/v1/config/image-engine/test
```

### 12.7 获取视频引擎配置

```
GET /api/v1/config/video-engine
```

### 12.8 更新视频引擎配置

```
PUT /api/v1/config/video-engine
```

### 12.9 测试视频引擎连接

```
POST /api/v1/config/video-engine/test
```

---

## 13. WebSocket接口

### 13.1 任务进度推送

```
WS /api/v1/ws/tasks/{task_id}
```

**连接方式：**
```javascript
const ws = new WebSocket('ws://localhost:8000/api/v1/ws/tasks/task_001');
```

**服务端推送消息格式：**

```json
{
  "type": "progress",
  "task_id": "task_001",
  "progress": 45,
  "current_step": "生成角色提示词",
  "message": "正在解析第2个角色...",
  "timestamp": "2026-07-14T09:01:30Z"
}
```

**消息类型（type字段）：**

| type | 说明 | 触发时机 |
|------|------|----------|
| `progress` | 进度更新 | 任务执行中 |
| `step_change` | 步骤切换 | 进入新阶段 |
| `completed` | 任务完成 | 任务成功完成 |
| `failed` | 任务失败 | 任务执行出错 |
| `log` | 日志消息 | 调试日志 |

**`completed`消息示例：**
```json
{
  "type": "completed",
  "task_id": "task_001",
  "result": {
    "characters_count": 3,
    "scenes_count": 5
  },
  "timestamp": "2026-07-14T09:02:00Z"
}
```

**`failed`消息示例：**
```json
{
  "type": "failed",
  "task_id": "task_001",
  "error": "LLM调用超时",
  "error_code": 2001,
  "timestamp": "2026-07-14T09:02:00Z"
}
```

### 13.2 剧本全局进度推送

```
WS /api/v1/ws/scripts/{script_id}
```

监听某个剧本下所有任务的进度，消息格式同上，额外包含 `task_type` 字段。

---

## 14. 接口汇总表

| 模块 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 认证 | POST | `/auth/register` | 用户注册 |
| 认证 | POST | `/auth/login` | 用户登录 |
| 剧本 | POST | `/scripts` | 创建剧本 |
| 剧本 | GET | `/scripts` | 剧本列表 |
| 剧本 | GET | `/scripts/{id}` | 剧本详情 |
| 剧本 | PUT | `/scripts/{id}` | 更新剧本 |
| 剧本 | DELETE | `/scripts/{id}` | 删除剧本 |
| 剧本 | POST | `/scripts/{id}/parse` | 解析剧本 |
| 角色 | GET | `/scripts/{id}/characters` | 角色列表 |
| 角色 | PUT | `/characters/{id}` | 更新角色 |
| 角色 | POST | `/characters/{id}/generate-reference` | 生成参考图 |
| 场景 | GET | `/scripts/{id}/scenes` | 场景列表 |
| 场景 | PUT | `/scenes/{id}` | 更新场景 |
| 分镜 | POST | `/scripts/{id}/storyboards` | 生成分镜表 |
| 分镜 | GET | `/scripts/{id}/storyboards` | 分镜列表 |
| 分镜 | PUT | `/storyboards/{id}` | 更新分镜 |
| 分镜 | DELETE | `/storyboards/{id}` | 删除分镜 |
| 分镜 | PUT | `/scripts/{id}/storyboards/reorder` | 调整顺序 |
| 分镜 | POST | `/scripts/{id}/storyboards/regenerate` | 重新生成 |
| 静态 | POST | `/scripts/{id}/static-prompts` | 生成静态提示词 |
| 静态 | GET | `/scripts/{id}/static-prompts` | 静态提示词列表 |
| 静态 | PUT | `/static-prompts/{id}` | 更新静态提示词 |
| 静态 | POST | `/static-prompts/{id}/generate-image` | 生成图片 |
| 静态 | POST | `/scripts/{id}/static-prompts/batch-generate` | 批量生成图片 |
| 动态 | POST | `/scripts/{id}/dynamic-prompts` | 生成动态提示词 |
| 动态 | GET | `/scripts/{id}/dynamic-prompts` | 动态提示词列表 |
| 动态 | PUT | `/dynamic-prompts/{id}` | 更新动态提示词 |
| 动态 | POST | `/dynamic-prompts/{id}/generate-video` | 生成视频 |
| 动态 | POST | `/scripts/{id}/dynamic-prompts/batch-generate` | 批量生成视频 |
| 视频 | GET | `/scripts/{id}/videos` | 视频列表 |
| 视频 | GET | `/videos/{id}` | 视频详情 |
| 视频 | GET | `/videos/{id}/download` | 下载单个视频 |
| 视频 | GET | `/scripts/{id}/videos/download-all` | 一键下载所有视频 |
| 视频 | POST | `/scripts/{id}/videos/merge` | 合并视频 |
| 任务 | GET | `/tasks/{id}` | 任务状态 |
| 任务 | GET | `/tasks` | 任务列表 |
| 任务 | POST | `/tasks/{id}/cancel` | 取消任务 |
| 任务 | POST | `/tasks/{id}/retry` | 重试任务 |
| 配置 | GET | `/config/llm` | 获取LLM配置 |
| 配置 | PUT | `/config/llm` | 更新LLM配置 |
| 配置 | POST | `/config/llm/test` | 测试LLM连接 |
| 配置 | GET | `/config/image-engine` | 获取图片引擎配置 |
| 配置 | PUT | `/config/image-engine` | 更新图片引擎配置 |
| 配置 | POST | `/config/image-engine/test` | 测试图片引擎 |
| 配置 | GET | `/config/video-engine` | 获取视频引擎配置 |
| 配置 | PUT | `/config/video-engine` | 更新视频引擎配置 |
| 配置 | POST | `/config/video-engine/test` | 测试视频引擎 |
| WS | WS | `/ws/tasks/{id}` | 任务进度推送 |
| WS | WS | `/ws/scripts/{id}` | 剧本全局进度 |

---

## 15. 版本历史

| 版本 | 日期 | 变更说明 | 作者 |
|------|------|----------|------|
| v1.0 | 2026-07-14 | 初始版本 | - |
