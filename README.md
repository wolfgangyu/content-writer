# content-writer 套件

融合 content-researcher（Hermes Agent 用，從 Obsidian 知識庫自動提議主題）和 content-writer（Claude Code 用，協作式寫作 → 四層自檢 → 共鳴診斷 → 平台輸出）的台灣本地化內容創作套件。

## 結構

```
content-writer/
├── SKILL.md                        # 套件總覽 + 交接協議
├── content-writer/
│   ├── SKILL.md                    # Claude Code 用：寫作技能
│   └── staging/                    # 接收 Hermes 產出的研究報告
└── content-researcher/
    └── SKILL.md                    # Hermes Agent 用：研究技能
```

## 設計理念

- **溫暖引導型** — 診斷有深度，語氣像寫作教練而非犀利編輯
- **Obsidian 原生** — 直接讀取你的知識庫結構，支援 macOS + Windows
- **雙平台輸出** — Hugo/Hinode 長文 + X/Threads 短帖
- **模組化** — 兩個 skill 獨立運作，透過標準化 YAML 交接
- **四層自檢** — L1 硬性規則 → L2 風格一致性 → L3 內容質量 → L4 活人感終審
- **HKR 評題** — 每個提議主題附 Happy / Knowledge / Resonance 三維評分
- **人來人AI** — 明確界定哪些交給 AI、哪些必須你親手寫

## 安裝

### Claude Code

```bash
# 機器 1 & 2
ln -sf /path/to/content-writer/content-writer ~/.claude/skills/content-writer
```

### Hermes Agent

```bash
# Hermes 所在機器
ln -sf /path/to/content-writer/content-researcher ~/.hermes/skills/content-researcher
```

### 跨機器同步

由於 Hermes 與 Claude Code 在三台不同機器上，**不需要橋接腳本**。交接透過 Obsidian vault 的 iCloud 同步完成：

```
Hermes 寫入 staging/  →  iCloud 同步  →  Claude Code 讀取
Claude Code 寫入 Outputs/  →  iCloud 同步  →  Hermes 讀取
```

### 注意事項

- 確保所有機器都能存取同一份 Obsidian vault（macOS iCloud 或 Windows iCloud Drive）
- 每個 SKILL.md 中已註明 macOS / Windows 的實際路徑
- content-writer 依賴 Claude Code，若不可用可改用 Hermes 單 agent 執行完整流程（詳見 [content-researcher/SKILL.md](content-researcher/SKILL.md)）

## 寫作流程

```
content-researcher (Hermes)
  1. 掃描知識庫 + 社群熱點
  2. HKR 評分提議主題
  3. 產出研究報告 → staging/
       ↓  Obsidian 同步
content-writer (Claude Code)
  4. 讀取研究報告
  5. AI 角色邊界界定
  6. 協作式大綱 → 分段寫作
  7. 四層自檢（L1 硬性 → L2 風格 → L3 內容 → L4 活人感）
  8. 共鳴診斷（五維框架）
  9. 平台輸出（Hugo / X Thread）
```

## 版本

v0.2.0 — 2026-07-12
