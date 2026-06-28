# TibaMe 雲端資料工程師 — 課程教材

> 帶狀課 21 集教學用 repo

## 學習路線

### Season 0：基礎建置（EP01-04）

| EP | 主題 | 資料夾 | 操作手冊 |
|----|------|--------|---------|
| EP01 | WSL + VS Code + Git | [ep01/](ep01/) | [環境建置](操作手冊_環境建置.md) |
| EP02 | Git + GitHub + uv + Module/Package | [ep02/](ep02/) | [環境建置](操作手冊_環境建置.md) · [Git 協作](Git_GitHub開發協作手冊.md) |
| EP03 | Docker 安裝 + Dockerfile | [ep03-04/](ep03-04/) | [Docker 安裝](Docker安裝教學手冊.md) · [Docker 操作](Docker_Compose開發操作手冊.md) |
| EP04 | Docker Compose + 完整系統 | [ep03-04/](ep03-04/) | [Docker 實作](Docker實作操作手冊.md) |

### Season 1+：爬蟲系統（EP05+）

| Repo | 內容 | 用在哪 |
|------|------|--------|
| [de-project-course](https://github.com/lu791019/de-project-course) | Hahow 課程爬蟲（Celery + RabbitMQ + MySQL + Airflow + BigQuery + Metabase） | EP05+ 主力 |
| [crawler](https://github.com/TibameSam/crawler) | FinMind 股價爬蟲（多 Queue、Scheduler） | 對照參考 |

> 課堂以 **de-project-course** 為主要動手專案（涵蓋 EP04-20），crawler 作為對照參考。兩者是同一套架構、不同資料源。

## 操作手冊

| 手冊 | 內容 | 什麼時候看 |
|------|------|-----------|
| [操作手冊](操作手冊_環境建置.md) | WSL → VS Code → Git → uv → Module/Package | EP01-02 跟著做 |
| [Docker 安裝手冊](Docker安裝教學手冊.md) | Docker Engine 安裝 + FAQ 26 題 + 課前預載 | 安裝 Docker 時 |
| [Docker 操作手冊](Docker_Compose開發操作手冊.md) | Daemon + Nginx 部署 + Port + Restart + Compose | Docker 日常操作 |
| [Docker 實作手冊](Docker實作操作手冊.md) | Dockerfile → Compose 多服務 → DockerHub → Portainer | Docker 實作課 |
| [Git 協作手冊](Git_GitHub開發協作手冊.md) | Git 基礎 + VS Code GUI + Branch + PR + 團隊協作 | Git 協作流程 |

## 快速開始

### 1. 環境準備（EP01）

```bash
# WSL Ubuntu Terminal
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip python3-venv git -y
```

### 2. Clone 本 repo

```bash
cd ~
mkdir de-01-projects && cd de-01-projects
git clone https://github.com/lu791019/de-test.git
cd de-test
```

### 3. Python 環境（EP02）

```bash
# 安裝 uv
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc

# 跑爬蟲測試
cd ep02
uv sync
uv run python ptt_crawler.py
```

### 4. Docker（EP03-04）

```bash
# 安裝 Docker Engine（完整步驟見 Docker安裝教學手冊.md）
# 安裝完成後：
cd ~/de-01-projects/de-test/ep03-04
docker compose up -d        # 起 4 個 infra 服務
docker compose ps           # 確認全部 running
docker compose down         # 停止
```

## 專案結構

```
de-test/
├── README.md                          ← 你在這裡
├── 操作手冊_環境建置.md
├── Docker安裝教學手冊.md
├── Docker實作操作手冊.md
├── Docker_Compose開發操作手冊.md
├── Git_GitHub開發協作手冊.md
├── ep01/                              ← EP01：環境建置
├── ep02/                              ← EP02：uv + Module/Package
│   ├── crawlers/                      ← Python Package
│   ├── pyproject.toml
│   └── uv.lock
└── ep03-04/                           ← EP03-04：Docker
    ├── Dockerfile
    ├── docker-compose.yml             ← 4 服務 infra
    ├── crawlers/
    ├── pyproject.toml
    └── uv.lock
```
