---
name: content-writer
description: |
  內容創作夥伴套件。由 content-researcher（Hermes Agent 用，從 Obsidian 知識庫自動提議主題）和 content-writer（Claude Code 用，協作式寫作 → 診斷 → 輸出）組成。
  觸發方式：/content-writer、/寫作夥伴、「幫我寫一篇文章」「從知識庫提議文章主題」
  Content creation partner suite. content-researcher (Hermes) proposes topics from Obsidian wiki; content-writer (Claude Code) co-writes with warm feedback, resonance diagnosis, and platform output.
  Trigger: /content-writer, "help me write an article", "suggest article topics from my knowledge base"
---

# 內容創作夥伴套件

這是一套雙 skill 架構，分別針對不同 agent 最佳化：

| Skill | Agent | 職責 |
|-------|-------|------|
| `content-researcher` | Hermes Agent | 從 Obsidian 知識庫自動提議主題、產出研究報告 |
| `content-writer` | Claude Code | 接收報告 → 協作寫作 → 共鳴診斷 → 平台輸出 |

## 交接協議

兩個 skill 透過標準化 YAML frontmatter 交接：

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

### 在 Claude Code 中

```bash
# 從研究報告開始
content-writer/research-topic.yaml  # 附上 Hermes 產出的報告

# 或直接開始寫作
「幫我寫一篇關於 [主題] 的文章」
```

### 在 Hermes Agent 中

```bash
# 自動提議
content-researcher/propose            # 掃描知識庫提議主題

# 產出研究報告
content-researcher/research <topic>   # 針對選定主題產出報告
```

## 核心原則

1. **溫暖引導型** — 建議而非命令，詢問而非假設
2. **台灣繁體中文** — 用語、範例、文化參考都台灣化
3. **知識庫感知** — 不建議寫過的事，利用現有 Wiki 和 Sources
4. **雙平台輸出** — 一次寫作，自動產出 Hugo + X Thread
5. **診斷不批判** — 用共鳴框架診斷，但語氣像寫作教練而非編輯

## 不知道下一步？

回到套件總覽：輸入 `/content-writer`，我會根據你目前的階段給建議。
