# 第二章 相關文獻探討 (Related Work)

## 2.1 空氣品質監測與時間序列預測 (Air Quality Monitoring and Forecasting)

空氣品質監測與短期預測是環境資訊科學長期關注的核心領域。傳統的數值空氣品質預報模式（Air Quality Numerical Models，如 WRF-Chem、CMAQ 及 CAMx）立足於大氣動力學、熱力學與複雜大氣光化學反應機制，能模擬大規模網格化之污染物擴散與傳輸。然而，數值模式高度依賴龐大的排放清冊（Emission Inventories）與複雜邊界條件，計算開銷極大且往往存在數小時之延遲，難以滿足個人化行動即時查詢所需的次秒級響應。

近年來，以資料驅動（Data-Driven）的機器學習與深度學習時間序列模型取得了顯著進展。
1. **梯度提升樹與傳統統計學習**：以 Ke 等人提出的 LightGBM 以及 Chen 與 Guestrin 提出的 XGBoost 為代表的高效梯度提升架構，在表格化時序特徵（滑動均值、空間測站相鄰特徵、氣象滯後變數）中展現出極高的計算效率與出色的預測穩定性。研究顯示，在 1 至 6 小時的超短期前瞻預測（Nowcasting / Short-Term Forecasting）任務中，特徵工程完善的 GBDT 模型在 MAE 與 RMSE 指標上常可媲美甚至超越未經精細校準的複雜神經網絡，且其單次前向推論時間常在數十毫秒以內，極為適合需要即時調用的在線系統。
2. **深度時序與時空圖類神經網路**：自 Zhou 等人（AAAI 2021）提出 Informer 模型以自注意力機制處理長序列時間依賴以來，以 Transformer 為基底的時序預測架構（如 PatchTST、TimesNet 等）及結合測站拓撲結構的時空圖神經網路（Spatio-Temporal Graph Neural Networks, ST-GNN）陸續被引入環境監測。此類模型在跨區域擴散捕捉與中長期（24–72 小時）趨勢上具有優勢，但存在模型量體龐大、對邊緣資源需求高以及難以在對話中動態調整前瞻步長等限制。

本研究在 AirGuard AI 中採取輕量級、高度可解釋且低延遲的 LightGBM 多步回歸架構作為即時預測核心，在維持預測精確度的同時，徹底消除推論延遲對後續 Agent 調度的阻塞。

## 2.2 人工智慧環境健康與決策支援系統 (AI Environmental Decision Support)

環境決策支援系統（Environmental Decision Support Systems, EDSS）長久以來旨在協助決策者或公眾將複雜環境數據轉換為防護行動。世界衛生組織（WHO, 2021）公布之《全球空氣品質指引》（Global Air Quality Guidelines）大幅收緊了 $\text{PM}_{2.5}$、$\text{PM}_{10}$ 及 $\text{O}_3$ 之日均與年均安全門檻；台灣環境部（MOENV）亦依據最新健康風險研究發布了包含六個級距之「空氣品質指標（AQI）及活動建議防護指引」，明確界定了不同污染等級下一般民眾與敏感族群（學童、長者、心血管與呼吸道疾病患者）之戶外體育鍛鍊限制。

然而，現有多數環境資訊系統仍停留在「靜態展示（Dashboards）」階段，鮮少深入「人體運動代謝當量（Metabolic Equivalent of Task, MET）」的交叉建模。醫學流行病學研究（如 Cepeda 等人於《Environmental Health Perspectives》探討之運動與空污暴露權衡）指出，高強度運動伴隨的通氣量劇增會顯著抵消運動帶來的生理益處。現有行動應用多僅提供通用文字提醒，缺乏依據預測曲線自動搜尋「當日最低暴露黃金時段」或「等效室內低代謝替代運動」的量化決策機制。AirGuard AI 透過將官方規則與個人化代謝模型程式化，實現真正閉環的個人化活動排程指導。

## 2.3 大型語言模型代理人與工具調度 (LLM Agents and Function Calling)

隨著基礎語言模型（Foundation Models）的突破，將 LLM 作為主控核心建構具備自主推理與外部環境互動能力的「AI Agent（智能體）」成為當前人工智慧的前沿研究焦點。
1. **ReAct 推理範式**：Yao 等人（ICLR 2023）提出的 ReAct（Synergizing Reasoning and Acting）架構，使語言模型能夠交替生成「思考（Thought）」、「行動（Action）」與「觀察（Observation）」，引導模型透過多步反思逐步解決複雜問題。
2. **工具使用與 Function Calling**：Schick 等人（NeurIPS 2023）發表的 Toolformer 證明了語言模型能夠自主學會呼叫外部 API 以彌補自身算術與時效性知識不足；Patil 等人（2023）推出的 Gorilla 模型更進一步深化了模型對豐富且多變的 API 定義之理解與結構化呼叫能力。現代商用模型（如 OpenAI GPT 系列、Google Gemini 系列）及開源生態（如 LangChain、LangGraph）已廣泛內建標準化之 Function Calling / Tool Calling 協議，利用 JSON Schema / Pydantic 實現強型別的參數傳遞。

