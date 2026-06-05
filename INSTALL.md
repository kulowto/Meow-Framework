---
文件類型: H_READ
版本: v1.0
建立: 2026-06-05
最後更新: 2026-06-05
適用對象: 新機器環境建置（AI 執行 / 人工操作皆適用）
---

# Meow-Env 環境安裝指南

本文件說明如何在新機器上完整架設 Meow-Env 開發環境。  
適用情境：換機、雙機同步、新成員加入，或 AI 代為執行環境建置。

### 文件閱讀說明

本文件包含兩種區塊，請根據以下標記判斷操作方式：

| 標記 | 意義 |
|------|------|
| **【AI 執行】** | AI 可直接在終端機執行的指令，無需人工確認 |
| **【人工操作】** | 需要登入帳號、輸入密碼、或做出人工判斷的步驟，AI 應提示使用者手動完成 |
| 無標記 | 說明文字，供參考理解，不需執行 |

---

## 一、系統概覽

Meow-Env 由五個 Git Repository 組成，各自獨立追蹤，統一放在同一個根目錄下：

| 子系統 | 縮寫 | GitHub Repository | 職責 |
|--------|------|-------------------|------|
| Meow-Framework | MF | `https://github.com/kulowto/Meow-Framework.git` | 通用框架，協作規則與模板的來源 |
| Meow-Agent | MA | `https://github.com/kulowto/Meow-agent.git` | 指揮官 / 個人秘書，Claude Code 主要啟動點 |
| Meow-Wiki | MW | `https://github.com/kulowto/Meow-Wiki.git` | 個人知識庫，已確認完成的知識存放處 |
| Meow-Tools | MT | `https://github.com/kulowto/Meow-tools.git` | 個人工具箱，工具與自動化流程 |
| Meow-Dev | MD | `https://github.com/kulowto/Meow-dev.git` | 開發暫存區（Playground），跨裝置同步 |

---

## 二、前置需求

在開始安裝前，確認以下工具已安裝並可正常執行：

### 通用（Mac / Windows 皆需）

| 工具 | 版本要求 | 確認指令 |
|------|---------|---------|
| Git | 任意近期版本 | `git --version` |
| Python | Mac：`python3`；Windows：`python` | `python3 --version` 或 `python --version` |
| Claude Code | 最新版 | `claude --version` |
| Node.js | 建議 v18+ | `node --version` |

### Mac 專屬

| 工具 | 安裝方式 | 用途 |
|------|---------|------|
| Homebrew | `/bin/bash -c "$(curl -fsSL https://brew.sh/install.sh)"` | 套件管理 |
| CotEditor | `brew install --cask coteditor` | Claude Code 外部編輯器（Ctrl+G 呼出） |
| trash-cli（npm） | `npm install -g trash-cli` | 安全刪除（替代 rm） |

### Windows 專屬

| 工具 | 安裝方式 | 用途 |
|------|---------|------|
| winget | 系統內建（Windows 10+） | 套件管理 |
| VS Code | `winget install Microsoft.VisualStudioCode` | Claude Code 外部編輯器（Ctrl+G 呼出） |
| jq | `winget install jqlang.jq` | Claude Code Status Line 解析 JSON |

---

## 三、目錄結構建立

> **規範**：根目錄路徑在 Mac 為 `~/Meow-Env`，Windows 為 `D:\Meow-Env`。  
> 各 CLAUDE.md 內的路徑以 Windows 為主，AI 在 Mac 上操作時需自行換算。

---

### 【AI 執行】Mac — 建立根目錄

```bash
mkdir -p ~/Meow-Env
```

---

### 【AI 執行】Windows — 建立根目錄（PowerShell）

```powershell
New-Item -ItemType Directory -Path "D:\Meow-Env" -Force
```

---

## 四、Clone 所有 Repository

五個 repo 需依序 clone 至根目錄下。若 repo 已存在，改執行 `git pull` 更新。

Clone 完成後，目錄結構應如下：

```
Meow-Env/
├── Meow-Framework/    ← MF：通用框架
├── Meow-agent/        ← MA：個人秘書
├── Meow-Wiki/         ← MW：知識庫
├── Meow-tools/        ← MT：工具箱
└── Meow-dev/          ← MD：開發暫存區
```

