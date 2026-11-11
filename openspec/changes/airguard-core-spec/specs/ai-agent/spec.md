# Spec Delta

## Purpose

基於 Function Calling 與工具編排架構，實現理解使用者意圖、自主調用環境與預測工具，並生成脈絡化建議的 AI 代理人核心。

## ADDED Requirements

### Requirement: 結構化工具調用 (Tool Orchestration)
AI Agent MUST 透過嚴格的 Pydantic 結構化 Schema 調用外部工具，嚴禁在未調用工具之情況下憑空捏造即時數值。

#### Scenario: 使用者詢問特定時段與地點之戶外活動可行性
- **WHEN** 使用者輸入包含地點、預定活動時間與活動類型的諮詢語句
- **THEN** Agent 自主發起 `get_current_station_aqi` 或 `forecast_short_term_aqi` 工具調用以取得客觀數據

#### Scenario: 缺少關鍵參數的主動澄清
- **WHEN** 使用者未提供活動地點或活動時段
- **THEN** Agent 識別缺失欄位並主動反問使用者以補齊必要上下文

