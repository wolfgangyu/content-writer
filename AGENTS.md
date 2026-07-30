# content-writer — 雙 Agent 內容創作套件

雙 skill 架構的內容創作套件，透過 Obsidian vault 交接，跨機器（Windows/macOS）協作。

## 速查

| 項目 | 值 |
|------|----|
| 類型 | Claude Code / Hermes skill collection |
| 輸出平台 | Hugo/Hinode（長文）、X/Threads（短帖） |
| 語言 | 繁體中文（台灣用語） |
| 交接目錄 | Obsidian vault `50_Outputs/staging/`（iCloud 同步） |
| Vault 名稱 | `KM` |

## 結構

```
content-writer/
├── SKILL.md              # 套件總覽 + 雙 Agent 交接協議
├── content-researcher/   # Hermes Agent skill
│   ├── SKILL.md          #   → propose / research / pulse
│   └── config.yaml        #   → 品牌關鍵字、社群快報設定
├── content-writer/        # Claude Code skill
│   ├── SKILL.md          #   → 四層自檢 + 共鳴診斷 + 平台輸出
│   └── staging/          #   → 接收 Hermes 研究報告範例
└── docs/
    ├── superpowers/specs/  #   → 設計文件
    ├── social-pulse-deprecated.md
    └── hermes-auto-propose-plan.md
```

## 雙 Agent 架構

```
Hermes (研究員)                 Claude Code (寫作者)
content-researcher              content-writer
├─ pulse --auto → 社群快報      ├─ 讀取 staging/ 研究報告
├─ propose → 提議主題           ├─ 四層自檢 (L1→L2→L3→L4)
├─ research → 產出報告           ├─ 共鳴診斷 (五維)
│              ↓                ├─ Hugo/X Thread 輸出
└─ 寫入 Obsidian staging/ ←── iCloud 同步 ──→ Claude Code 讀取
```

## 工作流速查

```
/content-researcher pulse --auto     # 社群快報（6h 間隔，自動推 Discord）
/content-researcher pulse <主題>     # 手動主題查詢（30 天範圍）
/content-researcher propose          # 從知識庫提議 3-5 個主題
/content-researcher research <主題>   # 深度研究產出報告

/content-writer                      # 從 staging/ 報告開始寫作
/content-writer 幫我寫一篇<主題>文章   # 直接寫作
```

## 社群快報（Social Pulse）

之前是獨立的 `social-pulse` REPO（Python 腳本 + JSON 輸出），已被歸入 content-researcher 為 `pulse` 指令。15 個平台平行搜尋，全 Markdown 產出。舊 REPO 已歸檔：`github.com/wolfgangyu/social-pulse`。

## 交接協議

- content-researcher 輸出 → `{KM-vault}/50_Outputs/staging/<topic-slug>.yaml`
- content-writer 讀取同上目錄
- Vault 路徑自動偵測（macOS iCloud / Windows iCloudDrive / WSL），找不到時詢問

## Do NOT introduce

- 硬寫死 vault 路徑（平台不同路徑差異大，用自動偵測）
- 用 AI 編造第一手經歷與情緒（活人感終審會擋下）
- 產出 JSON/YAML 機器格式取代 Markdown（人讀不可瀏覽）
- 獨立 Python 外部依賴（純 prompt 即可）
- 把歷史敘事塞進本檔案頂部（→ git log / docs）

## 深入文件

| 想了解 | 去這裡 |
|--------|-------|
| 四層自檢（L1-L4） | `content-writer/SKILL.md` § Step 5 |
| HKR 多維評分 | `content-researcher/SKILL.md` § 步驟 3 |
| 社群快報設計 | `docs/superpowers/specs/2026-07-30-social-pulse-integration-design.md` |
| social-pulse 歸檔 | `docs/social-pulse-deprecated.md` / `github.com/wolfgangyu/social-pulse` |
| Hermes 自動提議 | `docs/hermes-auto-propose-plan.md` |

> 版本：v0.3.0 · 最後更新 2026-07-30