---

### 【AI 執行】Mac — Clone / 更新（自動判斷）

> 實測指令（2026-06-05，Mac）：以下腳本在 Meow-Framework 已存在、其餘四個 repo 尚未 clone 的混合狀態下驗證通過。

```bash
cd ~/Meow-Env
for repo in Meow-Framework Meow-agent Meow-Wiki Meow-tools Meow-dev; do
  echo "=== $repo ==="
  if [ -d "$repo" ]; then
    git -C "$repo" pull
  else
    git clone "https://github.com/kulowto/$repo.git"
  fi
done
```

---

### 【AI 執行】Windows — Clone / 更新（自動判斷，PowerShell）

```powershell
cd D:\Meow-Env
foreach ($repo in @("Meow-Framework","Meow-agent","Meow-Wiki","Meow-tools","Meow-dev")) {
  Write-Host "=== $repo ===" -ForegroundColor Cyan
  if (Test-Path $repo) {
    git -C $repo pull
  } else {
    git clone "https://github.com/kulowto/$repo.git"
  }
}
```

---

## 五、Claude Code 設定

### 5.1 確認全域 settings.json

Claude Code 全域設定存放於：

- Mac：`~/.claude/settings.json`
- Windows：`C:\Users\<使用者名稱>\.claude\settings.json`

此檔案已透過 Git 同步（包含於 Meow-agent repo 的 `.claude/` 目錄）。  
確認檔案存在且內容正確即可。

### 5.2 建立平台專屬 settings.local.json

`settings.local.json` **不同步至 Git**，每台機器需獨立建立。

- Mac：`~/.claude/settings.local.json`
- Windows：`C:\Users\<使用者名稱>\.claude\settings.local.json`

建立初始內容（之後依需要逐步補充）：

```json
{
  "permissions": {
    "allow": [],
    "deny": []
  }
}
```

### 5.3 設定外部編輯器

Claude Code 按 `Ctrl+G` 可開啟外部編輯器撰寫長段 prompt。  
需在環境變數或 Claude Code 設定中指定編輯器指令：

| 平台 | 工具 | CLI 旗標 |
|------|------|---------|
| Mac | CotEditor | `cot -w` |
| Windows | VS Code | `code --wait` |

### 5.4 設定 cc alias（啟動捷徑）

#### Mac（加入 `~/.zshrc`）

```bash
alias cc='cd ~/Meow-Env/Meow-agent && CLAUDE_CODE_NO_FLICKER=1 claude'
```

#### Windows（加入 PowerShell Profile）

查看 Profile 路徑：執行 `$PROFILE` 取得路徑，加入以下內容：

```powershell
function cc {
    Set-Location 'D:\Meow-Env\Meow-agent'
    $env:CLAUDE_CODE_NO_FLICKER = '1'
    claude @args
}
```

> 若需要上下文旗標（`cc -ma` 等），請使用 5.5 的完整版本取代此段，兩者不可並存。

設定完成後重新載入 Shell，輸入 `cc` 即可啟動 Claude Code 並進入 Meow-agent 工作目錄。

---

### 5.5 【選配】設定 cc 上下文快速啟動旗標

> **誰需要這個？**  
> 有在使用各子系統 `context.md` 做跨對話進度追蹤的使用者。  
> 不使用 context.md 流程者可跳過，直接進入第六章。

啟用後，`cc` 可帶旗標直接讀取指定子系統的 `context.md`，Claude 啟動後會自動告知上次進度。

| 指令 | 行為 |
|------|------|
| `cc` | 切到 Meow-Agent，直接啟動，不讀任何文件 |
| `cc -ma` | 讀 MA `context.md`，告知上次進度，聚焦 Meow-Agent |
| `cc -mw` | 讀 MW `context.md`，告知上次進度，聚焦 Meow-Wiki |
| `cc -mt` | 讀 MT `context.md`，告知上次進度，聚焦 Meow-Tools |
| `cc -md` | 讀 MD `context.md`，告知上次進度，聚焦 Meow-Dev |

#### 【AI 執行】Mac — 將 `~/.zshrc` 內的 cc 改為 function

將原本的 `alias cc=...` 替換為以下 function（兩者不可並存）：

