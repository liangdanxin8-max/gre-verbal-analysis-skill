---
name: gre-verbal-analysis
description: >
  Automates full GRE Verbal mock exam analysis. Accepts user-provided GRE question data or a gre.viplgw.cn mock result page and, when browser automation is available, uses low-frequency human-paced visual page navigation with screenshots/OCR rather than direct API crawling. Generates a complete
  analysis report in both MD and HTML formats. Each question gets: categorization,
  full option-by-option explanation, sentence vocabulary with memory tips, mistake
  diagnosis (even for correct answers - user may have guessed), and methodology summary.
  Also produces an overall summary with ability matrix, SE/TC/SC methodology sections,
  vocabulary weakness list, and test-taking strategy. Use this skill when the user
  wants to analyze a GRE Verbal mock exam, review their performance, or build a
  personalized GRE study system.
---

# GRE Verbal Analysis Skill

Automates the complete workflow of analyzing GRE Verbal mock exam results. When a gre.viplgw.cn page is used, prefer human-paced browser visual navigation and screenshots over direct URL/API crawling.

## When to Use This Skill

Trigger on phrases like:
- "帮我分析GRE Verbal模考"
- "分析这个GRE Verbal结果"
- "做GRE做题体系"
- Any request involving `gre.viplgw.cn/mockResult` URL + Verbal analysis

## 多平台使用说明

本 Skill 的核心分析逻辑（Phase 2/3）与平台无关，可在任何 AI 平台上使用。

### WorkBuddy 用户
直接使用本 Skill，Phase 1 和 Phase 4 会自动调用 WorkBuddy 工具。

### Claude / OpenClaw / 其他 LLM 用户
1. 将 `SYSTEM_PROMPT.md` 的内容作为 system prompt 输入
2. 按提示提供题目数据（支持 URL / JSON / 截图粘贴三种格式）
3. AI 会按统一标准输出完整解析报告

### 无浏览器自动化能力时
手动粘贴题目数据，格式参考 `references/data_schema.json`。

---

## Overview

The skill executes a 4-phase workflow:

```
Phase 1: Collect — Prefer safe visual browser navigation, screenshots, and OCR/vision extraction for all questions
Phase 2: Analyze — For each question, generate full analysis (5 sections)
Phase 3: Report — Generate both MD and HTML reports
Phase 4: Deliver — Use deliver_attachments to send files to user
```

## Phase 1: Collect Questions Safely

Goal: collect the same structured question data while minimizing risk of rate limits or anti-bot triggers. Prefer user-provided text, uploaded screenshots, PDFs, saved HTML, or exported JSON when available. If the user wants the agent to operate the website, use **visual browser mode**: open the page like a normal user, click through questions, take screenshots, and extract text with vision/OCR.

### Safe-efficiency policy

Use the fastest workflow that still looks like normal reading behavior:

1. **Prefer visual navigation over crawling**
   - Navigate by visible buttons/links on the page.
   - Take screenshots of the question area and answer/status area.
   - Use vision/OCR to extract question text, options, correct answer, user's answer, timing, and RC passage text.
   - Do not fetch backend APIs, inspect hidden JSON endpoints, enumerate IDs, or construct direct question URLs unless the user explicitly supplies an exported dataset.

2. **Keep one browser session**
   - Reuse the same logged-in browser tab/session and cookies.
   - Do not open many tabs.
   - Do not repeatedly reload the page.
   - Do not clear cookies, rotate identities, or switch IPs during collection.

3. **Human-paced timing**
   - Wait for the page to fully render before each screenshot.
   - Use small randomized pauses between actions: normally 4-9 seconds per question; 8-15 seconds for RC passages or heavy pages.
   - Do not use fixed sub-second delays.
   - Do not collect more than one page/question at the same exact time.

4. **Efficient screenshot strategy**
   - Capture only what is needed: question stem, passage, options, selected answer/correct answer, time spent, and question number.
   - For RC, first capture the full passage, then capture each associated question.
   - Avoid duplicate screenshots unless OCR/vision confidence is low.
   - If a screenshot is unclear, retry once after zooming or scrolling; do not loop repeatedly.

