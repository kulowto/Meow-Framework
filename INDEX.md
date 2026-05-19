---
文件類型: index
版本: v0.1
建立: 2026-05-19
---

# Meow-Framework INDEX

> 本文件是 Meow-Framework 的對外介面，供 `REGISTRY.md` 與歸檔 Agent 引用。

---

## 本框架提供的能力

| 能力 | 路徑 | 適用對象 |
|------|------|---------|
| 通用 AI 協作規則（@include 繼承） | `CLAUDE.md` | 所有客戶 |
| AI 行為規則 / 資料傳遞契約 | `governance/collaboration/` | 所有客戶 |
| context.md 維護規範（何時刪除 / 壓縮 / 保留）| `governance/collaboration/(AI_Read) context 維護規範.md` | 所有有 context.md 的系統 |
| 角色 & 領域骨架模板 | `governance/roles/` | 需要多角色系統的客戶 |
| 通用 Prompt 模板（學習框架、壓縮格式等）| `governance/prompts/` | 所有客戶 |
| 知識管理架構概念（三層知識庫結構）| `governance/knowledge/` | **有 Wiki 子系統的客戶才需讀** |

---

## 使用方式

客戶在自己的 Agent CLAUDE.md 第一行加入：
```
@../[Name]-Framework/CLAUDE.md
```
後續個人覆蓋層寫在 include 之後。

---

## 歸檔接受準則（供歸檔 Agent 使用）

### 接受
- 普遍適用的 AI 協作規則（無個人名稱、無個人路徑）
- 角色骨架模板（通用骨架，不含個別角色定義）
- 任何領域通用的 Prompt 框架
- 知識管理架構概念

### 不接受
- 特定系統的路徑（如 `D:\Meow-Env\...`）
- 個人語氣 / 語言偏好設定
- 單一工具的操作步驟
- 任何 Meow 特有的設定或歷史決策
