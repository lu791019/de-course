# EP01 — 環境建置：WSL + VS Code + Git

> EP01 的目標：建好開發環境，讓你能在 Linux 上寫程式、用 VS Code 編輯、用 Git 做版本控制。

## 本集做了什麼

| 段落 | 內容 |
|------|------|
| ETL 觀念 | 什麼是 Extract → Transform → Load，課程的完整架構 |
| 架構圖 | 用 diagrams.net 畫 ETL 架構圖 |
| WSL 安裝 | 在 Windows 上裝 WSL + Ubuntu（Mac 用內建 Terminal） |
| Linux 基礎指令 | pwd / ls / cd / mkdir / touch / cat / cp / mv / rm |
| VS Code | 安裝 + WSL Extension + 內建 Terminal |
| Git + Git Graph | clone 本 repo，用 Git Graph 看 commit 歷史 |

## 環境建置步驟

### Windows：安裝 WSL + Ubuntu

```bash
# PowerShell（系統管理員）
wsl --install -d Ubuntu-22.04

# 重開機後，設定使用者名稱和密碼
# 完成後確認：
wsl -l -v
# NAME            STATE    VERSION
# Ubuntu-22.04    Running  2
```

### 更新系統 + 確認工具

```bash
# WSL Ubuntu Terminal
sudo apt update && sudo apt upgrade -y

# 確認 Python
python3 --version     # 預期：Python 3.10.x

# 確認 Git
git --version         # 預期：git version 2.x.x

# 如果沒有 Git：
sudo apt install git -y
```

### Mac 使用者

Mac 不用裝 WSL，直接用 Terminal。確認 Python 和 Git：

```bash
python3 --version
git --version
# 如果沒有 → 系統會跳出「Install Command Line Developer Tools」→ 按 Install
```

### 安裝 VS Code

1. 下載安裝 [VS Code](https://code.visualstudio.com/)
2. **Windows**：在 VS Code 裝 **WSL** Extension（讓 VS Code 連進 WSL）
3. 在 WSL Terminal 輸入 `code .` 打開 VS Code

### 必裝 VS Code Extension

| Extension | 用途 |
|-----------|------|
| WSL（Windows 必裝） | 讓 VS Code 連進 WSL |
| Python | Python 語法支援 |
| Git Graph | 圖形化 Git 歷史 |

### Clone 本 Repo

```bash
cd ~
mkdir de-01-projects && cd de-01-projects
git clone https://github.com/lu791019/de-test.git
cd de-test
code .
```

### Git 初始設定

```bash
git config --global user.name "你的名字"
git config --global user.email "你的email"
```

## EP01 結束後的專案狀態

clone 下來後，你的 `de-test/` 裡會有這些檔案：

| 檔案 | 說明 |
|------|------|
| `ptt_crawler.py` | PTT 看板爬蟲（EP02 會用到） |
| `hahow_crawler.py` | Hahow 課程爬蟲 |
| `finmind.py` | FinMind 股票資料查詢 |
| `test.py` | requests 基本測試 |
| `main.py` | 主程式入口 |
| `requirements.txt` | pip 套件清單 |

> 這些是 clone 來的，EP01 不會修改它們。EP02 才會開始跑爬蟲和整理成 Package。

## 詳細安裝指南

如果遇到問題，參考完整的安裝手冊：

- [操作手冊 — 環境建置](../操作手冊_EP01-04_完整路線.md)（WSL 安裝步驟 + Linux 指令 + VS Code 設定）
- [Git GitHub 開發協作手冊](../Git_GitHub開發協作手冊.md)（Git 基礎操作 + VS Code GUI）

## 常見問題

| 問題 | 解法 |
|------|------|
| `wsl --install` 報錯 | 確認 Windows 10 2004+ 或 Windows 11，BIOS 開啟虛擬化 |
| 開 WSL 發現在 `/mnt/c/Users/...` | 執行 `cd ~` 回到 Linux 家目錄 |
| `code .` 沒反應 | 確認 VS Code 有裝 WSL Extension |
| 登入是 root 不是個人帳號 | PowerShell：`ubuntu config --default-user 你的帳號` |