```bash
function cc() {
  cd ~/Meow-Env/Meow-agent
  case "$1" in
    -ma) CLAUDE_CODE_NO_FLICKER=1 claude "請讀取 context.md 告知上次進度，接下來聚焦 Meow-Agent 任務。" ;;
    -mw) CLAUDE_CODE_NO_FLICKER=1 claude "請讀取 ~/Meow-Env/Meow-Wiki/context.md 告知上次進度，接下來聚焦 Meow-Wiki 任務。" ;;
    -mt) CLAUDE_CODE_NO_FLICKER=1 claude "請讀取 ~/Meow-Env/Meow-tools/context.md 告知上次進度，接下來聚焦 Meow-Tools 任務。" ;;
    -md) CLAUDE_CODE_NO_FLICKER=1 claude "請讀取 ~/Meow-Env/Meow-dev/context.md 告知上次進度，接下來聚焦 Meow-Dev 任務。" ;;
    *)   CLAUDE_CODE_NO_FLICKER=1 claude ;;
  esac
}
```

#### 【AI 執行】Windows — 將 PowerShell Profile 內的 cc 改為以下完整版本

> 已在 Windows 實測驗證（2026-06-05）。  
> 此版本使用 `$args[0]` 接收 positional 參數（非 `param`），為 PowerShell 的正確寫法。  
> 每個旗標均會先確認 `context.md` 是否存在，依結果給出不同提示。

```powershell
function cc {
    Set-Location 'D:\Meow-Env\Meow-agent'
    $env:CLAUDE_CODE_NO_FLICKER = '1'
    $mode = if ($args.Count -gt 0) { $args[0] } else { '' }
    switch ($mode) {
        '-ma' {
            $ctx = 'D:\Meow-Env\Meow-agent\context.md'
            if (Test-Path $ctx) {
                claude "請閱讀 $ctx，摘要告訴我上次做到哪、待續事項，然後等待指示。本次聚焦 Meow-Agent（MA）。"
            } else {
                claude "目前沒有 MA 工作進度記錄，本次聚焦 Meow-Agent，等待指示。"
            }
        }
        '-mw' {
            $ctx = 'D:\Meow-Env\Meow-wiki\context.md'
            if (Test-Path $ctx) {
                claude "請閱讀 $ctx，摘要告訴我上次做到哪、待續事項，然後等待指示。本次聚焦 Meow-Wiki（MW）。"
            } else {
                claude "目前沒有 MW 工作進度記錄，本次聚焦 Meow-Wiki，等待指示。"
            }
        }
        '-mt' {
            $ctx = 'D:\Meow-Env\Meow-tools\context.md'
            if (Test-Path $ctx) {
                claude "請閱讀 $ctx，摘要告訴我上次做到哪、待續事項，然後等待指示。本次聚焦 Meow-Tools（MT）。"
            } else {
                claude "目前沒有 MT 工作進度記錄，本次聚焦 Meow-Tools，等待指示。"
            }
        }
        '-md' {
            $ctx = 'D:\Meow-Env\Meow-Dev\context.md'
            if (Test-Path $ctx) {
                claude "請閱讀 $ctx，摘要告訴我上次做到哪、待續事項，然後等待指示。本次聚焦 Meow-Dev（MD）。"
            } else {
                claude "目前沒有 MD 工作進度記錄，本次聚焦 Meow-Dev，等待指示。"
            }
        }
        default {
            claude @args
        }
    }
}
```

設定完成後重新載入 Shell，旗標即生效。`context.md` 的格式與維護規範見：  
`Meow-Framework/governance/collaboration/(AI_Read) context 維護規範.md`

---

### 5.6 【選配】設定 cs 指令（手動壓縮上下文）

`cs` 是手動觸發的上下文保存指令，用於將本次對話的關鍵進度寫入 `context.md`。  
對應 CLAUDE.md 中 `cs` 的行為定義。

#### 【AI 執行】Mac — 加入 `~/.zshrc`

```bash
function cs() {
  cd ~/Meow-Env/Meow-agent
  CLAUDE_CODE_NO_FLICKER=1 claude "工作進度保存模式：請先執行 git status 查看本次有哪些檔案異動，再閱讀現有的 context.md 了解上次進度，然後詢問我本次完成了什麼、有什麼待續事項，最後更新對應的 context.md，並視情況更新 workingData 的長期文件。"
}
```

