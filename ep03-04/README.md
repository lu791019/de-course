# EP03-04 — Docker 基礎 + Docker Compose 多服務

> EP03-04 的目標：學會用 Docker 打包應用、用 Docker Compose 一鍵啟動多服務架構。

## 本集做了什麼

| 段落 | 內容 |
|------|------|
| EP03 上午 | Docker 概念 → 安裝 Docker Engine → Dockerfile → Build + Run |
| EP04 下午 | Docker Compose 概念 → 4 服務 infra → hahow-crawler 完整系統 |

## 環境需求：安裝 Docker Engine

> 不用 Docker Desktop，直接在 WSL 裡裝 Docker Engine。

```bash
# 安裝 Docker Engine（WSL Ubuntu）
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# 加入 docker group
sudo usermod -aG docker $USER

# 啟動 Docker
sudo service docker start

# ⚠️ 讓 docker group 生效（在 Windows PowerShell 執行，不是 Ubuntu 裡）
# wsl --shutdown
# 然後重新開啟 Ubuntu

# 驗證
docker --version          # 預期：Docker version 29.x.x
docker compose version    # 預期：Docker Compose version v5.x.x
docker run hello-world    # 預期：Hello from Docker!
```

**Mac 替代方案：** `brew install colima docker docker-compose && colima start`

> 完整安裝步驟 + FAQ + Mac Colima → 見 [Docker 安裝教學手冊](../Docker安裝教學手冊.md)

## 快速開始

### Dockerfile — Build + Run

```bash
cd ~/de-01-projects/de-course/ep03-04

# Build image
docker build -t test-app .

# 跑 container（互動模式）
docker run -it test-app

# 在 container 裡跑爬蟲
uv run python crawlers/ptt_crawler.py
exit
```

### Docker Compose — 4 服務 infra

```bash
# 啟動所有服務（背景）
docker compose up -d

# 查看服務狀態
docker compose ps        # 預期：4 個服務都 running

# 停止所有服務
docker compose down
```

## Dockerfile 解讀

```dockerfile
FROM python:3.13-slim              # 基底 image
RUN apt-get update && ...          # 安裝系統工具
RUN curl ... | sh                  # 安裝 uv
WORKDIR /app                       # 設定工作目錄
COPY pyproject.toml uv.lock ./     # 先複製依賴檔（利用 Docker 快取）
RUN uv sync                        # 安裝套件
COPY crawlers/ ./crawlers/         # 複製 package
COPY *.py ./                       # 複製腳本
CMD ["/bin/bash"]                  # 預設開 bash
```

## 服務總覽

| 服務 | Image | Port | 用途 |
|------|-------|------|------|
| RabbitMQ | rabbitmq:3-management | 5672 / 15672 | 訊息佇列（管理介面 localhost:15672） |
| Flower | mher/flower | 5555 | Celery 任務監控（localhost:5555） |
| MySQL | mysql:8.0 | 3306 | 資料庫 |
| phpMyAdmin | phpmyadmin:latest | 8000 | 資料庫管理介面（localhost:8000） |

### 預設帳號密碼

| 服務 | 帳號 | 密碼 |
|------|------|------|
| RabbitMQ | worker | worker |
| MySQL | root | 1234 |

### Web 介面

| 服務 | 網址 |
|------|------|
| RabbitMQ 管理 | http://localhost:15672 |
| Flower 監控 | http://localhost:5555 |
| phpMyAdmin | http://localhost:8000 |

## EP03-04 結束後的專案狀態

相比 EP02，新增了：

| 檔案 | 說明 | EP02 就有？ |
|------|------|:---------:|
| `Dockerfile` | Docker 映像檔配置 | 新增 |
| `docker-compose.yml` | 4 服務 infra 設定 | 新增 |

## 整合版 vs 拆開版

本資料夾使用**整合版** docker-compose.yml（所有服務寫在同一個檔案）。

[hahow-crawler](https://github.com/lu791019/hahow-crawler-de-course-materials) 有**拆開版**（每個服務一個 compose file）。差別：

| | 整合版（本資料夾） | 拆開版（hahow-crawler） |
|---|---|---|
| 啟動 | `docker compose up` 一行 | 每個服務各一行 |
| network | 自動建 | 手動 `docker network create` |
| 適合 | 一鍵部署、教學 | 開發測試、逐步除錯 |

## 詳細指南

- [Docker 安裝教學手冊](../Docker安裝教學手冊.md)（完整安裝 + systemd + FAQ 26 題）
- [Docker Compose 開發操作手冊](../Docker_Compose開發操作手冊.md)（Daemon + Nginx 部署 + Restart 策略 + 指令速查）
- [Docker 實作操作手冊](../Docker實作操作手冊.md)（Dockerfile → Compose 多服務 → DockerHub → Portainer）

## 常見問題

| 問題 | 解法 |
|------|------|
| `docker: command not found` | `sudo service docker start`；確認已安裝 Docker Engine |
| `Cannot connect to Docker daemon` | `sudo service docker start` |
| `permission denied` | `sudo usermod -aG docker $USER` → PowerShell `wsl --shutdown` → 重開 |
| `port already in use` | `docker ps` 找到佔用的 container → `docker stop` |
| build 很慢 | 第一次要下載基底 image，之後有快取會快 |
| `iptables: No chain/target/match` / `DOCKER-FORWARD` | `sudo service docker restart` 即可。新版 Docker 預設用 nftables，不需要切 iptables-legacy |
| phpMyAdmin `exec format error` | Apple Silicon Mac 限定。compose 裡已改用 `phpmyadmin:latest`（多架構） |
