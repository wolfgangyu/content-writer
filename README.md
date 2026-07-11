# content-writer 套件

融合 content-research-writer（協作式寫作流程）與 dbskill（診斷框架）的台灣本地化內容創作套件。

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
- **Obsidian 原生** — 直接讀取你的知識庫結構
- **雙平台輸出** — Hugo/Hinode 長文 + X/Threads 短帖
- **模組化** — 兩個 skill 獨立運作，透過標準化 YAML 交接

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

- 確保三台機器都能存取同一份 Obsidian vault（iCloud 同步路徑）
- 如果 Obsidian vault 路徑不同，每個 SKILL.md 中已註明實際位置
- 不需要 dbs-bridge 式的軟鏈接，因為交接層在 Obsidian 而非 agent 系統目錄

## 版本

v0.1.0 — 2026-07-11