#### 【AI 執行】Windows — 加入 PowerShell Profile（已在 Windows 實測驗證）

```powershell
function cs {
    Set-Location 'D:\Meow-Env\Meow-agent'
    $env:CLAUDE_CODE_NO_FLICKER = '1'
    claude "工作進度保存模式：請先執行 git status 查看本次有哪些檔案異動，再閱讀現有的 context.md 了解上次進度，然後詢問我本次完成了什麼、有什麼待續事項，最後更新對應的 context.md，並視情況更新 workingData 的長期文件。"
}
```

---

## 六、Claude Code Skills 與 Plugins 安裝

Meow-Env 使用兩種不同機制安裝工具，**兩者不互通**：

| 類型 | 儲存位置 | Git 同步 | 安裝方式 |
|------|---------|---------|---------|
| **Git-based Skill** | `Meow-agent/.claude/skills/` | ✅ clone repo 後自動取得 | 無需額外安裝 |
| **Marketplace Plugin** | `~/.claude/plugins/cache/` | ❌ 各機器獨立安裝 | 需手動執行安裝指令 |

---

### 6.1 Git-based Skills（clone 後自動取得）

以下 Skills 已進 Git，`git clone` 或 `git pull` 後即可使用，無需額外操作：

| Skill 名稱 | 路徑（相對於 Meow-agent） | 說明 |
|------------|--------------------------|------|
| web-design-engineer | `.claude/skills/web-design-engineer/` | 網頁設計工程 |
| web-video-presentation | `.claude/skills/web-video-presentation/` | 文章轉影片簡報 |

---

### 6.2 Marketplace Plugins（每台機器手動安裝）

以下三個 Plugin **不會隨 Git 同步**，換機後必須重新安裝。

> **注意**：`/install-plugin` 指令在部分版本的 Claude Code 無效。  
> 正確方式是**手動 clone 到固定路徑，再寫入 `installed_plugins.json`**。

#### 已安裝 Plugin 清單

| Plugin 名稱 | Marketplace | Repo | 用途 |
|-------------|------------|------|------|
| `skill-creator` | `claude-plugins-official` | Anthropics 官方 | 自製 Skill 的完整流程工具 |
| `ui-ux-pro-max` | `ui-ux-pro-max-skill` | nextlevelbuilder/ui-ux-pro-max-skill | 配色、字體、設計系統參考 |
| `andrej-karpathy-skills` | `karpathy-skills` | forrestchang/andrej-karpathy-skills | Karpathy 程式碼原則（自動套用）|

---

#### Step 1 — 【AI 執行】Mac：手動 clone Plugin 到 cache 目錄

```bash
# ui-ux-pro-max（版本 2.5.0，鎖定 commit）
mkdir -p ~/.claude/plugins/cache/ui-ux-pro-max-skill/ui-ux-pro-max
git clone https://github.com/nextlevelbuilder/ui-ux-pro-max-skill.git \
  ~/.claude/plugins/cache/ui-ux-pro-max-skill/ui-ux-pro-max/2.5.0
cd ~/.claude/plugins/cache/ui-ux-pro-max-skill/ui-ux-pro-max/2.5.0
git checkout b7e3af80f6e331f6fb456667b82b12cade7c9d35

# andrej-karpathy-skills（版本 1.0.0，鎖定 commit）
mkdir -p ~/.claude/plugins/cache/karpathy-skills/andrej-karpathy-skills
git clone https://github.com/forrestchang/andrej-karpathy-skills.git \
  ~/.claude/plugins/cache/karpathy-skills/andrej-karpathy-skills/1.0.0
cd ~/.claude/plugins/cache/karpathy-skills/andrej-karpathy-skills/1.0.0
git checkout 2c606141936f1eeef17fa3043a72095b4765b9c2
```

#### Step 1 — 【AI 執行】Windows（PowerShell）：手動 clone Plugin 到 cache 目錄

