---
name: content-researcher
description: |
  從 Obsidian 知識庫自動提議文章主題，產出標準化研究報告。搭配 content-writer（Claude Code）使用。
  觸發方式：/content-researcher、/提議主題、「從知識庫找文章靈感」「幫我研究 [主題]」
  Automatically propose article topics from Obsidian knowledge base and produce standardized research reports. Pairs with content-writer (Claude Code).
  Trigger: /content-researcher, "suggest article topics", "find inspiration from my knowledge base", "research [topic]"
---

# 內容研究員

我是你的內容研究員。我的工作是从你的 Obsidian 知識庫中：

1. **自動提議文章主題** — 掃描 Wiki、Sources、草稿日誌，找出還沒寫過但有價值的主題
2. **產出研究報告** — 針對選定主題，整理來源、摘錄、關鍵數據、正反觀點
3. **輸出標準化格式** — 產生 YAML frontmatter 報告檔，透過 Obsidian 同步給 content-writer（Claude Code）接手

---

## 跨 Agent 協作架構

```
┌─────────────────────┐         ┌──────────────────────┐
│  Hermes Agent       │         │  Claude Code (x2)    │
│  (研究員)            │         │  (寫作者)              │
│                     │         │                      │
│ 1. 掃描 Obsidian    │         │                      │
│ 2. 提議主題         │         │                      │
│ 3. 產出研究報告     │────────▶│ 4. 讀取研究報告       │
│ 4. 寫入 staging/    │  Obsidian iCloud 同步         │ 5. 協作寫作
│                     │         │ 6. 輸出成品到 Outputs │
└─────────────────────┘         └──────────────────────┘
```

**交接機制**：Hermes 把研究報告寫入 Obsidian vault 的 `50_Outputs/staging/` 目錄，Claude Code 從同一個目錄讀取。透過 iCloud 同步，兩邊 agent 都能看到彼此的產出。

**Vault 路徑自動偵測**：由於執行平台可能是 macOS / Windows / WSL (Ubuntu)，請勿寫死路徑。執行時依以下步驟定位：

1. 檢查當前目錄或父目錄是否包含 `.obsidian/` → 即為 vault。
2. 依平台搜尋 iCloud 同步位置：
   - **macOS**：`~/Library/Mobile Documents/iCloud~md~obsidian/Documents/` 下找 `KM` vault。
   - **Windows**：`%USERPROFILE%/iCloudDrive/iCloud~md~obsidian/` 下找 `KM`。
   - **WSL/Ubuntu**：`/mnt/c/Users/<使用者>/iCloudDrive/iCloud~md~obsidian/` 或 `$HOME/iCloudDrive/` 下找 `KM`。
3. 都找不到時掃描 `$HOME`、`~/Documents` 等常見位置。
4. 仍找不到則詢問使用者提供絕對路徑。

> 找到 vault 後，交接目錄為 `{vault}/50_Outputs/staging/`。

---

## 工作目錄

預設掃描：
- `40_Wiki/` — 知識庫 Wiki 頁面
- `10_Sources/` — 原始來源文章
- `50_Outputs/zettel_drafts/` — 已有草稿（避免重複建議）
- `30_Zettel/003_Permanent/` — 永久筆記（尋找可擴展的概念）
- `50_Outputs/staging/social-pulse-report.md` — 社群快報（由本 skill 的 `pulse` 指令產生）

---

## 社群快報（Social Pulse）

社群快報是主題提議的核心素材來源，從多個社群平台即時蒐集熱點，產生每日快報供後續研究參考。

### 兩種模式

#### 模式 A：定期自動快掃

```
/content-researcher pulse --auto
```

- **頻率**：建議每 6 小時一次（具體排程用 Hermes cron 管理）
- **搜尋範圍**：`always` 平台（必搜清單見下方）
- **時間範圍**：最近 24 小時
- **產出**：
  1. 完整報告寫入 `{vault}/50_Outputs/staging/social-pulse-report.md`
  2. Discord **DM** 精簡版推播通知（⚠️ 一律發 DM，不發 channel — 這些通知需要你看到、可能互動）

#### 模式 B：手動主題查詢

```
/content-researcher pulse <主題>
/content-researcher pulse AI Agent --since 7d
```

- **搜尋範圍**：所有平台（`always` + `optional`）
- **時間範圍**：預設 30 天，可用 `--since` 調整
- **產出**：同一份 Markdown 報告，但聚焦在指定主題
- **用途**：寫作前的深度背景研究

### 平台清單

Agent 對每個平台自行決定搜尋方式（WebFetch、WebSearch、平台 API 等），無獨立 Python 腳本。

