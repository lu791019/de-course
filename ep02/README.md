# EP02 — Git + GitHub + Python 環境 + Module/Package

> EP02 的目標：學會 Git 協作流程、用 uv 管理 Python 環境、把散落的 .py 整理成 Package。

## 本集做了什麼

| 段落 | 內容 |
|------|------|
| Git GUI | VS Code Source Control 操作（stage → commit → diff） |
| GitHub | 建帳號 → 建 repo → push → clone |
| VS Code Extension | GitLens、Error Lens、autoDocstring |
| Python 環境管理 | 為什麼需要虛擬環境 → uv 安裝與操作 |
| Module / Package | `if __name__` → `__init__.py` → crawlers package |

## 環境建置

### 安裝 uv

```bash
# macOS / Linux（含 WSL）
curl -LsSf https://astral.sh/uv/install.sh | sh

# 重新載入 shell
source ~/.bashrc    # 或 source ~/.zshrc

# 驗證
uv --version        # 預期：uv 0.x.x
```

### 用 uv 管理本專案

```bash
cd ~/de-01-projects/de-course/ep02

# 根據 pyproject.toml + uv.lock 安裝所有套件
uv sync

# 跑爬蟲（不用手動 activate！）
uv run python ptt_crawler.py
uv run python finmind.py
```

### uv 常用指令

```bash
uv init my-project          # 建立新專案
uv add requests pandas      # 安裝套件（自動更新 pyproject.toml）
uv remove pandas            # 移除套件
uv run python main.py       # 在虛擬環境中執行
uv sync                     # 根據 lock 檔同步套件
uv python install 3.11      # 安裝指定 Python 版本
```

## Module / Package

### Module = 一個 .py 檔

```bash
# 直接執行
uv run python ptt_crawler.py

# 在 Python 裡 import
>>> from ptt_crawler import ptt_crawler
```

### `if __name__ == "__main__"` 的作用

```python
def ptt_crawler(board):
    ...

# 直接跑 → 會執行；被 import → 不會執行
if __name__ == "__main__":
    result = ptt_crawler("Stock")
    print(result)
```

### Package = 資料夾 + `__init__.py`

```
crawlers/                    ← Package
├── __init__.py              ← 讓 Python 認這是 package
├── ptt_crawler.py           ← 加了 if __name__
├── finmind.py
└── hahow_crawler.py
```

```python
# 從 package import（不會自動跑爬蟲）
from crawlers import ptt_crawler
from crawlers import crawler_finmind
from crawlers import crawler_hahow_course
```

## EP02 結束後的專案狀態

相比 EP01，新增了：

| 檔案 / 資料夾 | 說明 | EP01 就有？ |
|--------------|------|:---------:|
| `crawlers/` | 整理後的 Python Package | 新增 |
| `crawlers/__init__.py` | Package 匯出設定 | 新增 |
| `pyproject.toml` | uv 專案設定（dependencies） | 新增 |
| `uv.lock` | 套件版本鎖定 | 新增 |
| `.python-version` | Python 版本設定 | 新增 |
| `.gitignore` | Git 忽略規則 | 新增 |

## 詳細指南

- [Git GitHub 開發協作手冊](../Git_GitHub開發協作手冊.md)（Branch → PR → Review & Merge → 團隊協作）
- [操作手冊 — 環境建置](../操作手冊_環境建置.md)（uv 完整教學 + Module/Package 詳解）

## 常見問題

| 問題 | 解法 |
|------|------|
| `uv: command not found` | `source ~/.bashrc` 或重新安裝 uv |
| `uv sync` 失敗 | 確認在有 `pyproject.toml` 的目錄下 |
| `ModuleNotFoundError` | 用 `uv run python` 執行，不是直接 `python` |
| import 時爬蟲自動跑了 | 檢查 .py 底部有沒有加 `if __name__ == "__main__"` |
