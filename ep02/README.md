# EP02 — Python 環境管理 + Module / Package

本資料夾是 EP02 教完後的狀態：在 EP01 的基礎上加了 uv 環境管理和 crawlers package。

## 環境需求

- Python 3.10+
- [uv](https://docs.astral.sh/uv/)（Python 套件管理工具）

## 快速開始

```bash
# 1. 安裝 uv（如果還沒裝）
curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. 安裝套件
uv sync

# 3. 執行程式
uv run python ptt_crawler.py
uv run python finmind.py
```

## uv 環境管理

```bash
# 查看 Python 版本
uv python list --only-installed

# 安裝指定 Python 版本
uv python install 3.10

# 建立虛擬環境
uv venv --python 3.10

# 安裝套件
uv add pandas
uv add requests

# 移除套件
uv remove pandas

# 執行程式（自動使用虛擬環境）
uv run python main.py
```

## Module vs Package

### Module = 一個 .py 檔

每個 .py 檔都是一個 module，可以直接執行或被 import：

```bash
# 直接執行
uv run python ptt_crawler.py

# 在 Python 裡 import
>>> from ptt_crawler import ptt_crawler
>>> result = ptt_crawler("Stock")
```

⚠️ 注意：原始的 ptt_crawler.py 底部有直接執行的程式碼，import 時會自動跑爬蟲。

### Package = 資料夾 + `__init__.py`

`crawlers/` 資料夾就是一個 package：

```
crawlers/
├── __init__.py          ← 讓 Python 認這是 package
├── ptt_crawler.py       ← 加了 if __name__ == "__main__"
├── finmind.py           ← 加了 if __name__ == "__main__"
└── hahow_crawler.py     ← 加了 if __name__ == "__main__"
```

```python
# 從 package import（不會自動跑爬蟲）
from crawlers import ptt_crawler
from crawlers import crawler_finmind
from crawlers import crawler_hahow_course

# 手動呼叫
result = ptt_crawler("Tech_Job")
print(result)
```

### `if __name__ == "__main__"` 的作用

```python
# crawlers/ptt_crawler.py
def ptt_crawler(board):
    ...

# 只有直接執行才跑，被 import 時不跑
if __name__ == "__main__":
    result = ptt_crawler("Stock")
    print(result)
```

| 情境 | `__name__` 的值 | 底部程式碼會跑嗎 |
|------|----------------|----------------|
| `python ptt_crawler.py`（直接執行） | `"__main__"` | ✅ 會跑 |
| `from crawlers import ptt_crawler`（被 import） | `"ptt_crawler"` | ❌ 不會跑 |

### `__init__.py` 的作用

```python
# crawlers/__init__.py
from .ptt_crawler import ptt_crawler
from .finmind import crawler_finmind
from .hahow_crawler import crawler_hahow_course
```

- **標記**：告訴 Python 這個資料夾是 package
- **匯出**：把函式「往上提」，讓使用者可以 `from crawlers import ptt_crawler` 直接拿到函式

## 檔案說明

| 檔案 | 說明 |
|------|------|
| `ptt_crawler.py` | PTT 看板爬蟲（原始版，底部會直接跑） |
| `finmind.py` | FinMind 股票資料查詢（原始版） |
| `hahow_crawler.py` | Hahow 課程爬蟲（原始版） |
| `crawlers/` | 整理後的 package（加了 `if __name__` + `__init__.py`） |
| `pyproject.toml` | uv 專案設定（dependencies） |
| `uv.lock` | 套件版本鎖定 |
| `.python-version` | Python 版本設定 |
| `requirements.txt` | pip 用的套件清單（備用） |

## uv vs pip 對照

| 功能 | uv | pip |
|------|-----|-----|
| 安裝套件 | `uv add pandas` | `pip install pandas` |
| 移除套件 | `uv remove pandas` | `pip uninstall pandas` |
| 建虛擬環境 | `uv venv --python 3.10` | `python -m venv .venv` |
| 執行程式 | `uv run main.py` | `python main.py` |
| 鎖定版本 | `uv.lock`（自動） | `pip freeze > requirements.txt` |
