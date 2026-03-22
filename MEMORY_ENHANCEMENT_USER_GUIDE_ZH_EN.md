# CoPaw 记忆增强系统使用文档 / CoPaw Memory Enhancement System User Guide

**版本 / Version**: v1.0
**创建日期 / Created**: 2026-03-19
**作者 / Author**: 大葱 / DaCong
**最后更新 / Last Updated**: 2026-03-19 07:10 (北京时间 / CST)

---

## 📖 目录 / Table of Contents

1. [系统概述 / System Overview](#系统概述--system-overview)
2. [快速开始 / Quick Start](#快速开始--quick-start)
3. [功能详解 / Features](#功能详解--features)
4. [API 参考 / API Reference](#api-参考--api-reference)
5. [Skill 工具 / Skill Tools](#skill-工具--skill-tools)
6. [Web 界面 / Web Interface](#web-界面--web-interface)
7. [最佳实践 / Best Practices](#最佳实践--best-practices)
8. [故障排查 / Troubleshooting](#故障排查--troubleshooting)
9. [技术架构 / Technical Architecture](#技术架构--technical-architecture)

---

## 系统概述 / System Overview

### 什么是记忆增强系统？ / What is Memory Enhancement System?

CoPaw 记忆增强系统是一个**本地化、高性能、支持向量检索**的记忆管理系统，旨在增强 CoPaw 原生记忆功能，提供：

CoPaw Memory Enhancement System is a **localized, high-performance, vector retrieval-supported** memory management system designed to enhance CoPaw's native memory capabilities, providing:

- 🧠 **向量检索 / Vector Search** - 语义搜索，而非仅关键词匹配 / Semantic search, not just keyword matching
- ⚡ **快速响应 / Fast Response** - 50-200ms 检索速度（vs 原生 500-2000ms）/ 50-200ms retrieval speed (vs native 500-2000ms)
- 🌐 **可视化管理 / Visual Management** - Web 界面，直观操作 / Web interface for intuitive operation
- 🔄 **自动同步 / Auto Sync** - 与原生 MEMORY.md 保持同步 / Synchronized with native MEMORY.md
- 🔒 **完全本地 / Fully Local** - 数据不上传外部服务 / Data never uploaded to external services

### 核心特性 / Core Features

| 特性 / Feature | 原生系统 / Native System | 增强系统 / Enhanced System |
|-----------------|--------------------------|---------------------------|
| 检索方式 / Search Method | 关键词/向量 (需 API) / Keyword/Vector (API required) | 向量 (本地) + 关键词 / Vector (local) + Keyword |
| 检索速度 / Search Speed | 500-2000ms | 50-200ms |
| 离线可用 / Offline Available | ❌ | ✅ |
| 可视化管理 / Visual Management | ❌ | ✅ |
| 去重机制 / Deduplication | ❌ | ✅ |
| 热度追踪 / Hotness Tracking | ❌ | ✅ |
| Web 界面 / Web Interface | ❌ | ✅ |

### 系统架构 / System Architecture

```
┌─────────────────────────────────────────────────────┐
│            CoPaw 记忆增强系统 / Memory Enhancement     │
├─────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │  Skill 工具  │  │  Web 界面   │  │  REST API   │  │
│  │ Skill Tools │  │ Web Interface│  │ /api/*      │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## 快速开始 / Quick Start

### 1. 系统检查 / System Check

```bash
# 检查 Web 服务是否运行 / Check if Web service is running
curl http://127.0.0.1:8089/api/stats

# 检查 Skill 是否可用 / Check if Skill is available
@CoPawAI 使用 get_memory_stats 查看记忆库统计
@CoPawAI Use get_memory_stats to view memory stats
```

### 2. 首次使用 / First Use

**步骤 1: 同步原生记忆 / Step 1: Sync Native Memory**

```bash
# 方法 1: 使用 Skill 工具 / Method 1: Using Skill Tool
@CoPawAI 使用 sync_memories 同步原生记忆
@CoPawAI Use sync_memories to sync native memory

# 方法 2: 使用 Web 界面 / Method 2: Using Web Interface
访问 http://127.0.0.1:8089 → 点击"🔄 同步原生记忆"
Visit http://127.0.0.1:8089 → Click "🔄 Sync Native Memory"

# 方法 3: 使用 API / Method 3: Using API
curl -X POST http://127.0.0.1:8089/api/sync \
  -H "Content-Type: application/json" \
  -d '{"days": 30}'
```

**步骤 2: 测试搜索 / Step 2: Test Search**

```bash
# Web 界面 / Web Interface
访问 http://127.0.0.1:8089 → 输入关键词 → 点击搜索
Visit http://127.0.0.1:8089 → Enter keywords → Click search

# Skill 工具 / Skill Tool
@CoPawAI 使用 enhanced_memory_search 搜索"项目配置"
@CoPawAI Use enhanced_memory_search to search "project config"

# API
curl "http://127.0.0.1:8089/api/search?q=项目配置&k=5"
```

### 3. 添加第一条记忆 / Add First Memory

```bash
# Skill 工具 / Skill Tool
@CoPawAI 使用 add_to_enhanced_memory 保存"用户偏好使用北京时间" category="preference" tags="偏好，时间"
@CoPawAI Use add_to_enhanced_memory to save "User prefers Beijing time" category="preference" tags="preference,time"

# API
curl -X POST http://127.0.0.1:8089/api/memories \
  -H "Content-Type: application/json" \
  -d '{
    "content": "用户偏好使用北京时间",
    "category": "preference",
    "tags": ["偏好", "时间"]
  }'
```

---

## 功能详解 / Features

### 1. 向量检索 / Vector Search

**原理 / Principle**: 使用 `shibing624/text2vec-base-chinese` 模型将文本转换为 768 维向量，通过 FAISS 进行相似度搜索。/ Uses `shibing624/text2vec-base-chinese` model to convert text into 768-dimensional vectors, searching via FAISS similarity.

**优势 / Advantages**:
- 理解语义，而非仅匹配关键词 / Understand semantics, not just keyword matching
- 支持模糊查询 / Support fuzzy queries
- 返回相关度评分 / Return relevance scores

**示例 / Example**:
```
搜索"时间设置" → 返回"用户偏好使用北京时间" (即使不包含"时间设置"关键词)
Search "time settings" → Returns "User prefers Beijing time" (even without "time settings" keyword)
```

### 2. 关键词搜索 / Keyword Search

**原理 / Principle**: 传统 SQL LIKE 查询，作为向量检索的补充。/ Traditional SQL LIKE queries, as a supplement to vector search.

**使用场景 / Use Cases**:
- 精确匹配特定词汇 / Exact match for specific terms
- 向量检索结果不足时自动补充 / Auto-supplement when vector results are insufficient

### 3. 记忆分类 / Memory Categories

支持 5 种分类 / Supports 5 categories:

| 分类 / Category | 说明 / Description | 示例 / Example |
|-----------------|-------------------|----------------|
| `fact` | 事实信息 / Factual Info | 配置文件位置、系统架构 / Config file location, System architecture |
| `event` | 事件记录 / Event Record | 会议记录、系统更新 / Meeting notes, System updates |
| `decision` | 决策记录 / Decision Record | 技术选型、方案选择 / Technology selection, Solution choice |
| `preference` | 用户偏好 / User Preference | 沟通风格、时间偏好 / Communication style, Time preference |
| `task` | 任务信息 / Task Info | 待办事项、项目任务 / Todos, Project tasks |

### 4. 标签系统 / Tag System

**格式 / Format**: 逗号分隔的字符串或数组 / Comma-separated string or array

**示例 / Example**:
```json
{
  "tags": ["会议", "项目 A", "重要"]
}
{
  "tags": ["meeting", "project A", "important"]
}
```

**热门标签 / Hot Tags**: 系统自动统计使用频率最高的标签 / System automatically counts most frequently used tags

### 5. 同步机制 / Synchronization Mechanism

**同步内容 / Sync Content**:
- MEMORY.md 中的记忆记录 / Memory records in MEMORY.md
- memory/daily_log/*.md 中的每日日志 / Daily logs in memory/daily_log/*.md

**同步策略 / Sync Strategy**:
- 文件锁机制 (`fcntl.flock`) 防止竞争写入 / File lock mechanism (`fcntl.flock`) to prevent race conditions
- 原子写入 (`.tmp` → 原子替换) 防止数据损坏 / Atomic writes (`.tmp` → atomic replacement) to prevent data corruption
- 去重检测 防止重复记录 / Deduplication to prevent duplicate records

**同步频率 / Sync Frequency**: 建议每天一次，或修改 MEMORY.md 后手动同步 / Recommended once daily, or manually sync after modifying MEMORY.md

### 6. 热度追踪 / Hotness Tracking

系统自动记录每条记忆的访问次数，用于 / System automatically records access count for each memory, used for:
- 识别高频使用记忆 / Identify frequently used memories
- 优化检索排序 / Optimize retrieval ranking
- 清理低价值记忆 / Clean up low-value memories

---

## API 参考 / API Reference

### 基础信息 / Basic Info

- **Base URL**: `http://127.0.0.1:8089`
- **Content-Type**: `application/json`
- **CORS**: 已启用 / Enabled

### 端点列表 / Endpoints

#### GET /api/stats

获取记忆库统计信息。/ Get memory database statistics.

**响应示例 / Response Example**:
```json
{
  "database": {
    "total": 13,
    "by_category": {
      "fact": 8,
      "event": 3,
      "decision": 1,
      "preference": 1
    },
    "top_tags": {
      "memory_md": 7,
      "daily_log": 3
    }
  },
  "faiss": {
    "total_vectors": 15,
    "index_type": "IndexFlatL2",
    "dimension": 768,
    "index_file_size": 46125
  }
}
```

#### GET /api/search

搜索记忆。/ Search memories.

**参数 / Parameters**:
| 参数 / Param | 类型 / Type | 必填 / Required | 说明 / Description |
|--------------|-------------|-----------------|-------------------|
| `q` | string | ✅ | 搜索查询 / Search query |
| `k` | integer | ❌ | 返回数量，默认 10 / Return count, default 10 |
| `vector` | boolean | ❌ | 是否使用向量检索，默认 true / Use vector search, default true |

**响应示例 / Response Example**:
```json
{
  "query": "记忆",
  "total": 6,
  "results": [
    {
      "id": 4,
      "content": "记忆系统结构...",
      "category": "fact",
      "tags": "Memory Structure,memory_md",
      "score": 0.8523,
      "source": "vector"
    }
  ]
}
```

#### POST /api/memories

添加记忆。/ Add memory.

**请求体 / Request Body**:
```json
{
  "content": "记忆内容 / Memory content",
  "category": "fact",
  "tags": ["标签 1 / Tag 1", "标签 2 / Tag 2"]
}
```

**响应 / Response**:
```json
{
  "id": 15,
  "status": "ok",
  "message": "记忆已保存 (ID: 15)"
}
```

#### GET /api/memories

获取所有记忆。/ Get all memories.

**参数 / Parameters**:
| 参数 / Param | 类型 / Type | 说明 / Description |
|--------------|-------------|-------------------|
| `category` | string | 按分类过滤 / Filter by category |
| `limit` | integer | 返回数量限制，默认 100 / Return limit, default 100 |

#### DELETE /api/memories/:id

删除记忆。/ Delete memory.

**响应 / Response**:
```json
{
  "status": "ok",
  "message": "记忆 15 已删除 / Memory 15 deleted"
}
```

#### GET /api/categories

获取所有分类。/ Get all categories.

**响应 / Response**:
```json
{
  "categories": ["fact", "event", "decision", "preference", "task"]
}
```

#### GET /api/tags

获取热门标签。/ Get hot tags.

**响应 / Response**:
```json
{
  "tags": [
    {"tag": "memory_md", "count": 7},
    {"tag": "daily_log", "count": 3}
  ]
}
```

#### POST /api/sync

同步原生记忆。/ Sync native memory.

**请求体 / Request Body**:
```json
{
  "days": 30
}
```

**响应 / Response**:
```json
{
  "status": "ok",
  "total": 10,
  "message": "同步完成：10 条记录 / Sync complete: 10 records"
}
```

---

## Skill 工具 / Skill Tools

### enhanced_memory_search

**功能 / Function**: 搜索记忆（向量 + 关键词）/ Search memories (vector + keyword)

**参数 / Parameters**:
| 参数 / Param | 类型 / Type | 必填 / Required | 默认值 / Default | 说明 / Description |
|--------------|-------------|-----------------|-------------------|-------------------|
| `query` | string | ✅ | - | 搜索查询 / Search query |
| `top_k` | integer | ❌ | 5 | 返回数量 / Return count |
| `use_vector` | boolean | ❌ | true | 是否使用向量检索 / Use vector search |

**使用示例 / Usage Examples**:
```
@CoPawAI 使用 enhanced_memory_search 搜索"项目配置"
@CoPawAI 使用 enhanced_memory_search 搜索"会议记录" top_k=10
@CoPawAI 使用 enhanced_memory_search 搜索"测试" use_vector=false
```

**返回格式 / Return Format**:
```
🔍 找到 5 条相关记忆：
🔍 Found 5 related memories:

1. 🧠 [fact] 记忆内容预览... / Memory content preview...
   相关度：0.852 / Relevance: 0.852 | 访问次数：3 / Access count: 3
   标签：标签 1, 标签 2 / Tags: tag1, tag2

2. 📝 [event] 记忆内容预览... / Memory content preview...
   相关度：0.500 / Relevance: 0.500 | 访问次数：1 / Access count: 1
   标签：标签 3 / Tags: tag3
```

### add_to_enhanced_memory

**功能 / Function**: 添加到记忆库 / Add to memory database

**参数 / Parameters**:
| 参数 / Param | 类型 / Type | 必填 / Required | 默认值 / Default | 说明 / Description |
|--------------|-------------|-----------------|-------------------|-------------------|
| `content` | string | ✅ | - | 记忆内容 / Memory content |
| `category` | string | ❌ | fact | 分类 / Category |
| `tags` | string | ❌ | '' | 逗号分隔的标签 / Comma-separated tags |

**使用示例 / Usage Examples**:
```
@CoPawAI 使用 add_to_enhanced_memory 保存"用户偏好使用北京时间"
@CoPawAI 使用 add_to_enhanced_memory 保存"项目 A 启动会议" category="event" tags="会议，项目 A"
```

**返回格式 / Return Format**:
```
✅ 记忆已保存 (ID: 15) / Memory saved (ID: 15)

分类：preference
标签：['偏好', '时间'] / Tags: ['preference', 'time']
```

### get_memory_stats

**功能 / Function**: 获取记忆库统计 / Get memory database statistics

**参数 / Parameters**: 无 / None

**使用示例 / Usage Example**:
```
@CoPawAI 使用 get_memory_stats 查看记忆库统计
```

**返回格式 / Return Format**:
```
📊 记忆库统计 / Memory Stats:

总记忆数：13 条 / Total memories: 13
向量索引：15 条 / Vector indices: 15

按分类统计 / By category:
  - fact: 8 条 / 8
  - event: 3 条 / 3
  - decision: 1 条 / 1
  - preference: 1 条 / 1

热门标签 / Hot tags:
  - #memory_md: 7 次 / 7 times
  - #daily_log: 3 次 / 3 times

索引类型 / Index type：IndexFlatL2
向量维度 / Vector dimension：768
索引文件大小 / Index file size：46,125 bytes
```

### sync_memories

**功能 / Function**: 同步原生记忆 / Sync native memory

**参数 / Parameters**:
| 参数 / Param | 类型 / Type | 必填 / Required | 默认值 / Default | 说明 / Description |
|--------------|-------------|-----------------|-------------------|-------------------|
| `days` | integer | ❌ | 30 | 同步最近 N 天的每日日志 / Sync daily logs from last N days |

**使用示例 / Usage Examples**:
```
@CoPawAI 使用 sync_memories 同步原生记忆
@CoPawAI 使用 sync_memories days=7
```

**返回格式 / Return Format**:
```
✅ 同步完成 / Sync Complete!

同步记录数：10 条 / Synced records: 10
时间：2026-03-19 07:10:00 (北京时间) / Time: 2026-03-19 07:10:00 (CST)
```

---

## Web 界面 / Web Interface

### 访问地址 / Access URL

```
http://127.0.0.1:8089
```

### 功能模块 / Function Modules

#### 1. 统计看板 / Statistics Dashboard

显示 / Display:
- 总记忆数 / Total memories
- 向量索引数 / Vector indices
- 分类数量 / Category count
- 标签数量 / Tag count

#### 2. 搜索框 / Search Box

- 支持语义搜索 / Support semantic search
- 实时显示结果 / Real-time display results
- 显示相关度评分 / Show relevance scores

#### 3. 结果列表 / Result List

每条结果显示 / Each result displays:
- 分类标签（颜色区分）/ Category tags (color-coded)
- 内容预览 / Content preview
- 相关度百分比 / Relevance percentage
- 标签列表 / Tag list

#### 4. 添加记忆 / Add Memory

点击"➕ 添加记忆"按钮，弹出模态框 / Click "➕ Add Memory" button, popup modal:
- 内容（必填）/ Content (required)
- 分类（下拉选择）/ Category (dropdown)
- 标签（逗号分隔）/ Tags (comma-separated)

#### 5. 同步按钮 / Sync Button

点击"🔄 同步原生记忆"按钮 / Click "🔄 Sync Native Memory" button:
- 同步 MEMORY.md / Sync MEMORY.md
- 同步每日日志 / Sync daily logs
- 更新向量索引 / Update vector indices

### 界面截图 / Interface Screenshot

```
┌─────────────────────────────────────────────────────┐
│  🧠 CoPaw 记忆增强系统 / Memory Enhancement          │
│  支持向量检索、可视化管理、自动同步                    │
│  Vector search, Visual management, Auto sync        │
├─────────────────────────────────────────────────────┤
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │
│  │   13    │ │   15    │ │    4    │ │    8    │  │
│  │ 总记忆数 │ │ 向量索引 │ │ 分类数量 │ │ 标签数量 │  │
│  │ Total   │ │Vectors  │ │Categories│ │  Tags   │  │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘  │
├─────────────────────────────────────────────────────┤
│  [🔍 搜索记忆... / Search memories...    ] [🔍 搜索] │
├─────────────────────────────────────────────────────┤
│  搜索结果 / Search Results           找到 6 条结果   │
│  ┌───────────────────────────────────────────────┐  │
│  │ 🧠 [fact] 记忆内容预览... / Memory preview... │  │
│  │    相关度 / Relevance：85.2% | 访问 / Access  │  │
│  └───────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────┤
│  [🔄 同步原生记忆 / Sync Native Memory]              │
└─────────────────────────────────────────────────────┘
```

---

## 最佳实践 / Best Practices

### 1. 记忆添加规范 / Memory Addition Guidelines

**好的记忆 / Good Memories**:
```
✅ 用户偏好使用北京时间 (Asia/Shanghai)，除非特殊说明
✅ User prefers Beijing time (Asia/Shanghai), unless specified
✅ 项目 A 技术选型：使用 Python 3.11 + FastAPI + PostgreSQL
✅ Project A tech stack: Python 3.11 + FastAPI + PostgreSQL
✅ 2026-03-19 会议决定：记忆增强系统采用并行方案，不替换原生系统
✅ 2026-03-19 Meeting decision: Memory enhancement adopts parallel solution, not replacing native system
```

**不好的记忆 / Bad Memories**:
```
❌ 测试 / Test
❌ 用户偏好 / User preference (too vague)
❌ 会议记录（内容过于简略）/ Meeting notes (too brief)
```

### 2. 分类使用指南 / Category Usage Guide

| 场景 / Scenario | 推荐分类 / Recommended Category |
|-----------------|-------------------------------|
| 用户偏好、习惯 / User preferences, habits | `preference` |
| 会议、活动记录 / Meetings, events | `event` |
| 技术选型、方案决策 / Tech selection, decisions | `decision` |
| 配置信息、系统架构 / Config, system architecture | `fact` |
| 待办事项、项目任务 / Todos, project tasks | `task` |

### 3. 标签命名规范 / Tag Naming Conventions

- 使用简短、有意义的词汇 / Use short, meaningful words
- 避免过长标签（建议 < 20 字符）/ Avoid long tags (recommended < 20 chars)
- 使用统一命名（如日期格式：`2026-03-19`）/ Use consistent naming (e.g., date format: `2026-03-19`)
- 项目相关：`项目 A`、`项目 B` / Project-related: `Project A`, `Project B`
- 类型相关：`会议`、`配置`、`决策` / Type-related: `meeting`, `config`, `decision`

### 4. 同步策略 / Sync Strategy

| 场景 / Scenario | 建议操作 / Recommended Action |
|-----------------|------------------------------|
| 每日首次使用 / First daily use | 执行一次同步 / Run one sync |
| 修改 MEMORY.md 后 / After modifying MEMORY.md | 手动同步 / Manual sync |
| 发现搜索结果不全 / Finding incomplete search results | 检查同步状态 / Check sync status |
| 系统重启后 / After system restart | 验证 Web 服务运行 / Verify Web service running |

### 5. 性能优化 / Performance Optimization

- **首次使用 / First use**: 向量模型下载约 500MB，后续无需下载 / Vector model ~500MB download, no re-download needed
- **大量记忆 / Large memory**: 支持 10 万 + 条记忆，无需担心容量 / Supports 100k+ memories, no capacity concerns
- **检索速度 / Retrieval speed**: 向量检索 50-200ms，关键词检索 < 50ms / Vector search 50-200ms, keyword search < 50ms

---

## 故障排查 / Troubleshooting

### 问题 1: 搜索返回空 / Problem 1: Search Returns Empty

**可能原因 / Possible Causes**:
- 未同步原生记忆 / Native memory not synced
- 记忆库为空 / Memory database empty

**解决方案 / Solutions**:
```bash
# 执行同步 / Execute sync
@CoPawAI 使用 sync_memories 同步原生记忆

# 检查统计 / Check statistics
@CoPawAI 使用 get_memory_stats 查看记忆库统计
```

### 问题 2: 工具未找到 / Problem 2: Tool Not Found

**可能原因 / Possible Causes**:
- Skill 未启用 / Skill not enabled
- 配置文件错误 / Config file error

**解决方案 / Solutions**:
```bash
# 检查配置文件 / Check config file
cat ~/.copaw/config.json

# 确保 skills 配置包含 memory_enhanced
# Ensure skills config includes memory_enhanced
{
  "skills": {
    "enabled": ["memory_enhanced"]
  }
}
```

### 问题 3: 向量模型下载失败 / Problem 3: Vector Model Download Failed

**可能原因 / Possible Causes**:
- 网络连接问题 / Network connection issue
- 模型服务器不可达 / Model server unreachable

**解决方案 / Solutions**:
```bash
# 检查网络连接 / Check network
ping huggingface.co

# 手动下载模型（可选）/ Manual download (optional)
# 参考 / Reference: https://huggingface.co/shibing624/text2vec-base-chinese
```

### 问题 4: 数据库错误 / Problem 4: Database Error

**可能原因 / Possible Causes**:
- 数据库文件损坏 / Database file corrupted
- 权限问题 / Permission issue

**解决方案 / Solutions**:
```bash
# 备份当前数据库 / Backup current database
cp ~/.copaw/memory_enhanced/enhanced_memory.db ~/.copaw/backup/

# 删除并重新初始化 / Delete and reinitialize
rm ~/.copaw/memory_enhanced/enhanced_memory.db
python3 ~/.copaw/memory_enhanced/db_init.py

# 重新同步 / Resync
@CoPawAI 使用 sync_memories 同步原生记忆
```

### 问题 5: Web 服务无法访问 / Problem 5: Web Service Unreachable

**可能原因 / Possible Causes**:
- 服务未启动 / Service not started
- 端口被占用 / Port occupied

**解决方案 / Solutions**:
```bash
# 检查服务状态 / Check service status
ps aux | grep web_api

# 检查端口 / Check port
netstat -tlnp | grep 8089

# 重启服务 / Restart service
cd /app/src/copaw/agents/skills/memory_enhanced
pkill -f web_api.py
python3 web_api.py &
```

---

## 技术架构 / Technical Architecture

### 依赖库 / Dependencies

| 库 / Library | 版本 / Version | 用途 / Purpose |
|-------------|----------------|----------------|
| `faiss-cpu` | 1.13.2 | 向量索引 / Vector indexing |
| `flask` | 3.1.3 | Web 框架 / Web framework |
| `flask-cors` | 6.0.2 | CORS 支持 / CORS support |
| `sentence-transformers` | latest | 向量模型 / Vector model |

### 文件结构 / File Structure

```
~/.copaw/
├── memory_enhanced/
│   ├── db_init.py          # 数据库初始化脚本 / DB initialization script
│   ├── db_manager.py       # SQLite 数据库操作 / SQLite operations
│   ├── faiss_manager.py    # FAISS 向量索引管理 / FAISS vector index management
│   ├── sync_manager.py     # 原生记忆同步管理 / Native memory sync management
│   └── enhanced_memory.db  # SQLite 数据库文件 / SQLite database file
├── skills/
│   └── memory_enhanced/
│       ├── SKILL.md        # Skill 文档 / Skill documentation
│       ├── enhanced_memory.py  # Skill 工具实现 / Skill tool implementation
│       ├── web_api.py      # Web API 服务 / Web API service
│       └── test_e2e.py     # 端到端测试脚本 / E2E test script
├── web/
│   └── templates/
│       └── memory_dashboard.html  # Web 界面模板 / Web interface template
└── memory_enhanced_faiss.index    # FAISS 向量索引文件 / FAISS vector index file
```

### 数据模型 / Data Model

#### 记忆表 (memories) / Memory Table (memories)

| 字段 / Field | 类型 / Type | 说明 / Description |
|--------------|-------------|-------------------|
| `id` | INTEGER | 主键，自增 / Primary key, auto-increment |
| `content` | TEXT | 记忆内容 / Memory content |
| `category` | TEXT | 分类 (fact/event/decision/preference/task) |
| `source_file` | TEXT | 来源文件 / Source file |
| `created_at` | DATETIME | 创建时间 / Created time |
| `updated_at` | DATETIME | 更新时间 / Updated time |
| `access_count` | INTEGER | 访问次数 / Access count |

#### 标签表 (memory_tags) / Tag Table (memory_tags)

| 字段 / Field | 类型 / Type | 说明 / Description |
|--------------|-------------|-------------------|
| `memory_id` | INTEGER | 外键，关联 memories.id / Foreign key, references memories.id |
| `tag` | TEXT | 标签名称 / Tag name |

### 向量模型 / Vector Model

- **模型 / Model**: `shibing624/text2vec-base-chinese`
- **维度 / Dimension**: 768
- **语言 / Language**: 中文 / Chinese
- **来源 / Source**: HuggingFace
- **大小 / Size**: ~500MB

### 索引类型 / Index Type

- **类型 / Type**: `IndexFlatL2` (精确搜索 / Exact search)
- **距离度量 / Distance Metric**: L2 (欧几里得距离 / Euclidean distance)
- **特点 / Features**: 精度高，适合中小规模数据 / High precision, suitable for small to medium data

---

## 更新日志 / Changelog

### v1.0 (2026-03-19)

**初始版本 / Initial Version**:
- ✅ 向量检索功能 / Vector search
- ✅ 关键词搜索功能 / Keyword search
- ✅ 添加记忆功能 / Add memory
- ✅ 记忆统计功能 / Memory statistics
- ✅ 同步原生记忆功能 / Sync native memory
- ✅ Web 可视化管理界面 / Web visual management interface
- ✅ REST API
- ✅ Skill 工具集成 / Skill tool integration
- ✅ 文件锁机制 / File lock mechanism
- ✅ 原子写入机制 / Atomic write mechanism
- ✅ 去重检测 / Deduplication
- ✅ 热度追踪 / Hotness tracking

---

## 联系与支持 / Contact & Support

- **文档位置 / Documentation**: `/app/working/MEMORY_ENHANCEMENT_USER_GUIDE.md`
- **Skill 位置 / Skill Location**: `/app/src/copaw/agents/skills/memory_enhanced/`
- **Web 界面 / Web Interface**: http://127.0.0.1:8089
- **测试脚本 / Test Script**: `/app/src/copaw/agents/skills/memory_enhanced/test_e2e.py`

---

**文档结束 / End of Document**

*最后更新 / Last Updated：2026-03-19 07:10 (北京时间 / CST)*