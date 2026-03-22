# CoPaw Memory Enhancement System User Guide

**Version**: v1.0
**Created**: 2026-03-19
**Author**: DaCong
**Last Updated**: 2026-03-19 07:10 (CST)

---

## 📖 Table of Contents

1. [System Overview](#system-overview)
2. [Quick Start](#quick-start)
3. [Features](#features)
4. [API Reference](#api-reference)
5. [Skill Tools](#skill-tools)
6. [Web Interface](#web-interface)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)
9. [Technical Architecture](#technical-architecture)

---

## System Overview

### What is Memory Enhancement System?

CoPaw Memory Enhancement System is a **localized, high-performance, vector retrieval-supported** memory management system designed to enhance CoPaw's native memory capabilities, providing:

- 🧠 **Vector Search** - Semantic search, not just keyword matching
- ⚡ **Fast Response** - 50-200ms retrieval speed (vs native 500-2000ms)
- 🌐 **Visual Management** - Web interface for intuitive operation
- 🔄 **Auto Sync** - Synchronized with native MEMORY.md
- 🔒 **Fully Local** - Data never uploaded to external services

### Core Features

| Feature | Native System | Enhanced System |
|---------|---------------|----------------|
| Search Method | Keyword/Vector (API required) | Vector (local) + Keyword |
| Search Speed | 500-2000ms | 50-200ms |
| Offline Available | ❌ | ✅ |
| Visual Management | ❌ | ✅ |
| Deduplication | ❌ | ✅ |
| Hotness Tracking | ❌ | ✅ |
| Web Interface | ❌ | ✅ |

### System Architecture

```
┌─────────────────────────────────────────────────────┐
│          CoPaw Memory Enhancement System            │
├─────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │ Skill Tools │  │Web Interface│  │  REST API  │  │
│  │ enhanced_   │  │  memory_   │  │   /api/*   │  │
│  │ memory.py   │  │ dashboard  │  │            │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## Quick Start

### 1. System Check

```bash
# Check if Web service is running
curl http://127.0.0.1:8089/api/stats

# Check if Skill is available
@CoPawAI Use get_memory_stats to view memory stats
```

### 2. First Use

**Step 1: Sync Native Memory**

```bash
# Method 1: Using Skill Tool
@CoPawAI Use sync_memories to sync native memory

# Method 2: Using Web Interface
Visit http://127.0.0.1:8089 → Click "🔄 Sync Native Memory"

# Method 3: Using API
curl -X POST http://127.0.0.1:8089/api/sync \
  -H "Content-Type: application/json" \
  -d '{"days": 30}'
```

**Step 2: Test Search**

```bash
# Web Interface
Visit http://127.0.0.1:8089 → Enter keywords → Click search

# Skill Tool
@CoPawAI Use enhanced_memory_search to search "project config"

# API
curl "http://127.0.0.1:8089/api/search?q=project%20config&k=5"
```

### 3. Add First Memory

```bash
# Skill Tool
@CoPawAI Use add_to_enhanced_memory to save "User prefers Beijing time" category="preference" tags="preference,time"

# API
curl -X POST http://127.0.0.1:8089/api/memories \
  -H "Content-Type: application/json" \
  -d '{
    "content": "User prefers Beijing time",
    "category": "preference",
    "tags": ["preference", "time"]
  }'
```

---

## Features

### 1. Vector Search

**Principle**: Uses `shibing624/text2vec-base-chinese` model to convert text into 768-dimensional vectors, searching via FAISS similarity.

**Advantages**:
- Understands semantics, not just keyword matching
- Supports fuzzy queries
- Returns relevance scores

**Example**:
```
Search "time settings" → Returns "User prefers Beijing time" (even without "time settings" keyword)
```

### 2. Keyword Search

**Principle**: Traditional SQL LIKE queries, as a supplement to vector search.

**Use Cases**:
- Exact match for specific terms
- Auto-supplement when vector results are insufficient

### 3. Memory Categories

Supports 5 categories:

| Category | Description | Example |
|----------|-------------|---------|
| `fact` | Factual Information | Config file location, System architecture |
| `event` | Event Records | Meeting notes, System updates |
| `decision` | Decision Records | Technology selection, Solution choice |
| `preference` | User Preferences | Communication style, Time preference |
| `task` | Task Information | Todos, Project tasks |

### 4. Tag System

**Format**: Comma-separated string or array

**Example**:
```json
{
  "tags": ["meeting", "project A", "important"]
}
```

**Hot Tags**: System automatically counts most frequently used tags

### 5. Synchronization Mechanism

**Sync Content**:
- Memory records in MEMORY.md
- Daily logs in memory/daily_log/*.md

**Sync Strategy**:
- File lock mechanism (`fcntl.flock`) to prevent race conditions
- Atomic writes (`.tmp` → atomic replacement) to prevent data corruption
- Deduplication to prevent duplicate records

**Sync Frequency**: Recommended once daily, or manually sync after modifying MEMORY.md

### 6. Hotness Tracking

System automatically records access count for each memory, used for:
- Identifying frequently used memories
- Optimizing retrieval ranking
- Cleaning up low-value memories

---

## API Reference

### Basic Info

- **Base URL**: `http://127.0.0.1:8089`
- **Content-Type**: `application/json`
- **CORS**: Enabled

### Endpoints

#### GET /api/stats

Get memory database statistics.

**Response Example**:
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

Search memories.

**Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | ✅ | Search query |
| `k` | integer | ❌ | Return count, default 10 |
| `vector` | boolean | ❌ | Use vector search, default true |

**Response Example**:
```json
{
  "query": "memory",
  "total": 6,
  "results": [
    {
      "id": 4,
      "content": "Memory system structure...",
      "category": "fact",
      "tags": "Memory Structure,memory_md",
      "score": 0.8523,
      "source": "vector"
    }
  ]
}
```

#### POST /api/memories

Add memory.

**Request Body**:
```json
{
  "content": "Memory content",
  "category": "fact",
  "tags": ["tag 1", "tag 2"]
}
```

**Response**:
```json
{
  "id": 15,
  "status": "ok",
  "message": "Memory saved (ID: 15)"
}
```

#### GET /api/memories

Get all memories.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `category` | string | Filter by category |
| `limit` | integer | Return limit, default 100 |

#### DELETE /api/memories/:id

Delete memory.

**Response**:
```json
{
  "status": "ok",
  "message": "Memory 15 deleted"
}
```

#### GET /api/categories

Get all categories.

**Response**:
```json
{
  "categories": ["fact", "event", "decision", "preference", "task"]
}
```

#### GET /api/tags

Get hot tags.

**Response**:
```json
{
  "tags": [
    {"tag": "memory_md", "count": 7},
    {"tag": "daily_log", "count": 3}
  ]
}
```

#### POST /api/sync

Sync native memory.

**Request Body**:
```json
{
  "days": 30
}
```

**Response**:
```json
{
  "status": "ok",
  "total": 10,
  "message": "Sync complete: 10 records"
}
```

---

## Skill Tools

### enhanced_memory_search

**Function**: Search memories (vector + keyword)

**Parameters**:
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | string | ✅ | - | Search query |
| `top_k` | integer | ❌ | 5 | Return count |
| `use_vector` | boolean | ❌ | true | Use vector search |

**Usage Examples**:
```
@CoPawAI Use enhanced_memory_search to search "project config"
@CoPawAI Use enhanced_memory_search to search "meeting notes" top_k=10
@CoPawAI Use enhanced_memory_search to search "test" use_vector=false
```

**Return Format**:
```
🔍 Found 5 related memories:

1. 🧠 [fact] Memory content preview...
   Relevance: 0.852 | Access count: 3
   Tags: tag1, tag2

2. 📝 [event] Memory content preview...
   Relevance: 0.500 | Access count: 1
   Tags: tag3
```

### add_to_enhanced_memory

**Function**: Add to memory database

**Parameters**:
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `content` | string | ✅ | - | Memory content |
| `category` | string | ❌ | fact | Category |
| `tags` | string | ❌ | '' | Comma-separated tags |

**Usage Examples**:
```
@CoPawAI Use add_to_enhanced_memory to save "User prefers Beijing time"
@CoPawAI Use add_to_enhanced_memory to save "Project A kickoff meeting" category="event" tags="meeting,project A"
```

**Return Format**:
```
✅ Memory saved (ID: 15)

Category: preference
Tags: ['preference', 'time']
```

### get_memory_stats

**Function**: Get memory database statistics

**Parameters**: None

**Usage Example**:
```
@CoPawAI Use get_memory_stats to view memory stats
```

**Return Format**:
```
📊 Memory Stats:

Total memories: 13
Vector indices: 15

By category:
  - fact: 8
  - event: 3
  - decision: 1
  - preference: 1

Hot tags:
  - #memory_md: 7 times
  - #daily_log: 3 times

Index type: IndexFlatL2
Vector dimension: 768
Index file size: 46,125 bytes
```

### sync_memories

**Function**: Sync native memory

**Parameters**:
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `days` | integer | ❌ | 30 | Sync daily logs from last N days |

**Usage Examples**:
```
@CoPawAI Use sync_memories to sync native memory
@CoPawAI Use sync_memories days=7
```

**Return Format**:
```
✅ Sync Complete!

Synced records: 10
Time: 2026-03-19 07:10:00 (CST)
```

---

## Web Interface

### Access URL

```
http://127.0.0.1:8089
```

### Function Modules

#### 1. Statistics Dashboard

Display:
- Total memories
- Vector indices
- Category count
- Tag count

#### 2. Search Box

- Support semantic search
- Real-time display results
- Show relevance scores

#### 3. Result List

Each result displays:
- Category tags (color-coded)
- Content preview
- Relevance percentage
- Tag list

#### 4. Add Memory

Click "➕ Add Memory" button, popup modal:
- Content (required)
- Category (dropdown)
- Tags (comma-separated)

#### 5. Sync Button

Click "🔄 Sync Native Memory" button:
- Sync MEMORY.md
- Sync daily logs
- Update vector indices

---

## Best Practices

### 1. Memory Addition Guidelines

**Good Memories**:
```
✅ User prefers Beijing time (Asia/Shanghai), unless specified
✅ Project A tech stack: Python 3.11 + FastAPI + PostgreSQL
✅ 2026-03-19 Meeting decision: Memory enhancement adopts parallel solution, not replacing native system
```

**Bad Memories**:
```
❌ Test
❌ User preference (too vague)
❌ Meeting notes (too brief)
```

### 2. Category Usage Guide

| Scenario | Recommended Category |
|----------|---------------------|
| User preferences, habits | `preference` |
| Meetings, events | `event` |
| Tech selection, decisions | `decision` |
| Config, system architecture | `fact` |
| Todos, project tasks | `task` |

### 3. Tag Naming Conventions

- Use short, meaningful words
- Avoid long tags (recommended < 20 chars)
- Use consistent naming (e.g., date format: `2026-03-19`)
- Project-related: `Project A`, `Project B`
- Type-related: `meeting`, `config`, `decision`

### 4. Sync Strategy

| Scenario | Recommended Action |
|----------|-------------------|
| First daily use | Run one sync |
| After modifying MEMORY.md | Manual sync |
| Finding incomplete search results | Check sync status |
| After system restart | Verify Web service running |

### 5. Performance Optimization

- **First use**: Vector model ~500MB download, no re-download needed
- **Large memory**: Supports 100k+ memories, no capacity concerns
- **Retrieval speed**: Vector search 50-200ms, keyword search < 50ms

---

## Troubleshooting

### Problem 1: Search Returns Empty

**Possible Causes**:
- Native memory not synced
- Memory database empty

**Solutions**:
```bash
# Execute sync
@CoPawAI Use sync_memories to sync native memory

# Check statistics
@CoPawAI Use get_memory_stats to view memory stats
```

### Problem 2: Tool Not Found

**Possible Causes**:
- Skill not enabled
- Config file error

**Solutions**:
```bash
# Check config file
cat ~/.copaw/config.json

# Ensure skills config includes memory_enhanced
{
  "skills": {
    "enabled": ["memory_enhanced"]
  }
}
```

### Problem 3: Vector Model Download Failed

**Possible Causes**:
- Network connection issue
- Model server unreachable

**Solutions**:
```bash
# Check network
ping huggingface.co

# Manual download (optional)
# Reference: https://huggingface.co/shibing624/text2vec-base-chinese
```

### Problem 4: Database Error

**Possible Causes**:
- Database file corrupted
- Permission issue

**Solutions**:
```bash
# Backup current database
cp ~/.copaw/memory_enhanced/enhanced_memory.db ~/.copaw/backup/

# Delete and reinitialize
rm ~/.copaw/memory_enhanced/enhanced_memory.db
python3 ~/.copaw/memory_enhanced/db_init.py

# Resync
@CoPawAI Use sync_memories to sync native memory
```

### Problem 5: Web Service Unreachable

**Possible Causes**:
- Service not started
- Port occupied

**Solutions**:
```bash
# Check service status
ps aux | grep web_api

# Check port
netstat -tlnp | grep 8089

# Restart service
cd /app/src/copaw/agents/skills/memory_enhanced
pkill -f web_api.py
python3 web_api.py &
```

---

## Technical Architecture

### Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| `faiss-cpu` | 1.13.2 | Vector indexing |
| `flask` | 3.1.3 | Web framework |
| `flask-cors` | 6.0.2 | CORS support |
| `sentence-transformers` | latest | Vector model |

### File Structure

```
~/.copaw/
├── memory_enhanced/
│   ├── db_init.py          # DB initialization script
│   ├── db_manager.py       # SQLite operations
│   ├── faiss_manager.py    # FAISS vector index management
│   ├── sync_manager.py     # Native memory sync management
│   └── enhanced_memory.db  # SQLite database file
├── skills/
│   └── memory_enhanced/
│       ├── SKILL.md        # Skill documentation
│       ├── enhanced_memory.py  # Skill tool implementation
│       ├── web_api.py      # Web API service
│       └── test_e2e.py     # E2E test script
├── web/
│   └── templates/
│       └── memory_dashboard.html  # Web interface template
└── memory_enhanced_faiss.index    # FAISS vector index file
```

### Data Model

#### Memory Table (memories)

| Field | Type | Description |
|-------|------|-------------|
| `id` | INTEGER | Primary key, auto-increment |
| `content` | TEXT | Memory content |
| `category` | TEXT | Category (fact/event/decision/preference/task) |
| `source_file` | TEXT | Source file |
| `created_at` | DATETIME | Created time |
| `updated_at` | DATETIME | Updated time |
| `access_count` | INTEGER | Access count |

#### Tag Table (memory_tags)

| Field | Type | Description |
|-------|------|-------------|
| `memory_id` | INTEGER | Foreign key, references memories.id |
| `tag` | TEXT | Tag name |

### Vector Model

- **Model**: `shibing624/text2vec-base-chinese`
- **Dimension**: 768
- **Language**: Chinese
- **Source**: HuggingFace
- **Size**: ~500MB

### Index Type

- **Type**: `IndexFlatL2` (Exact search)
- **Distance Metric**: L2 (Euclidean distance)
- **Features**: High precision, suitable for small to medium data

---

## Changelog

### v1.0 (2026-03-19)

**Initial Version**:
- ✅ Vector search
- ✅ Keyword search
- ✅ Add memory
- ✅ Memory statistics
- ✅ Sync native memory
- ✅ Web visual management interface
- ✅ REST API
- ✅ Skill tool integration
- ✅ File lock mechanism
- ✅ Atomic write mechanism
- ✅ Deduplication
- ✅ Hotness tracking

---

## Contact & Support

- **Documentation**: `/app/working/MEMORY_ENHANCEMENT_USER_GUIDE.md`
- **Skill Location**: `/app/src/copaw/agents/skills/memory_enhanced/`
- **Web Interface**: http://127.0.0.1:8089
- **Test Script**: `/app/src/copaw/agents/skills/memory_enhanced/test_e2e.py`

---

**End of Document**

*Last Updated: 2026-03-19 07:10 (CST)*