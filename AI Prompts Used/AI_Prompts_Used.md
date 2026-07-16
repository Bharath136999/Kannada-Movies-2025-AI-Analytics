# AI Prompts Used — Kannada Movies 2025 Project

This file documents every prompt used to build this project with Claude (Anthropic), in sequence, so the workflow is reproducible and the AI's role is transparently recorded. Recommended location in the repo: `docs/AI_PROMPTS_USED.md` or the repo root as `PROMPTS.md`.

---

## 1. Data Cleaning
**Prompt:**
> "Clean this movie dataset. Remove duplicates, standardize month names, replace blank values with N/A, convert all numeric columns to numbers, check spelling mistakes, and ensure consistency."

**Input:** `Kannada_Movies_2025_AI_Project_Template.xlsx` (raw)
**Output:** `Kannada_Movies_2025_Cleaned.xlsx`
**What it did:** Deduplication check, month/date consistency check, blank-vs-N/A audit, text-to-numeric conversion for currency/rating columns, ROI% standardization, and fact-checked spelling corrections (director/title names) against public sources.

---

## 2. SQL Analysis
**Prompt:**
> "Generate 25 interview-level SQL queries for this Kannada movie dataset."

**Output:** `Kannada_Movies_2025_SQL_Interview_Questions.md`
**What it did:** Produced a `CREATE TABLE` schema plus 25 graduated SQL questions (filtering, aggregation, GROUP BY/HAVING, subqueries, window functions, CTEs, self-joins, NULL handling) with worked solutions and a topic-coverage map.

---

## 3. Power BI Dashboard Design (3-page)
**Prompt:**
> "Design a professional three-page Power BI dashboard for this dataset."

**Output:** `Kannada_Movies_2025_PowerBI_Dashboard_Spec.md` + inline visual mockups
**What it did:** Designed a 3-page report (Executive Overview, Financial Performance, Ratings & Talent) with DAX measures, slicer behavior, visual placement, and a theme/color system.

---

## 4. Power BI Dashboard Consolidation (single-page)
**Prompt:**
> "make all these 3 pages into one dashboard report of power bi"

**Output:** `Kannada_Movies_2025_PowerBI_SinglePage_Spec.md` + inline visual mockup
**What it did:** Merged the 3-page report into one dense single-page layout, with exact x/y/width/height coordinates for every visual and KPI card, and clarified the Power BI "Report vs. Dashboard" distinction.

---

## 5. Business Insights
**Prompt:**
> "Act as a Senior Business Analyst. Analyze this dataset and provide 20 business insights with supporting evidence."

**Output:** delivered inline in chat (20 grouped insights: revenue concentration, financial disclosure, critical reception vs. commercial performance, seasonality, talent/production patterns, portfolio-level takeaways), each backed by specific figures from the cleaned dataset.

---

## 6. Full Project Report (PDF)
**Prompt:**
> "Write a professional report with the following sections: Executive Summary, Project Objective, Data Cleaning, SQL Analysis, Power BI Dashboard, Key Findings, Business Insights, Recommendations, Conclusion. Create me a single pdf with 4 pages of power bi report and complete this data set."

**Output:** `Kannada_Movies_2025_Full_Report.pdf` (17 pages)
**What it did:** Assembled all prior work into one PDF — full written report with all 9 requested sections, an Appendix A with 4 rendered Power BI dashboard pages (Executive Overview, Financial Performance, Ratings & Talent, Consolidated Summary), and an Appendix B with the complete 15-row cleaned dataset on a landscape page.

---

## 7. PBIX Clarification
**Prompt:**
> "build 4 pages of report which is pbix file and also create a pdf for this project done."
> "Provide me one pbix file to download."

**Output:** clarification only — no file produced.
**What it did:** Explained that `.pbix` is a proprietary binary format only authored inside Power BI Desktop, and cannot be generated programmatically in this environment. Pointed back to the layout spec (§3/§4) and rendered PNG mockups as the equivalent deliverable, and offered a manual build walkthrough.

---

## 8. PNG Exports of Dashboard Pages
**Prompt:**
> "Provide me 3 png images for the power bi dashboard created above"

**Output:**
- `PowerBI_Page1_Executive_Overview.png`
- `PowerBI_Page2_Financial_Performance.png`
- `PowerBI_Page3_Ratings_Talent.png`

**What it did:** Exported the 3 original dashboard-report pages as standalone high-resolution PNGs for direct use in slide decks or documentation.

---

## Suggested repo structure

```
kannada-movies-2025-analysis/
├── README.md
├── data/
│   ├── Kannada_Movies_2025_AI_Project_Template.xlsx   (raw)
│   └── Kannada_Movies_2025_Cleaned.xlsx               (cleaned)
├── sql/
│   └── Kannada_Movies_2025_SQL_Interview_Questions.md
├── powerbi/
│   ├── Kannada_Movies_2025_PowerBI_Dashboard_Spec.md
│   ├── Kannada_Movies_2025_PowerBI_SinglePage_Spec.md
│   ├── PowerBI_Page1_Executive_Overview.png
│   ├── PowerBI_Page2_Financial_Performance.png
│   └── PowerBI_Page3_Ratings_Talent.png
├── report/
│   └── Kannada_Movies_2025_Full_Report.pdf
└── docs/
    └── AI_PROMPTS_USED.md   (this file)
```

## Suggested README snippet
```markdown
## AI-Assisted Workflow
This project was built iteratively with Claude (Anthropic) — see
[docs/AI_PROMPTS_USED.md](docs/AI_PROMPTS_USED.md) for the exact
sequence of prompts used to produce each deliverable, from data
cleaning through to the final PDF report.
```
