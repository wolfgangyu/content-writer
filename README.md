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
ln -s /path/to/content-writer/content-writer ~/.claude/skills/content-writer
```

### Hermes Agent

```bash
ln -s /path/to/content-writer/content-researcher ~/.hermes/skills/content-researcher
```

或使用 dbs-bridge 風格的橋接腳本：

```bash
./content-writer/scripts/bridge.sh link content-writer
./content-writer/scripts/bridge.sh link content-researcher
```

## 版本

v0.1.0 — 2026-07-11
