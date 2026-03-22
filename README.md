# CoPaw 记忆增强系统 / CoPaw Memory Enhancement System

[中文](#简介) | [English](#introduction)

---

## 简介 / Introduction

<a name="简介"></a>

**CoPaw 记忆增强系统** 是一个本地化、高性能、支持向量检索的记忆管理系统，旨在增强 CoPaw 的原生记忆功能。

**CoPaw Memory Enhancement System** is a localized, high-performance, vector retrieval-supported memory management system designed to enhance CoPaw's native memory capabilities.

### 核心特性 / Key Features

- 🧠 **向量检索 / Vector Search** - 语义搜索，精准理解意图 / Semantic search for accurate intent understanding
- ⚡ **快速响应 / Fast Response** - 50-200ms 检索速度 / 50-200ms retrieval speed
- 🌐 **可视化管理 / Visual Management** - Web 界面，直观操作 / Web interface for intuitive operation
- 🔄 **自动同步 / Auto Sync** - 与原生 MEMORY.md 保持同步 / Synchronized with native MEMORY.md
- 🔒 **完全本地 / Fully Local** - 数据不上云，保护隐私 / Local-only data, privacy protected

---

## 快速开始 / Quick Start

### 环境要求 / Requirements

- Python 3.8+
- pip

### 安装 / Installation

```bash
# 克隆仓库 / Clone the repository
git clone https://github.com/dgzmt/Openclaw_CoPaw_MEMORY_ENHANCEMENT.git
cd Openclaw_CoPaw_MEMORY_ENHANCEMENT

# 安装依赖 / Install dependencies
pip install faiss-cpu flask flask-cors sentence-transformers
```

### 使用 / Usage

```bash
# 启动 Web 服务 / Start Web service
cd memory_enhanced
python web_api.py

# 访问 Web 界面 / Access Web interface
# http://127.0.0.1:8089
```

---

## 文档 / Documentation

详细的用户指南请查看 / For detailed user guide, please refer to:

📄 [MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md)

文档包含以下内容 / Documentation includes:

| 章节 / Chapter | 内容 / Content |
|----------------|----------------|
| [系统概述 / System Overview](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md#系统概述--system-overview) | 系统介绍和架构 / System intro and architecture |
| [快速开始 / Quick Start](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md#快速开始--quick-start) | 快速上手指南 / Getting started guide |
| [功能详解 / Features](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md#功能详解--features) | 核心功能详细介绍 / Detailed feature descriptions |
| [API 参考 / API Reference](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md#api-参考--api-reference) | REST API 完整文档 / Complete REST API documentation |
| [Skill 工具 / Skill Tools](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md#skill-工具--skill-tools) | Skill 工具使用指南 / Skill tools usage guide |
| [Web 界面 / Web Interface](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md#web-界面--web-interface) | Web 界面功能说明 / Web interface guide |
| [最佳实践 / Best Practices](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md#最佳实践--best-practices) | 使用建议和技巧 / Usage tips and best practices |
| [故障排查 / Troubleshooting](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md#故障排查--troubleshooting) | 常见问题解答 / FAQ and troubleshooting |
| [技术架构 / Technical Architecture](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md#技术架构--technical-architecture) | 技术细节说明 / Technical details |

---

## 技术栈 / Tech Stack

| 技术 / Technology | 用途 / Purpose |
|------------------|----------------|
| Python | 主语言 / Main language |
| FAISS | 向量索引 / Vector indexing |
| Flask | Web 框架 / Web framework |
| SQLite | 数据存储 / Data storage |
| sentence-transformers | 向量模型 / Vector model |

---

## 项目结构 / Project Structure

```
.
├── MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md  # 中英对照用户指南 / Bilingual user guide
└── README.md                                 # 本文件 / This file
```

---

## 支持 / Support

如有问题，请提交 Issue 或联系作者 / For questions, please submit an Issue or contact the author.

---

## 许可证 / License

MIT License

---

<a name="introduction"></a>

**English Version**

**CoPaw Memory Enhancement System** is a localized, high-performance, vector retrieval-supported memory management system designed to enhance CoPaw's native memory capabilities.

For detailed documentation, please refer to [MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH_EN.md).

---

⭐ 如果这个项目对你有帮助，请给个 Star！/ If this project is helpful, please give it a Star!