---
title: "沒有真實 stopping condition 的迴圈會靜悄悄地失敗"
date: 2026-07-11
author: "Wolfgang"
description: "AI agent 自動化迴圈中最容易被忽略的一塊拼圖：stopping condition。沒有它的迴圈不會爆炸，它只會靜悄悄地浪費 token 和時間。"
tags: ["AI-agent", "loop-engineering", "hugin", "obsidian", "knowledge-management"]
category: tutorial
draft: true
thumbnail:
  url: img/notepad.jpg
  author: "Frederick Medina"
  authorURL: "https://unsplash.com/@frederickjmedina"
  origin: "Unsplash"
  originURL: "https://unsplash.com/photos/PdfRE-xB--s"
---

# 沒有真實 stopping condition 的迴圈會靜悄悄地失敗

> **摘要**：這篇文章探討 AI agent 自動化迴圈中「stopping condition」的重要性。核心論點：沒有真實 stopping condition 的迴圈不會爆炸，它只會靜悄悄地浪費 token 和時間。

## 引言：你以為在跑迴圈，其實在空轉

上週我在 Hermes Agent 上跑了一個 CI failure triage 的 loop。設定很簡單：每 5 分鐘掃描一次 failing tests，自動分類並開 fix PR。

三天後我回頭看，發現它跑了 144 次。

不是因為有 144 個 bug 要修。是因為連續兩天沒有任何 failing test，但 loop 的 stopping condition 是「時間」——每 5 分鐘跑一次，跑滿 24 小時就停。

**它沒有靜悄悄地失敗。它靜悄悄地浪費了 144 次 token。**

這不是個例。大多數人設定 loop 時只關心「啟動」，不關心「停止」。

## 什麼是 stopping condition？

stopping condition 不是一個布林值。它是一組可驗證的狀態檢查，回答三個問題：

1. **做完了嗎？** — 所有項目都處理過了？
2. **做好了嗎？** — 每個項目的品質達標？
3. **還要繼續嗎？** — 還有新的東西要處理？

常見的錯誤：用時間當 stopping condition。

「跑 5 分鐘」vs「做完為止」。這兩者之間的差別是：時間不關心任務有沒有真的做完。

## 三種 stopping condition 模式

### 模式 1：數量型

> 「做完 10 個 issue 為止」

簡單，但可能不夠（10 個不夠）或太多（10 個太多了）。

**適用場景**：任務明確、數量可預測。

### 模式 2：質量型

> 「所有測試通過且 linter 無警告為止」

可靠，但需要好的驗證器。如果驗證器本身有 bug，你就陷入了無限迴圈。

**適用場景**：有自動化測試的程式碼任務。

### 模式 3：混合型

> 「做完 20 個或連續 3 次沒有找到新 bug 為止」

兼顧效率與覆蓋率。20 個保證最低工作量，連續 3 次空結果保證不會無限跑。

**適用場景**：探索型任務（bug hunting、security scan）。

## 反方：有時候你不需要 stopping condition

Karpathy 的 LLM Wiki 模式中，Q&A 和 Lint 是持續進行的，沒有明確的 stopping condition。為什麼？因為它們是**探索型任務**。

你無法預知一個知識庫明天會不會有新文章要 ingest。你無法預知一個迴圈明天會不會有新的 lint 問題要修。

**關鍵是區分：你的任務是執行型還是探索型？**

| 類型 | 需要 stopping condition？ | 範例 |
|------|-------------------------|------|
| 執行型 | 是 | CI triage、dependency bump、lint-and-fix |
| 探索型 | 否（用時間或人工干預） | Wiki ingest、daily notes scan、research |

## 實作：在你的 loop 中加 stopping condition

檢查你的 loop，問自己三個問題：

1. **你知道它什麼時候該停嗎？** — 如果答案是「我不知道」，你需要加 stopping condition
2. **你設定的 stopping condition 是狀態檢查還是時間？** — 狀態檢查 > 時間
3. **如果驗證器失敗了，loop 會停嗎？** — 如果不會，你需要 fallback stopping condition

### 檢查清單

- [ ] 我的 loop 有 stopping condition 嗎？
- [ ] stopping condition 檢查的是狀態，不是時間？
- [ ] 如果驗證器失敗，loop 有 fallback 嗎？
- [ ] 我設定了 token budget 嗎？
- [ ] 我知道什麼情況下應該手動停掉它嗎？

## 結語：設計 stopping condition 是 loop engineering 的最後一塊拼圖

Loop Engineering 的四條件測試（Repetition / Automated verification / Token budget / Senior tools）中，token budget 本身就是一種 stopping condition。

但大多數人沒有設定 budget，或者設得太高。

**下一個 loop 啟動前，先問自己：它什麼時候該停？**

這個簡單的習慣，可以幫你省下數百個 token 和無數個「為什麼這個迴圈還在跑？」的疑問。

---

> **參考**：[[Loop_Engineering]]、[[LLM_Wiki_Pattern]]、[[Claude_Loops]]
>
> **草稿狀態**：初稿完成，待審閱
> **來源**：研究報告 `example-stopping-condition.yaml`
