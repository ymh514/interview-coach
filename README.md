# Interview Prep 2026

在台外商 Senior/Staff C++ / 跨平台 SDK 工程師面試的準備系統。
單一真實來源 = 這個 git repo;Claude Code 當主引擎。

## 怎麼用(每場面試的儀式)

1. **建公司資料夾**:`cp -r companies/_template companies/<公司名>`
2. **貼 JD** 進 `companies/<公司名>/job-description.md`
3. **做研究**:用 `prompts/company-research.md` 產出 `research-memo.md`
4. **讀書計畫**:`study-plan.md` 排這場要補的東西(JD 技術棧 + 我的弱項)
5. **Cheat sheet**:面試前用 `prompts/cheat-sheet.md` 產 `cheat-sheet.md`
6. **面試後**:把內容(onsite 口述 / online 逐字稿)給 Claude → 寫進 `feedback-log.md` → `git commit`
7. 下一場的 cheat sheet 自動帶入上次的修正點

## 結構

```
cv.md                  # canonical CV
corpus/                # 會進 git、會 push 的非敏感語料
  story-bank.md        # STAR 技術故事庫 ← 面試彈藥的核心
  bio.md               # 英文 + 中文自我介紹
  weak-areas.md        # 弱項清單
prompts/               # 可重用 prompt
companies/
  _template/           # 每場真面試複製一份
private/               # gitignored:薪資、各公司坦白負評
```

## 規則

- 敏感資料(薪資、負評)只放 `private/`,已 gitignore,不會上 remote。
- 跨機器同步:`corpus/` 走 git;`private/` 不走 git(用 iCloud/手動 copy,或只留主力機)。
- 教練原則、雙模式、雙語設定都在 `CLAUDE.md`。
