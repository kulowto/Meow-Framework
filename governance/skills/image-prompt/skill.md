---
name: image-prompt
description: 萬用圖片提示詞架構師。用 JSON 結構化思考畫面，再翻譯成目標 AI 平台的最佳格式（Midjourney / DALL-E 3 / Stable Diffusion / Flux / Firefly / Ideogram 等）。
version: v1.0
source_repo: https://github.com/ConardLi/garden-skills
source_file: skills/gpt-image-2/SKILL.md
installed_date: 2026-06-01
derived_from: gpt-image-2 Skill 的 JSON 提示詞方法論與 18 大圖像分類，提煉為平台無關的萬用版本，擴充六大平台翻譯規則
---

# Image Prompt — 萬用圖片提示詞架構

## 觸發方式

使用者描述想生成的圖片，或輸入 `/image-prompt [想做什麼]`。

---

## 執行流程

### Step 1：解析需求，確認圖像類型

從使用者描述中判斷對應的圖像類型（18 大類，見附錄 A），確認是否為特殊場景：

| 特殊場景 | 觸發條件 | 生效規則 |
|---------|---------|---------|
| Storyboard（分鏡敘事） | 多格連續畫面、有故事發展 | 規則 A |
| Lineup / Catalog（排列對比） | 多個主體並排比較 / 商品目錄 | 規則 B |
| Split 排版（左右對稱） | 行程地圖、左右對位設計 | 規則 C |

---

### Step 2：填入 JSON 骨架

依照三種參數類型填寫：

| 類型 | 說明 | 處理方式 |
|------|------|---------|
| **核心（必問）** | 缺失嚴重影響結果 | 主動向使用者詢問，不猜測 |
| **可默認** | 有合理預設，不影響主要結果 | 自動填入，告知使用者預設值 |
| **可隨機** | 允許在風格範圍內自動補全 | 自動填入，可接受使用者更正 |

```json
{
  "type": "圖像類型（必問）",
  "goal": "圖像用途與使用場景（必問）",
  "subject": {
    "main": "主要主體描述（必問）",
    "secondary": "次要元素（可隨機）"
  },
  "scene": {
    "background": "背景環境（可默認：與主體相符的中性場景）",
    "lighting": "光線條件（可默認：自然光）",
    "atmosphere": "整體氛圍（可默認：與 goal 相符）"
  },
  "layout": {
    "composition": "構圖方式（可默認：黃金比例 / 三分法）",
    "ratio": "畫面比例（必問：1:1 / 16:9 / 9:16 / 4:3 等）",
    "focus": "視覺焦點位置（可默認：中央或三分點）"
  },
  "style": {
    "aesthetic": "整體美學風格（必問）",
    "medium": "媒材質感（可默認：與風格相符）",
    "color_palette": "色彩基調（可默認：與氛圍相符）",
    "quality_tags": "品質描述詞（可默認：依平台自動填入）"
  },
  "details": {
    "text_overlay": "畫面上的文字（若有）",
    "special_elements": "特殊元素或道具"
  },
  "constraints": {
    "must_include": ["必須出現的元素"],
    "must_avoid": ["必須避免的元素或風格"]
  }
}
```

---

### Step 3：特殊場景額外規則

**規則 A — Storyboard（分鏡）**
- 必須有 `continuity` 聲明：幾格 panel、連續敘事主題
- 每格三要素缺一不可：景別（遠 / 中 / 近 / POV）+ 主體動作 + 光線/情緒
- 景別必須混搭，不可全格相同
- JSON 中 `subject.main` 在各格需保持外觀一致性描述

**規則 B — Lineup / Catalog（排列對比）**
- 每個 panel 必須各自指定 `theme_color` + `symbol`
- 缺少 per-panel 差異描述 → 模型會把所有格畫成同一張
- 必須有 `legend`：說明每格代表的等級、類別或意義

**規則 C — Split 排版（左右對稱）**
- 必須有 `alignment_rule`：左右兩欄的編號或主題對位方式
- 左右欄的視覺元素數量和順序必須嚴格對齊

---

### Step 4：確認目標平台

詢問使用者使用哪個圖片生成平台，或從上下文判斷。
若使用者未說明，預設詢問：「請問這張圖要用哪個 AI 生成？」

支援平台：Midjourney / DALL-E 3 / Stable Diffusion / Flux / Adobe Firefly / Ideogram / 其他

---

### Step 5：翻譯為目標平台格式

依照附錄 B 的翻譯規則，將填好的 JSON 轉為對應格式輸出。

---

### Step 6：輸出

```
## 最終提示詞（直接貼入 {平台名稱}）

{翻譯後的提示詞}

---
## JSON 記錄（日後切換平台 / 微調使用）

{原始 JSON}
```

---

## 附錄 A：18 大圖像類型