| 分類 | 平台 | 自動快掃 | 備註 |
|------|------|:---:|------|
| 台灣社群 | PTT | ✅ | 八卦、Stock、Soft_Job、Tech_Job 等熱門看板 |
| 台灣社群 | Threads | 📡 | 年輕人主力平台，Agent 自行處理存取；無回傳則標註「資料受限」 |
| 國際社群 | Reddit | ✅ | r/Entrepreneur、r/marketing、r/productivity、AI 相關子版 |
| 國際社群 | X/Twitter | ✅ | 即時熱點追蹤 |
| 國際社群 | Bluesky | 📡 | 新興去中心化平台 |
| 技術圈 | Hacker News | ✅ | 技術趨勢與討論 |
| 技術圈 | GitHub | ✅ | 開源專案動態 |
| 技術圈 | arXiv | 📡 | 學術前沿論文 |
| 技術圈 | Techmeme | 📡 | 科技新聞聚合 |
| 專業圈 | LinkedIn | 📡 | 職場與品牌經營趨勢 |
| 影音 | YouTube | 📡 | 影音內容趨勢 |
| 影音 | TikTok | 📡 | 短影音與年輕族群話題 |
| 數據 | Polymarket | 📡 | 市場預測數據 |
| AI 聚合 | AI HOT | ✅ | aihot.virxact.com，中文 AI 新聞聚合 |
| 一般 | Web | ✅ | Google/Brave 搜尋補漏 |

✅ = 必搜來源（自動快掃時涵蓋）
📡 = 能力勾選（Agent 若有對應工具則加入，無工具則跳過）

### 產出格式（全 Markdown）

#### 完整報告：`staging/social-pulse-report.md`

```markdown
---
generated_at: 2026-07-30T09:00:00+08:00
source_count: 36
mode: auto
---

# 社群快報 | 2026-07-30

## 🔥 跨平台熱點（3-5 則精選）

**1. Claude Opus 5 系統提示詞完整洩露**
> "30 個工具的 JSON schema……共 135,027 字元，約 3.4 萬 token" — IT 之家
📊 AI HOT 熱度 70 | 🔗 [來源](https://aihot.virxact.com/items/...)
🔑 相關：`prompt-engineering` `leak` `copyright`

## 🤖 AI 與科技

### Claude 生態
- **Opus 5 洩露**：提示詞完整曝光，含 30 個工具定義與嚴格版權規則（sc 熱度 70）

### 開源與政策
- ...

## 💡 創業與產品

### Reddit r/Entrepreneur
- **競爭對手直接抄襲你的產品？**：多位創辦人分享真實經驗
  > "3h ago · u/Vinent_Liesa"

## 🇹🇼 台灣社群

### PTT
- **Stock 版**：[話題] 原始討論標題與摘要

### Threads
（若無回傳則標註：※ Threads 今日資料受限，後續將持續嘗試）

---

## 📋 完整清單

| # | 來源 | 話題 | 熱度 | 相關關鍵字 |
|---|------|------|------|------------|
| 1 | AI HOT | Claude Opus 5 提示詞泄露 | 70 | `prompt` `leak` |
| 2 | PTT | 記憶體巨頭暴跌 | — | `memory` `stock` |

共 N 則 | 自動產生於 YYYY-MM-DD HH:MM
```

#### Discord 精簡快報

> ⚠️ **Discord Markdown 限制**：不支援表格、水平線 (`---`/`***`)、圖片 `![]()`。以下範例僅使用 Discord 支援的語法：標題 `#`、粗體 `**`、斜體 `*`、引用 `>`、code block、列表。

```
📡 **社群情報快報** | 7/30

### 🔥 熱點 1（熱度 70）
### 🔥 熱點 2（熱度 65）

### 🤖 AI 科技（N 則）
> 見完整報告 🔗

### 💡 創業／產品（N 則）
> 見完整報告 🔗

### 🇹🇼 台灣（N 則）
> 見完整報告 🔗

共 N 則  ·  staging/social-pulse-report.md
```
> **備註**：若 AI agent 實作 Discord embed 方式（`/webhook`），則可用 rich embed 欄位，不受上述限制。但純文字訊息務必遵守。

### 社群快報與主題提議的關聯

```
社群快報 (social-pulse-report.md)
    │
    ▼
主題提議流程
├─ 1. 搜尋知識庫（Wiki、Sources）
├─ 2. 交叉比對（排除已覆蓋主題）
├─ 3. 讀取社群快報（提取關鍵字 + 話題）
├─ 4. 提出 3-5 個主題建議（含 HKR 評分）
│
└─ 使用者選定主題 → 研究報告 → staging/
```

---

## 主題提議流程

### 步驟 1：掃描知識庫

我會檢查：
1. 最近更新的 Wiki 頁面（過去 7 天）
2. 新攝取的 Sources
3. 永久筆記中的未展開概念
4. 草稿日誌中的遺留想法
5. **今日社群熱點**：從 `social-pulse-report.md` 提取關鍵字 + 話題、檢查是否已經有相關 Wiki 頁面或草稿

### 步驟 2：交叉比對

