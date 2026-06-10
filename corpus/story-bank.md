# Story Bank — STAR 技術故事庫

面試彈藥的核心。每則故事對應「面試官想驗證的某種能力」。

---

## 怎麼豐富這份檔案(讀我)

這份是**半成品**:我(Claude)已用你 `cv.md` 的真實內容把每則故事的骨架填好,並把「只有你知道、我猜不出來」的地方標成 `〔需要你補:…〕`。你的任務是把這些缺口補成具體、可口說的內容。

**每則故事要能回答這五件事**(這也是 Staff 面試官真正在挖的):

1. **S/T(情境+任務)**:一句話講清楚問題本身有多難、影響多大。別鋪陳超過 30 秒。
2. **A(行動)**:**你個人**做了什麼,而不是「我們團隊」。要有技術機制——你怎麼下假設、怎麼驗證、為什麼選這條路而不是另一條。
3. **R(結果)**:量化。before → after 數字。沒數字的故事在 Staff 面試裡很弱。
4. **Trade-off / 決策**:你放棄了什麼、為什麼這個取捨對。你怎麼說服別人接受。
5. **反思**:再做一次你會怎麼改。

**最快的豐富方式**:跟我說「grill 我第 N 則故事」,我會用 `grill-me` 一條一條拷問你,把你的口頭回答整理回這個檔案。建議從你最強、最常被問的兩則開始:**#4 iOS DTS 時間戳 bug**(經典 debugging 故事)和 **#1 LangStreamer v3 架構**(system design / ownership)。

**標記圖例**:`〔需要你補:…〕` = 我需要你的輸入  ·  `★ key number` = cheat sheet 的「關鍵數字」欄彈藥

---

## 能力對照表(story → 證明什麼)

| # | 故事 | 主要證明 | 強度 |
|---|------|---------|------|
| 1 | LangStreamer v3 架構 | System design、ownership、Staff scope | ★★★ |
| 2 | 延遲 4–5s → 2–3s | 效能優化、量化影響、使用者導向 | ★★★ |
| 3 | A/V sync 監控 | 深度 debugging、敢下假設並驗證 | ★★★ |
| 4 | iOS DTS 時間戳 bug | Root-cause debugging(經典題) | ★★★ |
| 5 | 三方 SDK 整合衝突 | 跨 SDK 整合、執行緒/GL 疑難 | ★★ |
| 6 | Gradle → Bazel 遷移 | Infra ownership、Staff 系統思維 | ★★ |
| 7 | Android 15 16KB page 遷移 | 平台級遷移、零停機交付 | ★★ |
| 8 | Box2D Bonk Gift / Live2D Vtuber | 圖形/渲染、產品交付 | ★★ |
| 9 | Quanta 三款兒童手錶 | 跨時區協作、出貨消費性硬體 | ★★ |
| — | 140K DAU / 99% crash-free | 貫穿所有故事的 scope & impact | — |

---

## #1 — LangStreamer v3 架構(MediaPipe C++ 跨平台)

**證明**:Staff 級 system design 與 ownership。

- **S/T**:〔需要你補:接手前的 streaming 架構長怎樣?痛點是什麼——是 Android/iOS 兩套程式碼難維護?還是 edge-AI model 部署太慢?〕你重新架構成 C++ 跨平台(Android & iOS)、跑在 Google MediaPipe Framework 上,支援多執行緒即時 CV pipeline。
- **A**:〔需要你補:為什麼選 MediaPipe 而不是自己寫 pipeline / 用別的框架?這是你的決策還是上面定的?你怎麼說服團隊接受這個大改?多執行緒 pipeline 的關鍵設計(graph 結構、calculator 切分)是什麼?〕
- **R**:單一 C++ codebase 同時供 Android/iOS;簡化 edge-AI model 部署。★ 支撐到 **140K+ DAU**、**99% crash-free(從 96%)**。〔需要你補:v3 上線後具體改善了什麼可量化的東西——維護成本?新 model 上線時間從幾天變幾天?〕
- **Trade-off**:〔需要你補:用 MediaPipe 的代價是什麼(學習曲線、binary size、debug 難度)?你怎麼權衡?〕
- **反思**:〔需要你補〕

## #2 — 端到端延遲 4–5s → 2–3s

**證明**:效能優化能力 + 量化影響 + 使用者導向。

- **S/T**:直播端到端延遲 4–5 秒,影響觀眾互動體驗。
- **A**:RTMP 協定優化 + adaptive bitrate control。〔需要你補:RTMP 上你具體調了什麼(GOP、chunk size、buffer 策略)?adaptive bitrate 的演算法邏輯?你怎麼量延遲——量測方法是什麼,才知道是哪一段慢?〕
- **R**:★ **4–5s → 2–3s**(約砍半);在不同網路條件下都改善觀眾體驗。〔需要你補:有沒有對應的留存/互動指標變化?〕
- **Trade-off**:〔需要你補:降延遲通常犧牲畫質或穩定度,你怎麼取捨 latency vs quality vs stability?〕
- **反思**:〔需要你補〕

## #3 — A/V 同步監控(diff threshold / MIC_ROUTE_CHANGED / SEI)

**證明**:深度 debugging、敢下假設並驗證。

- **S/T**:音視訊不同步,且需要即時診斷線上問題。〔需要你補:最初的症狀是什麼?用戶怎麼回報?多嚴重?〕
- **A**:做了 A/V diff threshold 警告、MIC_ROUTE_CHANGED 事件偵測、把 bitrate 用 SEI 嵌進串流做即時診斷。〔需要你補:這是你原本計畫裡提到的 "burst-read 理論驗證 + threshold-bypass 修復" 的故事嗎?root cause 到底是什麼?你怎麼從一堆現象推到那個假設、又怎麼驗證它?〕
- **R**:〔需要你補:不同步的量級從多少 ms 降到多少 ms?能即時抓到問題後 MTTR 改善多少?〕
- **Trade-off / 反思**:〔需要你補〕