```powershell
# ui-ux-pro-max
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\plugins\cache\ui-ux-pro-max-skill\ui-ux-pro-max"
git clone https://github.com/nextlevelbuilder/ui-ux-pro-max-skill.git `
  "$env:USERPROFILE\.claude\plugins\cache\ui-ux-pro-max-skill\ui-ux-pro-max\2.5.0"
cd "$env:USERPROFILE\.claude\plugins\cache\ui-ux-pro-max-skill\ui-ux-pro-max\2.5.0"
git checkout b7e3af80f6e331f6fb456667b82b12cade7c9d35

# andrej-karpathy-skills
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\plugins\cache\karpathy-skills\andrej-karpathy-skills"
git clone https://github.com/forrestchang/andrej-karpathy-skills.git `
  "$env:USERPROFILE\.claude\plugins\cache\karpathy-skills\andrej-karpathy-skills\1.0.0"
cd "$env:USERPROFILE\.claude\plugins\cache\karpathy-skills\andrej-karpathy-skills\1.0.0"
git checkout 2c606141936f1eeef17fa3043a72095b4765b9c2
```

#### Step 2 — 【AI 執行】寫入 `installed_plugins.json`

> `installPath` 必須使用**絕對路徑**（不可用 `~`）。  
> Mac 執行 `echo $HOME` 取得實際路徑後填入。

**Mac** — 寫入 `~/.claude/plugins/installed_plugins.json`：

```bash
HOME_PATH=$(echo $HOME)
cat > ~/.claude/plugins/installed_plugins.json << EOF
{
  "version": 2,
  "plugins": {
    "ui-ux-pro-max@ui-ux-pro-max-skill": [
      {
        "scope": "user",
        "installPath": "${HOME_PATH}/.claude/plugins/cache/ui-ux-pro-max-skill/ui-ux-pro-max/2.5.0",
        "version": "2.5.0",
        "installedAt": "2026-04-18T13:53:54.034Z",
        "lastUpdated": "2026-04-18T13:53:54.034Z",
        "gitCommitSha": "b7e3af80f6e331f6fb456667b82b12cade7c9d35"
      }
    ],
    "andrej-karpathy-skills@karpathy-skills": [
      {
        "scope": "user",
        "installPath": "${HOME_PATH}/.claude/plugins/cache/karpathy-skills/andrej-karpathy-skills/1.0.0",
        "version": "1.0.0",
        "installedAt": "2026-05-02T09:05:29.034Z",
        "lastUpdated": "2026-05-02T09:05:29.034Z",
        "gitCommitSha": "2c606141936f1eeef17fa3043a72095b4765b9c2"
      }
    ]
  }
}
EOF
```

#### Step 3 — 【AI 執行】更新 `settings.json` 加入 Marketplace 來源與啟用清單

確認 `~/.claude/settings.json` 包含以下兩個區塊（若已有則合併，不可重複）：

```json
"extraKnownMarketplaces": {
  "ui-ux-pro-max-skill": {
    "source": { "source": "git", "url": "https://github.com/nextlevelbuilder/ui-ux-pro-max-skill.git" }
  },
  "karpathy-skills": {
    "source": { "source": "git", "url": "https://github.com/forrestchang/andrej-karpathy-skills.git" }
  }
},
"enabledPlugins": {
  "ui-ux-pro-max@ui-ux-pro-max-skill": true,
  "andrej-karpathy-skills@karpathy-skills": true
}
```

#### Step 4 — 【人工操作】重新啟動 Claude Code

完整重啟後，`/ui-ux-pro-max` 與 `/karpathy-guidelines` 指令應可正常觸發。

---

### 6.3 全域自訂指令（`~/.claude/commands/`）

全域指令放在 `~/.claude/commands/`，每台機器獨立存放，**不隨 Git 同步**。

#### 【AI 執行】Mac — 建立 inquiry.md

```bash
mkdir -p ~/.claude/commands
```

建立 `~/.claude/commands/inquiry.md`，內容如下（Mac 版，路徑與指令已調整）：

````markdown
你現在進入「結構化詢問模式」（Inquiry Model v0.2）。

主題：$ARGUMENTS
（若 $ARGUMENTS 為空，請先問：「你想釐清的主題是什麼？」再繼續）

---

## 你的角色

你是結構化詢問引導者，運用金字塔原理和分而治之框架，協助使用者釐清問題、發現盲點、沉澱洞見。一次只問一個問題，等完整回答後再推進。

---

## 核心框架規則

**金字塔原理（必須遵守）**
- 橫向 MECE：同層問題不重不漏
- 縱向嚴謹：子問題是上層問題的真實支撐
- 整體真知灼見：問題能解決核心，不是形式上拆解

**三驗證框架（追問時優先使用）**
- 定義驗證：「如何有效定義你說的這個核心？」
- 方法驗證：「具體的實現方法是什麼？請提供數值或計畫。」
- 數據驗證：「這個論點背後有數據或案例支持嗎？」

**深度上限：最多 3 層**（原始問題=深度1，追問=深度2，再追問=深度3）

**盲點提醒（explore 模式）**：偵測到使用者未意識到的盲點時，標示 ⚠️ 並說明（Johari Window 盲目我象限）。

**問題追蹤器**：每個問題標記「已解決」或「待續」，主迴圈結束後列出待續清單。

---

## 執行流程

### Step 0：確認模式
詢問：「**explore** 模式（完整探索，含盲點揭露）或 **clean** 模式（只跟著你說的方向走，不討論盲區）？直接 Enter 為 explore。」

### Step 1（可選）：Reality Check
詢問：「要先說明一下你目前的狀況嗎？（背景 / 已試過的 / 上次 AI 的進度）跳過請直接 Enter。」

### Step 2：Decompose
內部分析主題，輸出：
```
【核心主張】一句話

