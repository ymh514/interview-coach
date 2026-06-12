# 弱項清單

誠實列出來,讀書計畫優先打這些。每補強一項就標日期。

## 技術弱項

### Multithreading / Deadlock
- ✅ **Deadlock 定義**:了解(兩個 thread 互持對方要的資源,永遠卡住)
- ⚠️ **Deadlock 四條件**:練習時沒答出來 → 現在要背起來
  1. Mutual Exclusion、2. Hold and Wait、3. No Preemption、4. Circular Wait
- ⚠️ **Thread-safe queue**:練習時答 `atomic`(錯) → 正解是 `mutex` + `condition_variable`
- ✅ **Condition variable + spurious wakeup**:了解,要傳 predicate 給 `cv.wait()`
- ✅ **`atomic` 適用場景**:單一變數操作(counter),不能保護 queue 這種複合操作
- 待補:lock-free 資料結構(面試不一定考,有空再看)

### const / pointer 語法
- ⚠️ 基本規則懂了,進階用法待練習(見下方 sample code)
- 待補:函式參數、回傳值的 const 慣用法、`const` member function

### System Design
- 〔需要你補:你最沒把握的環節?分散式?storage?〕

### NeetCode 150
- 〔需要你補:哪幾類最弱?(DP、graph、interval…)〕

## 面試表現弱項(從 feedback-log 回填)

- **答題太破碎**:技術點列得出來但沒連成論點,面試官聽不到「所以呢」(題三 ASR 橋接)
- **四個條件沒背起來**:被問到 deadlock 條件時漏答(2026-06-12 練習)

## 補強進度

| 項目 | 狀態 | 最後練習日 |
|------|------|-----------|
| deadlock 四條件 | ⚠️ 練習時漏答,需再背 | 2026-06-12 |
| thread-safe queue (mutex+condvar) | ⚠️ 練習時答錯(atomic),已學正解 | 2026-06-12 |
| const/pointer 語法 | 🔄 進行中 | 2026-06-12 |
| multithreading/deadlock 整體 | 🔄 進行中 | 2026-06-12 |
