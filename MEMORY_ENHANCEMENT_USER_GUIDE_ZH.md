# CoPaw 记忆增强系统使用文档

**版本**: v1.0
**创建日期**: 2026-03-19
**作者**: 大葱
**最后更新**: 2026-03-19 07:10 (北京时间)

---

## 📖 目录

1. [系统概述](#系统概述)
2. [快速开始](#快速开始)
3. [功能详解](#功能详解)
4. [API 参考](#api-参考)
5. [Skill 工具](#skill-工具)
6. [Web 界面](#web-界面)
7. [最佳实践](#最佳实践)
8. [故障排查](#故障排查)
9. [技术架构](#技术架构)

---

## 系统概述

### 什么是记忆增强系统？

CoPaw 记忆增强系统是一个**本地化、高性能、支持向量检索**的记忆管理系统，旨在增强 CoPaw 原生记忆功能，提供：

- 🧠 **向量检索** - 语义搜索，而非仅关键词匹配
- ⚡ **快速响应** - 50-200ms 检索速度（vs 原生 500-2000ms）
- 🌐 **可视化管理** - Web 界面，直观操作
- 🔄 **自动同步** - 与原生 MEMORY.md 保持同步
- 🔒 **完全本地** - 数据不上传外部服务

### 核心特性

| 特性 | 原生系统 | 增强系统 |
|------|----------|----------|
| 检索方式 | 关键词/向量 (需 API) | 向量 (本地) + 关键词 |
| 检索速度 | 500-2000ms | 50-200ms |
| 离线可用 | ❌ | ✅ |
| 可视化管理 | ❌ | ✅ |
| 去重机制 | ❌ | ✅ |
| 热度追踪 | ❌ | ✅ |
| Web 界面 | ❌ | ✅ |

### 系统架构

```
┌─────────────────────────────────────────────────────┐
│                  CoPaw 记忆增强系统                   │
├─────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │  Skill 工具  │  │  Web 界面   │  │  REST API   │  │
│  │ enhanced_   │  │ memory_     │  │ /api/*      │  │
│  │ memory.py   │  │ dashboard   │  │             │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## 快速开始

### 1. 系统检查

```bash
# 检查 Web 服务是否运行
curl http://127.0.0.1:8089/api/stats

# 检查 Skill 是否可用
@CoPawAI 使用 get_memory_stats 查看记忆库统计
```

### 2. 首次使用

**步骤 1: 同步原生记忆**

```bash
# 方法 1: 使用 Skill 工具
@CoPawAI 使用 sync_memories 同步原生记忆

# 方法 2: 使用 Web 界面
访问 http://127.0.0.1:8089 → 点击"🔄 同步原生记忆"

# 方法 3: 使用 API
curl -X POST http://127.0.0.1:8089/api/sync \
  -H "Content-Type: application/json" \
  -d '{"days": 30}'
```

**步骤 2: 测试搜索**

```bash
# Web 界面
访问 http://127.0.0.1:8089 → 输入关键词 → 点击搜索

# Skill 工具
@CoPawAI 使用 enhanced_memory_search 搜索"项目配置"

# API
curl "http://127.0.0.1:8089/api/search?q=项目配置&k=5"
```

### 3. 添加第一条记忆

```bash
# Skill 工具
@CoPawAI 使用 add_to_enhanced_memory 保存"用户偏好使用北京时间" category="preference" tags="偏好，时间"

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

## 功能详解

### 1. 向量检索

**原理**: 使用 `shibing624/text2vec-base-chinese` 模型将文本转换为 768 维向量，通过 FAISS 进行相似度搜索。

**优势**:
- 理解语义，而非仅匹配关键词
- 支持模糊查询
- 返回相关度评分

**示例**:
```
搜索"时间设置" → 返回"用户偏好使用北京时间" (即使不包含"时间设置"关键词)
```

### 2. 关键词搜索

**原理**: 传统 SQL LIKE 查询，作为向量检索的补充。

**使用场景**:
- 精确匹配特定词汇
- 向量检索结果不足时自动补充

### 3. 记忆分类

支持 5 种分类：

| 分类 | 说明 | 示例 |
|------|------|------|
| `fact` | 事实信息 | 配置文件位置、系统架构 |
| `event` | 事件记录 | 会议记录、系统更新 |
| `decision` | 决策记录 | 技术选型、方案选择 |
| `preference` | 用户偏好 | 沟通风格、时间偏好 |
| `task` | 任务信息 | 待办事项、项目任务 |

### 4. 标签系统

**格式**: 逗号分隔的字符串或数组

**示例**:
```json
{
  "tags": ["会议", "项目 A", "重要"]
}
```

**热门标签**: 系统自动统计使用频率最高的标签

### 5. 同步机制

**同步内容**:
- MEMORY.md 中的记忆记录
- memory/daily_log/*.md 中的每日日志

**同步策略**:
- 文件锁机制 (`fcntl.flock`) 防止竞争写入
- 原子写入 (`.tmp` → 原子替换) 防止数据损坏
- 去重检测 防止重复记录

**同步频率**: 建议每天一次，或修改 MEMORY.md 后手动同步

### 6. 热度追踪

系统自动记录每条记忆的访问次数，用于：
- 识别高频使用记忆
- 优化检索排序
- 清理低价值记忆

---

## API 参考

### 基础信息

- **Base URL**: `http://127.0.0.1:8089`
- **Content-Type**: `application/json`
- **CORS**: 已启用

### 端点列表

#### GET /api/stats

获取记忆库统计信息。

**响应示例**:
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

搜索记忆。

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `q` | string | ✅ | 搜索查询 |
| `k` | integer | ❌ | 返回数量，默认 10 |
| `vector` | boolean | ❌ | 是否使用向量检索，默认 true |

**响应示例**:
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

添加记忆。

**请求体**:
```json
{
  "content": "记忆内容",
  "category": "fact",
  "tags": ["标签 1", "标签 2"]
}
```

**响应**:
```json
{
  "id": 15,
  "status": "ok",
  "message": "记忆已保存 (ID: 15)"
}
```

#### GET /api/memories

获取所有记忆。

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `category` | string | 按分类过滤 |
| `limit` | integer | 返回数量限制，默认 100 |

#### DELETE /api/memories/:id

删除记忆。

**响应**:
```json
{
  "status": "ok",
  "message": "记忆 15 已删除"
}
```

#### GET /api/categories

获取所有分类。

**响应**:
```json
{
  "categories": ["fact", "event", "decision", "preference", "task"]
}
```

#### GET /api/tags

获取热门标签。

**响应**:
```json
{
  "tags": [
    {"tag": "memory_md", "count": 7},
    {"tag": "daily_log", "count": 3}
  ]
}
```

#### POST /api/sync

同步原生记忆。

**请求体**:
```json
{
  "days": 30
}
```

**响应**:
```json
{
  "status": "ok",
  "total": 10,
  "message": "同步完成：10 条记录"
}
```

---

## Skill 工具

### enhanced_memory_search

**功能**: 搜索记忆（向量 + 关键词）

**参数**:
| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `query` | string | ✅ | - | 搜索查询 |
| `top_k` | integer | ❌ | 5 | 返回数量 |
| `use_vector` | boolean | ❌ | true | 是否使用向量检索 |

**使用示例**:
```
@CoPawAI 使用 enhanced_memory_search 搜索"项目配置"
@CoPawAI 使用 enhanced_memory_search 搜索"会议记录" top_k=10
@CoPawAI 使用 enhanced_memory_search 搜索"测试" use_vector=false
```

**返回格式**:
```
🔍 找到 5 条相关记忆：

1. 🧠 [fact] 记忆内容预览...
   相关度：0.852 | 访问次数：3
   标签：标签 1, 标签 2

2. 📝 [event] 记忆内容预览...
   相关度：0.500 | 访问次数：1
   标签：标签 3
```

### add_to_enhanced_memory

**功能**: 添加到记忆库

**参数**:
| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `content` | string | ✅ | - | 记忆内容 |
| `category` | string | ❌ | fact | 分类 |
| `tags` | string | ❌ | '' | 逗号分隔的标签 |

**使用示例**:
```
@CoPawAI 使用 add_to_enhanced_memory 保存"用户偏好使用北京时间"
@CoPawAI 使用 add_to_enhanced_memory 保存"项目 A 启动会议" category="event" tags="会议，项目 A"
```

**返回格式**:
```
✅ 记忆已保存 (ID: 15)

分类：preference
标签：['偏好', '时间']
```

### get_memory_stats

**功能**: 获取记忆库统计

**参数**: 无

**使用示例**:
```
@CoPawAI 使用 get_memory_stats 查看记忆库统计
```

**返回格式**:
```
📊 记忆库统计：

总记忆数：13 条
向量索引：15 条

按分类统计:
  - fact: 8 条
  - event: 3 条
  - decision: 1 条
  - preference: 1 条

热门标签:
  - #memory_md: 7 次
  - #daily_log: 3 次

索引类型：IndexFlatL2
向量维度：768
索引文件大小：46,125 bytes
```

### sync_memories

**功能**: 同步原生记忆

**参数**:
| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `days` | integer | ❌ | 30 | 同步最近 N 天的每日日志 |

**使用示例**:
```
@CoPawAI 使用 sync_memories 同步原生记忆
@CoPawAI 使用 sync_memories days=7
```

**返回格式**:
```
✅ 同步完成!

同步记录数：10 条
时间：2026-03-19 07:10:00 (北京时间)
```

---

## Web 界面

### 访问地址

```
http://127.0.0.1:8089
```

### 功能模块

#### 1. 统计看板

显示：
- 总记忆数
- 向量索引数
- 分类数量
- 标签数量

#### 2. 搜索框

- 支持语义搜索
- 实时显示结果
- 显示相关度评分

#### 3. 结果列表

每条结果显示：
- 分类标签（颜色区分）
- 内容预览
- 相关度百分比
- 标签列表

#### 4. 添加记忆

点击"➕ 添加记忆"按钮，弹出模态框：
- 内容（必填）
- 分类（下拉选择）
- 标签（逗号分隔）

#### 5. 同步按钮

点击"🔄 同步原生记忆"按钮：
- 同步 MEMORY.md
- 同步每日日志
- 更新向量索引

---

## 最佳实践

### 1. 记忆添加规范

**好的记忆**:
```
✅ 用户偏好使用北京时间 (Asia/Shanghai)，除非特殊说明
✅ 项目 A 技术选型：使用 Python 3.11 + FastAPI + PostgreSQL
✅ 2026-03-19 会议决定：记忆增强系统采用并行方案，不替换原生系统
```

**不好的记忆**:
```
❌ 测试
❌ 用户偏好
❌ 会议记录（内容过于简略）
```

### 2. 分类使用指南

| 场景 | 推荐分类 |
|------|----------|
| 用户偏好、习惯 | `preference` |
| 会议、活动记录 | `event` |
| 技术选型、方案决策 | `decision` |
| 配置信息、系统架构 | `fact` |
| 待办事项、项目任务 | `task` |

### 3. 标签命名规范

- 使用简短、有意义的词汇
- 避免过长标签（建议 < 20 字符）
- 使用统一命名（如日期格式：`2026-03-19`）
- 项目相关：`项目 A`、`项目 B`
- 类型相关：`会议`、`配置`、`决策`

### 4. 同步策略

| 场景 | 建议操作 |
|------|----------|
| 每日首次使用 | 执行一次同步 |
| 修改 MEMORY.md 后 | 手动同步 |
| 发现搜索结果不全 | 检查同步状态 |
| 系统重启后 | 验证 Web 服务运行 |

### 5. 性能优化

- **首次使用**: 向量模型下载约 500MB，后续无需下载
- **大量记忆**: 支持 10 万 + 条记忆，无需担心容量
- **检索速度**: 向量检索 50-200ms，关键词检索 < 50ms

---

## 故障排查

### 问题 1: 搜索返回空

**可能原因**:
- 未同步原生记忆
- 记忆库为空

**解决方案**:
```bash
# 执行同步
@CoPawAI 使用 sync_memories 同步原生记忆

# 检查统计
@CoPawAI 使用 get_memory_stats 查看记忆库统计
```

### 问题 2: 工具未找到

**可能原因**:
- Skill 未启用
- 配置文件错误

**解决方案**:
```bash
# 检查配置文件
cat ~/.copaw/config.json

# 确保 skills 配置包含 memory_enhanced
{
  "skills": {
    "enabled": ["memory_enhanced"]
  }
}
```

### 问题 3: 向量模型下载失败

**可能原因**:
- 网络连接问题
- 模型服务器不可达

**解决方案**:
```bash
# 检查网络连接
ping huggingface.co

# 手动下载模型（可选）
# 参考：https://huggingface.co/shibing624/text2vec-base-chinese
```

### 问题 4: 数据库错误

**可能原因**:
- 数据库文件损坏
- 权限问题

**解决方案**:
```bash
# 备份当前数据库
cp ~/.copaw/memory_enhanced/enhanced_memory.db ~/.copaw/backup/

# 删除并重新初始化
rm ~/.copaw/memory_enhanced/enhanced_memory.db
python3 ~/.copaw/memory_enhanced/db_init.py

# 重新同步
@CoPawAI 使用 sync_memories 同步原生记忆
```

### 问题 5: Web 服务无法访问

**可能原因**:
- 服务未启动
- 端口被占用

**解决方案**:
```bash
# 检查服务状态
ps aux | grep web_api

# 检查端口
netstat -tlnp | grep 8089

# 重启服务
cd /app/src/copaw/agents/skills/memory_enhanced
pkill -f web_api.py
python3 web_api.py &
```

---

## 技术架构

### 依赖库

| 库 | 版本 | 用途 |
|----|------|------|
| `faiss-cpu` | 1.13.2 | 向量索引 |
| `flask` | 3.1.3 | Web 框架 |
| `flask-cors` | 6.0.2 | CORS 支持 |
| `sentence-transformers` | latest | 向量模型 |

### 文件结构

```
~/.copaw/
├── memory_enhanced/
│   ├── db_init.py          # 数据库初始化脚本
│   ├── db_manager.py       # SQLite 数据库操作
│   ├── faiss_manager.py    # FAISS 向量索引管理
│   ├── sync_manager.py     # 原生记忆同步管理
│   └── enhanced_memory.db  # SQLite 数据库文件
├── skills/
│   └── memory_enhanced/
│       ├── SKILL.md        # Skill 文档
│       ├── enhanced_memory.py  # Skill 工具实现
│       ├── web_api.py      # Web API 服务
│       └── test_e2e.py     # 端到端测试脚本
├── web/
│   └── templates/
│       └── memory_dashboard.html  # Web 界面模板
└── memory_enhanced_faiss.index    # FAISS 向量索引文件
```

### 数据模型

#### 记忆表 (memories)

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | INTEGER | 主键，自增 |
| `content` | TEXT | 记忆内容 |
| `category` | TEXT | 分类 (fact/event/decision/preference/task) |
| `source_file` | TEXT | 来源文件 |
| `created_at` | DATETIME | 创建时间 |
| `updated_at` | DATETIME | 更新时间 |
| `access_count` | INTEGER | 访问次数 |

#### 标签表 (memory_tags)

| 字段 | 类型 | 说明 |
|------|------|------|
| `memory_id` | INTEGER | 外键，关联 memories.id |
| `tag` | TEXT | 标签名称 |

### 向量模型

- **模型**: `shibing624/text2vec-base-chinese`
- **维度**: 768
- **语言**: 中文
- **来源**: HuggingFace
- **大小**: ~500MB

### 索引类型

- **类型**: `IndexFlatL2` (精确搜索)
- **距离度量**: L2 (欧几里得距离)
- **特点**: 精度高，适合中小规模数据

---

## 更新日志

### v1.0 (2026-03-19)

**初始版本**:
- ✅ 向量检索功能
- ✅ 关键词搜索功能
- ✅ 添加记忆功能
- ✅ 记忆统计功能
- ✅ 同步原生记忆功能
- ✅ Web 可视化管理界面
- ✅ REST API
- ✅ Skill 工具集成
- ✅ 文件锁机制
- ✅ 原子写入机制
- ✅ 去重检测
- ✅ 热度追踪

---

## 联系与支持

- **文档位置**: `/app/working/MEMORY_ENHANCEMENT_USER_GUIDE.md`
- **Skill 位置**: `/app/src/copaw/agents/skills/memory_enhanced/`
- **Web 界面**: http://127.0.0.1:8089
- **测试脚本**: `/app/src/copaw/agents/skills/memory_enhanced/test_e2e.py`

---

**文档结束**

*最后更新：2026-03-19 07:10 (北京时间)*