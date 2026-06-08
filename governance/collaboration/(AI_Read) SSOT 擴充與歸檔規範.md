---
type: protocol
scope: 全域（所有涉及 SSOT 新增、修改、歸檔的操作）
version: v1.0
created: 2026-06-08
usage: AI 執行任何 SSOT 新增 section、更新內容或歸檔操作前必須閱讀並遵守。
---

# (AI_Read) SSOT 擴充與歸檔規範

> 適用範圍：**Meow-Agent SSOT 系統**（`D:\Meow-Env\Meow-agent\ssot\`）。
> 任何對 SSOT 的新增、修改或歸檔操作，都必須遵守本規範。

---

## 一、SSOT 架構概覽

```
Meow-Agent/ssot/
  core/                       ← 核心知識（每次啟動可引用）
    persona.md                    人設、語氣、語言、協作方式
    environment.md                設備、路徑、GitHub Repos、平台差異
    tools.md                      MCP 工具清單
    skills-registry.md            Skills 來源矩陣
    methodology.md                方法論索引（指向 MF/governance/）
    links.md                      常用連結索引
    timeline.md                   事件與專案時間軸
  content-profiles/           ← 受眾定義（按需載入，不全部引用）
    _index.md                     Profile 清單與使用說明
    {profile-name}.md             各受眾定義文件
  archive/                    ← 退役內容歸檔區
    _README.md                    歸檔區說明
    content-profiles/             退役 Profile
    skills/                       廢棄 Skill 記錄
    timeline-YYYY.md              年度 Timeline 歸檔
    snapshots/                    重構前版本快照
```

### 與其他層的關係

| 層 | 位置 | 內容 | SSOT 角色 |
|----|------|------|----------|
| 規則 SSOT | `MF/governance/` | AI 行為規則、格式規範 | 通用規則的唯一來源 |
| 知識 SSOT | `MA/ssot/` | 個人知識、路徑、工具、受眾 | 個人資料的唯一來源 |

**判斷標準**：「別人 fork 這個框架，他會想要一樣的內容嗎？」
- 會 → 放 `MF/governance/`
- 不會 → 放 `MA/ssot/`

---

## 二、各 AI 工具的 SSOT 消費方式

| AI 工具 | 同步方式 | 說明 |
|---------|---------|------|
| Claude Code | `@include ssot/core/*.md`（即時生效） | 修改 SSOT 後，下次啟動自動讀取，不需要 build |
| Codex | `python ssot/build/build.py`（手動生成 `AGENTS.md`） | 不支援 @include，需執行 build script |
| 其他 AI | 新增對應 template，執行 build | 擴充 build script 的 templates 目錄 |

> **Build script 位置**：`MA/ssot/build/build.py`（待建立，AGENTS.md 需求確認後再實作）

---

## 三、新增 Section 的流程

### 3.1 新增 core/ 文件

1. 在 `MA/ssot/core/` 建立新的 `.md` 文件
2. frontmatter 必填欄位：

```yaml
---
ssot_section: {section-name}    # 唯一識別，snake_case
version: v1.0
created: YYYY-MM-DD
description: 一行說明此文件的用途與內容範圍
---
```

3. 在 `MA/CLAUDE.md` 的 SSOT include 區塊補一行 `@ssot/core/{filename}.md`
4. 在 `MA/ssot/core/methodology.md` 或適當的索引文件補上引用

### 3.2 新增 Content Profile

1. 在 `MA/ssot/content-profiles/` 建立 `{profile-name}.md`
2. frontmatter 必填欄位：

```yaml
---
profile: {profile-name}
version: v1.0
created: YYYY-MM-DD
description: 一行說明適用情境
---
```

3. 在 `_index.md` 的 Profile 清單表格補一行

### 3.3 判斷應放哪裡

- 屬於「我是誰、怎麼說話」→ `persona.md`
- 屬於「在哪裡工作、用什麼設備」→ `environment.md`
- 屬於「用什麼工具」→ `tools.md` 或 `skills-registry.md`
- 屬於「要做給誰看」→ `content-profiles/{name}.md`
- 屬於「什麼時候發生過什麼」→ `timeline.md`
- 屬於「通用 AI 行為規則」→ `MF/governance/`（不放 SSOT）

---

## 四、更新現有 Section 的流程

1. 直接修改對應 SSOT 文件
2. 更新 frontmatter 的 `version`（v1.0 → v1.1）
3. 若修改影響 Codex，執行 `python ssot/build/build.py` 重新生成 `AGENTS.md`
4. Commit 訊息格式：`docs(ssot): 更新 {section-name} - 簡述改了什麼`

**重要**：修改 SSOT 文件後，Claude Code 下次啟動即生效（via @include）。Codex 需要手動 build。

---

## 五、歸檔協議

### 5.1 觸發歸檔的情境

| 情境 | 處理方式 |
|------|---------|
| Content Profile 不再使用 | 移至 `archive/content-profiles/` |
| Skill 廢棄或被替換 | 在 `skills-registry.md` 標記，詳細記錄移至 `archive/skills/` |
| Timeline 超過一年的事件 | 移至 `archive/timeline-YYYY.md` |
| SSOT 某 section 整個重構 | 舊版存入 `archive/snapshots/` |

### 5.2 歸檔步驟

1. **在原文件加退役標記**（不要直接刪除原文件）：

```yaml
# 若整個文件退役，在 frontmatter 加：
archived: YYYY-MM-DD
archived_reason: 說明為何退役（停產 / 被替換 / 重構）
archived_to: archive/{path}/filename.md
```

2. **移動文件至 archive/ 對應子目錄**

3. **在原位留一個簡短的 note**（如果該位置還會被引用到）：

```markdown
> ⚠ 此 Profile 已歸檔（YYYY-MM-DD）。  
> 原因：[說明]  
> 歸檔位置：`archive/content-profiles/{filename}.md`
```

4. **更新 _index.md 或 skills-registry.md** 中對應的行，標記為 `已歸檔`

5. **更新 CLAUDE.md**：若該文件原本在 @include 列表中，移除對應行

### 5.3 歸檔文件的維護

- 歸檔後的文件**不再主動更新**，只供歷史查閱
- 歸檔文件的 frontmatter `version` 凍結，不再遞增

### 5.4 年度 Timeline 歸檔（每年 1 月執行）

1. 將前一年的事件從 `timeline.md` 剪下
2. 貼入 `archive/timeline-YYYY.md`（YYYY 為前一年年份）
3. `timeline.md` 保留當年及未來的事件

---

## 六、快速自查清單

### 新增 Section 時

```
□ frontmatter 有 ssot_section（或 profile）、version、created、description
□ 在 CLAUDE.md 補 @include（若屬於 core/）
□ 在索引文件（_index.md 或 methodology.md）補上引用
□ 判斷過「放 MA 還是 MF」，選擇正確位置
```

### 更新 Section 時

```
□ frontmatter version 已更新
□ 若影響 Codex → 已執行 build script
□ Commit 訊息有 docs(ssot): 前綴
```

### 歸檔時

```
□ 原文件 frontmatter 加了 archived / archived_reason / archived_to
□ 文件已移至 archive/ 對應子目錄
□ 原位有 note（若該位置仍被引用）
□ 索引文件（_index.md / skills-registry.md）已更新
□ CLAUDE.md @include 列表已移除（若適用）
```
