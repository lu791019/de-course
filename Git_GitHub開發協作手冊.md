# Git + GitHub 開發協作手冊

> 本手冊涵蓋 Git 日常開發操作與 GitHub 團隊協作流程。
> 每個操作同時提供 **Terminal 指令** 和 **VS Code GUI** 兩種方式。

## 目錄

- [第一部分 Git 基礎觀念](#第一部分-git-基礎觀念)
- [第二部分 Git 基本操作](#第二部分-git-基本操作)
- [第三部分 VS Code Git 操作](#第三部分-vs-code-git-操作)
- [第四部分 Branch 分支開發](#第四部分-branch-分支開發)
- [第五部分 GitHub PR 流程](#第五部分-github-pr-流程)
- [第六部分 完整開發流程](#第六部分-完整開發流程)
- [第七部分 團隊協作](#第七部分-團隊協作)
- [指令速查表](#指令速查表)

---

## 第一部分 Git 基礎觀念

### Git 是什麼

Git 是**版本控制系統**，幫你：

- **記錄每次修改**：誰在什麼時候改了什麼
- **隨時回到過去**：改壞了可以回到之前的版本
- **多人同時開發**：每個人在自己的分支上工作，不互相干擾

### Git vs GitHub

| | Git | GitHub |
|------|-----|--------|
| **是什麼** | 版本控制工具（裝在你電腦上） | 雲端平台（存放 Git 專案的網站） |
| **在哪裡** | 本地電腦 | 網路上（github.com） |
| **功能** | 追蹤修改、建分支、合併 | 存放程式碼、Code Review、PR |

### Git 的三個區域

```
工作目錄              暫存區              本地倉庫             遠端倉庫
(Working Dir)    (Staging Area)    (Local Repo)       (Remote/GitHub)
    │                 │                 │                    │
    │── git add ─────→│                 │                    │
    │                 │── git commit ──→│                    │
    │                 │                 │── git push ───────→│
    │←─────────────────────────────────────── git pull ──────│
```

- **工作目錄**：你正在編輯的檔案
- **暫存區**：準備要 commit 的檔案（用 `git add` 放進去）
- **本地倉庫**：已經 commit 的歷史紀錄
- **遠端倉庫**：GitHub 上的備份

---

## 第二部分 Git 基本操作

### 設定 Git（首次使用）

```bash
git config --global user.name "你的名字"
git config --global user.email "你的email"
```

### git init — 初始化專案

```bash
mkdir my-project
cd my-project
git init
```

**預期輸出：**
```
Initialized empty Git repository in /home/user/my-project/.git/
```

### git status — 查看狀態

```bash
git status
```

**預期輸出（有修改時）：**
```
Changes not staged for commit:
  modified:   ptt_crawler.py

Untracked files:
  .gitignore
```

- `modified`：已修改但還沒加入暫存區
- `Untracked`：新檔案，Git 還不認識

### git add — 加入暫存區

```bash
# 加入指定檔案
git add ptt_crawler.py

# 加入所有修改過的檔案
git add .
```

### git commit — 提交變更

```bash
git commit -m "create ptt_crawler.py"
```

**預期輸出：**
```
[main abc1234] create ptt_crawler.py
 1 file changed, 50 insertions(+)
 create mode 100644 ptt_crawler.py
```

> **提交訊息**要簡潔描述「做了什麼」，讓隊友一看就懂。

### git log — 查看歷史

```bash
# 完整 log
git log

# 精簡 log（一行一筆）
git log --oneline
```

**預期輸出（--oneline）：**
```
abc1234 create ptt_crawler.py
def5678 my first commit
```

### git diff — 查看差異

```bash
# 查看工作目錄 vs 暫存區的差異
git diff

# 查看暫存區 vs 上次 commit 的差異
git diff --staged
```

---

## 第三部分 VS Code Git 操作

VS Code 內建 Git 支援，左側邊欄的 **Source Control**（第三個圖示）可以做所有 Git 操作。

### 查看修改狀態

打開 Source Control 面板，會看到：

- **Changes**：已修改的檔案
- 檔案後面的標記：`M`（Modified）、`U`（Untracked）、`A`（Added）

### Stage（加入暫存區）

- 點擊檔案旁的 **+** 按鈕 → 該檔案移到 **Staged Changes**
- 等同於 `git add <file>`

### Commit（提交）

1. 在上方訊息欄輸入 commit message（例：`add uv init files`）
2. 點擊 **✓ Commit** 按鈕
3. 等同於 `git commit -m "add uv init files"`

### Push（推送到 GitHub）

1. 點擊 Source Control 面板右上角的 **⋯** 選單
2. 選擇 **Push**
3. 如果分支還沒有遠端追蹤，VS Code 會問「Would you like to publish this branch?」→ 點 **OK**

### Pull（拉取最新）

1. 點擊 Source Control 面板右上角的 **⋯** 選單
2. 選擇 **Pull**
3. 等同於 `git pull`

---

## 第四部分 Branch 分支開發

### 為什麼需要 Branch

在 `main` 分支上直接開發有風險——如果改壞了，所有人都受影響。

**Branch 的概念**：從 main 複製一份出來，在自己的分支上開發，測試通過後再合併回 main。

```
main:          ●───●───●──────────────●  (Merge PR)
                        \            /
feature/xxx:             ●───●───●
                         (在分支上開發)
```

### 分支命名規則

| 類型 | 命名 | 範例 |
|------|------|------|
| 新功能 | `feature/功能名稱` | `feature/init_uv` |
| 修 bug | `fix/問題描述` | `fix/login_error` |
| 重構 | `refactor/說明` | `refactor/api_cleanup` |

### Terminal 操作

**建立並切換到新分支：**

```bash
# 方法一：兩步驟
git branch feature/init_uv
git checkout feature/init_uv

# 方法二：一步完成（推薦）
git checkout -b feature/init_uv
```

**預期輸出：**
```
Switched to a new branch 'feature/init_uv'
```

**查看所有分支：**

```bash
git branch
```

**預期輸出：**
```
* feature/init_uv
  main
```

`*` 標記代表你目前所在的分支。

**切換分支：**

```bash
git checkout main
```

### VS Code 操作

**建立新分支：**

1. 點擊左下角的分支名稱（例：`main`）
2. 在彈出的選單中選擇 **+ Create new branch...**
3. 輸入分支名稱（例：`feature/init_uv`）
4. 按 Enter → 自動切換到新分支

> 左下角會顯示當前分支名稱，可以隨時確認你在哪個分支。

**切換分支：**

1. 點擊左下角的分支名稱
2. 從選單中選擇要切換的分支（例：`main`）

---

## 第五部分 GitHub PR 流程

### 什麼是 Pull Request（PR）

PR 是 GitHub 上的「合併請求」，流程：

1. 你在分支上開發完成
2. 推送到 GitHub
3. 在 GitHub 上建立 PR：「請把我的分支合併到 main」
4. 團隊成員 Review 你的程式碼
5. 通過後 Merge 到 main

### 完整 PR 流程

#### Step 1：在分支上開發和 commit

```bash
# 確認在 feature 分支
git branch

# 修改程式碼後...
git add .
git commit -m "add uv init files"
```

**VS Code**：在 Source Control 面板 Stage + Commit。

#### Step 2：推送分支到 GitHub

```bash
git push -u origin feature/init_uv
```

| 參數 | 說明 |
|------|------|
| `-u` | 設定遠端追蹤（第一次 push 才需要） |
| `origin` | 遠端倉庫名稱 |
| `feature/init_uv` | 要推送的分支 |

**VS Code**：點 **⋯** → **Push**。第一次推送新分支時，VS Code 會彈出對話框「The branch has no remote branch. Would you like to publish this branch?」→ 點 **OK**。

#### Step 3：在 GitHub 建立 PR

推送後，到 GitHub repo 頁面會看到黃色提示：

```
feature/init_uv had recent pushes — [Compare & pull request]
```

點擊 **Compare & pull request**，進入 PR 建立頁面：

1. 確認方向：`base: main` ← `compare: feature/init_uv`
2. **Title**：簡短描述（例：`add uv init files`）
3. **Description**：詳細說明改了什麼
   ```
   - 使用 uv 管理專案
   - 新增 uv init 的初始檔案
   - 新增套件 requests, beautifulsoup4, pandas
   ```
4. 點擊 **Create pull request**

#### Step 4：Review & Merge

團隊成員（或自己）在 PR 頁面：

1. 查看 **Files changed** 確認程式碼
2. 確認沒有 conflict（「No conflicts with base branch」）
3. 可以留下 comment
4. 點擊 **Merge pull request** → **Confirm merge**

Merge 完成後會看到：

```
Pull request successfully merged and closed
The feature/init_uv branch can be safely deleted.
```

點擊 **Delete branch** 清理已合併的分支。

#### Step 5：切回 main 並拉取最新

**Terminal：**

```bash
# 切回 main
git checkout main

# 拉取合併後的最新程式碼
git pull
```

**VS Code：**

1. 點左下角分支名稱 → 選 `main`
2. 點 **⋯** → **Pull**

### Git Graph 查看分支歷史

安裝 VS Code 擴充套件 **Git Graph**：

1. 在 Source Control 面板上方點擊 **View Git Graph (git log)** 圖示
2. 可以看到完整的分支圖：
   - 每個點是一個 commit
   - 分支的分叉和合併一目了然
   - 不同顏色代表不同分支

---

## 第六部分 完整開發流程

### 一個功能的完整生命週期

```
1. 建分支 → 2. 開發 → 3. Commit → 4. Push → 5. PR → 6. Review → 7. Merge → 8. 回 main
```

### Terminal 版（完整指令）

```bash
# 1. 確認在 main 且是最新的
git checkout main
git pull

# 2. 建立功能分支
git checkout -b feature/init_uv

# 3. 開發...（修改程式碼）

# 4. 查看修改了什麼
git status
git diff

# 5. 加入暫存區
git add .

# 6. 提交
git commit -m "add uv init files"

# 7. 推送到 GitHub
git push -u origin feature/init_uv

# 8. 到 GitHub 建立 PR → Review → Merge

# 9. 合併完成後，切回 main
git checkout main

# 10. 拉取最新（包含剛合併的程式碼）
git pull

# 11. 刪除本地已合併的分支（選做）
git branch -d feature/init_uv
```

### VS Code 版（GUI 操作）

| 步驟 | 操作 |
|------|------|
| 1. 切到 main | 點左下角分支名 → 選 `main` |
| 2. 拉取最新 | **⋯** → **Pull** |
| 3. 建新分支 | 點左下角分支名 → **+ Create new branch** → 輸入名稱 |
| 4. 開發 | 正常寫程式 |
| 5. Stage | Source Control → 點 **+** 加入暫存 |
| 6. Commit | 輸入訊息 → **✓ Commit** |
| 7. Push | **⋯** → **Push**（首次推送點 OK 發布分支） |
| 8. 建 PR | 到 GitHub 網站操作 |
| 9. 切回 main | 點左下角 → 選 `main` |
| 10. Pull | **⋯** → **Pull** |

---

## 第七部分 團隊協作

### Fork 工作流程

如果你要貢獻程式碼到**別人的專案**（你沒有直接 push 的權限），需要使用 Fork：

```
原始 repo (DataEngCamp/test)
    │
    │── Fork ──→ 你的 repo (你的帳號/test)
    │                 │
    │                 │── Clone ──→ 本地開發
    │                 │
    │                 │←── Push
    │                 │
    │←── Pull Request ─┘
```

**步驟：**

1. **Fork**：在 GitHub 上點原始 repo 的 Fork 按鈕，複製一份到你的帳號下
2. **Clone**：把你 Fork 的 repo clone 到本地

```bash
git clone https://github.com/你的帳號/test.git
cd test
```

3. **開發**：建分支 → 修改 → commit → push（推到你自己的 repo）

```bash
git checkout -b feature/my_change
# 修改程式碼...
git add .
git commit -m "add my change"
git push -u origin feature/my_change
```

4. **Pull Request**：在 GitHub 上從你的 repo 建立 PR 到原始 repo

### 同步原始 repo 的更新

原始 repo 可能在你 Fork 之後有新的更新，需要同步：

```bash
# 加入原始 repo 為 upstream
git remote add upstream https://github.com/DataEngCamp/test.git

# 拉取原始 repo 的更新
git fetch upstream

# 合併到你的 main
git checkout main
git merge upstream/main

# 推到你的 GitHub
git push
```

### 團隊開發守則

| 規則 | 說明 |
|------|------|
| 不要直接在 main 上開發 | 一律開分支 |
| 分支命名有意義 | `feature/xxx`、`fix/xxx` |
| commit 訊息要清楚 | 讓隊友看得懂你做了什麼 |
| Push 前先 Pull | 避免 conflict |
| PR 要寫描述 | 說明改了什麼、為什麼改 |
| Merge 後刪分支 | 保持 repo 整潔 |

---

## 指令速查表

### Git 基礎

| 指令 | 說明 |
|------|------|
| `git init` | 初始化 Git 專案 |
| `git clone <url>` | Clone 遠端 repo 到本地 |
| `git status` | 查看工作目錄狀態 |
| `git add <file>` | 加入暫存區 |
| `git add .` | 加入所有修改 |
| `git commit -m "msg"` | 提交變更 |
| `git log --oneline` | 查看精簡歷史 |
| `git diff` | 查看修改差異 |

### 分支操作

| 指令 | 說明 |
|------|------|
| `git branch` | 列出所有分支 |
| `git branch <name>` | 建立新分支 |
| `git checkout <name>` | 切換分支 |
| `git checkout -b <name>` | 建立並切換分支 |
| `git branch -d <name>` | 刪除已合併的分支 |
| `git merge <name>` | 合併指定分支到當前分支 |

### 遠端操作

| 指令 | 說明 |
|------|------|
| `git remote -v` | 查看遠端倉庫 |
| `git push -u origin <branch>` | 推送分支到遠端（首次） |
| `git push` | 推送到遠端（已追蹤） |
| `git pull` | 拉取遠端最新 |
| `git fetch` | 下載遠端資料（不合併） |

### VS Code 操作對照

| 操作 | Terminal | VS Code |
|------|----------|---------|
| 查看狀態 | `git status` | Source Control 面板 |
| 加入暫存 | `git add <file>` | 點 **+** 按鈕 |
| 提交 | `git commit -m "msg"` | 輸入訊息 → **✓ Commit** |
| 推送 | `git push` | **⋯** → **Push** |
| 拉取 | `git pull` | **⋯** → **Pull** |
| 建分支 | `git checkout -b <name>` | 左下角 → **+ Create new branch** |
| 切分支 | `git checkout <name>` | 左下角 → 選分支名 |
| 查看歷史 | `git log` | Git Graph 擴充套件 |