5. **Stop conditions**
   Stop immediately and tell the user the site may have rate-limited or blocked access if any of these occur:
   - HTTP 403, 429, 502, 503, captcha, access denied, forced login, blank content, or repeated network errors.
   - More than 2 consecutive failed page loads.
   - Page content stops changing after navigation.
   - The browser begins auto-refreshing or redirecting unexpectedly.

6. **Retry limits**
   - Retry a failed action at most once.
   - After a second failure, pause collection and ask the user to provide screenshots/exported data or try again later.
   - Never create an infinite retry loop.

### Browser automation guidelines

If agent-browser or another browser controller is available:

- Use click/scroll/screenshot actions on visible UI elements whenever possible.
- Keep `headless: false` or use a normal visible browser profile if the environment supports it.
- Use a normal viewport size and zoom level that makes screenshots readable.
- Avoid direct DOM scraping as the primary method. It may be used only to verify visible text already shown on screen, not to discover hidden data or backend endpoints.
- Avoid direct `open` calls to fabricated question URLs. Use page navigation controls instead.

### Data to Extract Per Question

Store as a structured JSON array following `references/data_schema.json`. For each question, collect:

- section number and question number
- question type: SC, SE, TC, or RC
- full stem/passage text and all options
- user's answer
- correct answer
- time spent
- whether the answer was correct
- highlighted sentence for RC questions, if visibly highlighted

**RC 高亮句提取（重要）**：
- Prefer extracting the visibly highlighted sentence from the screenshot/page view.
- **绝对不能猜测**高亮句内容。
- If the highlighted sentence is unclear or not visible, mark `[高亮句待确认]` and explain that the screenshot/page view did not show it clearly.
- Reference `references/rc_question_types.md` for highlighted-sentence question strategy.

### If visual collection is not possible

Ask the user for one of these inputs instead:

- pasted question text
- screenshots of each question/result page
- saved HTML files
- exported JSON matching `references/data_schema.json`

Do not fall back to bulk crawling or API enumeration.

## Phase 2: Analyze Each Question

### Required Analysis Sections (ALL 5 must be present for EVERY question)

#### Section 1: 题目解析 (Question Analysis)

Write full sentence analysis with translation. Identify:
- Logical connectives (yet/because/although/instead/unlike)
- Key signal words
- Direction of each blank (positive/negative/neutral)
- The "reveal word" that gives away the answer

#### Section 2: 逐选项分析 (Option-by-Option Explanation)

**MANDATORY for all 6 options (SE) or all options (SC/TC)**:
- Write a table with columns: 选项 | 词义 | 为什么对/错
- Every option must have its meaning explained
- Mark correct answers with ✅ and wrong with ❌
- Explain WHY each wrong option is wrong (not just "wrong")

Example format:
```
| (A) abstract | 抽象的 | ❌ "末日修辞"不是"抽象"的，方向偏差 |
| (B) complacent | 自满的 | ❌ 你选了这个。complacent = 对现状满足... |
| (D) overblown | 夸张过头的 | ✅ 与 apocalyptic rhetoric 同义复现 |
```

#### Section 3: 句中重点词/生词 (Key Vocabulary)

**MANDATORY table for EVERY question** (even correct ones):

| 词汇 | 词性 | 词义 | 记忆方法 |
|------|------|------|---------|
| word | pos. | meaning | memory tip |

Include ALL important words from the sentence, not just answer-related words.
Mark high-frequency GRE words with ⭐.

#### Section 4: 错因诊断 (Mistake Diagnosis)

**ALWAYS include this section, even for correct answers**:
- If correct: analyze whether user may have guessed (e.g., "⚠️ 仅耗时29s，可能蒙对")
- If incorrect: categorize the mistake type

Mistake types:
| 错误类型 | 说明 |
|---------|------|
| 词汇不懂 | 不认识关键词 |
| 逻辑方向搞反 | 正负方向判断错误 |
| 形近词混淆 | comely vs comic |
| 自行推断（RC细节题） | 没有原文依据 |
| 做题太快 | <60s for SE, <45s for SC |
| 未作答 | 白白丢分 |
| SE两词不近义 | 选了不是同义的两个词 |
| Unlike逻辑搞反 | 选了同义词而非反义词 |

#### Section 5: 方法论总结 (Methodology Summary)

