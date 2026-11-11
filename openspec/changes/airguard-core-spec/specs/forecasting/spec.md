# Spec Delta

## Purpose

基於歷史監測與氣象時序特徵，提供未來 1 至 6 小時之短程污染物濃度（PM2.5, PM10, O3）與 AQI 級別預測，包含預測信賴區間輸出。

## ADDED Requirements

### Requirement: 多步前瞻時序預測
預測引擎 MUST 支援針對指定測站進行未來 1 小時、3 小時與 6 小時之污染物濃度連續回歸預測。

#### Scenario: 產生 1 至 6 小時污染預測曲線
- **WHEN** 呼叫預測服務並傳入目標測站識別碼與當前時間特徵
- **THEN** 系統返回包含各前瞻步長之預估濃度值、預期 AQI 級別以及上下信賴區間邊界

### Requirement: 模型推論延遲限制
為了支援 Agent 的流暢對話調度，單次時序預測工具的推論執行時間 SHALL 不得超過 200 毫秒。

#### Scenario: Agent 即時工具調用
- **WHEN** AI Agent 發起 short-term forecasting 工具調用
- **THEN** 預測模組於 200ms 內完成特徵組裝與模型前向計算並回傳結構化 JSON
