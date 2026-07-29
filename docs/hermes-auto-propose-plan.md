# Hermes 自動提議機制優化計畫

**目標**：提升 Hermes 自動提議的品牌一致性、時效性和差異化，確保內容與社群/Blog 經營目標對齊。

**範圍**：
- 調整 `content-researcher` 的提議邏輯。
- 改進 `social-pulse` 的輸出格式。
- 確保跨機器交接流程順暢。

---

## 階段 1：基礎設定（1-2 天）
**目的**：確保路徑正確、配置文件就緒，為後續優化打基礎。

### 任務 1.1：更新路徑設定
- **行動**：
  - 修改 `content-researcher/SKILL.md` 和 `content-writer/SKILL.md`，確保 Windows 路徑正確指向：
    ```
    C:\Users\chimi\iCloudDrive\iCloud~md~obsidian\KM\50_Outputs\staging
    ```
  - 確認 macOS 路徑（如有需要）同步更新。
- **驗證**：
  - 在 Hermes 和 Claude Code 中手動觸發 `/content-researcher propose`，確認報告正確寫入 `staging/`。

### 任務 1.2：新增配置檔
- **行動**：
  - 在 `content-researcher/` 目錄下新增 `config.yaml`，定義：
    - 品牌關鍵字白名單（`brand_keywords`）。
    - 自動提議頻率（`auto_propose.interval`）。
    - 熱點過期時間（`trend_expiry_days`）。
  - 範例：
    ```yaml
    brand_keywords:
      - "AI"
      - "Agent"
      - "醫療"
      - "產品管理"
      - "循環經濟"
      - "永續"
    auto_propose:
      enabled: true
      default_interval: 7d
      dynamic_adjust: true
    trend_expiry_days: 7
    ```
- **驗證**：
  - 確認 Hermes 讀取 `config.yaml` 時不報錯。

---

## 階段 2：改進 `social-pulse`（2-3 天）
**目的**：提升熱點資料的可讀性和時效性，對齊 `last30days-skill` 的設計。

### 任務 2.1：調整輸出格式
- **行動**：
  - 將 `social-pulse.json` 改為 YAML 格式，加入：
    - `expires_at`（熱點過期時間）。
    - `sentiment`（情緒分析）。
    - `related_keywords`（相關關鍵字）。
  - 範例：
    ```yaml
    ---
    generated_at: "2026-07-30T09:00:00+08:00"
    platforms:
      - name: "X (Twitter)"
        trends:
          - keyword: "AI Agent"
            volume: 1500
            related_keywords: ["Multi-agent", "Claude", "Autogen"]
            sentiment: "positive"
            expires_at: "2026-08-02T09:00:00+08:00"
    ```
- **驗證**：
  - 執行 `social-pulse` cron job，確認輸出符合新格式。

### 任務 2.2：新增可視化報告
- **行動**：
  - 新增 `social-pulse-report.md`，自動產生：
    - 熱點趨勢圖（ASCII 或 Mermaid）。
    - 與 KM Wiki 的交叉比對結果。
  - 範例：
    ```markdown
    # 社群熱點報告 (2026-07-30)
    ## 熱點趨勢
    ```mermaid
    graph TD
      A[AI Agent] -->|1500 提及| B[高熱度]
      C[永續醫療] -->|892 提及| D[中熱度]
    ```
    
    ## 與 KM Wiki 相關
    - **AI Agent**：與 `40_Wiki/AI_Agent.md` 相關。
    - **永續醫療**：與 `40_Wiki/循環經濟.md` 相關。
    ```
- **驗證**：
  - 確認報告自動更新且內容正確。

---

## 階段 3：優化 Hermes 提議邏輯（3-4 天）
**目的**：提升提議的品牌一致性、時效性和差異化。

### 任務 3.1：品牌關鍵字過濾
- **行動**：
  - 修改 `content-researcher` 的提議邏輯，只提議包含 `brand_keywords` 的主題。
  - 範例程式碼（偽代碼）：
    ```python
    def is_brand_aligned(topic):
        return any(keyword in topic for keyword in config["brand_keywords"])
    ```
- **驗證**：
  - 執行 `/content-researcher propose`，確認提議主題均符合品牌關鍵字。

### 任務 3.2：熱點時效性過濾
- **行動**：
  - 優先提議 `social-pulse-report.md` 中「未來 3 天預測」的熱點。
  - 排除已過期的熱點（`expires_at` < 當前時間）。
  - 範例程式碼（偽代碼）：
    ```python
    def filter_trends(trends):
        now = datetime.now()
        return [t for t in trends if t["expires_at"] > now]
    ```
- **驗證**：
  - 確認提議主題均為有效熱點。

### 任務 3.3：調整 HKR 評分
- **行動**：
  - 修改 HKR 評分維度，強化品牌一致性：
    - **Happy**：是否能吸引目標受眾？
    - **Knowledge**：是否能展現專業深度？
    - **Resonance**：是否能強化品牌形象？
  - 範例：
    ```yaml
    hkr_score:
      happy: "高 — 故事性強，能吸引 AI 從業者"
      knowledge: "高 — 有獨家見解，展現技術深度"
      resonance: "高 — 與品牌定位一致"
    ```
- **驗證**：
  - 執行 `/content-researcher propose`，確認評分邏輯正確。

---

## 階段 4：整合與測試（2 天）
**目的**：確保所有改動協同運作，並驗證效果。

### 任務 4.1：端到端測試
- **行動**：
  - 手動觸發 Hermes 自動提議（`/content-researcher propose`）。
  - 確認：
    1. 報告正確寫入 `staging/`。
    2. Claude Code 能讀取報告並開始寫作。
    3. 提議主題符合品牌關鍵字和熱點時效性。
- **驗證**：
  - 檢查 `staging/` 和 `50_Outputs/` 的檔案。

### 任務 4.2：使用者反饋機制
- **行動**：
  - 新增提議後詢問使用者評分（1-5 分）：
    ```
    這個主題符合你的需求嗎？ (1-5 分)
    ```
  - 將反饋記錄到 `feedback.log`，用於後續優化。
- **驗證**：
  - 確認反饋記錄正確寫入 `feedback.log`。

---

## 階段 5：文件更新（1 天）
**目的**：確保團隊和未來的你理解新邏輯。

### 任務 5.1：更新 `SKILL.md`
- **行動**：
  - 在 `content-researcher/SKILL.md` 中補充：
    - 自動提議的觸發條件。
    - HKR 評分的新定義。
    - `social-pulse` 的改進說明。
- **驗證**：
  - 確認文件清晰易懂。

---

## 時間表
| 階段 | 任務 | 預計時間 | 依賴關係 |
|------|------|----------|----------|
| 1 | 基礎設定 | 1-2 天 | - |
| 2 | 改進 `social-pulse` | 2-3 天 | 階段 1 |
| 3 | 優化提議邏輯 | 3-4 天 | 階段 2 |
| 4 | 整合與測試 | 2 天 | 階段 3 |
| 5 | 文件更新 | 1 天 | 階段 4 |

---

## 風險與緩解
| 風險 | 影響 | 緩解措施 |
|------|------|----------|
| 路徑錯誤 | 報告無法交接 | 階段 1 驗證路徑正確性。 |
| `social-pulse` 格式不兼容 | Hermes 無法讀取 | 階段 2 確認輸出格式。 |
| 提議邏輯過濾過嚴 | 提議主題過少 | 階段 3 調整關鍵字白名單。 |
| 使用者反饋不足 | 無法優化模型 | 階段 4 主動詢問反饋。 |