本研究借鏡上述 Tool-Orchestrated 典範，將空氣品質監測 API、短程預測模型與官方規則庫包裝為標準化工具，使 Agent 在面對使用者自然語言諮詢時，能自主解構意圖並發起確定性工具呼叫，杜絕內部參數臆造。

## 2.4 知識落地與生成幻覺防範 (Knowledge Grounding and Hallucination Prevention)

大型語言模型在垂直高風險領域（如醫學診斷、環境安全）的主要落地瓶頸在於「事實性幻覺（Factuality Hallucination）」。模型經常以高度流暢且自信的語句輸出錯誤的數值或捏造不存在的法規。

為解決此問題，檢索增強生成（Retrieval-Augmented Generation, RAG，Lewis et al., 2020）被廣泛應用以提供靜態背景知識。然而，空氣品質屬於**高度時變（High-Frequency Time-Varying）**的動態數值，傳統基於向量資料庫的語義檢索無法捕捉即時發生的氣象與污染波動。因此，「動態工具落地（Dynamic Tool Grounding）」成為唯一可靠解方。本研究更進一步在工具落地之上，設計了後處理數值一致性檢驗器（Post-generation Fact-checking Guard），透過正則表達式與結構化對象比對，嚴格審核生成文字中出現的每一次濃度值與指標數值是否與底層工具回傳之實體數據完全對齊，一旦偵測到數值偏差立即觸發覆寫。

## 2.5 系統安全性與可信任 AI 護欄 (Safety and Trustworthy AI)

在公眾健康防護決策中，系統的安全性與可信任度具有最高優先級。近年學界在可信任 AI 領域提出多種防護機制：
1. **Constitutional AI 與價值對齊**：Bai 等人（Anthropic, 2022）提出了 Constitutional AI，透過預先設定的原則原則規範模型的行為邊界。
2. **可程式化硬護欄（Deterministic Guardrails）**：以 NeMo Guardrails（Rebedea et al., 2023）及 Guardrails AI 為代表的研究指出，單純依賴提示詞（System Prompts）的軟性約制在對抗性攻擊或邊緣情境下存在高達數個百分點的失效率。在高風險決策情境中，必須引入由宿主程式執行的確定性程式邏輯（Deterministic Programmable Guardrails）。

AirGuard AI 採用確定性官方規則引擎作為「硬性熔斷器（Circuit Breaker）」。當預測或實測 AQI 達到嚴重危害等級（如紫爆警戒）時，系統繞過語言模型的任何委婉生成空間，直接強制輸出法定防護宣告與醫療中止提醒，確保系統在任何情況下均符合公眾安全最高標準。

## 2.6 文獻比較與研究定位 (Research Gap and Comparison)

下表系統性比較了現有主流方案與本研究提出之 AirGuard AI 架構在各核心維度上的能力差異：

| 評估維度 | 官方即時監測系統 (如環境部 App) | 通用大語言模型 (如 ChatGPT / Gemini) | 傳統環境 RAG 系統 (靜態檢索) | 數值模式預報 (WRF-Chem / CMAQ) | **AirGuard AI (本研究架構)** |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **時間前瞻性** | 僅即時/歷史觀測值 (滯後 1 小時) | 無即時性 (受限於訓練資料截止點) | 無法處理即時動態時間序列 | 支援未來 24–72 小時預報 | **精確 1–6 小時短程高頻預報** |
| **即時推論延遲** | 毫秒級 (查表) | 1–3 秒 (文字生成) | 1–2 秒 | 數小時 (大規模偏微分計算) | **< 200ms (預測工具) + Agent 實時流式輸出** |
| **個人化情境計算** | 無 (全體通用廣播) | 具對話能力，但無客觀數值依據 | 僅限靜態文檔匹配 | 無 (純物理大氣格網) | **綜合評估敏感病史 + 運動 MET 代謝當量** |
| **數值與法規真實性** | 100% 真實 (直接量測) | 頻繁產生數值幻覺與臆造風險 | 偶發事實抽取與語義漂移 | 具物理守恆依據 | **100% 確定性工具落地 + 生成後嚴格比對校驗** |
| **醫療安全防護機制** | 僅靜態警示語 | 僅軟性 Prompt 約束，易被繞過 | 無即時健康規則邊界 | 無直接決策機制 | **官方標準硬編碼引擎 + 嚴重污染強制熔斷機制** |
| **互動形式** | 圖表 / 數據儀表板 | 自然語言多輪對話 | 問答對話 (Q&A) | 專業大氣物理圖譜 | **自然語言對話 + 具體排程行動指南 (Actionable)** |

綜合上述文獻分析，AirGuard AI 正好座落於「即時大氣時序預報」、「確定性醫療健康硬護欄」與「Tool-Orchestrated 對話代理人」的交叉新興領域，填補了從「純環境數據量測」轉化為「可信任、個人化行動決策指引」的重大研究缺口。