## #4 — iOS DTS 時間戳 bug(ns/µs 單位不符)★ 經典 debugging 故事

**證明**:root-cause debugging。這是最容易講得漂亮、面試官最愛的那種故事——好好打磨。

- **S/T**:production-blocking 的 iOS bug,所有 iOS 裝置串流輸出不穩。
- **A**:定位到 DTS 時間戳的 **nanosecond / microsecond 單位不符**。〔需要你補:這是整個故事的精華——你怎麼縮小範圍到時間戳?症狀(花屏?卡頓?串流中斷?)→ 你排除了哪些假設 → 怎麼最終發現是 ns vs µs(差 1000 倍)?為什麼這個 bug 只在 iOS 出現?〕
- **R**:修復後所有 iOS 裝置恢復穩定串流。〔需要你補:影響多少使用者/多少 % iOS 流量?從發現到修復多久?〕
- **反思**:〔需要你補:有沒有加什麼防護(unit type、assert、test)避免再犯?Staff 面試官會問「你怎麼讓這類 bug 系統性地不再發生」〕

## #5 — 三方 SDK 整合衝突(Bytedance / SenseMe / Agora)

**證明**:跨 SDK 整合、執行緒優先權與 GL context 疑難。

- **S/T**:把三家 SDK 整進同一條 pipeline,彼此在 shared EGL 環境互相污染。
- **A**:解 thread-priority 衝突、消除 3 個三方 SDK 間的 GL context pollution。〔需要你補:GL context pollution 具體現象是什麼(畫面錯亂?crash?)?你怎麼診斷出是哪家 SDK 在污染?thread-priority 衝突怎麼解的——重排優先權?加同步?〕
- **R**:三家 SDK 在同一 pipeline 穩定共存。〔需要你補:量化——crash 下降?穩定度?〕
- **Trade-off / 反思**:〔需要你補〕

## #6 — Gradle → Bazel 遷移 + build 系統

**證明**:infra ownership、Staff 級系統思維。

- **S/T**:〔需要你補:build 慢到什麼程度成為瓶頸?團隊多大、一天 build 幾次?〕
- **A**:把 ffmpeg-kit、ExoPlayer 等三方庫從 Gradle 遷到 Bazel,做到 iOS framework / Android AAR 的 reproducible build。〔需要你補:這是你主動推的還是被指派?最難的部分是什麼(三方庫沒有 Bazel 支援要自己寫 BUILD?)?你怎麼說服團隊吞下遷移成本?〕
- **R**:★ Android build time **−50%**、iOS **−30%**;reproducible builds。
- **Trade-off**:〔需要你補:Bazel 學習曲線 vs Gradle 生態,你怎麼說服這個取捨?〕
- **反思**:〔需要你補〕

## #7 — Android 15 16KB page-size 遷移(零停機)

**證明**:平台級遷移、跨整個 native stack 的協調。

- **S/T**:Android 15 要求 16KB page size,整個 native stack 都要動。
- **A**:跨整個 native stack 遷移、零停機部署。〔需要你補:哪些三方 .so 不相容要重編?你怎麼確保零停機(灰度?相容層?)?〕
- **R**:零停機交付。〔需要你補:影響的 binary 數量 / 涵蓋多少 DAU〕
- **反思**:〔需要你補〕

## #8 — Box2D Bonk Gift / Live2D Vtuber(圖形渲染)

**證明**:圖形/渲染深度、產品交付。

- **Bonk Gift**:Box2D 互動禮物系統 + SenseMe 手勢辨識,三種互動模式(Normal/Merge/Gesture);gift-manifest protobuf schema + 從 ZIP 載入 WebP/PNG/WAV 的記憶體高效快取。
- **Vtuber**:Live2D 端到端串流,mesh-attached 裝飾的 sprite anchoring、即時換髮色/服裝的 palette shader、submodel 換裝系統,**★ 25fps on-device**。
- 〔需要你補:這兩個哪個面試更想帶?選一個深挖——技術難點是什麼?25fps 是怎麼從更低 fps 優化上來的?premultiplied alpha / grayscale-to-color 的坑?〕

## #9 — Quanta 三款兒童智慧手錶(GizmoWatch 2 / Disney / T-Mobile SyncUp)

**證明**:跨時區跨文化協作、出貨消費性硬體、scale。

- **S/T**:交付 3 款電信商通路的兒童手錶 App,零售上架(Verizon、T-Mobile)。
- **A**:整 Azure IoT SDK + DPS(★ **1M+ 裝置 provisioning**);擁有 T-Mobile SyncUp 端到端架構(MVVM + Repository、DB 層、API、source control、scrum)。MQTT 長連線 ★ **省電 30%**;DB query ★ **快 5 倍**。
- 〔需要你補:跟美國/法國團隊協作的具體挑戰(時差、規格來回、文化)?MQTT 省電 30% 是怎麼做到的(心跳調整?連線策略?)——這題很可能被細問。〕
- **反思**:〔需要你補〕

---

## 待辦缺口總覽

最該優先補的(影響最多場面試):**#4 的 root-cause 推理過程**、**#1 的 MediaPipe 決策理由**、**所有故事的「你個人 vs 團隊」界線**。
跟我說「grill 我第 N 則」開始。
