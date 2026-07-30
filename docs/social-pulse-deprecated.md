# DEPRECATED — social-pulse 獨立 REPO 已歸檔

**歸檔日期**：2026-07-30
**原 REPO**：`wolfgangyu/social-pulse`（未在 GitHub 上找到，可能從未被推到遠端）

## 遷移說明

社群快報功能已完全遷入 [content-writer](https://github.com/wolfgangyu/content-writer) 的 `content-researcher` skill。

### 舊架構

```
social-pulse (獨立 Python REPO)
├─ social-pulse.py                ← Python 腳本
├─ cron: 每日 07:00               ← Betelgeuse 機器
├─ 依賴: firecrawl, opencc
├─ 產出 → staging/social-pulse.json
│           ↓ iCloud 同步
└─ content-researcher 讀取 JSON
```

### 新架構

```
content-researcher (content-writer REPO)
├─ SKILL.md                       ← 「社群快報」章節
├─ config.yaml                    ← 社群快報設定
│
├─ 模式 A: 定期自動快報
│   └─ Hermes cron → /content-researcher pulse --auto
│        → 產出 social-pulse-report.md → staging/
│        → Discord 精簡快報 → 推播
│
└─ 模式 B: 手動查詢
    └─ /content-researcher pulse <主題> [--since Nd]
```

### 主要改變

| 舊 | 新 |
|---|---|
| Python 腳本 + 外部依賴 | 純 prompt 指令（Agent 自行處理） |
| 產出 JSON | 全 Markdown |
| 獨立 REPO | content-researcher 的子功能 |
| betelgeuse cron | Hermes cron |
| 只支援 PTT 轉換 | 多平台平行搜尋 |

### 詳情

見設計文件：[docs/superpowers/specs/2026-07-30-social-pulse-integration-design.md](docs/superpowers/specs/2026-07-30-social-pulse-integration-design.md)