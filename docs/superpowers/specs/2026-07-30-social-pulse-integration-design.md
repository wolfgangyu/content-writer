# Social-Pulse 社群快報整合設計

**日期**：2026-07-30
**狀態**：設計定稿
**決策者**：Wolfgang

---

## 1. 背景與動機

目前 `social-pulse` 是獨立的 Python REPO，每天 07:00 透過 cron 蒐集 PTT、Reddit、AI HOT 等社群熱點，產出 JSON 到 Obsidian staging/ 目錄，供 `content-researcher` 提議主題時參考。

但實際操作流程中，**所有決策都由人（Wolfgang）在 Claude Code 內進行** — 沒有人會自動讀取 YAML/JSON 觸發寫作。因此保留機器格式是多餘的。

### 設計目標

1. **消滅獨立 REPO** — `social-pulse` 改為 `content-researcher` 的一段 prompt 指令，不再有獨立程式碼或倉庫
2. **融合 `last30days-skill` 精神** — 多平台平行搜尋、社群原句引用、30 天時間範圍
3. **加入台灣社群** — PTT 為必備，Threads 由 Agent 自行設法存取
4. **產出全改 Markdown** — 砍掉 JSON/YAML，人讀得舒服、AI 也能解析
5. **雙模式** — 定時自動快報（Discord 推播）+ 手動查詢（深度研究）

---

## 2. 架構變更

### 現狀（Before）

```
social-pulse (獨立 REPO)
├─ social-pulse.py                ← Python 腳本
├─ cron: 每日 07:00               ← Betelgeuse 機器
├─ 依賴: firecrawl, opencc
├─ 產出 → staging/social-pulse.json
│           ↓ iCloud 同步
└─ content-researcher 讀取 JSON
```

### 設計（After）

```
content-researcher (本 REPO)
├─ SKILL.md                       ← 新增「社群快報」章節
├─ config.yaml                    ← 加入社群快報設定
│
├─ 模式 A: 定期自動快報
│   └─ Hermes cron 觸發 → Agent 搜尋多平台
│        → 產出 social-pulse-report.md → staging/
│        → 產出 Discord 快報 → 推播
│
└─ 模式 B: 手動查詢
    └─ /content-researcher pulse <主題>
         → Agent 搜尋多平台 → 合成 Markdown 簡報
```

### 檔案變更

| 檔案 | 變更 | 說明 |
|------|------|------|
| `social-pulse/` (獨立 REPO) | **歸檔** | 不再維護，README 加註遷移說明 |
| `content-researcher/SKILL.md` | **新增「社群快報」章節** | 描述平台清單、產出格式、兩種模式 |
| `content-researcher/config.yaml` | **擴充** | 加入社群快報參數 |
| 產出: `staging/social-pulse-report.md` | **新格式** | 取代原 `social-pulse.json` |

---

## 3. 平台清單

採用 `last30days-skill` 原清單，外加 `PTT` 一個台灣來源。Agent 對每個平台自行決定搜尋方式。

| 分類 | 平台 | 自動快掃 | 備註 |
|------|------|:---:|------|
| 台灣社群 | PTT | ✅ | 八卦、Stock、Soft_Job、Tech_Job |
| 台灣社群 | Threads | 📡 | 年輕人主力平台，Agent 自行處理存取 |
| 國際社群 | Reddit | ✅ | 關聯子版（Entrepreneur、marketing、productivity、AI 相關） |
| 國際社群 | X/Twitter | ✅ | 即時熱點 |
| 國際社群 | Bluesky | 📡 | 新興平台 |
| 技術圈 | Hacker News | ✅ | 技術趨勢 |
| 技術圈 | GitHub | ✅ | 開源動態 |
| 技術圈 | arXiv | 📡 | 學術前沿 |
| 技術圈 | Techmeme | 📡 | 科技新聞 |
| 專業圈 | LinkedIn | 📡 | 職場/品牌經營 |
| 影音 | YouTube | 📡 | 影音趨勢 |
| 影音 | TikTok | 📡 | 短影音/年輕人話題 |
| 數據 | Polymarket | 📡 | 市場預測 |
| AI 聚合 | AI HOT | ✅ | aihot.virxact.com，中文 AI 新聞 |
| 一般 | Web | ✅ | Google/Brave 搜尋補漏 |

✅ = 必搜來源
📡 = 能力勾選（Agent 若有工具則加入，無工具則跳過）

---

## 4. 產出格式（全 Markdown）

### 4.1 每日快報：`staging/social-pulse-report.md`

Agent 寫入 Obsidian vault 的 `50_Outputs/staging/social-pulse-report.md`。

