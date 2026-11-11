# AirGuard AI: 工具調度代理人與短程空污預報之個人化戶外活動決策系統

> **AirGuard AI: A Tool-Orchestrated Agent for Short-Term Air Quality Forecasting and Personalized Outdoor Activity Guidance**  
> 本專案採用 **OpenSpec (Spec-Driven Development, 規格驅動開發)** 進行學術計畫書重構與系統規格工程化。

[![OpenSpec](https://img.shields.io/badge/OpenSpec-v1.17.0-blue.svg)](https://openspec.dev)
[![Architecture](https://img.shields.io/badge/Architecture-Tool--Orchestrated-green.svg)](#系統架構)
[![GitHub](https://img.shields.io/badge/GitHub-kashak068%2Fdic3-lightgrey.svg)](https://github.com/kashak068/dic3.git)

---

## 目錄

- [專案簡介](#專案簡介)
- [研究核心 (A–F 框架與 5 Cores)](#研究核心-af-框架與-5-cores)
- [三大技術權衡 (Trade-offs)](#三大技術權衡-trade-offs)
- [系統架構圖](#系統架構圖)
- [OpenSpec 規格體系導覽](#openspec-規格體系導覽)
- [學術文件導覽](#學術文件導覽)
- [文獻比較矩陣](#文獻比較矩陣)
- [常用操作指令](#常用操作指令)
- [版本里程碑 (Git Checkpoints)](#版本里程碑-git-checkpoints)

---

## 專案簡介

當前環境空氣品質資訊多為「被動式、滯後一小時之測站歷史數值廣播」，且通用大型語言模型（LLMs）缺乏即時動態數據支援，容易在公眾健康領域產生「數值與醫療幻覺」。

**AirGuard AI** 提出全新的典範轉移：
1. **即時環境管線**：串接台灣環境部（MOENV）測站即時與歷史觀測 API。
2. **超短期預報**：以 LightGBM 機器學習模型提供未來 **1–6 小時** 之 $\text{PM}_{2.5}$、$\text{PM}_{10}$、$\text{O}_3$ 濃度多步預測（推論延遲 $< 200\text{ms}$）。
3. **工具調度代理人**：透過 Pydantic 強型別 Function Calling，自主調用測站數據與預報工具。
4. **雙層確定性安全護欄**：將台灣環境部官方 AQI 活動健康防護規範程式化，並於生成回覆前執行數值對齊審查（Post-generation Fact-checking Guard），杜絕幻覺。

---

## 研究核心 (A–F 框架與 5 Cores)

### 1. A–F 研究邏輯

```
[A] 誘餌 (Attention Getter)   戶外運動換氣量劇增帶來的突發性空污極端暴露風險
       │
       ▼
[B] 衝突 (But)                觀測滯後性 + 齊頭式廣播 + 通用 LLM 醫療數值幻覺
       │
       ▼
[C] 解方 (Cure)               環境即時管線 + 1-6h 輕量預報 + 官方規則 + 工具調度代理人
       │
       ▼
[D] 開發 (Development)        FastAPI 異步微服務 + Pydantic Function Calling + 雙層安全護欄
       │
       ▼
[E] 實驗 (Experiments)        2024–2026 台中測站滾動回測 (MAE/RMSE) + 30 組情境基準測試
       │
       ▼
[F] 發現 (Findings)           0% 數值幻覺率 + 100% 官方標準依從率 + 推論延遲 < 200ms
```

### 2. 五大研究核心 (5 Cores)
1. **工具落地自主調度 (Tool-Grounded Orchestration)**：杜絕依賴模型內部參記憶體，所有數據強制源自即時工具調用。
2. **短程 1–6 小時動態預報 (1–6h Horizon Forecasting)**：變革被動看盤模式，提供出門前瞻暴露預期。
3. **多因子個人化風險綜合評估**：量化融合使用者易感分級（氣喘、長者、兒少）與活動代謝當量（METs）。
4. **確定性安全硬護欄 (Deterministic Safety Guardrails)**：官方醫療健康防護標準硬編碼，遇嚴重污染強制熔斷。
5. **可重現之滾動回測與基準測試**：以 2024–2026 台中數據為實證基線，釋出 30 組對話情境測試集。

---

## 三大技術權衡 (Trade-offs)

| 權衡維度 | 衝突極端 | AirGuard AI 的工程決策與解法 |
| :--- | :--- | :--- |
| **1. 預測精度 vs. 推論延遲** | 複雜時空圖神經網路 (ST-GNN) vs. 輕量回歸 | 採用 **LightGBM 多步回歸 + 10 分鐘滑動特徵快取**，單次推論延遲 $< 200\text{ms}$，在保障對話流暢度的同時具備出色的 MAE/RMSE。 |
| **2. 對話彈性 vs. 臨床安全** | 自由發揮同理心對話 vs. 呆板機械式警示 | 採用**「語氣軟生成、數值與規則硬約束」**：LLM 負責同理心互動，底層由確定性規則引擎與生成後校驗器把關醫療邊界。 |
| **3. 個人化粒度 vs. 隱私保護** | 常駐 GPS 軌跡追蹤 vs. 完全匿名無脈絡 | 採用**臨時會話級生理特徵抽象標籤**（如敏感分類），會話結束即銷毀，不持久化儲存可識別個人資訊（PII）。 |

---

## 系統架構圖

```
                      [ 使用者自然語言輸入 ]
                                │
                                ▼
┌─────────────────────────────────────────────────────────────┐
│ 1. 互動與對話代理層 (Agentic Interaction Layer)            │
│    - FastAPI 異步路由 & WebSocket 通訊                      │
│    - LLM (ReAct 思考鏈 + Pydantic Function Calling)         │
│    - 會話狀態與意圖解析 (抽取地點、時間、活動類型、敏感群體) │
└──────────────┬──────────────────────────────▲───────────────┘
               │ 結構化工具調用                │ 觀測/預測/規則結果
               ▼                              │
┌─────────────────────────────────────────────┴───────────────┐
│ 2. 確定性服務與預測引擎層 (Deterministic Services Layer)    │
│    - 數據服務: MOENV API 串接、異常清洗、時序滑動特徵快取   │
│    - 預測引擎: 1–6 小時 LightGBM 多步回歸 + 信心區間計算     │
│    - 規則引擎: 台灣環境部官方 AQI 活動防護規範對照矩陣      │
└─────────────────────────────────────────────┬───────────────┘
                                              │ 生成初步回覆
                                              ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. 雙層安全驗證攔截層 (Safety & Guardrail Layer)            │
│    - 數值對齊校驗器: 檢查生成文本與工具回傳數據之一致性     │
│    - 臨床安全熔斷器: 嚴重污染時強制覆蓋為官方安全模板       │
└─────────────────────────────────────────────┬───────────────┘
                                              │ 安全回覆
                                              ▼
                       [ 使用者行動決策回饋 ]
```

---

## OpenSpec 規格體系導覽

本專案完全遵循 **OpenSpec** 規範驅動開發流程，所有規劃與規格均位於 [`openspec/`](file:///c:/Users/User/Desktop/00/dic3/openspec/)：

```
openspec/
├── config.yaml                            # OpenSpec 專案配置 (schema: spec-driven)
└── changes/
    └── airguard-core-spec/                # 核心架構變更提案
        ├── proposal.md                    # 提案動機、範圍與能力清單
        ├── design.md                      # 技術決策、架構設計與 3 大 Trade-offs
        ├── tasks.md                       # 5 大階段實作任務與檢核清單
        └── specs/                         # 5 大核心能力規範 (Delta Specs)
            ├── air-quality-data/spec.md   # 環境數據串接、清洗與快取規格
            ├── forecasting/spec.md        # 1-6 小時時序預測與延遲約束規格
            ├── ai-agent/spec.md           # Function Calling 與對話編排規格
            ├── activity-guidance/spec.md  # 個人化活動防護與排程建議規格
            └── safety-evaluation/spec.md  # 官方規則硬護欄與數值校驗規格
```

---

## 學術文件導覽

學術計畫書詳細內容撰寫於 [`docs/`](file:///c:/Users/User/Desktop/00/dic3/docs/) 目錄：

* **[docs/00_project_overview.md](file:///c:/Users/User/Desktop/00/dic3/docs/00_project_overview.md)**：專案綜合總覽、A–F 框架摘要與驗證報告。
* **[docs/01_introduction.md](file:///c:/Users/User/Desktop/00/dic3/docs/01_introduction.md)**：學術第一章「緒論」，涵蓋研究背景、問題陳述、動機、3 大研究缺口、5 項研究目標、提出方法與 5 大預期貢獻。
* **[docs/02_related_work.md](file:///c:/Users/User/Desktop/00/dic3/docs/02_related_work.md)**：學術第二章「相關文獻探討」，綜述時序預報、環境決策系統、LLM 工具調度、防範幻覺與可信任 AI 護欄，並附五方系統比較矩陣。

---

## 文獻比較矩陣

| 評估維度 | 官方監測 App | 通用 LLM (ChatGPT等) | 傳統靜態 RAG | 數值模式 (WRF-Chem) | **AirGuard AI (本專案)** |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **時間前瞻性** | 僅滯後 1 小時觀測值 | 受限訓練截止日，無即時性 | 無法預測未來趨勢 | 支援 24–72 小時預報 | **精確 1–6 小時短程高頻預報** |
| **推論延遲** | 毫秒級 (查表) | 1–3 秒 (文字生成) | 1–2 秒 | 數小時 (大規模偏微分) | **< 200ms (預測工具) + Agent 流式** |
| **個人化計算** | 無 (全體通用廣播) | 具對話能力，但無客觀數值 | 僅限文字語義比對 | 無 (純大氣格網) | **綜合評估病史 + 運動 MET 代謝** |
| **數值真實性** | 100% 真實量測 | 頻繁產生數值幻覺 | 偶發事實抽取錯誤 | 具物理守恆依據 | **100% 工具落地 + 生成後嚴格比對** |
| **醫療安全防護** | 僅靜態文字警示 | 軟提示詞約束，易被突破 | 無即時規則邊界 | 無直接決策機制 | **官方規則硬編碼 + 嚴重污染強制熔斷** |
| **互動形式** | 圖表 / 數據儀表板 | 自然語言多輪對話 | 靜態 Q&A 問答 | 專業大氣物理圖譜 | **自然語言對話 + 具體排程行動指南** |

---

## 常用操作指令

### 1. OpenSpec 驗證與狀態
```powershell
# 檢視當前變更進度
openspec status --change airguard-core-spec

# 執行全域規格驗證
openspec validate --all

# 檢視規格清單
openspec list --specs
```

### 2. Git 操作
```powershell
# 查看遠端同步狀態
git status

# 拉取最新提交
git pull origin main
```

---

## 版本里程碑 (Git Checkpoints)

* `7ba681b` — `chore(openspec): initialize openspec with antigravity support`
* `10a5cc8` — `feat(spec): add airguard-core-spec openspec proposal, specs, design, and tasks`
* `1ad9c5d` — `style: normalize line endings`
* `b16a17b` — `docs(academic): add refactored introduction chapter (Phase 4)`
* `2f21021` — `docs(academic): add refactored related work chapter with comparison matrix (Phase 5)`
* `09eca7e` — `docs(overview): add project overview and final validation summary`
* 最新提交 — `docs(readme): add comprehensive project README and navigation guide`
