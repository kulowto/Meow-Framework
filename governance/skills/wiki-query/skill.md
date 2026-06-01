---
name: wiki-query
description: 使用分層索引從知識庫精準檢索內容，保護 context 不被大量文件撐爆。整合品質機制（reviewed 狀態）與特殊格式處理（PDF / Excel）。
version: v1.0
適用系統: Meow-Wiki 及遵循相同架構的知識庫
source_repo: https://github.com/ConardLi/garden-skills
source_file: skills/kb-retriever/SKILL.md
installed_date: 2026-06-01
derived_from: kb-retriever 的分層索引與迭代檢索機制，擴充 reviewed 品質機制、5 輪上限後使用者決策節點、PDF/Excel 特殊格式處理
---

# Wiki Query Skill

## 觸發方式

使用者輸入 `/wiki-query [主題或問題]` 或請 AI 查詢知識庫。

---

## 執行流程

### Step 0：確認知識庫根目錄

- 優先使用使用者指定的路徑
- 預設路徑：`D:\Meow-Env\Meow-wiki\`
- 根目錄下若無 `MENU.md` → 停下來告知使用者，不猜測路徑

---

### Step 1：讀 MENU.md — 確認目標域

讀取 `{wiki_root}/MENU.md`，找出目標資訊最可能落在哪個域（concepts / entities / sources / syntheses / assets）。

**禁止直接讀 wiki/index.md 或任何具體文件**，必須先完成此步驟。

---

### Step 2：讀域索引 — 縮小範圍

讀取對應域的 `{wiki_root}/wiki/{domain}/_DataStructure_INDEX.md`。

從索引表找出候選文件，同時**預讀每份文件的 reviewed 狀態**：

| reviewed | 處理方式 |
|----------|---------|
| `true` | 可直接作為執行依據 |
| `false` 或欄位缺失 | 讀取後**必須告知使用者**：「此文件尚未人工審核，若要作為執行依據請確認後再使用」 |

---

### Step 3：讀目標文件

讀取候選文件，提取與問題相關的內容。

**特殊格式處理（PDF / Excel）**：
- 遇到 PDF 文件前，必須先讀 `{wiki_root}/references/pdf_reading.md`
- 遇到 Excel / CSV 文件前，必須先讀 `{wiki_root}/references/excel_reading.md`
- 禁止在未讀規範的情況下直接處理特殊格式

---

### Step 4：判斷是否足夠

- **資訊足夠** → 整合後回答使用者，標注引用來源（文件路徑 + reviewed 狀態）
- **資訊不足** → 換域、換關鍵詞，回到 Step 1，進入下一輪

---

### Step 5：5 輪上限處理

每輪 = 一次「讀索引 → 讀文件 → 判斷」的完整循環。

累積 **5 輪**仍未找到足夠資訊時：

1. 停止自動檢索
2. 明確告知使用者：
   - 已嘗試哪些域 / 文件（列出路徑）
   - 目前找到的最接近資訊（若有）
3. **讓使用者決定下一步**，選項包括：
   - 換關鍵詞重新查詢
   - 指定特定文件直接讀取
   - 接受現有資訊
   - 結束查詢

**禁止**在超過 5 輪後繼續自動查詢，也禁止僅回傳「找不到」而不讓使用者選擇方向。

---

### Step 6（選填）：詢問是否保存

若本次查詢產生了有價值的洞見或合成，詢問使用者是否要存回 `wiki/syntheses/`。

保存格式參照 `wiki/syntheses/_DataStructure_INDEX.md` 的更新規則。

---

## 回答格式

```
## 查詢結果

**來源**：{域}/{文件名}（reviewed: true/false）
**內容**：

{整合後的回答內容}

---
⚠️ 未審核文件警告（若有）：以下文件尚未人工審核，內容作為參考，請確認後再作為執行依據。
```

---

## 注意事項

- 永遠先讀 MENU.md，不跳步直接讀具體文件
- 每次引用都要標注 reviewed 狀態，讓使用者知道資料品質
- 遇到 PDF / Excel 必讀對應規範，不猜測處理方式
- 不可刪除或修改任何文件（查詢為唯讀操作）
- `raw/` 目錄 AI 不主動讀取，除非使用者明確指定

---

## 來源 & 更新追蹤

| 項目 | 內容 |
|------|------|
| 上游 Repo | https://github.com/ConardLi/garden-skills |
| 對應 Skill | `kb-retriever` |
| 對應文件 | `skills/kb-retriever/SKILL.md` |
| 本地版本 | v1.0（2026-06-01 建立） |
| 衍生說明 | 保留分層索引與迭代檢索機制，加入 reviewed 品質機制、5 輪上限後使用者決策、PDF/Excel 規範引導 |

**更新時檢查項目**：
1. 前往上游 Repo 查看 `kb-retriever` Skill 的最新版本
2. 對比分層索引流程是否有調整
3. 對比迭代輪數上限或失敗處理邏輯是否更新
4. 有實質更新時同步修改本文件並更新 `version`，同步更新 `MW/.claude/skills/query/skill.md`