接下來依序詢問三個支柱問題，請盡量具體回答。
```

### Step 3：支柱詢問（對三個支柱逐一進行）

對每個支柱：
```
────────────────────────────────────────────────────
  支柱 N  [問題內容]
────────────────────────────────────────────────────
```
1. 等待使用者回答
2. 評估回答完整度（三驗證角度）
3. 若有盲點（explore 模式）：`⚠️ 盲點：[說明]`
4. 若不完整：`↳ 追問（深度 N）[問題類型]：[追問內容]`
5. 到達深度 3 或已完整：`💡 [洞見]`，進入下一支柱

### Step 4：問題追蹤器
若有待續問題：
```
📋 有 N 個問題尚未完整討論：
   • [支柱 X] [問題]

要繼續處理這些問題嗎？（y / Enter 跳過）
```

### Step 5：彙整輸出
```markdown
## 核心主張
[一句話]

## 三個支柱

### 支柱一：[標題]
[洞見與摘要]

### 支柱二：[標題]
[洞見與摘要]

### 支柱三：[標題]
[洞見與摘要]

## 彙整結論
[段落]

## 盲點提醒
[段落]

## 行動建議
- [條列]

## 待續問題（若有）
- [條列]
```

### Step 6：存檔
先執行 `date '+%Y-%m-%d %H:%M'` 確認當下時間，再用 Write 工具存至：

`~/Meow-Env/Meow-Dev/active/inquiry_sessions/YYYY-MM-DD_主題前20字.md`

檔案開頭格式：
```markdown
# [主題]
討論時間：YYYY-MM-DD HH:MM

