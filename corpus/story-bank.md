# Story Bank — STAR 技術故事庫

面試彈藥的核心。每則故事對應「面試官想驗證的某種能力」。

---

## 怎麼豐富這份檔案(讀我)

這份是**半成品**:我(Claude)已用你 `corpus/resume.md` 的真實內容把每則故事的骨架填好,並把「只有你知道、我猜不出來」的地方標成 `〔需要你補:…〕`。你的任務是把這些缺口補成具體、可口說的內容。

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
| 10 | PAG GPU-native 渲染優化 | GPU pipeline 效能優化、量化(CPU 81→61%) | ★★ |
| 11 | 動態 delegate 選擇(GPU vendor) | edge-AI 異質裝置健壯性、資料驅動黑名單 | ★★★ |
| — | 140K DAU / 99% crash-free | 貫穿所有故事的 scope & impact | — |

---

## #1 — LangStreamer v3 架構(MediaPipe C++ 跨平台)

**證明**:Staff 級 system design 與 ownership。

- **S/T**:v3 之前 **Android / iOS 各維護一份 codebase**,雙端實現容易有落差(修一個 bug 改兩邊);舊版 **v2 是 single-thread**,難擴展新特效/禮物,且 **edge-AI model 部署很慢**。團隊(前主管主導)決定重構成 **C++ 跨平台、跑在 Google MediaPipe Framework 上的多執行緒即時 CV pipeline**,Android & iOS 共用同一份。
- **A**:〔需要你補:為什麼選 MediaPipe 而不是自己寫 pipeline / 用別的框架?這是你的決策還是上面定的?你怎麼說服團隊接受這個大改?〕多執行緒 pipeline 的關鍵設計(graph 結構、calculator 切分):
  > ✅ **codebase 查證(`mediapipe/graphs/langstreamer/langstreamer.pbtxt`,約 33 個 calculator)**:整條直播鏈是**單一 MediaPipe graph**,同一份 C++ 編成 Android AAR + iOS framework。一路是 `FlowLimiterCalculator`(背壓/丟舊 frame)→ `BundleSyncCalculator`(對齊音視訊)→ `FiniteStateMachineCalculator`(禮物/狀態機驅動)→ `SenseMEModelCalculator`/`SenseMECalculator`(美顏+人臉)→ `BonkLangStreamerCalculator`(Box2D 禮物)→ `SenseARCalculator` → `UIRendererCalculator` → `AgoraRtcCalculator`/`AgoraRtcVideoMixCalculator`/`AgoraRtcAudioMixCalculator`(連麥混流)→ 平台編碼器(`AmediaH264EncoderCalculator` / iOS VideoToolbox)→ `RtmpStreamerCalculator`。
  > 並行設計:靠 MediaPipe executor + 在 render/encode node 上掛 `SyncSetInputStreamHandler` 做 sync set;近期還做了「依 GPU vendor 動態選 inference delegate」(commit `ce69a270b`)與 PowerVR 上 `LOAD_ASYNC` per-delegate 防呆(`51b10e4cc`)。這些都是可口說的「Staff 級 graph 切分」具體例子。
  > **你個人的 ownership 邊界(誠實版,很關鍵)**:重構**方向由前主管主導**;你是**跨平台窗口 + platform 層的建置者**——Android/iOS 介面層與平台邏輯(camera / audio 採集)、**Android JNI ↔ C++ 橋接**、與 MediaPipe 串接、**把原有業務邏輯 migrate 進新架構**、整體 **FSM 狀態機與 report-event 管理**。後期愈來愈熟,**幾乎每個模組都經手修改維護**,新 feature 也由你實作。→ 講法:別說「我設計了整個架構」,而是「**架構方向是團隊定的,但跨平台 platform 層、JNI/MediaPipe 串接、業務邏輯遷移與 FSM/事件系統是我從零建起並長期維護**」——精準反而更 Staff、更可信。
  > **為什麼 MediaPipe(你的真實理由)**:(a) **跨平台**——calculator graph 純 C++,雙端共用同一份 pipeline,連 render 都只要寫**一套 OpenGL ES shader**;(b) **即時 CV 並行**——原生多執行緒排程 + GPU delegate + TFLite,美顏/手勢/AI 去背各拆成獨立 calculator;(c) **效能**——C++ 層較好省 GPU,能做到 **GPU-to-GPU zero-copy**;(d) edge-AI 部署標準化、比自幹快。自己寫一套同等的 graph executor + 跨平台抽象,成本太高且要長期養。
  > 📓 **daily-memo 時間軸(可口說的 ownership 證據)**:v3 **2023-06-12 內部上線**——同日 Android 與 iOS 都跑同一個 `FiniteStateMachineCalculator`(證明同一份 C++ graph 雙端同時 live);**2023-08-11 Android SDK 1.0.0 公開發版**。期間我把推流模組重構成 **subgraph**(踩過「executor 要寫在『使用的 graph』而非 subgraph 裡才認得」的雷)——first-hand graph 架構 ownership。
- **R**:單一 C++ codebase 同時供 Android/iOS。★ 支撐到 **140K+ DAU**、**99% crash-free(從 96%,主要來自統一 codebase、消掉雙端各自的 bug)**。★ **開發成本約砍半**——主邏輯在 native 寫一次、雙端只串 API,不再花時間對齊雙端行為。★ **新禮物/特效上線約 1 個月 → 約 2 週**(架構從「改很多地方」變「掛一個 calculator」)。〔edge-AI 部署有變快但**無數據佐證,面試別報數字**,只講「標準化、可掛 TFLite calculator」〕
- **Trade-off**:代價是 **MediaPipe 學習曲線非常陡**,前期投入大量時間。權衡:用一次性的陡峭 ramp-up,換掉「雙端各維護一份」的長期重複成本 + 換到「**新特效 = 掛一個 calculator**」的可擴展架構。止血團隊風險的方式:**我把踩過的坑與經驗文件化、整理成上手指南**,讓新進同仁快速接手——把「只有我懂」變成「團隊都能動」(這也是 leadership / scale-yourself 的證據)。
- **反思**:① **更早投資 graph 的除錯工具與文件**——前期太多時間花在「graph 卡住但不知哪個 calculator 卡」,trace/log 規範(後來的 ATrace/Perfetto)早點建,團隊 ramp-up 會快很多。② **多讀 MediaPipe team 自己的寫法**——從框架作者的 calculator/graph 慣用法獲得很多啟發,少走很多彎路。

## #2 — 端到端延遲 4–5s → 2–3s

**證明**:效能優化能力 + 量化影響 + 使用者導向。

