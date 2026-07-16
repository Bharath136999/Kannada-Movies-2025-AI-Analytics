# SQL Interview Practice — Kannada Movies 2025 Dataset

## Table Schema

```sql
CREATE TABLE kannada_movies_2025 (
    movie_name          VARCHAR(100),
    release_date        DATE,
    month                VARCHAR(20),
    director             VARCHAR(100),
    lead_actor           VARCHAR(150),
    production_house     VARCHAR(200),
    budget_cr            DECIMAL(10,2),   -- NULL if unknown (was 'N/A')
    india_gross_cr       DECIMAL(10,2),
    overseas_gross_cr    DECIMAL(10,2),
    worldwide_gross_cr   DECIMAL(10,2),
    profit_cr            DECIMAL(10,2),
    roi_percent          DECIMAL(10,2),
    imdb_rating          DECIMAL(3,1),
    verdict              VARCHAR(50)
);
```

> Note: In the cleaned Excel file, unknown numeric values are stored as the text "N/A". When loading into SQL, those become `NULL` — every query below assumes that convention, and several questions specifically test `NULL` handling.

---

### Q1. List all movies released after June 2025, sorted by release date.
```sql
SELECT movie_name, release_date, director
FROM kannada_movies_2025
WHERE release_date > '2025-06-30'
ORDER BY release_date;
```

### Q2. Find the average IMDb rating across all movies.
```sql
SELECT ROUND(AVG(imdb_rating), 2) AS avg_imdb_rating
FROM kannada_movies_2025;
```

### Q3. Count how many movies fall into each verdict category.
```sql
SELECT verdict, COUNT(*) AS movie_count
FROM kannada_movies_2025
GROUP BY verdict
ORDER BY movie_count DESC;
```

### Q4. Find the movie with the highest worldwide gross (handle NULLs).
```sql
SELECT movie_name, worldwide_gross_cr
FROM kannada_movies_2025
WHERE worldwide_gross_cr = (
    SELECT MAX(worldwide_gross_cr) FROM kannada_movies_2025
);
```

### Q5. List movies where the budget is known but the gross is not (data-quality check).
```sql
SELECT movie_name, budget_cr, worldwide_gross_cr
FROM kannada_movies_2025
WHERE budget_cr IS NOT NULL
  AND worldwide_gross_cr IS NULL;
```

### Q6. Rank movies by IMDb rating using a window function.
```sql
SELECT movie_name, imdb_rating,
       RANK() OVER (ORDER BY imdb_rating DESC) AS rating_rank
FROM kannada_movies_2025;
```

### Q7. Find the top 3 highest-grossing movies (worldwide), skipping NULLs.
```sql
SELECT movie_name, worldwide_gross_cr
FROM kannada_movies_2025
WHERE worldwide_gross_cr IS NOT NULL
ORDER BY worldwide_gross_cr DESC
LIMIT 3;
```

### Q8. For each verdict, find the average IMDb rating, only for verdicts with more than 1 movie.
```sql
SELECT verdict, COUNT(*) AS num_movies, ROUND(AVG(imdb_rating), 2) AS avg_rating
FROM kannada_movies_2025
GROUP BY verdict
HAVING COUNT(*) > 1
ORDER BY avg_rating DESC;
```

### Q9. List directors who directed more than one movie in 2025.
```sql
SELECT director, COUNT(*) AS movie_count
FROM kannada_movies_2025
GROUP BY director
HAVING COUNT(*) > 1;
```

### Q10. Find all movies released in the same month as "Su From So".
```sql
SELECT movie_name, release_date
FROM kannada_movies_2025
WHERE month = (
    SELECT month FROM kannada_movies_2025 WHERE movie_name = 'Su From So'
)
AND movie_name <> 'Su From So';
```

### Q11. Calculate total profit generated across all movies with known profit figures.
```sql
SELECT SUM(profit_cr) AS total_profit_cr
FROM kannada_movies_2025
WHERE profit_cr IS NOT NULL;
```

### Q12. Classify each movie into a budget tier using CASE.
```sql
SELECT movie_name, budget_cr,
       CASE
           WHEN budget_cr IS NULL THEN 'Unknown'
           WHEN budget_cr < 10 THEN 'Low Budget'
           WHEN budget_cr BETWEEN 10 AND 50 THEN 'Mid Budget'
           ELSE 'High Budget'
       END AS budget_tier
FROM kannada_movies_2025;
```

### Q13. Find the movie(s) with the best ROI % (excluding NULLs).
```sql
SELECT movie_name, roi_percent
FROM kannada_movies_2025
WHERE roi_percent = (
    SELECT MAX(roi_percent) FROM kannada_movies_2025 WHERE roi_percent IS NOT NULL
);
```