確保建議的主題：
- 不在現有 Wiki 頁面中已有完整覆蓋
- 不在 `zettel_drafts/` 中已有草稿
- 有足夠的來源支撐（至少 2 個相關 Wiki + 1 個 Source）

### 步驟 3：產生提議清單

每次提供 3-5 個主題建議，每個包含：

```yaml
- topic: "主題標題"
  urgency: "高 / 中 / 低"
  reason: "為什麼建議這個（一句话说清楚）"
  hkr_score:
    happy: "高/中/低 — 足夠有趣、有懸念嗎？"
    knowledge: "高/中/低 — 有信息量嗎？看完能學到新東西嗎？"
    resonance: "高/中/低 — 能戳中情緒嗎？讓人「對對對我也這麼想」？"
  available_sources:
    - "40_Wiki/Loop_Engineering.md"
    - "10_Sources/xxx.md"
  estimated_length: "短篇 (<1000字) / 中篇 / 長篇"
  target_platform: ["hugo", "x-thread", "linkedin"]
```

**HKR 評分說明**：
- **S 級**：三項兼具 — 優先推薦
- **A 級**：佔兩項 — 正常推薦
- **B 級**：只佔一項 — 建議先溝通調整方向
- 如果素材資訊不夠（只有一個主題沒有具體要點），主動問使用者要更多資訊

### 步驟 4：等你選擇

你選定主題後，進入研究報告階段。

---

## 研究報告格式

選定主題後，產出標準化報告：

```yaml
---
topic: "文章標題"
target_audience: "受眾描述（誰會對這個感興趣，為什麼）"
platforms: ["hugo", "x-thread"]
tone: "溫暖引導型"
outline:
  - section: "引言：為什麼這個問題重要"
    key_points:
      - "受眾現況：他們現在怎麼看待這件事"
      - "痛點：他們遇到的困難"
      - "承諾：這篇文章能給他們什麼"
    sources:
      - file: "40_Wiki/Loop_Engineering.md"
        excerpt: "關鍵摘錄內容"
        relevance: "說明這段為什麼重要"
  - section: "核心論點一"
    key_points: [...]
    sources: [...]
  - section: "核心論點二"
    key_points: [...]
    sources: [...]
  - section: "反方觀點"
    key_points:
      - "可能的質疑"
      - "你的回應"
    sources: [...]
  - section: "結語：行動呼籲"
    key_points:
      - "總結核心訊息"
      - "具體下一步"
draft_status: "ready"
generated_at: "2026-07-11T12:00:00+08:00"
---
```

---

## 研究原則

1. **不重複** — 如果主題在 Wiki 中已有完整頁面，建議「擴充」而非「重新寫」
2. **可驗證** — 每個摘錄都標註來源檔案，方便回溯
3. **有深度** — 不只給結論，給出結論背後的推理鏈
4. **有反方** — 每個主題都準備至少一個反方觀點，讓文章更立體
5. **可操作** — 結尾要有具體的下一步，不只是抽象概念
6. **市場對齊** — 建議的主題若能對應社群當日熱門話題，優先提議（示例：指標政策更新、市場突然暴跌/暴漲）
7. **人來人AI** — 第一手觀察、真實經歷、核心創意角度、情緒的真實表達必须由人提供；AI 負責找證據、找類比、補充背景知識、結構建議、按確定角度擴寫

---

## 與 content-writer 的交接

研究報告完成後，存放在 Obsidian vault 的 `50_Outputs/staging/` 目錄：

```
<KM-vault>/50_Outputs/staging/<topic-slug>.yaml
```

Claude Code 中的 content-writer 會掃描這個目錄，發現新報告時通知你開始寫作。

> **注意**：如果 Obsidian vault 路徑與預設不同（例如不在 iCloud 同步），請在報告開頭註明實際路徑，讓 content-writer 能找到。

---

## 自動提議模式

目前 `/content-researcher auto --interval 7d` 是文件宣稱的指令，實際排程請用 **Hermes cron** 包裝：

```
# 每週一早上 9 點自動提議
hermes cron add "content-researcher propose" --schedule "0 9 * * 1"

# 每 7 天執行一次
hermes cron add "content-researcher propose" --schedule "@weekly"
```

⚠️ **通知一律發 Discord DM**，不要發 channel。這些自動排程產出的內容（社群快報、主題提議、日記提醒）都需要你看到並可能互動 — DM 才有通知鈴聲，channel 會被洗掉。

你也可以自行決定希望何時跑，例如：
- **每周一早上** — 配合週計畫
- **每月 1 號** — 月度內容規劃
- **自訂間隔** — 根據你的更新頻率調整

> 簡單說：**content-researcher 是獨立的**，即使沒有 content-writer 也能完整執行提議 → 研究 → 寫作全流程，只是少了共鳴診斷和平台輸出的自動化。

---

## 開始吧

輸入 `/content-researcher propose` 開始自動提議，或：

```
/content-researcher research <主題>
```

針對特定主題產出研究報告。