Write a box for EVERY question with:
- The core method used (e.g., "同义复现法", "SE配对验证法", "instead转折结构")
- A memorable formula or checklist
- Connection to GRE test strategy

Format:
```
> **方法名**：
> - Key point 1
> - Key point 2
>
> **解题公式**：...
> **陷阱**：...
```

### Question Type Classification

Classify each question:
- **SC** (Single blank, 1-of-5): choose 1 from 5 options
- **SE** (Sentence Equivalence): choose 2 synonymous options from 6
- **TC** (Text Completion, 2-blank or 3-blank): fill in blanks
- **RC** (Reading Comprehension): reading questions

Further classify RC questions:
- 主旨题 (Main idea)
- 细节题 (Detail / According to the passage)
- 推断题 (Inference / implies / suggests)
- 目的题 (in order to / primary purpose)
- 多选题 (Select all that apply)
- 类比题 (analogy)

## Phase 3: Generate Reports

### Markdown Report

File: `GRE_Verbal_解析报告.md`

Structure:
```
# GRE Verbal 全题解析报告（完整版）
**日期/分数/总分**
**目标**：每题深度解析 + 方法论总结 → 形成做题体系

## 总览：你的弱点地图
（题型 | 次数 | 错题数 | 错误率 table）

# SECTION 1 · VERBAL（共11题）
---
## ❌/✅ S1-Qn | ID:xxxx | 题型 | 耗时
（5 analysis sections above）
---
（repeat for all 11 questions）

# SECTION 3 · VERBAL（共13题）
（same for all 13 questions）

# 综合总结：你的做题体系
## 一、题型能力矩阵
## 二、SE题方法论
## 三、TC题方法论
## 四、词汇弱点清单
## 五、RC题注意事项
## 六、下次模考策略
```

### HTML Report

File: `GRE_Verbal_解析报告.html`

Use the template in `assets/report_template.html` as a starting point.
The HTML report should have:
- Hero section with score display
- Color-coded question cards (red for wrong, green for correct)
- Collapsible sections for each analysis part
- Print-friendly styling

## Phase 4: Deliver

After both files are generated:
1. Use `deliver_attachments` to send both `.md` and `.html` files to user
2. Use `preview_url` to show the HTML report in the built-in browser
3. Append a brief note to working memory (memory/YYYY-MM-DD.md)

## Important Notes

### For Correct Answers

**NEVER skip analysis for correct answers**. The user may have guessed. Always include all 5 sections. Add a "✅ 做题检验" subsection:
```
### ✅ 做题检验
| 检查项 | 结果 |
|--------|------|
| 是否能找到原文依据？ | ✅ ... |
| 是否可能蒙对的？ | ⚠️ 耗时XXs，可能蒙的 |
| 排除法是否有效？ | ✅ ... |
```

### RC Question Principles

1. **细节题 (According to the passage)**: MUST find direct textual evidence. Never infer.
2. **推断题 (implies/suggests)**: Find the closest paraphrase of the text. Choose the most conservative inference.
3. **主旨题**: Summarize in one sentence. Eliminate "too broad" and "too specific" options.
4. **多选题**: Check EVERY option against the text. "According to the passage" = only what's in the text.
5. **Never leave a question blank**: Always guess if unsure.

### SE Question Principles

1. The two correct answers MUST be near-synonyms
2. Verify: can the two words be used interchangeably in the sentence?
3. "Unlike" = contrast signal → the two things are opposites
4. Minimum 60-90 seconds per SE question

### TC Question Principles

1. Find logical connectives first (yet/because/although/instead)
2. Determine direction (positive/negative) for each blank
3. The three blanks must form a coherent logical chain
4. Use known blanks to verify unknown blanks

## Reference Files

Load these files as needed during analysis:

- `references/gre_high_freq_words.md` — High-frequency GRE vocabulary (load when analyzing vocabulary sections)
- `references/se_synonym_clusters.md` — SE synonym word clusters to memorize (load when analyzing SE questions)
- `references/rc_question_types.md` — RC question type strategies (load when analyzing RC questions)
- `references/data_schema.json` — JSON schema for scraped question data

## Assets

These files are used in the output reports, not loaded into context:

- `assets/report_template.html` — HTML report template
- `assets/report_styles.css` — CSS styles for the report