| 類型 | 適合場景 |
|------|---------|
| `ui-mockups` | App 介面、網頁截圖風格 |
| `product-visuals` | 電商商品展示、產品圖 |
| `infographics` | 資訊圖表、數據視覺化 |
| `poster-and-campaigns` | 海報、廣告素材、IG 貼文封面 |
| `slides-and-visual-docs` | 簡報封面、文件視覺化 |
| `portraits-and-characters` | 人物照、角色設計 |
| `scenes-and-illustrations` | 場景插圖、概念藝術 |
| `editing-workflows` | 圖片後製合成工作流 |
| `avatars-and-profile` | 頭像、個人檔案圖 |
| `storyboards-and-sequences` | 分鏡、連續敘事畫面 |
| `grids-and-collages` | 拼貼、格狀排版 |
| `branding-and-packaging` | 品牌識別、包裝設計 |
| `typography-and-text-layout` | 字體排版為主的設計 |
| `assets-and-props` | 素材、道具、去背圖 |
| `academic-figures` | 學術圖表、研究示意圖 |
| `technical-diagrams` | 技術架構圖、流程圖 |
| `maps` | 地圖、路線、空間示意 |
| `（其他）` | 上述未涵蓋的類型 |

---

## 附錄 B：各平台翻譯規則

### Midjourney

```
格式：[主體描述], [場景描述], [風格詞], [品質詞] --ar [比例] --style [風格] --no [避免元素]

規則：
- 描述短而精確，避免長句
- style 欄 → 風格關鍵詞（cinematic / watercolor / photorealistic 等）
- layout.ratio → --ar（例：16:9 → --ar 16:9）
- constraints.must_avoid → --no 參數（例：--no text, watermark, blur）
- 品質詞固定加：--q 2（或 --q 1 節省速度）
```

**範例輸出**：
`a confident businesswoman in a minimalist office, soft morning light, editorial photography style, clean composition --ar 3:2 --style raw --no clutter, text, watermark`

---

### DALL-E 3 / GPT-Image-2

```
格式：一段詳細的自然語言描述段落

規則：
- 越詳細越好，把 JSON 所有欄位都整合進一段話
- 明確描述光線、氛圍、構圖、色彩
- constraints.must_avoid 直接寫進描述（「避免...」或「不包含...」）
- 比例透過 API 的 size 參數設定，不寫進 prompt
```

**範例輸出**：
`A confident businesswoman standing in a minimalist modern office with floor-to-ceiling windows. Soft morning light filters through, casting warm golden tones. The composition uses rule of thirds with the subject slightly left of center. Clean, editorial photography aesthetic with a muted, professional color palette. No text overlays, no cluttered background, no harsh shadows.`

---

### Stable Diffusion / Flux

```
格式：
正面提示詞（Positive）：[主體], [場景], [風格詞], [品質 tag]
負面提示詞（Negative）：[constraints.must_avoid 的所有項目]

規則：
- 品質 tag 固定加：masterpiece, best quality, highly detailed
- 風格詞用英文逗號分隔的 tag 形式（不用長句）
- constraints.must_avoid → 全部移入 negative prompt
- 比例透過生成介面的 Width/Height 設定
```

**範例輸出**：
- Positive: `businesswoman, minimalist office, morning light, editorial photography, professional, clean composition, masterpiece, best quality, highly detailed`
- Negative: `text, watermark, blur, clutter, low quality, distorted face, extra limbs`

---

### Adobe Firefly

```
格式：自然語言，風格保守

規則：
- 描述比 DALL-E 3 簡短，避免過於複雜的場景
- 不描述具體真實人物（會被過濾）
- 商業安全場景表現最好（辦公室 / 產品 / 自然場景）
- constraints.must_avoid 改寫為正面描述（Firefly 無 negative prompt）
```

---

### Ideogram

```
格式：自然語言，強調文字排版需求

規則：
- 最適合 typography-and-text-layout 類型
- 畫面中的文字直接寫進描述，用引號標示（例：with the text "Hello World" in bold sans-serif）
- style 欄強調設計感（graphic design, poster art, vector illustration 等）
```

---

### Leonardo AI

```
格式：自然語言 + 平台 Style Preset 選擇

規則：
- style 欄先對應到 Leonardo 的內建 preset（Cinematic / Anime / Photography 等）
- Preset 選好後，prompt 可以較短，讓 preset 主導風格
- 適合需要一致性角色的多張圖（用 Image Guidance 功能）
```

---

## 注意事項

- 遇到模糊需求（「做個好看的」）→ 先問清楚 goal 和 subject，不自行假設
- constraints.must_avoid 是品質保障：沒有明確要避免的，也要問使用者「有沒有不想要的元素？」
- 輸出 JSON 記錄的目的是讓使用者可以隨時切換平台，不需要重新描述需求

---

## 來源 & 更新追蹤

| 項目 | 內容 |
|------|------|
| 上游 Repo | https://github.com/ConardLi/garden-skills |
| 對應 Skill | `gpt-image-2` |
| 對應文件 | `skills/gpt-image-2/SKILL.md` |
| 本地版本 | v1.0（2026-06-01 建立） |
| 衍生說明 | 提取 JSON 提示詞方法論、18 大圖像分類、特殊場景規則，移除 GPT-Image-2 API 相依，擴充為六大平台翻譯版本 |

**更新時檢查項目**：
1. 前往上游 Repo 查看 `gpt-image-2` Skill 的最新版本
2. 對比 JSON 骨架欄位是否新增 / 移除
3. 對比 18 大圖像類型是否有調整
4. 對比特殊場景規則是否有新增場景
5. 若有實質更新，同步修改本文件並更新 `version`，同步更新 `MW/wiki/concepts/ref-image-prompt-architecture.md`