```markdown
---
generated_at: 2026-07-30T09:00:00+08:00
source_count: 36
mode: auto
---

# 社群快報 | 2026-07-30

## 🔥 跨平台熱點（3-5 則精選）

**1. Claude Opus 5 系統提示詞完整洩露**
> "30 個工具的 JSON schema...共 135,027 字元，約 3.4 萬 token" — IT 之家
📊 AI HOT 熱度 70 | 🔗 [來源](https://aihot.virxact.com/items/...)
🔑 相關：`prompt-engineering` `leak` `copyright`

**2. OpenAI 與 Anthropic 遊說限制中國開源模型**
> "黃仁勳與馬斯克公開反對...近 200 家矽谷創業公司聯名信" — AI HOT
📊 AI HOT 熱度 71 | 🔗 [來源](https://aihot.virxact.com/items/...)
🔑 相關：`opensource` `china` `ai-regulation`

---

## 🤖 AI 與科技

### Claude 生態
- **Opus 5 洩露**：提示詞完整曝光，含 30 個工具定義與嚴格版權規則（sc 熱度 70）
- **ESP32 跑 LLM**：8 美元微控制器運行 28.9M 參數模型，9.5 tok/s（sc 熱度 76）

### 開源與政策
- ...

## 💡 創業與產品

### Reddit r/Entrepreneur
- **競爭對手直接抄襲你的產品？**：r/Entrepreneur 上熱議，多位創辦人分享真實經驗
  > "3h ago · u/Vinent_Liesa"

### Reddit r/marketing
- ...

## 🇹🇼 台灣社群

### PTT
- **Stock 版**：[話題]... 原始標題

### Threads
（若無回傳則標註：※ Threads 今日資料受限，後續將持續嘗試）

---

## 📋 完整清單

| # | 來源 | 話題 | 熱度 | 相關關鍵字 |
|---|------|------|------|------------|
| 1 | AI HOT | Claude Opus 5 提示詞泄露 | 70 | `prompt` `leak` |
| 2 | AI HOT | 限制中國開源模型 | 71 | `opensource` |
| 3 | PTT | 記憶體7巨頭斷魂 | — | `memory` |
| ... | ... | ... | ... | ... |

共 36 則 | 自動產生於 2026-07-30 09:00
```

### 4.2 Discord 精簡快報

Discord 單一訊息長度限制內（< 2000 字），適合快速掃過。

```
📡 社群情報快報 | 7/30

🔥 Claude Opus 5 提示詞完整泄露（熱度 70）
🔥 大廠遊說限制中國開源 AI（熱度 71）
🔥 ESP32-S3 微控制器跑 LLM（熱度 76）

🤖 AI 科技（12 則）
見完整報告 🔗

💡 創業／產品（8 則）
見完整報告 🔗

🇹🇼 台灣（5 則）
見完整報告 🔗

────────────────────────
共 36 則 | 完整報告：staging/social-pulse-report.md
```

---

## 5. 兩種模式

### 模式 A：定期自動快掃

```
/content-researcher pulse --auto
```

- **頻率**：建議每 6 小時一次
- **搜尋**：`always` 平台（見上表）
- **時間範圍**：最近 24 小時
- **產出**：
  1. 完整報告 → staging/social-pulse-report.md
  2. Discord 精簡版 → 推播

### 模式 B：手動主題查詢

```
/content-researcher pulse <主題>
/content-researcher pulse AI Agent --since 7d
```

- **搜尋**：所有平台（`always` + `optional`）
- **時間範圍**：預設 30 天，可用 `--since` 調整
- **產出**：同一份 Markdown 報告，但聚焦在指定主題
- **用途**：寫作前的深度背景研究

---

## 6. 與本套件的關聯

社群快報是 `content-researcher` 提議主題的**素材來源之一**，不是獨立流程：

```
社群搜尋 (social-pulse-report.md)
    │
    ▼
content-researcher 提議主題
├─ 1. 搜尋知識庫（Wiki、Sources）s）
├─ 2. 交叉比對（排除已覆蓋主題）
├─ 3. 讀取社群快報（從報告提取關鍵字 + 話題）
├─ 4. 提出 3 ~ 5 個主題建議（含 HKR 評分）
│
└─ 使用者選定主題 → 研究報告 → staging/
        │
        ▼ iCloud 同步
content-writer 寫作 → 四層自檢 → 共鳴診斷 → 平台輸出
```

---

## 7. 配置擴充（config.yaml）

擴充現有的 `content-researcher/config.yaml`：

```yaml
# 社群快報設定
social_pulse:
  enabled: true
  auto_interval: 6h
  # Discord 通知
  discord_notify: true
  # 手動查詢預設時間
  default_lookback: 30d
  # 平台啟用
  platforms:
    always: [ptt, threads, reddit, x, hackernews, github, aihot, web]
    optional: [bluesky, linkedin, youtube, tiktok, polymarket, arxiv, techmeme]
```

---

## 8. 舊 REPO 退場

`social-pulse` 獨立倉庫加入 `DEPRECATED.md`：

```markdown
# DEPRECATED

本 REPO 已歸檔。部落快報功能已導入 `context-writer` 的 `content-researcher`skill：
- https://github.com/wolfgangyu/content-writer

新的部落快報是一段純提示指令，不再需要獨立 Python 腳本。
```

---

## 9. 風險與對策

| 風險 | 減緩 |
|------|------|
| Threads 無公開 API，搜尋可能空返回 | Agent 執行時自行嘗試或標註「資料受限」 |
| 純提示無程式碼，不同 Agent 的執行結果不一致 | 若品質不穩，後續可補參考範例（不改核心政策） |
| 一次搜太多平台，Agent 可能超載超時 | 自動快掃只搜 `always` 平台；手動模式才全開 |
| Discord 2000 字限制 | 自動版只放摘要 + 連結到完整報告 |
| 沒有 Python 腳本後，Hermes cron 如何觸發 | 直接排 `/content-researcher pulse --auto` |

---

## 10. 不需做的事（明確排除）

- ❌ 不保留獨立 Python 所取腳本
- ❌ 不輸出機器格式（JSON/XML）
- ❌ 不保留 `social-pulse` 獨立 REPO（歸檔）
- ❌ 不保留 `last30days-skill` 的 LAWs 風格規則
- ❌ 不保留 Queue / Library / Discover 三指令系統
- ❌ 不提供第三方 API keys