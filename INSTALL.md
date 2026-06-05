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

### 【AI 執行】Mac — 首次安裝（Clone）

```bash
cd ~/Meow-Env

git clone https://github.com/kulowto/Meow-Framework.git
git clone https://github.com/kulowto/Meow-agent.git
git clone https://github.com/kulowto/Meow-Wiki.git
git clone https://github.com/kulowto/Meow-tools.git
git clone https://github.com/kulowto/Meow-dev.git
```

---

### 【AI 執行】Mac — 已有 repo 時更新（Pull）

```bash
cd ~/Meow-Env
for dir in Meow-Framework Meow-agent Meow-Wiki Meow-tools Meow-dev; do
  echo "=== $dir ===" && git -C "$dir" pull
done
```

---

### 【AI 執行】Windows — 首次安裝（Clone，PowerShell）

```powershell
cd D:\Meow-Env

git clone https://github.com/kulowto/Meow-Framework.git
git clone https://github.com/kulowto/Meow-agent.git
git clone https://github.com/kulowto/Meow-Wiki.git
git clone https://github.com/kulowto/Meow-tools.git
git clone https://github.com/kulowto/Meow-dev.git
```

---

### 【AI 執行】Windows — 已有 repo 時更新（Pull，PowerShell）

```powershell
cd D:\Meow-Env
foreach ($dir in @("Meow-Framework","Meow-agent","Meow-Wiki","Meow-tools","Meow-dev")) {
  Write-Host "=== $dir ===" -ForegroundColor Cyan
  git -C $dir pull
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

查看 Profile 路徑：`echo $PROFILE`，加入以下內容：

```powershell
function cc {
    Set-Location D:\Meow-Env\Meow-agent
    claude
}
```

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

#### 【AI 執行】Windows — 將 PowerShell Profile 內的 cc 改為以下版本

```powershell
function cc {
    param([string]$flag = "")
    Set-Location D:\Meow-Env\Meow-agent
    switch ($flag) {
        "-ma" { claude "請讀取 context.md 告知上次進度，接下來聚焦 Meow-Agent 任務。" }
        "-mw" { claude "請讀取 D:\Meow-Env\Meow-Wiki\context.md 告知上次進度，接下來聚焦 Meow-Wiki 任務。" }
        "-mt" { claude "請讀取 D:\Meow-Env\Meow-tools\context.md 告知上次進度，接下來聚焦 Meow-Tools 任務。" }
        "-md" { claude "請讀取 D:\Meow-Env\Meow-dev\context.md 告知上次進度，接下來聚焦 Meow-Dev 任務。" }
        default { claude }
    }
}
```

設定完成後重新載入 Shell，旗標即生效。`context.md` 的格式與維護規範見：  
`Meow-Framework/governance/collaboration/(AI_Read) context 維護規範.md`

---

## 六、Claude Code Skills 安裝

以下 Skills 已記錄在 Meow-agent 的 CLAUDE.md 中，clone repo 後即可獲得檔案。  
如需在新機器上啟用，參照各 Skill 的 `manifest.json` 執行安裝指令。

| Skill 名稱 | 路徑（相對於 Meow-agent） | 說明 |
|------------|--------------------------|------|
| web-design-engineer | `.agents/skills/web-design-engineer/` | 網頁設計工程 |
| web-video-presentation | `.agents/skills/web-video-presentation/` | 文章轉影片簡報 |

> 安裝任何新 Skill 時，必須遵守來源追蹤規範：  
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

- [ ] 五個 repo 均已 clone 至正確路徑，`git status` 顯示 `nothing to commit`
- [ ] `~/.claude/settings.json` 存在且可讀取
- [ ] `~/.claude/settings.local.json` 已建立（即使是空框架）
- [ ] 輸入 `cc` 可成功啟動 Claude Code 並切入 Meow-agent 目錄
- [ ] Claude Code 內輸入 `Ctrl+G` 可開啟外部編輯器

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
