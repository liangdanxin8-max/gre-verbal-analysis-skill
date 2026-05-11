# GRE Verbal Analysis

AI-powered GRE Verbal mock exam analyzer. Scrapes exam results from gre.viplgw.cn, generates comprehensive per-question analysis with vocabulary breakdowns, mistake diagnosis, and methodology summaries.

Built as a [WorkBuddy Skill](https://www.codebuddy.cn) — an AI agent workflow that automates the entire analysis pipeline.

## Features

- **Full 24-question analysis**: Section 1 (11 Qs) + Section 3 (13 Qs)
- **Every question analyzed deeply** — even correct ones (to catch lucky guesses)
- **5 analysis sections per question**:
  1. Sentence analysis with logic breakdown
  2. Option-by-option explanation (all options, not just wrong ones)
  3. Key vocabulary with memory tips and word roots
  4. Mistake diagnosis with categorized error types
  5. Methodology summary building a test-taking system
- **HTML + Markdown reports**: Beautiful, print-friendly, mobile-responsive
- **Skill-based architecture**: Reusable AI workflow with reference knowledge base

## Project Structure

```
gre-verbal-analysis/
├── README.md                    # This file
├── LICENSE                      # MIT License
├── .gitignore                   # Git ignore rules
├── skill/
│   ├── SKILL.md                 # Core workflow definition (4 phases)
│   ├── scripts/                 # (Reserved for automation scripts)
│   ├── references/
│   │   ├── gre_high_freq_words.md       # GRE high-frequency vocabulary
│   │   ├── se_synonym_clusters.md       # SE synonym pair clusters
│   │   ├── rc_question_types.md         # RC question type strategies
│   │   └── data_schema.json             # Scraped data JSON schema
│   └── assets/
│       └── report_template.html         # HTML report template
└── samples/
    └── sample_report.html       # Sample output report (anonymized)
```

## How It Works

### Phase 1: Scrape
Uses browser automation (CDP) to fetch all 24 Verbal questions from the mock result page.

### Phase 2: Analyze
For each question, generates 5 mandatory analysis sections with logic parsing, vocabulary annotations, and error classification.

### Phase 3: Report
Produces both a detailed Markdown report and a styled HTML report with:
- Score overview with ability bars
- Color-coded question cards (red = wrong, green = correct)
- Collapsible analysis sections
- Vocabulary master grid with memory tips
- Methodology summary cards

### Phase 4: Deliver
Sends the generated files to the user.

## Sample Output

See [`samples/sample_report.html`](samples/sample_report.html) for a full example report.

## Question Types Supported

| Type | Description | Format |
|------|-------------|--------|
| **SC** | Single blank, choose 1 of 5 | Sentence with one blank |
| **SE** | Sentence Equivalence, choose 2 synonyms from 6 | Sentence with one blank |
| **TC2** | Text Completion, 2 blanks | Multiple option groups |
| **TC3** | Text Completion, 3 blanks | Multiple option groups |
| **RC** | Reading Comprehension | Various sub-types |

## Error Classification

The system categorizes mistakes into types:
- Vocabulary gap (unknown key words)
- Wrong logic direction (positive/negative reversed)
- Look-alike confusion (e.g., comely vs comic)
- Unsupported inference (RC detail questions)
- Speed error (too fast, likely guessing)
- Blank answer (free points lost)
- Non-synonymous SE pair (chose two words that aren't synonyms)
- Reversed "unlike" logic

## Tech Stack

- **AI Agent**: WorkBuddy Skill (LLM-driven workflow)
- **Browser Automation**: CDP (Chrome DevTools Protocol) via agent-browser
- **Output**: Static HTML + Markdown (no server required)
- **Styling**: Pure CSS with CSS variables, responsive design

## Setup as WorkBuddy Skill

1. Copy `skill/` directory to `~/.workbuddy/skills/gre-verbal-analysis/`
2. Trigger by saying "帮我分析GRE Verbal模考" or providing a gre.viplgw.cn URL
3. Ensure Chrome/Edge is running with `--remote-debugging-port=9222`

## License

MIT
