# Docker + Docker Compose 開發操作手冊

> 本手冊涵蓋 Docker 日常開發操作，從 daemon 管理到 Compose 多服務部署。
> 所有指令皆在 WSL Ubuntu 環境實測通過。

## 目錄

- [第一部分 Docker Daemon](#第一部分-docker-daemon)
- [第二部分 Docker 基本指令](#第二部分-docker-基本指令)
- [第三部分 什麼是 Port](#第三部分-什麼是-port)
- [第四部分 部署 Nginx 網頁伺服器](#第四部分-部署-nginx-網頁伺服器)
- [第五部分 容器管理指令](#第五部分-容器管理指令)
- [第六部分 Restart 重啟策略](#第六部分-restart-重啟策略)
- [第七部分 Docker Compose 介紹](#第七部分-docker-compose-介紹)
- [第八部分 Compose 部署 Nginx](#第八部分-compose-部署-nginx)
- [第九部分 Docker vs Docker Compose 比較](#第九部分-docker-vs-docker-compose-比較)
- [指令速查表](#指令速查表)
- [FAQ](#faq)

---

## 第一部分 Docker Daemon

### 什麼是 Docker Daemon

Docker Daemon（`dockerd`）是 Docker 的核心背景程式，負責：

- **管理容器**：建立、啟動、停止、刪除容器
- **管理映像檔**：從 DockerHub 下載、儲存、刪除映像檔
- **管理網路**：容器之間的網路通訊
- **管理儲存**：容器的資料持久化（volumes）

```
你的指令 ──→ Docker CLI (docker) ──→ Docker Daemon (dockerd) ──→ 容器
```

你在 Terminal 輸入 `docker run ...`，其實是 Docker CLI 把指令送給 Daemon，Daemon 才真正去建立容器。

### 啟動、停止、重啟 Docker

WSL Ubuntu 使用 `service` 指令管理 Docker Daemon：

```bash
# 啟動 Docker
sudo service docker start

# 查看 Docker 狀態
sudo service docker status
```

**預期輸出（status）：**
```
● docker.service - Docker Application Container Engine
     Active: active (running) since ...
```

看到 `active (running)` 代表 Docker 正在運行。

```bash
# 停止 Docker
sudo service docker stop

# 重啟 Docker（先停後啟）
sudo service docker restart
```

### 驗證 Docker 是否可用

```bash
# 檢查版本
docker --version
```

**預期輸出：**
```
Docker version 29.6.1, build 8900f1d
```

```bash
# 檢查詳細資訊
docker info
```

會顯示 Server Version、Storage Driver、CPUs、Total Memory 等資訊。如果看到 `Cannot connect to the Docker daemon`，代表 Daemon 沒有啟動，執行 `sudo service docker start`。

### 什麼時候需要啟動 Docker

| 情境 | 需要做什麼 |
|------|-----------|
| 剛開啟 WSL Terminal | `sudo service docker start` |
| 電腦重開機後 | `sudo service docker start` |
| Docker 指令報錯 `Cannot connect` | `sudo service docker restart` |
| 正常使用中 | 不需要做任何事 |

> **提示**：WSL 預設不會自動啟動 Docker，每次開 Terminal 都要手動啟動一次。

---

## 第二部分 Docker 基本指令

### 查看本地映像檔

```bash
docker images
```

**預期輸出：**
```
REPOSITORY   TAG             IMAGE ID       SIZE
nginx        stable-alpine   0d3b80406a13   92.5MB
```

### 查看容器

```bash
# 查看運行中的容器
docker ps

# 查看所有容器（包含已停止的）
docker ps -a
```

**預期輸出（docker ps）：**
```
NAMES      IMAGE                 STATUS         PORTS
my-nginx   nginx:stable-alpine   Up 5 seconds   0.0.0.0:8080->80/tcp
```

### 下載映像檔

```bash
docker pull nginx:stable-alpine
```

**預期輸出：**
```
stable-alpine: Pulling from library/nginx
...
Status: Downloaded newer image for nginx:stable-alpine
```

### 查看容器 Log

```bash
# 查看 log
docker logs my-nginx

# 持續追蹤 log（像 tail -f）
docker logs -f my-nginx
```

> 按 `Ctrl + C` 停止追蹤。

### 停止容器

```bash
docker stop my-nginx
```

### 刪除容器

```bash
# 刪除已停止的容器
docker rm my-nginx

# 強制刪除運行中的容器
docker rm -f my-nginx
```

### 刪除映像檔

```bash
docker rmi nginx:stable-alpine
```

---

## 第三部分 什麼是 Port

### IP 地址 + Port 號碼

當你在瀏覽器輸入 `127.0.0.1:8080`，這其實是兩個東西的組合：

```
127.0.0.1 : 8080
    IP    :  Port
```

- **IP 地址**：定位哪台電腦（伺服器），如同一棟大樓的**門牌號碼**
- **Port（埠號）**：定位該電腦上的哪個服務/應用程式，如同大樓裡的各個**房間號碼**

### 常見 Port 號碼

| Port | 服務 | 比喻 |
|------|------|------|
| 80 | 網頁服務（HTTP） | 大廳 |
| 443 | 安全網頁（HTTPS） | VIP 大廳 |
| 22 | SSH 遠端登入 | 員工入口 |
| 3306 | MySQL 資料庫 | 資料室 |
| 5672 | RabbitMQ 訊息佇列 | 郵務室 |

同一棟大樓裡不同房間的門牌號，讓多個服務可以共存在一台機器上。

### Docker 的 Port Mapping

在 Docker 裡，容器有自己的網路空間。要讓外部存取容器內的服務，需要做 **Port Mapping（端口映射）**：

```bash
docker run -p 8080:80 nginx:stable-alpine
#              ↑    ↑
#          主機Port  容器Port
```

- `80` = 容器裡 Nginx 預設服務埠
- `8080` = 本機對外提供服務的埠

映射之後，在瀏覽器打開 `http://localhost:8080` 就會連到容器內的 Nginx。

```
你的瀏覽器 → localhost:8080 → Docker 幫你轉到 → 容器內的 :80（Nginx）
```

---

## 第四部分 部署 Nginx 網頁伺服器

### Step 1：下載 Nginx 映像檔

去 Docker Hub 找 Nginx 官方 image，選擇 `stable-alpine`（穩定版 + 輕量 Alpine Linux）。

```bash
docker pull nginx:stable-alpine
```

**驗證：**
```bash
docker images | grep nginx
```

**預期輸出：**
```
nginx   stable-alpine   0d3b80406a13   92.5MB
```

### Step 2：前景模式執行

```bash
docker run --name my-nginx -p 8080:80 nginx:stable-alpine
```

| 參數 | 說明 |
|------|------|
| `--name my-nginx` | 容器名稱 |
| `-p 8080:80` | 主機 8080 → 容器 80 |
| `nginx:stable-alpine` | 使用的映像檔 |

你會看到 Nginx 的 log 直接印在 Terminal：

```
...start worker process 30
...start worker process 31
```

打開另一個 Terminal 測試：

```bash
curl http://localhost:8080
```

**預期輸出：**
```html
<h1>Welcome to nginx!</h1>
```

回到原本的 Terminal 按 `Ctrl + C` 停止。

> **前景模式**：容器的 log 會佔據你的 Terminal，關掉 Terminal 容器也會停止。適合除錯時使用。

### Step 3：背景模式執行（推薦）

```bash
# 先刪除剛才的容器
docker rm my-nginx

# 用 -d 背景模式執行
docker run -d --name my-nginx -p 8080:80 nginx:stable-alpine
```

| 參數 | 說明 |
|------|------|
| `-d` | detach，背景執行 |

**預期輸出：**
```
ae27bd93df58fc80f1a759d2dc0f49b844c4be947df3b3bfc92ffadff55e452e
```

回傳的是容器 ID，代表容器已在背景執行。

**驗證：**
```bash
docker ps
```

```
NAMES      IMAGE                 STATUS                  PORTS
my-nginx   nginx:stable-alpine   Up Less than a second   0.0.0.0:8080->80/tcp
```

```bash
curl http://localhost:8080
```

看到 `Welcome to nginx!` 就成功了。

### Step 4：用完即刪模式

```bash
# 先停止並刪除之前的
docker rm -f my-nginx

# 用 --rm 執行
docker run -d --rm --name my-nginx -p 8080:80 nginx:stable-alpine
```

| 參數 | 說明 |
|------|------|
| `--rm` | 容器停止後自動刪除 |

```bash
# 停止容器
docker stop my-nginx

# 確認容器已自動刪除
docker ps -a | grep nginx
```

**預期輸出**：（空，沒有任何結果 → 容器已被自動刪除）

### 三種模式比較

| 模式 | 指令 | 適用場景 |
|------|------|---------|
| 前景 | `docker run` | 除錯、看即時 log |
| 背景 | `docker run -d` | 正式運行 |
| 用完即刪 | `docker run -d --rm` | 測試、臨時任務 |

---

## 第五部分 容器管理指令

### 容器生命週期

```
pull → run → (running) → stop → (stopped) → rm
                ↑                    │
                └── start ───────────┘
```

### 完整操作流程

```bash
# 1. 啟動容器
docker run -d --name my-nginx -p 8080:80 nginx:stable-alpine

# 2. 確認運行中
docker ps

# 3. 查看 log
docker logs my-nginx

# 4. 停止容器
docker stop my-nginx

# 5. 確認已停止（在 ps -a 裡）
docker ps -a

# 6. 重新啟動（不用重新 run）
docker start my-nginx

# 7. 確認又在運行了
docker ps

# 8. 停止並刪除
docker stop my-nginx
docker rm my-nginx

# 9. 確認已刪除
docker ps -a
```

### stop vs rm vs rmi

| 指令 | 作用 | 類比 |
|------|------|------|
| `docker stop` | 停止容器（可以再 start） | 關機但電腦還在 |
| `docker rm` | 刪除容器（永久移除） | 把電腦搬走 |
| `docker rmi` | 刪除映像檔（安裝包） | 把安裝光碟丟掉 |

---

## 第六部分 Restart 重啟策略

當容器意外崩潰（crash）或 Docker Daemon 重啟時，Docker 可以自動幫你重啟容器。

### 四種策略

| 策略 | 說明 |
|------|------|
| `no`（預設） | 不會自動重啟容器 |
| `always` | 無論容器是正常結束還是非正常結束，都會自動重啟。即便是手動 `docker stop`，在 Docker Daemon 重啟後容器也會再跑起來 |
| `unless-stopped`（最常用） | 只要容器是「被動」停止（錯誤、崩潰、Daemon 重啟），都會再自動跑起來。但如果你手動停掉（`docker stop`），那 Docker 不會再幫你自動重啟 |
| `on-failure` | 只有當容器因「非 0 的退出碼」失敗退出時才會重啟，可以搭配重試次數 |

### 使用方式

```bash
# docker run 指定 restart 策略
docker run -d --name my-nginx --restart unless-stopped -p 8080:80 nginx:stable-alpine

# 查看容器的 restart 策略
docker inspect my-nginx --format '{{.HostConfig.RestartPolicy.Name}}'
```

**預期輸出：**
```
unless-stopped
```

```bash
# on-failure 搭配最多重試 5 次
docker run -d --name my-app --restart on-failure:5 my-app-image
```

### 怎麼選

| 情境 | 建議策略 |
|------|---------|
| 資料庫、Web Server（長期運行） | `unless-stopped` |
| 一次性任務（跑完就結束） | `no` |
| 不穩定的服務（偶爾會 crash） | `on-failure:5` |
| 關鍵服務（絕對不能停） | `always` |

```bash
# 清理
docker rm -f my-nginx
```

---

## 第七部分 Docker Compose 介紹

### 為什麼需要 Docker Compose

用 `docker run` 啟動一個容器時，可能要帶很多參數：

```bash
docker run -d --name my-nginx --restart unless-stopped -p 8080:80 -v ./html:/usr/share/nginx/html nginx:stable-alpine
```

一個容器就要這麼長一串，如果有 4-5 個服務（Web + DB + Cache + Worker），每個都要手動打一遍，不容易管理和編輯。

### Docker Compose 是什麼

Docker Compose 是能夠管理多個 containers 的工具：

- 將每個服務的各種參數都寫在一份 **YAML 檔**裡
- 一個指令就能啟動/停止所有服務
- 可以做到更彈性的設定（網路、依賴、volumes）

### 確認 Compose 可用

安裝好 Docker Engine 就自帶 Docker Compose（Plugin 版），不用另外安裝。

```bash
docker compose version
```

**預期輸出：**
```
Docker Compose version v5.2.0
```

### docker-compose.yaml 檔案結構

```yaml
services:
  服務名稱:
    image: 使用的映像檔
    container_name: 容器名稱
    ports:
      - "主機Port:容器Port"
    volumes:
      - 本機路徑:容器路徑
    restart: 重啟策略
    environment:
      - 環境變數=值
    depends_on:
      - 依賴的其他服務
```

---

## 第八部分 Compose 部署 Nginx

### Step 1：Clone nginx-demo 專案

```bash
cd ~
git clone https://github.com/DataEngCamp/nginx-demo.git
cd nginx-demo
```

**預期輸出：**
```
Cloning into 'nginx-demo'...
remote: Enumerating objects: 6, done.
...
Receiving objects: 100% (6/6), done.
```

### Step 2：查看專案結構

```bash
ls -la
```

```
docker-compose.yaml    ← Compose 設定檔
html/                  ← 網頁檔案
  └── index.html       ← 首頁
README.md
```

### Step 3：閱讀 docker-compose.yaml

```bash
cat docker-compose.yaml
```

```yaml
services:
  nginx:
    image: nginx:stable-alpine
    container_name: my-nginx
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    restart: unless-stopped
```

逐行解析：

| 設定 | 說明 |
|------|------|
| `image: nginx:stable-alpine` | 使用 Nginx 穩定版 Alpine 映像檔 |
| `container_name: my-nginx` | 容器叫做 `my-nginx` |
| `ports: "8080:80"` | 主機 8080 → 容器 80 |
| `volumes: ./html:...` | 把本地 `html/` 資料夾掛載到容器的網頁根目錄 |
| `restart: unless-stopped` | 崩潰自動重啟，手動停不重啟 |

### Step 4：前景模式啟動

```bash
docker compose up
```

會看到 Nginx 的 log 直接印出來，按 `Ctrl + C` 停止。

### Step 5：背景模式啟動（推薦）

```bash
docker compose up -d
```

**預期輸出：**
```
 ✔ Network nginx-demo_default  Created
 ✔ Container my-nginx          Started
```

### Step 6：驗證服務

```bash
# 查看容器狀態
docker compose ps
```

**預期輸出：**
```
NAME       IMAGE                 STATUS   PORTS
my-nginx   nginx:stable-alpine   Up ...   0.0.0.0:8080->80/tcp
```

```bash
# 測試網頁
curl http://localhost:8080
```

**預期輸出：**
```html
<h1>It works! 🚀</h1>
```

看到的是 `html/index.html` 的內容（不是 Nginx 預設頁面），因為 volumes 掛載了本地檔案。

### Step 7：查看 Log

```bash
# 查看 log
docker compose logs

# 持續追蹤 log
docker compose logs -f
```

按 `Ctrl + C` 停止追蹤。

### Step 8：停止服務

```bash
docker compose stop
```

**預期輸出：**
```
 ✔ Container my-nginx  Stopped
```

容器停止但還存在，可以用 `docker compose start` 重新啟動。

```bash
# 確認容器還在（已停止）
docker compose ps -a
```

```
NAME       IMAGE                 STATUS
my-nginx   nginx:stable-alpine   Exited (0) ...
```

### Step 9：停止並刪除

```bash
# 先重新啟動
docker compose start

# 停止 + 刪除容器 + 刪除網路
docker compose down
```

**預期輸出：**
```
 ✔ Container my-nginx          Stopped
 ✔ Container my-nginx          Removed
 ✔ Network nginx-demo_default  Removed
```

```bash
# 確認已清除
docker compose ps -a
```

**預期輸出**：（空，沒有容器）

### down 的不同選項

| 指令 | 刪除什麼 |
|------|---------|
| `docker compose down` | 容器 + 網路 |
| `docker compose down -v` | 容器 + 網路 + volumes |
| `docker compose down --rmi all` | 容器 + 網路 + 映像檔 |

> **注意**：`down -v` 會刪除資料，如果你的 volumes 裡有資料庫資料，執行前要確認。

### Step 10：其他實用指令

**啟動指定服務：**
```bash
docker compose up -d nginx
```

**用 -f 指定 compose 檔案（不在專案目錄時）：**
```bash
docker compose -f ~/nginx-demo/docker-compose.yaml up -d
```

**重新建立容器（改了設定檔之後）：**
```bash
docker compose up -d --force-recreate
```

---

## 第九部分 Docker vs Docker Compose 比較

### 功能對比

| | Docker（docker） | Docker Compose（docker compose） |
|------|-----------------|-------------------------------|
| **管理對象** | 單一容器 | 多個容器（整個應用） |
| **設定方式** | 指令參數 | YAML 檔案 |
| **適用場景** | 快速測試、單一服務 | 多服務應用、開發環境 |
| **啟動** | `docker run ...` | `docker compose up -d` |
| **停止** | `docker stop <name>` | `docker compose stop` |
| **刪除** | `docker rm <name>` | `docker compose down` |
| **查看** | `docker ps` | `docker compose ps` |
| **Log** | `docker logs <name>` | `docker compose logs` |

### 指令對照

啟動一個 Nginx，兩種方式寫法比較：

**Docker（手動打一長串）：**
```bash
docker run -d \
  --name my-nginx \
  -p 8080:80 \
  -v ./html:/usr/share/nginx/html \
  --restart unless-stopped \
  nginx:stable-alpine
```

**Docker Compose（寫進 YAML，一鍵啟動）：**
```yaml
# docker-compose.yaml
services:
  nginx:
    image: nginx:stable-alpine
    container_name: my-nginx
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    restart: unless-stopped
```

```bash
docker compose up -d
```

### 什麼時候用哪個

| 情境 | 用什麼 |
|------|--------|
| 快速跑一個容器測試 | `docker run` |
| 跑多個服務（Web + DB + Cache） | `docker compose` |
| 團隊共用同一套環境 | `docker compose`（YAML 放進 Git） |
| CI/CD 自動化 | `docker compose` |

> **結論**：學會 `docker run` 理解原理，日常開發用 `docker compose`。

---

## 指令速查表

### Docker 指令

| 指令 | 說明 |
|------|------|
| `sudo service docker start` | 啟動 Docker Daemon |
| `sudo service docker status` | 查看 Daemon 狀態 |
| `sudo service docker restart` | 重啟 Daemon |
| `docker pull <image>` | 下載映像檔 |
| `docker images` | 查看本地映像檔 |
| `docker rmi <image>` | 刪除映像檔 |
| `docker run -d --name <name> -p <host>:<container> <image>` | 背景執行容器 |
| `docker run --rm ...` | 執行容器，停止後自動刪除 |
| `docker ps` | 查看運行中的容器 |
| `docker ps -a` | 查看所有容器 |
| `docker logs <name>` | 查看容器 log |
| `docker logs -f <name>` | 持續追蹤 log |
| `docker stop <name>` | 停止容器 |
| `docker start <name>` | 啟動已停止的容器 |
| `docker rm <name>` | 刪除已停止的容器 |
| `docker rm -f <name>` | 強制刪除容器 |
| `docker inspect <name>` | 查看容器詳細資訊 |

### Docker Compose 指令

| 指令 | 說明 |
|------|------|
| `docker compose up` | 前景啟動所有服務 |
| `docker compose up -d` | 背景啟動所有服務 |
| `docker compose up -d <service>` | 背景啟動指定服務 |
| `docker compose -f <file> up -d` | 指定 compose 檔案啟動 |
| `docker compose ps` | 查看服務狀態 |
| `docker compose ps -a` | 查看所有服務（含已停止） |
| `docker compose logs` | 查看所有服務 log |
| `docker compose logs -f` | 持續追蹤 log |
| `docker compose logs <service>` | 查看指定服務 log |
| `docker compose stop` | 停止所有服務 |
| `docker compose start` | 啟動已停止的服務 |
| `docker compose down` | 停止 + 刪除容器和網路 |
| `docker compose down -v` | 停止 + 刪除容器、網路和 volumes |
| `docker compose up -d --force-recreate` | 重新建立容器 |

---

## FAQ

### Q1：docker compose up 後服務沒有啟動

```bash
# 1. 看狀態
docker compose ps
# 看哪個服務的 STATE 不是 running

# 2. 看 log
docker compose logs <service_name>
# 找出錯誤訊息

# 3. 驗證 yml 語法
docker compose config
```

### Q2：yml 格式錯誤

YAML 對縮排非常敏感：
- 用空格，不要用 Tab
- 同層級要對齊
- `:` 後面要有空格

### Q3：services 之間連不到彼此

```bash
# 確認 network 設定正確
docker network ls

# 如果用拆開版，確認都加入同一個 network
# 每個 compose file 都要有：
# networks:
#   my_network:
#     external: true

# 確認 network 已建好
docker network create my_network
```

### Q4：volumes 的資料不見了

```bash
# docker compose down 預設不會刪 volume
# 但如果加了 -v 就會刪！
docker compose down      # ✅ volume 保留
docker compose down -v   # ❌ volume 也刪掉
```

### Q5：depends_on 不保證服務「準備好」

`depends_on` 只保證啟動順序，不保證服務完全就緒。MySQL 可能已啟動但還在初始化。

**解法**：設定 `healthcheck` 或在應用程式裡加重試邏輯。

### Q6：怎麼只重建某個服務的 image

```bash
docker compose build <service_name>
docker compose up -d <service_name>

# 或一步完成
docker compose up -d --build <service_name>
```

### Q7：docker compose logs 太多怎麼過濾

```bash
# 只看最後 100 行
docker compose logs --tail 100 <service>

# 即時追蹤
docker compose logs -f <service>

# 加時間戳
docker compose logs -t <service>
```

### Q8：怎麼更新 docker-compose.yml 後套用

```bash
# 修改 yml 後，重新啟動
docker compose up -d
# Docker Compose 會自動偵測變更，只重建有改動的服務
```

### Q9：多個 compose file 怎麼整合

```bash
# 方法 1：逐個啟動（拆開版）
docker compose -f docker-compose-broker.yml up -d
docker compose -f docker-compose-mysql.yml up -d

# 方法 2：合併指定
docker compose -f docker-compose-broker.yml -f docker-compose-mysql.yml up -d
```

### Q10：docker compose 排錯 SOP

```
Step 1: docker compose ps           → 哪個服務掛了？
Step 2: docker compose logs <svc>   → 看 log 找錯誤訊息
Step 3: docker network ls           → network 建好了嗎？
Step 4: docker compose config       → yml 語法正確嗎？
Step 5: docker compose down && docker compose up -d → 砍掉重練
```