### Q14. List movies whose lead actor name contains more than one person (co-leads), using string functions.
```sql
SELECT movie_name, lead_actor
FROM kannada_movies_2025
WHERE lead_actor LIKE '%,%';
```

### Q15. Find the month with the most releases.
```sql
SELECT month, COUNT(*) AS releases
FROM kannada_movies_2025
GROUP BY month
ORDER BY releases DESC
LIMIT 1;
```

### Q16. For every movie, show how its IMDb rating compares to the overall average (difference).
```sql
SELECT movie_name, imdb_rating,
       ROUND(imdb_rating - (SELECT AVG(imdb_rating) FROM kannada_movies_2025), 2) AS diff_from_avg
FROM kannada_movies_2025
ORDER BY diff_from_avg DESC;
```

### Q17. Use a CTE to find movies that are "All-Time Blockbuster" or "Flop" only.
```sql
WITH extremes AS (
    SELECT movie_name, verdict, imdb_rating
    FROM kannada_movies_2025
    WHERE verdict IN ('All-Time Blockbuster', 'Flop')
)
SELECT * FROM extremes
ORDER BY verdict, imdb_rating DESC;
```

### Q18. Find the second-highest IMDb rating in the dataset (without LIMIT/OFFSET).
```sql
SELECT MAX(imdb_rating) AS second_highest_rating
FROM kannada_movies_2025
WHERE imdb_rating < (SELECT MAX(imdb_rating) FROM kannada_movies_2025);
```

### Q19. List each movie along with the previous movie's release date, ordered chronologically (LAG).
```sql
SELECT movie_name, release_date,
       LAG(release_date) OVER (ORDER BY release_date) AS previous_release_date
FROM kannada_movies_2025
ORDER BY release_date;
```

### Q20. Find production houses that produced a movie which turned out to be a "Flop".
```sql
SELECT DISTINCT production_house
FROM kannada_movies_2025
WHERE verdict = 'Flop';
```

### Q21. Calculate what percentage of movies have a known (non-NULL) budget.
```sql
SELECT
    ROUND(100.0 * SUM(CASE WHEN budget_cr IS NOT NULL THEN 1 ELSE 0 END) / COUNT(*), 2)
    AS pct_with_known_budget
FROM kannada_movies_2025;
```

### Q22. Self-join: find pairs of movies released on the exact same date.
```sql
SELECT a.movie_name AS movie_1, b.movie_name AS movie_2, a.release_date
FROM kannada_movies_2025 a
JOIN kannada_movies_2025 b
  ON a.release_date = b.release_date
 AND a.movie_name < b.movie_name;
```

### Q23. Find the running total of worldwide gross ordered by release date (window function).
```sql
SELECT movie_name, release_date, worldwide_gross_cr,
       SUM(worldwide_gross_cr) OVER (ORDER BY release_date
                                      ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total_gross
FROM kannada_movies_2025
WHERE worldwide_gross_cr IS NOT NULL;
```

### Q24. Identify movies that outperformed their budget (profit_cr > 0), sorted by ROI %.
```sql
SELECT movie_name, budget_cr, profit_cr, roi_percent
FROM kannada_movies_2025
WHERE profit_cr > 0
ORDER BY roi_percent DESC;
```

### Q25. Bucket movies into IMDb rating bands and count how many fall in each band (using a derived table).
```sql
SELECT rating_band, COUNT(*) AS movie_count
FROM (
    SELECT movie_name,
           CASE
               WHEN imdb_rating >= 8 THEN '8.0 - 10.0 (Excellent)'
               WHEN imdb_rating >= 6.5 THEN '6.5 - 7.9 (Good)'
               WHEN imdb_rating >= 5 THEN '5.0 - 6.4 (Average)'
               ELSE 'Below 5.0 (Poor)'
           END AS rating_band
    FROM kannada_movies_2025
) sub
GROUP BY rating_band
ORDER BY rating_band DESC;
```

---

## Topics Covered
| # | Concept |
|---|---|
| 1, 7 | Filtering + Sorting (`WHERE`, `ORDER BY`, `LIMIT`) |
| 2, 11, 21 | Aggregate functions (`AVG`, `SUM`, conditional counts) |
| 3, 9, 15, 20 | `GROUP BY` |
| 8 | `GROUP BY` + `HAVING` |
| 4, 10, 13, 16, 18 | Subqueries |
| 6, 19, 23 | Window functions (`RANK`, `LAG`, running `SUM`) |
| 12, 25 | `CASE` expressions / bucketing |
| 14 | String pattern matching (`LIKE`) |
| 17 | Common Table Expressions (CTE) |
| 22 | Self-join |
| 5, 21, 24 | `NULL` handling |
