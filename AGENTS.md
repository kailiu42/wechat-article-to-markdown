# AGENTS.md - 开发指南

本文件为 Agent 编程助手提供项目开发规范。

## 项目概述

- **项目类型**: Python CLI 工具
- **功能**: 微信公众号文章抓取并转换为 Markdown
- **依赖**: camoufox(反检测浏览器), markdownify, beautifulsoup4, httpx

---

## 构建与测试命令

### 环境配置

```bash
# 安装依赖 (推荐使用 uv)
pip install -e .
# 或
uv pip install -e .
```

### 测试命令

```bash
# 运行所有非 e2e 测试 (CI 默认方式)
pytest -m "not e2e"
uv run --with pytest pytest -m "not e2e"

# 运行单个测试文件
pytest tests/test_core.py
pytest tests/test_e2e_live.py

# 运行单个测试函数
pytest tests/test_core.py::test_extract_publish_time_supports_multiple_patterns

# 运行所有测试包括 e2e (需要设置环境变量)
export WECHAT_E2E_URLS="https://mp.weixin.qq.com/s/xxx"
export WECHAT_E2E_TIMEOUT="240"
pytest -m e2e
```

### 构建与发布

```bash
# 构建包
uv build

# 本地运行
python main.py <wechat-article-url>
```

---

## 代码风格指南

### 导入规范

- 使用 `from __future__ import annotations` 启用延迟注解
- 标准库导入放最前，第三方库次之，本地模块最后
- 每组导入之间空一行

```python
from __future__ import annotations

import asyncio
import re
import sys
from pathlib import Path

import httpx
import markdownify
from bs4 import BeautifulSoup
from camoufox.async_api import AsyncCamoufox

import wechat_article_to_markdown
```

### 类型注解

- 函数参数和返回值应使用类型注解
- 使用 `| None` 而非 `Optional`
- 使用 `str | dict | list` 而非 typing 模块

```python
async def download_image(
    client: httpx.AsyncClient,
    img_url: str,
    img_dir: Path,
    index: int,
    semaphore: asyncio.Semaphore,
) -> tuple[str, str | None]:
    ...
```

### 命名约定

- 函数/变量: `snake_case` (如 `extract_publish_time`)
- 类: `PascalCase` (如 `AsyncCamoufox`)
- 常量: `UPPER_SNAKE_CASE` (如 `OUTPUT_DIR`)
- 私有函数/变量: 前缀 `_` (如 `_internal_func`)

### Docstring 规范

- 使用中文 docstring
- 使用 Google 风格或简明描述

```python
def extract_publish_time(html: str) -> str:
    """从 HTML script 标签中提取发布时间"""
    ...
```

### 错误处理

- 捕获异常时使用具体异常类型
- 避免裸露的 `except:`
- 关键错误使用 `sys.exit(1)` 退出

```python
try:
    ts = int(val)
    if ts > 0:
        return format_timestamp(ts)
except ValueError:
    return val
```

### 代码组织

- 使用分节注释分隔功能模块 (如 `# Helpers`, `# Image Downloading`)

### 异步编程

- 使用 `async`/`await` 语法
- 使用 `asyncio.Semaphore` 控制并发
- 使用 `asyncio.gather` 并发执行任务

### 路径处理

- 使用 `pathlib.Path` 而非字符串拼接
- 创建目录使用 `mkdir(parents=True, exist_ok=True)`

### 测试规范

- 测试文件放在 `tests/` 目录
- 使用 `pytest.mark.e2e` 标记端到端测试
- e2e 测试需要 `WECHAT_E2E_URLS` 环境变量
- 非 e2e 测试应能离线运行

```python
import pytest

pytestmark = pytest.mark.e2e
```

---

## 项目结构

```
.
├── wechat_article_to_markdown.py  # 核心逻辑
├── main.py                        # CLI 入口
├── tests/
│   ├── test_core.py              # 单元测试
│   └── test_e2e_live.py          # 端到端测试
├── pyproject.toml                # 项目配置
└── requirements.txt              # 依赖
```

---

## 注意事项

1. **反检测浏览器**: 代码使用 Camoufox 浏览器，需要 Firefox 或 Chrome 浏览器支持
2. **e2e 测试**: 实际网络请求，可能不稳定，CI 默认跳过
3. **输出目录**: 默认输出到 `./output` 目录
4. **调试**: 失败时会保存 `output/debug.html` 供排查