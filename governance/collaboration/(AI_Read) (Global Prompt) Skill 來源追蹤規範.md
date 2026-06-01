---
type: protocol
scope: 全域（所有涉及 Skill 安裝、建立、更新的操作）
version: v1.0
created: 2026-06-01
usage: AI 執行任何 Skill 安裝、自建或更新操作前必須閱讀並遵守。
---

# (AI_Read) (Global Prompt) Skill 來源追蹤規範

> 適用範圍：**所有 Meow-Env 子系統**。
> 凡涉及 Skill 的安裝（外部引入）、自建（從零撰寫）、更新（同步上游變更），**每一步都必須留下可追溯的來源資訊**。
> 這不是建議，是**強制規範**。

---

## 規則一：安裝外部 Skill 時，Skill 文件必須記錄來源

**問題描述：**
Skill 安裝後若未記錄出處，日後上游 Repo 更新、或功能出問題時，無從追溯原始版本與差異，維護成本極高。

**規範：**

安裝來自 GitHub 或任何外部來源的 Skill 後，**Skill 文件的 frontmatter 必須包含以下欄位**：

```yaml
---
name: skill-name
description: ...
version: v1.0                        # 本地版本號，初次安裝為 v1.0
source_repo: https://github.com/...  # 上游 GitHub Repo URL（完整 URL）
source_file: path/to/SKILL.md        # Repo 內的具體文件路徑
installed_date: YYYY-MM-DD           # 安裝日期
---
```

若 Skill 內容有**衍生修改**（非原版直接使用），額外必填：

```yaml
derived_from: 說明做了哪些衍生修改，以及為什麼
```

**禁止：**
- 安裝後 frontmatter 不填任何來源欄位
- 用模糊描述替代 URL（例如「來自 ConardLi」而不填完整 URL）

---

## 規則二：自建 Skill 時，必須記錄設計靈感與參考來源

**問題描述：**
自建 Skill 若未說明設計靈感或參考來源，未來改版時無法判斷哪些部分是核心邏輯、哪些是參考借鑑，容易過度或不足修改。

**規範：**

自建 Skill 的 frontmatter 必須包含：

```yaml
---
name: skill-name
description: ...
version: v1.0
source_repo: 自建                    # 無外部來源時填「自建」
source_file: N/A
installed_date: YYYY-MM-DD
derived_from: 說明設計靈感、參考資料或觸發此 Skill 建立的背景
---
```

---

## 規則三：Skill 文件末尾必須附「來源 & 更新追蹤」區塊

除了 frontmatter，**Skill 文件正文末尾必須附一個「來源 & 更新追蹤」區塊**，供人工快速確認：

```markdown
---

## 來源 & 更新追蹤

| 項目 | 內容 |
|------|------|
| 上游 Repo | https://github.com/... |
| 對應文件 | path/to/SKILL.md |
| 本地版本 | v1.0（YYYY-MM-DD 建立） |
| 衍生說明 | （若有改動）說明做了哪些修改 |

**更新時檢查項目**：
1. （列出更新時需要對照的具體項目）
2. 有實質更新時：更新 frontmatter 的 `version`，追加 changelog 記錄
```

---

## 規則四：更新 Skill 時必須留下 changelog

**問題描述：**
Skill 版本更新後若未記錄「改了什麼、為什麼改」，下一個 AI 無法判斷當前版本是否仍符合需求。

**規範：**

每次更新 Skill 時，在「來源 & 更新追蹤」區塊下方追加：

```markdown
### Changelog

| 版本 | 日期 | 更新內容 |
|------|------|---------|
| v1.1 | YYYY-MM-DD | 說明改了什麼（對照上游哪個 commit / PR） |
| v1.0 | YYYY-MM-DD | 初次安裝 |
```

frontmatter 的 `version` 同步更新（v1.0 → v1.1）。

---

## 規則五：CLAUDE.md 技能表的「來源」欄必須填完整 URL

**問題描述：**
CLAUDE.md 的「已安裝 Plugins / Skills」表格若只寫名稱或描述，其他 AI 無法在不讀 Skill 文件的情況下判斷來源是否可信。

**規範：**

`CLAUDE.md` 中「已安裝 Plugins / Skills」表格的「來源」欄格式：

```markdown
| `skill-name` | [作者/repo-name](https://github.com/作者/repo-name) | 使用情境說明 |
```

- 外部 Skill：填 GitHub Repo 的完整超連結
- 自建 Skill：填 `本地自建（路徑）`，例如 `本地自建（MF/governance/skills/image-prompt/）`
- 本地自訂指令：填 `本地自訂指令（~/.claude/commands/xxx.md）`

---

## 快速自查清單（安裝 / 建立 Skill 完成後逐項確認）

```
□ Skill 文件 frontmatter 有 source_repo（完整 URL 或「自建」）
□ Skill 文件 frontmatter 有 source_file（具體路徑或「N/A」）
□ Skill 文件 frontmatter 有 version 和 installed_date
□ 有衍生修改 → frontmatter 有 derived_from 說明
□ Skill 文件末尾有「來源 & 更新追蹤」區塊
□ 「來源 & 更新追蹤」有具體的更新時檢查項目
□ CLAUDE.md 技能表的「來源」欄填完整 URL（或自建路徑）
□ 若為更新操作 → Changelog 已追加一行，version 已更新
```