- **S/T**:直播端到端延遲 4–5 秒,影響觀眾互動體驗。
- **A**:RTMP 協定優化 + 碼率調整策略 + **編碼器去掉 B-frame**(直播場景不適合,B-frame 的重排序會增加延遲)+ v3 整體推流框架比 v2 更 robust。**碼率策略是與主管一起實作**(你-vs-團隊:共同實作,非獨力)。
  **量測方法(誠實版,務必照這個講)**:4–5s→2–3s 是 **v3 與 v2 的對比**,做法是**直接並排比對「推流端 preview 畫面」vs「player 端畫面」** 的時間差(經驗實測對比,非分段儀器量測)。→ 別宣稱做了精密分段量測;若被追問可補「下次會用畫面打 marker 做分段歸因」當加分。
  > ✅ **codebase 查證(adaptive bitrate / 低延遲機制現存於碼庫)**:
  > • **網路自適應**:`PUSH_NETWORK_WEAK_WARN` 觸發時回報 **average frame QP** 來評估 target bitrate(commit `3ca50bd55` / issue 958);把 `VARIABLE_ENCODER_CONFIG` 指派進 CONFIG sync_set 避免誤判弱網(`8669123e5`);MediaCodec 明確設 **VBR** 模式(`98c848001`)。
  > • **低延遲路徑**:encoder **zero-copy** 優化(`4b10d8333`);AHardwareBuffer 配置失敗時**丟該 frame 而非殺掉 graph**(`e1cfef2eb`);RTMP 重連時 `ln_Buffer::clear()` 補 mutex 防 data race / SIGSEGV(`46e452dd3`)。
  > • 量測面:av-sync 用每 5 分鐘 `avdf` log(見 #3),SEI 帶實際 bitrate 讓拉流端對帳(見 #3)。
  > ⚠️ **注意**:本碼庫 git 史**沒有**直接對應「4–5s → 2–3s」那次優化(可能在更早的 repo/時期),所以**延遲數字與量測方法只有你知道**——這題務必準備好「你怎麼把端到端延遲拆段量出來」,否則 −50% 會空泛。
- **R**:★ **4–5s → 2–3s**(約砍半);在不同網路條件下都改善觀眾體驗。〔需要你補:有沒有對應的留存/互動指標變化?〕
- **Trade-off**:碼率策略是 **「急降緩升」**——網路一差立刻把碼率砍下去(畫質明顯變糟),網路回來時**緩慢爬升**。明確**以順暢為優先、犧牲尖峰畫質**:寧可糊一點也不要卡(觀眾對卡頓的容忍遠低於對畫質)。📓 **加分決策(daily-memo)**:我們**評估過用 AI 預測模型**做碼率自適應,但 **2025 Q4 確認效果不佳、不如急降緩升**而捨棄——真實的「試了更花俏的方案、最後用數據選回簡單可靠的」取捨。(也佐證 daily-memo:2023-09-26 `bitrate 400–1300 飄、觀眾端卡` 是當初發現問題的具體訊號。)
- **反思**:〔可補:「緩升」會不會太保守,讓網路恢復後畫質回不來、被主播抱怨?如果有,你怎麼微調爬升曲線——這種「我後來怎麼調參數」的細節很加分〕

## #3 — A/V 同步監控(diff threshold / MIC_ROUTE_CHANGED / SEI)

**證明**:深度 debugging、敢下假設並驗證。

- **S/T**:**觀眾回報看直播卡頓**,同時主播端 **CPU 偏高、手機發燙**。問題模糊、難複現——得先把「感覺卡」變成可量測的線上指標才追得動。
- **A**:做了 A/V diff threshold 警告、MIC_ROUTE_CHANGED 事件偵測、把 bitrate 用 SEI 嵌進串流做即時診斷。**完整偵錯敘事見下方「你的推理」**。
  > ✅ **codebase 查證**:
  > • **A/V diff 監控**(`rtmp_streamer_calculator.cc`):`av_diff_threshold_ms = 300`,比對 `last_audio_pts` vs `last_video_pts`,超過就把 `avdf`(audio-video diff)寫進 push log;每 5 分鐘回報一次(commit `79fe176f1`「add push av sync warn and audio video diff every 5 mins, threshold 300ms」)。
  > • **MIC_ROUTE_CHANGED_WARN** 事件(`a19fc2820` 新增,`474dbfd1b` 改名):偵測 iOS 麥克風 route 切換——這正是 lip-sync 跑掉的元兇之一。
  > • **threshold-bypass 修復**(commit `cd77dbee4` / issue 929):音訊 PTS 平滑在 route change 那一刻會把缺口攤平、反而**連鎖插入靜音**;修法是「diff ≥ 150ms 時直接 bypass 平滑」。這應該就是你說的 "threshold-bypass" 故事——root cause = PTS smoothing 把單一大跳變抹成連續小偏移。
  > • **SEI 嵌 bitrate**(`7f534008a`,UUID `LANGBITRATE_V1.0`;`a73efabe7` 加 EPB emulation-prevention-byte):把 target bitrate 用 H.264 SEI NAL 帶進串流,拉流端即時讀到實際下發碼率做交叉診斷;路徑 FSM→RtmpStreamer(`7569e063d`)。
  > **你的推理(假設驅動、含一個被自己否證的假設——這段最 Staff)**:
  > ① **先讓問題可量測**:卡頓很模糊,我加了一組觀察指標(avdf、CPU、溫度),並**自己寫了一個 ffmpeg 的 skill 去量 frame PTS**,把「觀眾覺得卡」變成可追的數據——約 **300ms 不同步觀眾就有感**。
  > ② **多重 root cause(橫跨三層)**:(a) 主播常接**外部聲卡、但廠商品質參差**,採集端 timestamp 本身就壞;(b) 壞掉的 ts 會**污染 MediaPipe 每個 node 的 frame pacing**;(c) iOS audio route change 時 **PTS smoothing 連鎖插入靜音**(→ threshold-bypass 修)。
  > ③ **誠實的一筆**:我原本假設是 **burst-read**(音訊 buffer 批次進來、ts 擠在一起),**驗證後發現它從來沒被觸發**——於是**否證自己的假設**,改判定為 frame pacing 問題。面試講這段超加分:hypothesis → test → 推翻 → pivot。
  > 📓 **daily-memo 佐證 + 更深的洞察**:burst 否證有實測——memo 直接寫 **`但沒有收到 burst`**(2026 Q1)。更 Staff 的點(可加進故事):**publish_queue 的 0.3/frame 平滑反而「把告警藏起來」**——一個乾淨的 300ms 缺口被攤成 8 幀往前擠(聽起來像 stutter),但**每幀 diff 都壓在 300ms 門檻下 → `PUSH_AV_SYNC_WARN` 根本沒觸發**。「**本來要幫忙的平滑,既讓 UX 更糟、又把症狀對監控藏起來**」。修法:diff 超門檻就 bypass 平滑、snap 回原始 ts(門檻**以 git 為準 = 150ms**;memo 曾記 ~95ms 應是早期 iteration,**口說一律用 150ms**)。
- **R**:這套監控把「觀眾喊卡」變成可量測訊號——**卡頓發生時 avdf 通常會飆很大**,讓我們能**快速 narrow down**(MTTR 有改善但無硬數據)。修法綜合:**threshold-bypass(≥150ms)** + **優化各 calculator node 執行時間改善 frame pacing** + **跟 CDN 廠商協作做轉碼重推、重新對齊 timestamp**,卡頓回報下降。〔可補:卡頓投訴下降幾成 / avdf 超標頻率下降多少〕
- **Trade-off / 反思**:root cause 不只一個、且橫跨**裝置(外接聲卡)× 框架(node pacing)× CDN(轉碼重推)**,單點修不完——所以價值在「**先建可觀測性、再逐層收斂**」而非急著改 code。反思:**敢否證自己的假設(burst-read)** 是這次最值得的習慣;另外若早點讓外接音訊裝置走**統一的 ts 正規化入口**,可少踩很多廠商品質的坑。

## #4 — iOS DTS 時間戳 bug(ns/µs 單位不符)★ 經典 debugging 故事

**證明**:root-cause debugging。這是最容易講得漂亮、面試官最愛的那種故事——好好打磨。

- **S/T**:production bug——**iPhone 16 上市後**,高階 iOS 主播的串流時間戳異常,**觀眾端(跨 web / Android / iOS 都看得到)出現馬賽克破圖**,觀看數下滑、主播投訴。影響約 **28% 的 iOS 主播**(高階機種佔比)。
- **A**:定位到 DTS 時間戳的 **nanosecond / microsecond 單位不符**。
  **偵錯實況(你的口述,這段是精華)**:症狀先在**播放端出現馬賽克** → 我用 **ffmpeg parse 串流,發現 DTS 錯亂(時序亂跳)** → 回採集源頭追,發現 **MediaPipe timestamp 用 µs,但採集端打進去的是 ns**。而 ns 是先前為了 **iPhone 16** 上市、**效能太好、µs scale 下會連續兩幀採到同一個 timestamp(timestamp mismatch)** 才改的。**歸屬手法**:本地用 **ffplay / VLC 等 reference player 複現**,反證問題在 **streamer 端、不是播放 SDK**(這招務必講——「拿已知正確的參考實作做歸屬」)。最終正解見下方第三幕。
  > ✅ **codebase 查證 — 真實是三幕劇(比「單位不符」精彩,但和原草稿相反,務必改用這版)**。檔案 `mediapipe/objc/streamer_lib/LangStreamer.mm`:`NOW` 巨集 + `synctimestamp:` 方法。
  > **第一幕(bug 浮現,`6f6a20649`,2025-04-23)**:iOS v3 原本 `#define NOW (CACurrentMediaTime()*1000000)`(**微秒**)。**iPhone 16 的音訊** callback 太密,µs 解析度撞同一刻度 → **重複時間戳** → MediaPipe「timestamp mismatch」(契約要求嚴格單調遞增)→ 串流崩。
  > **第二幕(天真修法反噬,`3d289532f`,2025-05-27)**:第一直覺把 `NOW` 拉到 **奈秒**(`clock_gettime(CLOCK_MONOTONIC_RAW)`,`tv_sec*1e9+tv_nsec`)。重複消失,**但拉流端觀眾開始 mosaic 馬賽克**——因為 MediaPipe 的 `Timestamp` 與下游 encoder PTS/GOP 全部**以微秒為單位**(程式碼註解明寫 "represents packet timestamps in units of microseconds"),餵奈秒(數量級 ×1000)把時間運算/編碼節奏打亂。
  > **第三幕(正解)**:`NOW` **改回微秒**(尊重單位契約),改在 `synctimestamp:` 加**明確單調性守衛**——記 `lastSyncedTimestamp_`,`pts <= lastSyncedTimestamp_` 就**丟該幀**(`return 0`,video/audio 兩路都檢查回傳),並用 `_avsyncSemaphore` 保護(音視訊來自不同執行緒)。
  > 💡 **真正的 root cause(Staff 級洞察)**:從來不是「解析度」問題,是**兩個約束打架**——(a) 時間戳必須單調遞增、(b) 必須是微秒(MediaPipe 契約 + encoder 依賴)。用 ns 修 (a) 等於破壞 (b)。正解=「守住單位 + 用顯式去重保單調」,而不是「拉高時鐘解析度去賭不撞」。
  > ⚠️ **原草稿/履歷若寫「改成 nanosecond 就修好」是反的** —— ns 正是造成 mosaic 那一步、後來被 revert。面試千萬別這樣講。
  > 📓 **daily-memo 佐證(精確時間軸 + 更利的講法)**:**4/21** 首現 `Packet timestamp mismatch ... expected 17220940 but received 17220939`(audio 流)→ **4/22** 改 ns 暫解 → **5/22** 確認 mosaic 是 ns 造成 → **5/26** 抓到根因:**MediaPipe TimestampAPI 一律把傳入整數當「微秒」**,餵 ns 不是「精度 ×1000」而是「**尺度 ×1000 → MediaPipe 以為每幀間隔 15 秒**」,priority queue 還印出 `Pop frame timestamp: 18640 ← WRONG ORDER!`(幀序錯亂的鐵證)→ **5/27–28** 正解:回微秒 + 守衛(考慮過 `+1` 但會無限加一,最後選 **drop**)。**最利的一句**:「不是精度問題,是**把奈秒餵進假設微秒的 API、尺度差 1000 倍,讓 MediaPipe 以為幀距 15 秒**」。
- **R**:**由觀眾投訴發現**。影響所有 **iPhone 16 等級以上高階機種主播(估 ~28% iOS 主播)**;馬賽克**跨所有平台觀眾(web/Android/iOS)可見**,造成觀看數下降 + 主播投訴。處置兩段:**① 約一週 workaround 止血**——推流前用 **atomic 重排/檢查送出的 DTS,非遞增就 drop**,先壓住錯亂;**② 約一個月**鎖定 root cause(ns 違反 µs 單位契約),上**微秒 + 單調守衛**正解,觀眾畫質與開播品質回穩。〔可再補:止血後觀看數回升 / 投訴歸零的具體數字;28% 的分母是「iOS 主播」還是「全平台主播」要講清楚〕
- **反思**:① **執行期防線**:此後**任何要打進 MediaPipe 的 timestamp,我都加一道單位/單調性檢查**,把正確性收斂到單一 choke point。② **真正的 takeaway——「先別假設成熟 framework 有 bug,去讀它的契約」**:當初卡很久、一度懷疑是 MediaPipe framework 本身壞掉,後來邊追 source code 邊翻文件,才在文件看到簡短一行 *"MediaPipe timestamps are in units of microseconds"*,並說明用其他單位 pipeline 可能出錯——**root cause 一直藏在一行文件裡**。從此養成「動共享契約 / 第三方框架前,先確認它的單位與前後條件假設」的習慣。〔可加但非必要:把 µs 包成 strong-typedef 讓單位用錯直接編不過,當作「如果再做一次」的加分項〕

## #5 — 三方 SDK 整合衝突(Bytedance / SenseMe / Agora)

**證明**:跨 SDK 整合、執行緒優先權與 GL context 疑難。

- **S/T**:多個三方圖形/特效套件(Agora、SenseME,加上 client 端的 **Spine**、騰訊 **PAG**)**共用同一個 GL context**,有套件用完沒把 GL state 清乾淨 → **互相污染、黑畫面**。
- **A**:解 thread-priority 衝突、消除三方 SDK / 套件間的 GL context pollution。〔需要你補:GL context pollution 具體現象是什麼(畫面錯亂?crash?)?你怎麼診斷出是哪家 SDK 在污染?thread-priority 衝突怎麼解的——重排優先權?加同步?〕
  > ✅ **codebase 查證(三家 SDK 都實際整合在同一條 graph)**:
  > • **Agora**:以 calculator 形式進 graph——`AgoraRtcCalculator` + `AgoraRtcVideoMixCalculator` + `AgoraRtcAudioMixCalculator`(`langstreamer.pbtxt` / `agora_rtc.pbtxt`),負責連麥/混流。
  > • **SenseME(商湯)**:`SenseMEModelCalculator` / `SenseMECalculator` / `SenseARCalculator` 美顏+人臉鏈;pbtxt 裡留有 `#executor: "senseme"`(被註解掉),正是「要不要把 SenseME 丟到獨立 executor / 執行緒」這個 thread 議題的痕跡——可當作你講 thread-priority 取捨的實證。近期還有 SenseME 在不同 GPU 的 delegate 處置(`51b10e4cc` 防 PowerVR 上 GPU 載 model)。
  > • 共享 EGL:SenseME(GPU 美顏)與 render/encode 同處一張 graph,GL context 共用問題就出在這。
  > **第三家 = ByteDance 火山引擎(Volcengine)**(Agora 連麥 + SenseME 美顏 + 火山引擎)。
  > **GL 污染實戰兩例(真正的故事,résumé 請對齊這兩個)**:
  > • **Spine(client App 用的 2D 骨骼動畫庫)**:render 某些特效時出現**黑畫面**。完整排查發現 **client App 也用 Spine、且與我們共用同一個 glcontext**——底層 render 時 GL state 已被 Spine 污染。修法:**讓 client 在 init SDK 時設一個 child context**,彼此隔離不互污;過程中我**還去 survey 了 client 的 codebase**,幫他們定位問題並提供配套解法(跨團隊 ownership)。
  > • **Tencent PAG(GPU decode)**:把素材丟給騰訊 PAG 套件做 **GPU decode**,流程結束後它**沒把 GL state 清乾淨**,造成後續**黑畫面**。修法:在該流程前後做 GL state 的保存/還原。
  > → 共同 root cause:**共用 GL context 時,某套件改了 binding/framebuffer 沒復原**;診斷靠**隔離法**(一次只開一個來源)+ 比對前後 GL state。
  > 📓 **daily-memo 佐證(更精確的機制,Staff 級細節)**:**Spine**(2023-04):GL error **`0x502`(GL_INVALID_OPERATION)** 出現在 `RealtimeCameraTextureConverter` 的 GL thread;根因是 **Spine/libgdx 重複使用 texture id,`dispose()` 時把我們 pipeline 的 texture 也一起刪掉**(不只是「沒清乾淨」這麼籠統)——我進到 client 的 `AvatarAdapter.java` 改 `dispose()` 不去清 base atlas。**PAG**(2025 Q4):PAG **bind 了 VAO/VBO 沒還原**,我們 renderer 假設沒 bind,直接 `glDrawElements` → INVALID_OPERATION;退背景再回來就掛。**兩例都用同一招隔離驗證**(關掉 Spine avatar / `pag_enabled_=false` → 黑畫面立刻消失)。後續(2026-06)再把 PAG 從 **CPU readback 改成 GPU-native render-to-texture**,單層 PAG 的 CPU 尖峰 **57%→81-85% 降到 57%→61-65%**(可獨立成「GPU pipeline 優化」故事,見底部候選)。
  > ⚠️ **thread-priority 衝突 → 已確認你沒有此故事,建議從 résumé 拿掉**。你的理由本身就是正解:MediaPipe 是 **pipeline 結構**,雖 multithread,但每個 node 仍要**等上一個 node 的輸出**;某條 pipeline 慢、**timestamp 落後就直接被 drop**——所以**不需要手動調 thread priority**,框架用「pipeline + 時間戳丟幀」自然處理了排程。→ 面試被問併發時**這樣講才準**:不是「我解了 thread-priority 衝突」,而是「**MediaPipe 的排程模型讓我不必手動管優先權,慢節點會被時間戳丟幀自然降級**」。
  > 📌 **第三家 SDK 的誠實版**:**火山引擎是當初評估用來「取代商湯 SenseME」、但最後仍採用 SenseME** 的一次**選型評估,並未 production 落地**。GL 污染的真實主角是 **Spine(client)+ PAG(Tencent)**。→ résumé 寫「Bytedance/SenseMe/Agora 整合衝突」與事實有落差,**建議改寫**為「**Agora + SenseME 整合,並解決 Spine / PAG 的 GL context 污染(黑畫面)**」。
- **R**:**消除黑畫面**,Agora / SenseME / 火山引擎 / Spine / PAG 在同一 GL context 穩定共存。〔可補:黑畫面客訴或相關 crash 下降數字〕
- **Trade-off / 反思**:教訓是「**共用 GL context 一定要有嚴格的 state save/restore 紀律,或乾脆給第三方套件獨立 child context**」——與其每次去抓誰污染,不如一開始就劃清 GL state 的 ownership 邊界。

## #6 — Gradle → Bazel 遷移 + build 系統

**證明**:infra ownership、Staff 級系統思維。

- **S/T**:因為 MediaPipe 的主要 build tool 就是 **Bazel**,團隊自然全用 Bazel。痛點是 ffmpeg-kit、ExoPlayer 這些三方庫原本走 Gradle:**要先 gradle build 出 AAR → 整合進來 → 再跑一次 build** 才能測,開發迴圈冗長,且**不能直接 native debug**。
- **A**:把 ffmpeg-kit、ExoPlayer 等三方庫從 Gradle 遷到 Bazel,做到 iOS framework / Android AAR 的 reproducible build。〔需要你補:這是你主動推的還是被指派?最難的部分是什麼(三方庫沒有 Bazel 支援要自己寫 BUILD?)?你怎麼說服團隊吞下遷移成本?〕
  > ✅ **codebase 查證(本 repo 本身就是 Bazel:`WORKSPACE` + `BUILD.bazel`)**:可佐證你 Bazel ownership 的近期工作——
  > • **Bazel 5.2.0 → 7.4.1** 升級以支援 macOS 26 / Xcode 26(commit `cf4c84ed8`、`0dbb5011e`),連帶處理 framework codesign / modulemap(`f2f457bfc`)、XNNPACK `select()` 預設平台條件(`c0b26ee10`)。
  > • **Tulsi → rules_xcodeproj** 換掉 Xcode 專案產生器(`66c10f752`)。
  > • ffmpeg-kit 以 Bazel 流程出 AAR(NDK 25 / cmake 3.31,`c87a9d02c`)。
  > **你的真實版本**:不是「說服團隊大遷移」的政治故事,而是「**因為要用 MediaPipe(Bazel),順手把 ffmpeg-kit / ExoPlayer 也整進 Bazel**」。最大的實際好處是 **native debug 的開發迴圈**:整進 Bazel 後可**直接改三方庫原始碼 → build → launch**,省掉「先 gradle 出 AAR → 再整合 → 再 build」的來回。**最難點**:這些庫沒現成 Bazel 支援,要**讀 Bazel 文件 + 套件結構**,自己寫 `cc_library`/`android_library`/`objc_library` 去包 prebuilt 並處理 header/ABI/`select()`。
- **R**:★ Android build time **−50%**、iOS **−30%**(**估計值**,以**發版/release build 時間**粗測;主要省在 **Bazel incremental build + 快取**,沒改到的檔案不重編)。更大的隱性收益是**開發迴圈變短**——能直接改三方庫 code 做 native debug。⚠️ **誠實標記**:這兩個數字是估計、非精密 benchmark,面試講「估計」即可,別當精確值報。
- **Trade-off**:Bazel 學習曲線 vs Gradle 生態——但這次**幾乎不用說服**,因為團隊本來就用 MediaPipe(Bazel),把三方庫也統一進 Bazel 反而讓大家開發/debug 更順,是順水推舟。
- **反思**:〔可補:要把 −50%/−30% 講得更硬,下次會在 CI 固定量「clean vs incremental」兩條基準線,而不是只看發版時間估計〕

## #7 — Android 15 16KB page-size 遷移(零停機)

**證明**:平台級遷移、跨整個 native stack 的協調。

- **S/T**:**Google Play policy 強制要求** Android 15 起 App 必須支援 **16KB page size**,整個 native stack 的三方 .so 都要相容,否則 Android 用戶無法正常使用 / 上架受阻。
- **A**:**獨力 own** 整個 native stack 的 16KB 遷移——逐一把三方 .so 升級 / 重編到 16KB-aligned。
  > ✅ **codebase 查證(哪些 .so 不相容、怎麼逐一處理——這題碼庫答得很完整)**:
  > • **SenseME**:`libst_mobile.so` 換成 **9.7.7**(支援 16KB,commit `e94961104`)。
  > • **ffmpeg-kit**:`ffmpeg-kit-min-aar` 升 **6.0**(16KB page,`015938002`)。
  > • **openh264**:`1.1.10` 不支援 16KB,需重編(issue 918,`2cfbbecb8`)。
  > • **libuvccamera**(UVC 外接相機):更新支援 16KB + 權限 workaround(`7da27715a`)。
  > • **Linker**:加 `common-page-size`(`-Wl,-z,common-page-size=16384` 類)防止靜態庫造成 crash(`d58b09915`);Android engine 產出腳本補 flag(`4c07c1df0`)。
  > 一句話 framing:「**盤點整條 native stack 每個 prebuilt .so 是否 16KB-aligned,逐一升版/重編(SenseME、ffmpeg-kit、openh264、libuvccamera)並在 linker 指定 page size**」。
  > ★ **最硬的亮點(務必講)**:**ffmpeg-kit 上游當時宣布不支援(專案後來也停更)**,沒有現成 16KB 版本可用——我**翻出之前 archive 的 repo、找到 build config、自己修改後重新 build** 出 16KB-aligned 的 ffmpeg-kit。這是「上游棄坑、我自己接手把它做出來」的 ownership 證據。
  > ⚠️ **「零停機」建議拿掉**:你自己也說不太懂這詞、在 App 場景也沒明確意義(16KB-aligned 本就向下相容 4KB 舊機,一份 build 通吃)。別在 résumé 放你無法解釋的詞。
  > 📓 **daily-memo 佐證(具體到版本/repo,可口說)**:ffmpeg-kit 用社群 fork **`AliAkhgar/ffmpeg-kit-16KB`** 自 build **6.0**;**openh264** 官方版不支援,我從 `openh264-roi-1.1.2_16` **改 `android.mk` 自己編**;**SenseME 9.7.2 不支援、9.7.6+ 才行**,我定位版本邊界後**找商湯升出 9.7.7**(廠商升級談判);連**靜態 link 進 `mediapipe_jni.so` 的 protobuf** 都因缺 `-Wl,-z,max-page-size=16384` 而要處理。背景:**MediaPipe 自己也是最新版(v0.10.26 / NDK r28)才預設支援 16KB** → 等於追一個移動標靶。採用率:某時點 **69% Android 用戶**已在 16KB 相容版(Android 1.5.2 / iOS 1.4.0 起內含)。
- **R**:讓 App 符合 Google 16KB 強制要求,**影響全部 Android 用戶**;遷移的 binary 含 **SenseME、ffmpeg-kit、openh264、libuvccamera**。〔可補:Android 佔 140K DAU 多少;沒做會被 Google Play 擋上架的期限壓力〕
- **反思**:〔可補:上游棄坑這種事之後怎麼防——把關鍵三方庫的 build config 自己 fork / archive 一份,不要只依賴上游 release〕

## #8 — Box2D Bonk Gift / Live2D Vtuber(圖形渲染)

**證明**:圖形/渲染深度、產品交付。

- **Bonk Gift**:Box2D 互動禮物系統 + SenseMe 手勢辨識,三種互動模式(Normal/Merge/Gesture);gift-manifest protobuf schema + 從 ZIP 載入 WebP/PNG/WAV 的記憶體高效快取。
- **Vtuber**:Live2D 端到端串流,mesh-attached 裝飾的 sprite anchoring、即時換髮色/服裝的 palette shader、submodel 換裝系統,**★ 25fps on-device**。
- **兩個都帶(都成功落地過)**:Bonk Gift + Vtuber 都上線且效果好;**Vtuber 後來下架**(可誠實提——重點是「曾成功落地」,下架是產品決策非技術失敗)。
- **最難 / 最有成就感(你的口述)**:難在**整合並吃透第三方 SDK(Live2D)**,有時還得**幫 SDK 補坑**;另一個難點是**寫 MVP(Model-View-Projection)矩陣、重做相機座標等運算**(為此**讀了相關論文**)——把 Live2D 模型/裝飾正確投影到相機座標。最有成就感的是**真的上架落地 + 解掉這些問題**。
- **渲染坑(可講)**:**premultiplied alpha**——貼圖 blend 邊緣出現黑邊/白邊,因為 RGB 沒先乘 alpha 就 blend(或 source 已 premultiplied 卻用 straight-alpha 的 blend func);修法是統一 blend equation / 在 shader 正確處理。**grayscale-to-color palette shader**——用一張灰階圖當 mask、在 shader 映射到目標色盤,**即時換髮色/服裝且省記憶體**。
  > 📓 **daily-memo 佐證(精確根因 + 日期)**:premultiplied alpha 根因 **2024-04-01** 找到——**iOS 載入貼圖時會 pre-multiply,但我的 shader 假設 straight-alpha**,換髮色時邊緣出現暗色光暈;palette shader 叫 `FragShaderChangeColorWithPalette`(灰階→index→paletteArray 查表)。Live2D **2023-12-26 先在 iOS 上線**;submodel 換色 2024-03 用 multiply/screen 完成。Bonk gift 有正式 Confluence 文件(屬正式 production 功能)。
- 〔25fps:依你說的**不深講**,頂多「on-device 即時渲染」一句帶過,沒有特別的優化故事〕
  > ✅ **codebase 查證**:
  > • **Bonk Gift**:`BonkLangStreamerCalculator` 在主 graph(`langstreamer.pbtxt`),三模式之一的 **Gesture** 靠 SenseME 手勢;近期做了**動態手部 frame-skipping 控制**(`SetHandProcessEveryFrame`,commit `e05c5287a`)——手勢禮物觸發時才每幀跑手部偵測、平時跳幀省算力。「圖形互動 × 即時 CV 取捨」的好深挖點。
  > • **Live2D Vtuber**:`live2dspritemanager` sprite cache key 改 `int64_t`、用 timestamp 當 key(`61ba3cf3d`)——對應 sprite anchoring / submodel 換裝。
  > **面試帶法**:Bonk Gift 講「Box2D 物理 + SenseME 手勢 + frame-skipping 取捨」;Vtuber 講「**第三方 SDK 整合 + 補坑 + MVP 矩陣/相機座標運算(讀論文)** + premultiplied alpha / palette shader」——兩個各一個技術亮點,不要混講。

## #9 — Quanta 三款兒童智慧手錶(GizmoWatch 2 / Disney / T-Mobile SyncUp)

**證明**:跨時區跨文化協作、出貨消費性硬體、scale。

- **S/T**:交付 3 款電信商通路的兒童手錶 App,零售上架(Verizon、T-Mobile)。
- **A**:擔任 **client 端 ↔ Azure backend 的註冊/串接整合**(Azure IoT SDK + **DPS 裝置 provisioning** 註冊流);參與 T-Mobile SyncUp 架構(**MVVM + RxKotlin**、Repository、DB 層、API、source control、scrum)+ **real-time location tracking**;跟 **美國(Verizon / T-Mobile)+ 法國** 團隊**跨時區開會、規格來回**做 E2E。三款手錶**累計出貨 1M+**。
  > ⚠️ **誠實校正(這次 grill 出來,務必改 résumé)**:
  > • **MQTT 長連線「省電 30%」+ DB query「快 5 倍」→ 你說都沒有這故事**。(註:這兩個**不在你真正的 `resume.md`**,是 **story bank 初稿被 AI 虛構**的;確認沒有 → **別補進履歷**。)
  > • **真履歷的 DPS「1M+ device provisioning」**:1M+ 是**三款手錶的出貨量**,你的角色是 **client↔Azure 註冊串接**;措辭可留「我串接的 provisioning 流程支撐出貨 1M+」,**別讓人讀成「我 provision 了 1M 台」**。
  > • **真履歷寫「Owned end-to-end architecture of T-Mobile SyncUp」**——你實際是**串接 + 參與**,「owned end-to-end」略微 overclaim,建議改「contributed to / built the client-side …」。
  > ℹ️ 此故事不在本 codebase(Quanta、不同 repo);已用 **2022 履歷 + portfolio** 佐證:GizmoWatch、Disney Edition、T-Mobile SyncUp、Azure、real-time location tracking 都確認屬實。
  > 💡 **改主打真實亮點**:① **出貨 1M+ 消費性硬體、3 款電信商通路 SKU(Verizon GizmoWatch / Disney / T-Mobile SyncUp)**——scale 與出貨能力;② **client↔Azure DPS/IoT Hub 註冊串接**;③ **跨時區(美國+法國)scrum 交付 + 全球團隊 E2E**;④ real-time location tracking。
- **反思**:〔可補:電信商通路硬體上架前的合規 / E2E 嚴謹度,或跨時區協作你後來找到的有效做法(非同步交接、規格先寫清楚再開會)——挑你真有體會的〕

---

## #10 — PAG GPU-native 渲染優化(CPU readback → render-to-texture)

**證明**:GPU pipeline 設計、量化效能優化、減少 CPU↔GPU 搬移。

- **S/T**:騰訊 PAG 特效原本走 **CPU readback**——`PAGDecoder.readFrame()` 解到 CPU RGBA `ImageFrame`,再上傳成 `GlTexture` 才能進 render。每幀一次 CPU↔GPU 來回,**單層 PAG 就讓 CPU 尖峰衝到 81–85%**(基線 57%)→ **真實禮物場景容易發燙 / jank(主要痛點)**;GL state 污染的黑畫面是次要,`ResetGlState` 清掉即可。
- **A**:改成 **GPU-native 渲染**——用 **PAGPlayer + PAGSurface** 取代 `PAGDecoder::readFrame()`,讓 PAG 直接 render 到 GPU texture。關鍵設計(commit `ea0aa5a40`):**① 用 MediaPipe buffer pool 之外的專屬 GL texture 避免 pool 爭用;② shader-based blit 回 pool GpuBuffer,避開 dual-FBO 衝突;③ flush/blit 後 `ResetGlState` 防止下游 VAO 被污染(呼應 #5 的 PAG 污染);④ `setMaxFrameRate(25)` 對齊 pipeline tick;⑤ flush-false 時 re-emit 上一幀做 PAG dedup**。**兩個坑(你的理解)**:**dual-FBO 衝突**——PAG 的 `PAGSurface` 自己綁了一個 FBO,跟 MediaPipe pool GpuBuffer 的 FBO 同時綁會打架 → 所以讓 PAG render 到專屬 texture、再 shader blit 回 pool。**GL state 污染**——PAG 內部的 **tgfx 引擎 render 完沒清 GL state**,直接接續 GL 操作就 invalid → 加 `ResetGlState` 清掉。
  > ⚠️ **誠實提醒**:commit `ea0aa5a40` 標了 `Co-Authored-By: Claude`——這塊是 AI 協作完成的。面試講沒問題(現在普遍),但**上面每個設計決策(專屬 texture / shader blit / ResetGlState / dedup)你都要能自己解釋**,別只背 commit message。另注:git 真實演進是 **PAGPlayer+PAGSurface(GPU 渲染)→ 再換 PAGDecoder 提速 → render-to-texture**,比「一步到位」複雜,口說可講「迭代了幾版」。
  > 💡 **你的角色(誠實且漂亮)**:**主思路、架構、debug 由你主導,AI 負責實作**。你的自有 debug 例子:**「沒有 output」時把底圖塗成紅色**,藉此分辨「真的沒 output」還是「其實有 output、只是 alpha 全 1.0 看起來像沒畫」。面試這樣講最穩:**「架構與除錯是我定的,實作我用 AI 加速」**——再補這個紅底圖 debug 細節證明你在驅動。
- **R**:★ 單層 PAG 的 **CPU 尖峰 81–85% → 61–65%**(基線 57%);同時根除 readback 路徑的 GL state 污染黑畫面。〔可補:多層 PAG / 真實禮物場景的 CPU、fps、發燙改善;影響哪些禮物〕
- **Trade-off / 反思**:先用 CPU readback 是**為了最快讓 PAG 禮物上線**(已知技術債);GPU-native 的代價是**更難 debug**(FBO 打架、tgfx 沒清 GL state 這些坑),但換到 CPU 大降、不發燙值得。反思:GPU pipeline 的 GL state 一定要有「**用完即 `ResetGlState`**」的紀律——跟 #5 的 Spine/PAG 污染是同一課。
  > 📓 **daily-memo 佐證(2026-06-05, 2026 Q2)**:`把從 cpu readback 改回到 gpu-native，減少每個 frame cpu ↔ gpu 的資料搬移`;`原版是 PAGDecoder 去 readFrame 回 CPU RGBA ImageFrame 後再轉成 GlTexture`;`新版是用 render to texture … 在 tgfx 引擎 gpu decode 後 render 完，直接寫進去傳進來的 gltexture`。CPU:舊 57%→81-85% vs 新 57%→61-65%。
  > ⚠️ **此故事尚未 grill**:上面 S/T、R 的機制由 memo + 程式邏輯推得,**人為決策/取捨/你個人 role 待補**。跟我說「**grill 我 #10**」可補完口說稿。

---

## #11 — 依 GPU vendor 動態選 inference delegate(heterogeneous GPU 健壯性)

**證明**:edge-AI 跨異質裝置部署的健壯性、infra ownership、資料驅動決策。

- **S/T**:AI mask gift 的 **portrait segmentation 推論**在**特定 GPU(尤其 PowerVR)** 上 **output 不穩**——有時整個 frame 全黑、甚至串流撐不久;查網路也有該晶片跑 inference model 的類似討論。結論:**不能一律用 GPU delegate**。
- **A**:設計**依 GPU vendor 動態切換 CPU / GPU inference delegate** 的機制——核心是**建黑名單**:仿 **Chromium 的 `gpu_driver_bug_list`(AND/OR 條件語意)** 做 `GpuBlockRule`,命中的 GPU 退回 CPU delegate;**GPU probe 失敗也 fallback CPU**;PowerVR 特別用 **LOAD_ASYNC** 避免 GPU 載 model 出事。**獨立設計**(無 AI 協作,對比 #10)。
  > 📓 git 佐證:`ce69a270b` 動態 delegate(新增 `background_subgraph_dynamic_delegate.pbtxt`、`portrait_segmentation_dynamic_delegate.pbtxt`)、`4f0c222de` Chromium 式 `GpuBlockRule`、`aad8f611e` probe 失敗 fallback CPU、`51b10e4cc` PowerVR LOAD_ASYNC、`315ef9946` PowerVR 串流時間變短。
- **R**:問題 GPU 上**恢復穩定**(消除全黑 frame / 串流變短);其餘 GPU 維持 GPU delegate → **segmentation 加速、fps 提升**。〔可補:影響機型數 / 比例、全黑或 crash 回報下降數字〕
- **Trade-off**:**預設走更快的 GPU delegate**(segmentation 加速、fps 升),**只有黑名單機型退回 CPU**(較慢、較耗 CPU)換穩定——是「**資料驅動的黑名單擋掉已知有問題的 GPU**」,不是保守地一律 CPU。
- **反思**:〔可補:黑名單需持續維護(新機型/新 driver),可加遙測自動標記異常 GPU,而非只靠手動回報〕

---

## 候選新故事(daily-memo 挖出來、可獨立成題)

> 這些是 grill 之外、memo 裡有扎實佐證的料,若某場面試需要更多彈藥可展開。

- **★ PAG GPU-native 渲染優化 → 已升格為 #10**;**★ 動態 delegate 選擇 → 已升格為 #11**(見上)。
- **A/V smoothing 悖論(debugging 故事)**:見 #3 ③ 的「平滑反而藏住告警」——「修法讓問題更糟、且對監控隱形」的經典 debugging 題,可從 #3 抽出獨立講。
- **SenseME 16KB 廠商升級談判**:見 #7——從第一性原理判斷三方 binary 的 16KB 相容性、定位版本邊界(9.7.2→9.7.6→談到 9.7.7),展現廠商關係與升級協調。
- **B. Xcode 26 / Bazel 7 / macOS 26 工具鏈升級 + LangEngineSDK module stability**(已評選,★★,未 grill):重大工具鏈跳版時保團隊不被卡;與 #6 重疊,可併入或獨立。git:`cf4c84ed8`、`f426983e0`(module stability)、`66c10f752`(Tulsi→rules_xcodeproj)。
- **C. WebP/PAG background subgraph 架構**(已評選,★★,未 grill):mux/demux 切換 WebP 與 PAG 解碼路徑、`BackgroundDecodePathSelectCalculator` + 抽象設計 spec → 可成「pluggable 解碼路徑架構」題。
- **其他較弱(feature-level,暫不單獨成題)**:Media Editor SDK、H265、UVC camera、GPU particle。需要時再 grill。

---

## 待辦缺口總覽

**✅ 已 grill 完、補成可口說的(2026-06-11 session)**:**#1–#11 全部十一則**(#10 PAG GPU-native、#11 動態 delegate 選擇,新增)——含 root-cause 推理、量測方法的誠實邊界、你-vs-團隊界線、AI 協作的誠實標記、以及多個**履歷校正**(見下)。

**⚠️ 這次 grill 出來、需要你回去改 résumé 的誠實校正(重要)**:
1. **#4 別講「改成 nanosecond 就修好」**——ns 是 bug、造成 mosaic、後來被 revert;正解是「微秒 + 單調守衛」。
2. **#5 第三家是 ByteDance 火山引擎,但只是「評估取代商湯、最後沒採用」**,非 production 整合;GL 污染真實主角是 Spine + PAG。**thread-priority 衝突你沒有故事,建議從履歷拿掉**。
3. **#7「零停機」建議拿掉**(無意義);真亮點是 ffmpeg-kit 上游棄坑、你自己 archive 重 build。
4. **#2 −50% / #6 −50%/−30% 是估計值**(preview-vs-player 目測 / 發版時間),面試講「估計」,別當精密 benchmark。
5. **#9**:**MQTT 省電 30% / DB ×5 是 story bank 初稿虛構的(真 `resume.md` 沒有),你也確認沒這故事 → 別補進履歷**;真履歷的 DPS「1M+ provisioning」是**出貨量**、你角色是 client↔Azure 串接;「Owned end-to-end architecture」略 overclaim 建議軟化。

**剩餘缺口(可選,有空再補)**:
- 各則的**回升數字**(#3 卡頓投訴下降、#4 止血後觀看回升、#7 Android DAU 佔比)——有就加,沒有不強掛。
- #9 換主打真實亮點(出貨 1M+、3 SKU、client↔Azure 串接、跨時區 E2E),已寫進 #9。

九則都 grill 過了。下一步可說「**幫我把校正 1–5 直接改進 résumé**」(給我履歷檔路徑),或挑某則要我**寫成完整逐字口說稿**。

---

# 逐字稿(60–90 秒口說版)

> 以下是可以直接念、直接練的口說稿(中英夾雜,符合實際面試講法)。已內建前面 grill 出來的誠實邊界。**先練 #4 與 #1**。括號是停頓/語氣提示,不用念出來。英文場合可直接把技術名詞留英文、敘述翻成英文。

## 逐字稿 #4 — iOS 時間戳 bug(經典 debugging,~90 秒)

有一次我們收到**觀眾投訴**,說看 iPhone 主播的直播會出現**馬賽克破圖**,而且是跨平台的——web、Android、iOS 觀眾都看得到。

我先用 **ffmpeg 去 parse 串流,發現 DTS 時間戳在亂跳**;為了確認問題是出在推流端而不是播放器,我用 **ffplay 跟 VLC 這種已知正確的參考播放器**去複現,確定是 streamer 端的問題。

回到源頭追,根因是:**iPhone 16 上市後效能太好**,我們原本用**微秒**解析度取時間戳,結果相鄰兩幀會落在同一個微秒、產生**重複時間戳**,而 MediaPipe 要求時間戳嚴格遞增,就報 timestamp mismatch。我第一直覺是把解析度提高到**奈秒**——重複是沒了,**但反而讓觀眾端開始馬賽克**。

(這裡是關鍵)問題從來不是解析度,是我**違反了 MediaPipe 的單位契約**。它的 Timestamp API 一律把傳入的整數當微秒,我餵奈秒不是精度差 1000 倍,是**尺度差 1000 倍**——MediaPipe 以為每一幀間隔 15 秒,整個編碼節奏就垮了。

所以最後正解是:**時間戳回到微秒、尊重單位契約**,另外在送進 graph 的**單一入口加一道單調性守衛**,時間戳沒遞增就直接丟那一幀。我先花一週做 workaround 止血,大約一個月後上完整正解。

我學到最重要的一課是:**不要假設成熟的 framework 壞掉——root cause 其實就藏在它文件的一行字裡。**

## 逐字稿 #1 — LangStreamer v3 架構(ownership,~85 秒)

我在 Lang Live 的核心工作是 **LangStreamer v3**。v3 之前 Android 跟 iOS **各維護一份 codebase**,雙端常常實現不一致;舊版又是 **single thread**,很難擴展新特效跟禮物。架構方向是我**前主管定的**——重構成跑在 **Google MediaPipe** 上的 C++ 跨平台 pipeline,雙端共用同一份。

我個人擁有的是 **platform 層**:Android/iOS 的 camera、audio 採集、Android 的 **JNI 跟 C++ 橋接**、跟 MediaPipe 的串接、把原本的業務邏輯 migrate 進新架構,還有整個 **FSM 狀態機跟事件回報系統**。後期幾乎每個模組我都經手維護。

選 MediaPipe 的理由很實際:calculator graph 是純 C++,雙端共用一份 pipeline,連 render 都只要寫一套 OpenGL ES shader;而且原生支援多執行緒、GPU delegate、TFLite,我們的美顏、手勢、AI 去背都能拆成獨立 calculator 掛上去。

成果是:**開發成本大概砍半**(主邏輯 native 寫一次、雙端只串 API);**新禮物特效從大約一個月縮短到兩週**;**crash-free 從 96% 提升到 99%**,主要來自統一 codebase;整個平台支撐到 **140K+ DAU**。代價是學習曲線很陡,我後來把踩過的坑**文件化、做成上手指南**,讓團隊都能接手。

## 逐字稿 #3 — A/V 同步監控(假設驅動 debugging,~90 秒)

有一陣子觀眾一直反映看直播**卡頓**,同時我們發現主播端 **CPU 偏高、手機發燙**。問題很模糊、難複現,所以我第一步是**先讓它可量測**:加了一組指標——audio/video diff、CPU、溫度——還**自己寫了一個 ffmpeg 的 skill 去量每一幀的 PTS**。大概不同步到 **300ms** 觀眾就有感。

追下去發現 root cause **不只一個、橫跨三層**:第一,主播常接**外部聲卡、廠商品質參差**,採集端時間戳本身就壞;第二,壞掉的時間戳會**污染 MediaPipe 每個 node 的 frame pacing**;第三,iOS 切換麥克風 route 時,audio PTS 的**平滑機制反而連鎖插入靜音**。

(這段我蠻自豪)我原本有一個假設叫 **burst-read**,以為是音訊 buffer 批次進來、時間戳擠在一起。但我把它儀器化之後,**實測發現它從來沒被觸發**,所以我**否證了自己的假設**,改判定成 frame pacing 問題。

更深的一個發現是:那個 PTS 平滑本來要幫忙,卻把一個乾淨的 300ms 缺口**攤成連續好幾幀的偏移**,既讓 UX 更糟,又因為**每幀 diff 都壓在門檻下、讓告警根本沒響**——等於把症狀對監控藏起來。

最後修法是綜合的:diff 超門檻就 bypass 平滑、優化各 node 執行時間改善 pacing、再跟 **CDN 廠商協作做轉碼重推重新對齊**,卡頓回報就下降了。

## 逐字稿 #2 — 端到端延遲 4–5s→2–3s(~70 秒)

我們的直播端到端延遲原本是 **4 到 5 秒**,很影響觀眾互動。在 v3 我做了幾件事把它降到 **2 到 3 秒**:優化 RTMP、調整碼率策略、**編碼器拿掉 B-frame**——因為 B-frame 的重排序會增加延遲,直播場景不適合——再加上整個推流框架比 v2 更 robust。

碼率策略我們用的是**「急降緩升」**:網路一差立刻把碼率砍下去,網路回來再緩慢爬升,**明確以順暢為優先**。我們其實也評估過用 **AI 預測模型**來做自適應,但實測效果不如急降緩升,就用數據選回了簡單可靠的版本。

(誠實補一句)這個 4–5 秒到 2–3 秒是 **v3 跟 v2 的對比**,做法是直接並排比對推流端 preview 跟 player 端的畫面差,是**經驗實測、不是精密分段量測**;如果再做一次,我會在畫面打 marker 做分段歸因,讓數字更硬。

## 逐字稿 #11 — 動態 delegate 選擇(edge-AI 健壯性,~65 秒)

我們的 AI 禮物會用到 **portrait segmentation** 的即時推論。我發現在特定 GPU 上——特別是 **PowerVR**——這個推論的 output 很不穩,有時候會**整個 frame 全黑**;我去查也發現網路上對這顆晶片跑 inference 有類似災情。

所以我設計了一套**依 GPU vendor 動態選 inference delegate** 的機制。核心是一個**資料驅動的黑名單**——我參考了 **Chromium 的 gpu_driver_bug_list**,用它那種 AND/OR 條件的語意去寫 GpuBlockRule;命中黑名單的 GPU 就退回 CPU delegate,GPU 偵測本身失敗也 fallback CPU,PowerVR 還特別用 **LOAD_ASYNC** 避免它載 model 時出事。

這裡的取捨是:**GPU delegate 其實是首選**,因為 segmentation 會加速、fps 更高;我只把**已知有問題的 GPU 退回 CPU**,換取穩定。所以不是保守地一律 CPU,而是用資料驅動的方式擋掉壞掉的 GPU、其他人照樣享受 GPU 的速度。這整套是我**獨立設計**的。

---

> **練法建議**:每則先照稿念 2–3 次抓節奏,再丟掉稿用自己的話講一次。面試官最常追問的點:#4「你怎麼確定是 streamer 不是 player」(答:reference player 反證)、#3「你怎麼證明 burst 假設錯了」(答:儀器化後沒觸發)、#11「為什麼不乾脆全用 CPU」(答:GPU 更快、只擋黑名單)。英文場合要我產一版 English script 再跟我說。