[Step 5 完整內容]
```

存完後告知完整路徑。

---

現在開始，從 Step 0 執行。
````

#### 【AI 執行】Windows — 建立 inquiry.md

Windows 版本路徑與日期指令不同，建立 `%USERPROFILE%\.claude\commands\inquiry.md`。  
Step 6 的存檔路徑改為 `D:\Meow-Env\Meow-Dev\active\inquiry_sessions\`，  
日期指令改為 `powershell -Command "Get-Date -Format 'yyyy-MM-dd HH:mm'"`。  
其餘內容與 Mac 版相同。

> 注意：`~/.claude/commands/` 這個目錄不隨 Git 同步，換機後需重新建立。

---

> 安裝任何新 Skill / Plugin 時，必須遵守來源追蹤規範：  
> `Meow-Framework/governance/collaboration/(AI_Read) (Global Prompt) Skill 來源追蹤規範.md`

---

## 七、平台相容性注意事項

### Windows Only（Mac 不可執行）

`Meow-tools/ps_tools/` 目錄下的所有程式碼依賴 **Windows COM 橋接**（photoshop-python-api），Mac 無法執行。

- 識別方式：docstring 第一行有 `[Windows Only]` 標記
- Mac 上請勿執行此目錄下任何 `.py` 腳本
- 閱讀與編輯程式碼本身不受影響

### 跨平台（Mac / Windows 皆可）

| 範圍 | 說明 |
|------|------|
| Meow-Framework/ | 全部可跨平台 |
| Meow-agent/ | 全部可跨平台 |
| Meow-wiki/ | 全部可跨平台 |
| Meow-Dev/ | 全部可跨平台 |
| Meow-tools/（ps_tools 除外） | 可跨平台 |

---

## 八、安裝驗證清單

完成以上步驟後，逐項確認：

**Repo 與設定**
- [ ] 五個 repo 均已 clone 至正確路徑，`git status` 顯示 `nothing to commit`
- [ ] `~/.claude/settings.json` 存在且可讀取，包含 `extraKnownMarketplaces` 三個 marketplace 來源
- [ ] `~/.claude/settings.local.json` 已建立（即使是空框架）
- [ ] 輸入 `cc` 可成功啟動 Claude Code 並切入 Meow-agent 目錄
- [ ] Claude Code 內輸入 `Ctrl+G` 可開啟外部編輯器

**Plugins**
- [ ] `~/.claude/plugins/cache/ui-ux-pro-max-skill/ui-ux-pro-max/2.5.0/` 目錄存在且含 `CLAUDE.md`
- [ ] `~/.claude/plugins/cache/karpathy-skills/andrej-karpathy-skills/1.0.0/` 目錄存在且含 `CLAUDE.md`
- [ ] `~/.claude/plugins/installed_plugins.json` 列出 `ui-ux-pro-max` 與 `andrej-karpathy-skills`（含 installPath 與 gitCommitSha）
- [ ] Claude Code 重啟後，`/ui-ux-pro-max` 與 `/karpathy-guidelines` 可正常觸發

**全域指令**
- [ ] `~/.claude/commands/inquiry.md` 存在
- [ ] Claude Code 內輸入 `/inquiry` 可啟動結構化詢問模式

**Mac 專屬**
- [ ] 執行 `trash --version` 確認 trash-cli 已全域安裝；若無，執行 `npm install -g trash-cli`

---

## 九、日常維護

### 【AI 執行】Mac — 同步所有 repo（Pull）

```bash
cd ~/Meow-Env
for dir in Meow-Framework Meow-agent Meow-Wiki Meow-tools Meow-dev; do
  echo "=== $dir ===" && git -C "$dir" pull
done
```

### 【AI 執行】Windows — 同步所有 repo（Pull，PowerShell）

```powershell
cd D:\Meow-Env
foreach ($dir in @("Meow-Framework","Meow-agent","Meow-Wiki","Meow-tools","Meow-dev")) {
  Write-Host "=== $dir ===" -ForegroundColor Cyan
  git -C $dir pull
}
```

### 【AI 執行】Mac — 檢查各 repo 狀態

```bash
for dir in Meow-Framework Meow-agent Meow-Wiki Meow-tools Meow-dev; do
  echo "=== $dir ===" && git -C ~/Meow-Env/$dir status
done
```

### 【AI 執行】Windows — 檢查各 repo 狀態（PowerShell）

```powershell
foreach ($dir in @("Meow-Framework","Meow-agent","Meow-Wiki","Meow-tools","Meow-dev")) {
  Write-Host "=== $dir ===" -ForegroundColor Cyan
  git -C "D:\Meow-Env\$dir" status
}
```

---

## 附錄：路徑對照表

| 項目 | Mac 路徑 | Windows 路徑 |
|------|---------|-------------|
| 根目錄 | `~/Meow-Env/` | `D:\Meow-Env\` |
| Meow-Agent | `~/Meow-Env/Meow-agent/` | `D:\Meow-Env\Meow-agent\` |
| Claude Code 全域設定 | `~/.claude/` | `C:\Users\<使用者>\.claude\` |
| settings.json | `~/.claude/settings.json` | `C:\Users\<使用者>\.claude\settings.json` |
| settings.local.json | `~/.claude/settings.local.json` | `C:\Users\<使用者>\.claude\settings.local.json` |
