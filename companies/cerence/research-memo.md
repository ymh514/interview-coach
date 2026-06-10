# Research Memo — Cerence · C++ SDK Engineer (Taiwan Office)

---

## 1. 公司背景

Cerence(NASDAQ: CRNC)2019 年從 Nuance Communications 車用部門分拆獨立。做**車用語音 AI**:ASR、NLU、TTS、Audio AI 整包套件,主張「Voice as Infrastructure」。客戶幾乎涵蓋所有主要 OEM(Mercedes、BMW、Toyota、Ford、GM 等),聲稱全球 500M+ 輛車搭載其平台。

**對你最重要的定位**:他們把自己定位成「可靠性優先、不是創新噱頭」的 production-grade voice AI。面試時對應你的 SDK 99% crash-free 這個角度。

---

## 2. 技術棧

**核心語言**:C++(SDK 主力)、Python、Java。部分 ML 角色需要 PyTorch / TensorFlow / ONNX Runtime / TensorRT。

**SDK V9 架構(模組化)**:
- **ASR**:Neural network 引擎,支援 streaming,40+ 語言
- **TTS**:65+ 語言,支援 cloud + embedded 雙部署
- **Audio Manager**:平台 audio 整合 + Speech Signal Enhancement(SSE)
- **NLU**:語意提取、filler word 過濾
- **Cloud Connectivity**:可選,embedded 優先

**設計原則**(可在面試中直接引用):
1. **模組化**:元件可單獨用、可組合,不需大改集成程式碼
2. **Embedded-First**:不需要外加 HW DSP,直接跑在 host processor
3. **API 抽象**:隱藏技術複雜性,對外提供高層 interface

**平台**:Android Automotive、Linux、QNX、嵌入式系統。Build 工具不明,Git 是標配。

---

## 3. Taiwan Office

**地址**:台北市內湖區洲子街 55 號 4 樓(內湖新世紀)

**做什麼**:R&D + production engineering。目前在招 Senior TTS Researcher(神經網路推論/生成優化、多語言 TTS 整合、SDK 設計維護、latency/RTF KPI)和 Software Engineer(research 成果轉 production)。

**協作語言**:英文(written 尤其強調)+ 中文本地協作。不確定與哪個總部/region 對接。

---

## 4. 面試流程(Glassdoor/Blind 彙整)

1. **HR 電話篩選**
2. **HackerRank 線上題**:約 2 題 easy level,有 C++ 和 Python 題
3. **Hiring Manager 面試**:~30 分鐘,職位 fit + 技術背景
4. **Technical Interview**:1–2 輪,按 JD 技術棧出題
5. **Technical Director / 主管面**:最終輪,文化 fit + 溝通 + 團隊適合度

**難度**:Glassdoor 評分 2.63–2.9 / 5(偏容易,不是 FAANG 的 hard)。重點在 **coding standards 和 code review 觀念**,有人特別提到這塊問得很深。**平均 timeline 43 天**,HR 回覆快但整體流程可能拖。

**Glassdoor 關鍵評語**:
- 正面:「well-structured, detailed questions across all relevant topics」、「strong focus on coding standards and code review practices」
- 公司文化:「cutting-edge tech, friendly culture, good work-life balance」
- 負面預警:部分人提到 higher management empathy 不足、有裁員疑慮(公司財務面需要關注)

---

## 5. 他們最看重的工程師特質

- **Coding standards + code review 文化**:面試中明確強調
- **模組化設計思維**:SDK 的核心哲學
- **Embedded / real-time 直覺**:latency、RTF、memory footprint 是 KPI
- **Cross-location 協作**:台灣 office 一定會跟其他地區對接
- **迭代改進心態**:官方文化關鍵字「constant and relentless thinking, iteration, and improvement」

---

## 6. 核心技術挑戰(面試對話素材)

以下是 Cerence 自己公開描述的難題,主動提起來展示你懂他們的領域:

1. **車內聲學環境**:引擎噪音、道路噪音、多乘客同時說話、開窗=不可預測噪音 profile。他們的 Audio AI 從 800+ 公尺外辨識緊急車輛警報聲。
2. **Demo → Production 的 gap**:「The gap between demos and production is where decades of voice AI innovation separate them.」這句話和你的 crash-free 99% 角度完全對接。
3. **Multi-zone voice**:前後排同時各有請求,audio routing + echo cancellation 複雜度高。
4. **斷網容錯**:隧道裡沒有網路,系統必須完全 embedded fallback。
5. **OTA 更新下的架構韌性**:模組化才能快速 OTA,不 rewrite 整個 integration。

**橋接話術**(你沒做過 ASR 時用):「我的 streaming SDK 做的是 real-time edge-AI embedded pipeline——MediaPipe on-device、低延遲、多執行緒、記憶體受限、跨平台交付。工程挑戰和語音 SDK 高度相鄰,domain knowledge 可以快速補。」

---

## 7. Fit 分析

| JD 要求 | 我的對應 | 強度 |
|---|---|---|
| 跨平台 C++ SDK 開發與維護 | LangStreamer v3:整套 C++ 跨平台 streaming SDK | ★★★ |
| Improve performance / latency / memory | 延遲 4–5s→2–3s、build time -50%/-30% | ★★★ |
| Support SDK users(feature req / bug triage) | SDK↔client 溝通窗口,一線問題排查 | ★★★ |
| Android + JNI | MediaPipe C++↔Android JNI | ★★ |
| Multi-thread + memory management | ⚠️ 概念在,口頭表達需加強(這週補) | ★★ |
| Coding standards / code review | 曾定義 team coding standards(Quanta) | ★★ |
| Speech/voice recognition 領域 | ✗ 沒做過 ASR;用 edge AI 工程經驗橋接 | ★ |
| Linux 開發環境 | mobile 背景,純 Linux 較淡 | ★ |

---

## 8. 找不到的資訊

- 具體 coding 面試題目(只知道 HackerRank easy 兩題)
- Taiwan office 確切團隊規模與組成
- 內部 build system / CI/CD 工具
- 是否有 system design 輪(Technical Director 輪可能觸及,不確定)
- 薪資 Taiwan 具體數據
