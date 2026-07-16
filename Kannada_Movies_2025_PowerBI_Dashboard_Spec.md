# Power BI dashboard design spec — Kannada Movies 2025

Source table: `kannada_movies_2025` (from `Kannada_Movies_2025_Cleaned.xlsx`)
Canvas size: 1280 × 720 (16:9), 3 report pages, consistent theme and slicer behavior across all pages via a bookmark-free sync (Power BI's native cross-page slicer sync, since Verdict/Month apply everywhere).

---

## 1. Data model prep (Power Query / DAX)

- Import the cleaned sheet; set data types: `budget_cr`, `india_gross_cr`, `overseas_gross_cr`, `worldwide_gross_cr`, `profit_cr`, `roi_percent`, `imdb_rating` → Decimal Number. Blank "N/A" text values load as `null` automatically once typed as Decimal.
- Add a calculated **Date table** (`CALENDAR` or `CALENDARAUTO()`) marked as the official date table, related to `release_date`.
- Split `lead_actor` and `production_house` into a bridge table if you want accurate actor/studio-level counts (they're comma-separated multi-value fields) — use Power Query "Split by delimiter → rows" into `movie_cast` and `movie_studios` tables, each with a `movie_name` foreign key.

### Key DAX measures
```dax
Total Movies            = DISTINCTCOUNT(kannada_movies_2025[movie_name])
Total Worldwide Gross   = SUM(kannada_movies_2025[worldwide_gross_cr])
Total Budget (Known)    = CALCULATE(SUM(kannada_movies_2025[budget_cr]), NOT(ISBLANK(kannada_movies_2025[budget_cr])))
Total Profit (Known)    = SUM(kannada_movies_2025[profit_cr])
Avg IMDb Rating         = AVERAGE(kannada_movies_2025[imdb_rating])
Movies w/ Known Budget  = CALCULATE([Total Movies], NOT(ISBLANK(kannada_movies_2025[budget_cr])))
% Data Completeness     = DIVIDE([Movies w/ Known Budget], [Total Movies])
Best ROI Movie          = CALCULATE(MAX(kannada_movies_2025[roi_percent]))
Blockbuster Count       = CALCULATE([Total Movies], kannada_movies_2025[verdict] = "All-Time Blockbuster")
Rating-Gross Correlation = CALCULATE(
    COMBINEVALUES(",", MIN(kannada_movies_2025[imdb_rating]), MAX(kannada_movies_2025[worldwide_gross_cr]))
) -- placeholder; better computed as a scatter trendline in-visual
```

---

## 2. Global design system

| Element | Spec |
|---|---|
| Theme | Custom JSON theme — navy header (`#1A2332`), accent blue (`#2A78D6`), success green (`#008300`), danger red (`#E34948`), amber (`#EDA100`), neutral grays for chrome |
| Font | Segoe UI (Power BI default) — 9pt body, 11pt titles, 20–24pt KPI card values |
| Card style | White background, 4px corner radius, 1px light-gray border, no drop shadow |
| Header bar | 40px navy strip across every page: report title (left) + page-navigation buttons (right) built with bookmarks/page navigator |
| Slicer panel | Fixed 130–160px left rail, consistent position/size on every page (copy-paste formatting) |
| Number format | `₹#,##0.0" Cr"` for currency measures, `0.0%` for ROI, `0.0` for ratings |
| Tooltips | Custom report-page tooltips showing movie poster placeholder, director, verdict, and all 3 gross figures on hover |

---

## 3. Page 1 — Executive Overview

**Goal:** answer "how did the year go?" in under 10 seconds.

| Zone | Visual | Fields |
|---|---|---|
| Top strip | 5 KPI cards | Total Movies · Total Worldwide Gross · Total Budget (Known) · Total Profit (Known) · Avg IMDb Rating |
| Left rail | Slicers | Month (dropdown), Verdict (checklist), Director (search list) |
| Mid-left | Horizontal bar chart | Top 6–8 movies by `worldwide_gross_cr`, sorted descending |
| Mid-right | Donut chart | Count of movies by `verdict`, direct-labeled with % |
| Bottom | Line/area chart | Count of releases by `month` (calendar order, not alphabetical — sort by a hidden `month_number` column) |
| Bottom-right (optional) | Table | Top 3 movies by `profit_cr` with conditional-formatting data bars |

**Interactions:** clicking a verdict slice on the donut filters the bar chart and line chart; clicking a month on the line chart filters everything else (standard Power BI cross-filter, no custom bookmarks needed).

---

## 4. Page 2 — Financial Performance

**Goal:** which movies made money, and how efficiently.

| Zone | Visual | Fields |
|---|---|---|
| Top strip | 4 KPI cards | Best ROI % · Worst ROI % · Biggest Budget · Movies w/ Known ROI |
| Left rail | Slicers | "Budget known" toggle (bin measure), Production House search |
| Mid-left | Scatter/bubble chart | X = `budget_cr`, Y = `worldwide_gross_cr`, size = `roi_percent`, color = `verdict`. Add a diagonal "break-even" reference line where gross = budget. |
| Mid-right | Diverging bar chart | `profit_cr` by movie, green for positive / red for negative, sorted by value |
| Bottom | Matrix / table | Movie · Budget · India Gross · Overseas Gross · Worldwide Gross · Profit · ROI% — with data bars on Profit and ROI% columns |

**Interactions:** a **drill-through** page (hidden, accessible by right-click) showing full financial detail for a single selected movie — useful for a "movie deep-dive" without cluttering the main page.

---

## 5. Page 3 — Ratings & Talent

**Goal:** critical reception and who's behind the camera/in front of it.

| Zone | Visual | Fields |
|---|---|---|
| Top strip | 3 KPI cards | Highest Rated Movie · Lowest Rated Movie · Repeat Directors count |
| Left rail | Slicers | Min-rating slider, Lead Actor search |
| Mid-left | Horizontal bar chart | `imdb_rating` by movie, all 15, sorted descending |
| Mid-right | Scatter chart | X = `imdb_rating`, Y = `worldwide_gross_cr` — tests the "good reviews ≠ box office" story; add a trendline |
| Bottom | Table | Production House · Movie Count · Avg Rating (built from the `movie_studios` bridge table) |
| Bottom-right (optional) | Word cloud or bar | Most frequent lead actors across the 15 releases (from `movie_cast` bridge table) |

---

## 6. Cross-cutting features

- **Page navigator** buttons in the header on all 3 pages for one-click switching.
- **Bookmark**: a "Reset filters" button on each page (clear-all-slicers bookmark) in the top-right corner.
- **Data quality badge**: since ~9 of 15 movies have missing box-office figures, add a small info icon/tooltip on Pages 1–2 noting "Financial figures reflect publicly reported data; N/A where undisclosed" so viewers don't mistake missing data for zero.
- **Mobile layout**: build a stacked mobile view for Page 1 (KPI cards → donut → bar → line) since executives often check this on phones.
- **Refresh cadence**: schedule a weekly refresh if this dataset will be extended with 2026 releases — model already supports it via the Date table.
