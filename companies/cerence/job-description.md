# JD — Cerence · C++ Engineer

## 原始 JD

**Responsibilities**
- Analyze and implement product requirements from projects.
- Integrate the latest voice recognition technologies.
- Develop unit tests to ensure the product quality.
- Improve the performance, latency, memory.
- Develop product for different platforms.
- Write technical documents (API doc, User Guide, Footprint doc, etc.).
- Support the projects which use our SDK.

**Education**: Bachelor's or master's in CS, EE, software engineering, etc.

**Min experience**: 3 years.

**Qualifications**
- Strong C/C++; proficient in **multi-thread programming, memory management**, etc.
- Familiar with Git.
- Familiar with **Linux** developing environment.
- Python a plus.
- Can-do attitude, organized, strong responsibility, handle peak demands, team-work.

**Preferred**
- Speech recognition or related tech experience.
- Android development a plus.
- **JNI** a plus.
- Voice interaction design/dev background; JavaScript/TypeScript; Python script.

---

## 拆解(我的分析)

Cerence = 車用語音 AI(從 Nuance 車用部門分拆)。這是一個 **C/C++ SDK 工程師**職,做語音辨識 SDK、跨平台交付、效能/記憶體優化、並支援使用 SDK 的專案。

**強匹配(主打這些)**
- **跨平台 C++ SDK ownership** — 你整份 CV 的核心。「Develop product for different platforms」「Support the projects which use our SDK」幾乎是照你工作寫的。
- **你是 SDK↔client 的溝通窗口**(bio 新增那段)→ 直接對應「Support the projects which use our SDK」+「Analyze and implement product requirements」。這是你的差異化優勢,務必主動帶出。
- **Performance / latency / memory** → 你的延遲 4–5s→2–3s(#2)、Bazel build 優化(#6)。
- **Android + JNI** → preferred 項目,你強。MediaPipe C++ ↔ Android 本來就走 JNI。
- **Unit tests / 技術文件** → 你 CV 有定 coding standards / API spec 的經驗。

**Gap / 他們會戳的點(必須先補)**
1. ⚠️ **multi-thread programming + memory management** — JD 明列為「basic C++ knowledge」核心要求,而這**正是你 `weak-areas.md` 的弱項**。這場最高優先級複習。預期會被白板/口頭問:race condition、deadlock 四條件、lock vs lock-free、RAII/智慧指標、記憶體洩漏怎麼抓。
2. **Speech / voice recognition 領域** — 你沒做過 ASR。橋接話術:你做的是**端上 edge AI 模型整合(MediaPipe on-device)**,即時、低延遲、跨平台——和語音辨識 SDK 的工程挑戰高度相鄰(模型部署、執行緒、延遲、記憶體足跡)。別假裝懂 ASR,但把工程框架對接過去。
3. **Linux 開發環境** — 你是 mobile(Android/iOS)背景,純 Linux 較淡。誠實 + 可遷移性(Bazel 跨平台、command-line build)。

**這場該主打的 story(先豐富這幾則)**
- **#1 LangStreamer v3**(跨平台 C++ SDK + 端上 edge AI = 最接近他們語音 SDK 的類比)
- **#5 三方 SDK 整合**(thread / memory / GL 疑難 → 打 multi-thread + memory 要求)
- **#2 延遲優化**(直接對應 performance/latency/memory bullet)
- 加 **#4 iOS DTS bug**(memory/timestamp 級別的 root-cause debugging)
