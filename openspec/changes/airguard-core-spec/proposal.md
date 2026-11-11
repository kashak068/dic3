# Proposal

## Why

戶外活動對公眾健康至關重要，但突發性空氣污染（如 PM2.5、PM10 與臭氧 O3 濃度驟升）對呼吸道與心血管脆弱族群構成嚴重健康威脅。現有空氣品質監測系統多僅提供歷史或落後觀測值，缺乏未來數小時（1–6 小時）的前瞻預測，且官方 AQI 建議多為齊頭式廣播，無法結合個人敏感病史與運動強度提供客製化指引。此外，現有生成式 AI 直接應用於環境健康時易產生數值幻覺與不安全的建議。

AirGuard AI 旨在透過工具調度（Tool Orchestration）架構，結合氣象與空氣品質時間序列回歸預測模型、官方標準規範引擎與 LLM 代理人，打造具備確定性安全護欄的短程預警與個人化戶外活動決策支援系統。

## What Changes

- 新增環境資料擷取與即時特徵工程管線（整合台灣環境部開放資料 API）。
- 新增 1–6 小時短期空氣品質時間序列預測模組（LightGBM / 回歸模型）。
- 新增基於 Function Calling 與 ReAct 模式的 AI 決策代理人核心。
- 新增多維度個人化戶外活動指引引擎（結合運動強度 MET 與敏感族群分級）。
- 新增確定性安全驗證與數值對齊護欄（Safety Guardrails & Post-generation Check）。
- 建立 2024–2026 台中測站滾動回測評估指標（MAE、RMSE）與 30 組場景基準測試集。

## Capabilities

### New Capabilities
- `air-quality-data`: 負責台灣環境部測站監測資料串接、時序特徵計算、異常值濾除與快取管理。
- `forecasting`: 負責 1–6 小時多步前瞻污染物（PM2.5、PM10、O3）時間序列預測與信賴區間估計。
- `ai-agent`: 負責語義理解、工具動態調度（Function Calling）、多輪對話狀態維護與結構化輸出。
- `activity-guidance`: 負責根據預測空氣品質、活動強度（MET）與個人敏感屬性，生成客製化運動風險評估與時段調整建議。
- `safety-evaluation`: 負責以確定性規則引擎對齊官方 AQI 指引，並對 Agent 生成內容進行數值與安全邊界攔截審查。

### Modified Capabilities
<!-- 本專案為初次規格建立，目前無修改之既有能力 -->

## Impact

- **後端系統**: 建立基於 FastAPI 與 Python 3.11 的微服務 API。
- **Agent 架構**: 依賴 LangChain / LangGraph 與 Pydantic v2 定義之工具協議。
- **資料與模型**: 依賴台灣環境部（MOENV）Open API，引入 LightGBM、Scikit-Learn 作為預測引擎。
- **外部界面**: 提供 RESTful 與 WebSocket 端點供前端即時對話與預測數據視覺化調用。

