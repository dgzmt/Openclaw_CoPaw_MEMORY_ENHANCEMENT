# CoPaw Memory Enhancement System

[中文](MEMORY_ENHANCEMENT_USER_GUIDE_ZH.md) | [English](MEMORY_ENHANCEMENT_USER_GUIDE_EN.md)

---

## Introduction

CoPaw Memory Enhancement System is a **localized, high-performance, vector retrieval-supported** memory management system designed to enhance CoPaw's native memory capabilities.

## Key Features

- 🧠 **Vector Search** - Semantic search for accurate intent understanding
- ⚡ **Fast Response** - 50-200ms retrieval speed
- 🌐 **Visual Management** - Web interface for intuitive operation
- 🔄 **Auto Sync** - Synchronized with native MEMORY.md
- 🔒 **Fully Local** - Local-only data, privacy protected

---

## Quick Start

### Requirements

- Python 3.8+
- pip

### Installation

```bash
# Clone the repository
git clone https://github.com/dgzmt/Openclaw_CoPaw_MEMORY_ENHANCEMENT.git
cd Openclaw_CoPaw_MEMORY_ENHANCEMENT

# Install dependencies
pip install faiss-cpu flask flask-cors sentence-transformers
```

### Usage

```bash
# Start Web service
cd memory_enhanced
python web_api.py

# Access Web interface
# http://127.0.0.1:8089
```

---

## Documentation

| Language | File |
|----------|------|
| 中文 | [MEMORY_ENHANCEMENT_USER_GUIDE_ZH.md](./MEMORY_ENHANCEMENT_USER_GUIDE_ZH.md) |
| English | [MEMORY_ENHANCEMENT_USER_GUIDE_EN.md](./MEMORY_ENHANCEMENT_USER_GUIDE_EN.md) |

---

## Tech Stack

| Technology | Purpose |
|------------|---------|
| Python | Main language |
| FAISS | Vector indexing |
| Flask | Web framework |
| SQLite | Data storage |
| sentence-transformers | Vector model |

---

## Project Structure

```
.
├── MEMORY_ENHANCEMENT_USER_GUIDE_ZH.md  # Chinese version
├── MEMORY_ENHANCEMENT_USER_GUIDE_EN.md  # English version
└── README.md
```

---

## Support

For questions, please submit an Issue or contact the author.

---

## License

MIT License

---

⭐ If this project is helpful, please give it a Star!