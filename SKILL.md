---
name: content-writer
description: |
  內容創作夥伴套件。由 content-researcher（Hermes Agent 用，從 Obsidian 知識庫自動提議主題）和 content-writer（Claude Code 用，協作式寫作 → 四層自檢 → 共鳴診斷 → 平台輸出）組成。
  觸發方式：/content-writer、/寫作夥伴、「幫我寫一篇文章」「從知識庫提議文章主題」
  Content creation partner suite. content-researcher (Hermes) proposes topics from Obsidian wiki; content-writer (Claude Code) co-writes with 4-layer self-check, resonance diagnosis, and platform output.
  Trigger: /content-writer, "help me write an article", "suggest article topics from my knowledge base"
---

# 內容創作夥伴套件

這是一套雙 skill 架構，分別針對不同 agent 最佳化：

| Skill | Agent | 職責 |
|-------|-------|------|
| `content-researcher` | Hermes Agent | 從 Obsidian 知識庫自動提議主題、產出研究報告 |
| `content-writer` | Claude Code | 接收報告 → 四層自檢 → 共鳴診斷 → 平台輸出 |

## 架構圖

```
┌─────────────────────┐         ┌──────────────────────┐
│  Hermes Agent       │         │  Claude Code (x2)    │
│  content-researcher │         │  content-writer      │
│                     │         │                      │
│  掃描 Obsidian      │         │                      │
│  提議主題           │         │                      │
│  產出研究報告       │────────▶│  讀取研究報告         │
│  寫入 staging/      │  Obsidian iCloud 同步         │  協作寫作
│                     │         │  輸出成品到 Outputs  │
└─────────────────────┘         └──────────────────────┘
```

## 交接協議

Hermes（content-researcher）與 Claude Code（content-writer）透過 **Obsidian vault 的 `50_Outputs/staging/` 目錄** 交接。研究報告以標準化 YAML frontmatter 存放，透過 iCloud 同步到所有 Claude Code 機器。

### Vault 路徑自動偵測

由於 Hermes 和 Claude Code 可能執行在 macOS / Windows / WSL (Ubuntu) 等不同平台，**請勿寫死路徑**。執行時依以下步驟定位 Obsidian vault：

1. 檢查當前工作目錄或其父目錄是否包含 `.obsidian/` 資料夾 → 即為 vault 根目錄。
2. 若無，依平台搜尋 iCloud 同步位置：
   - **macOS**：`~/Library/Mobile Documents/iCloud~md~obsidian/Documents/` 下找名為 `KM` 的 vault。
   - **Windows**：`%USERPROFILE%/iCloudDrive/iCloud~md~obsidian/` 下找 `KM`。
   - **WSL/Ubuntu**：從 `/mnt/c/Users/<使用者>/iCloudDrive/iCloud~md~obsidian/` 或 `$HOME/iCloudDrive/` 下找 `KM`。
3. 若上述都沒有，改為掃描 `$HOME`、`~/Documents`、`~/Obsidian` 等常見位置，尋找含有 `KM` 且內有 `.obsidian/` 的目錄。
4. 完全找不到時，**詢問使用者**提供 vault 絕對路徑。

> 找到 vault 後，交接目錄一律為 `{vault}/50_Outputs/staging/`。

### 雙 skill 相依 Claude Code

content-writer 依賴 Claude Code。如果 Claude Code 不可用，可改用 Hermes 單 agent 執行完整流程（提議 → 研究 → 寫作 → 輸出），詳細替代方案請見 [content-researcher/SKILL.md](content-researcher/SKILL.md)。

```yaml
# content-researcher 輸出 → content-writer 輸入
---
topic: "文章標題"
target_audience: "受眾描述"
platforms: ["hugo", "x-thread"]
tone: "溫暖引導型"
outline:
  - section: "標題"
    key_points: ["要點1", "要點2"]
    sources:
      - file: "40_Wiki/Loop_Engineering.md"
        excerpt: "摘錄內容"
draft_status: "pending"
generated_at: "2026-07-11T12:00:00+08:00"
---
```

## 使用方式

### 在 Hermes Agent 中（研究員）

```bash
/content-researcher propose    # 自動提議主題
/content-researcher research <主題>  # 產出研究報告
```

報告會寫入 Obsidian `50_Outputs/staging/`，iCloud 同步後 Claude Code 即可讀取。

### 在 Claude Code 中（寫作者）

```bash
# 從研究報告開始寫作
「讀取 staging 中的最新報告開始寫作」

# 或直接開始寫作
「幫我寫一篇關於 [主題] 的文章」
```

成品輸出回 Obsidian `50_Outputs/`，iCloud 同步後 Hermes 也能看到。

## 核心原則

1. **溫暖引導型** — 建議而非命令，詢問而非假設
2. **台灣繁體中文** — 用語、範例、文化參考都台灣化
3. **知識庫感知** — 不建議寫過的事，利用現有 Wiki 和 Sources
4. **雙平台輸出** — 一次寫作，自動產出 Hugo + X Thread
5. **診斷不批判** — 用共鳴框架診斷，但語氣像寫作教練而非編輯

## 不知道下一步？

回到套件總覽：輸入 `/content-writer`，我會根據你目前的階段給建議。
