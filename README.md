# data-cleaning-and-EDA-with-MY-SQL

Tech Layoffs Data Cleaning (SQL)
This project cleans and standardizes the Layoffs dataset (2020–2025) sourced from Layoffs.fyi via Kaggle, building an analysis‑ready table using staging, deduplication, normalization, and pruning steps.

Dataset
Source: Kaggle — Layoffs (11 Mar 2020 to 21 Apr 2025).

File: layoffs.csv (11 columns).

Attribution: Roger Lee / Layoffs.fyi.

Goals
Preserve raw data with staging tables for safe transformations.

Identify and remove true duplicates via window functions and composite keys.

Standardize categorical fields (industry, country) and normalize dates.

Keep analytically useful NULLs; drop rows with insufficient information only.

SQL Highlights
Staging tables: world_layoffs.layoffs_staging, world_layoffs.layoffs_staging2 for controlled edits.

Deduplication using ROW_NUMBER() over a wide partition key to avoid false matches.

String standardization (TRIM, controlled vocabularies) and date typing with STR_TO_DATE then ALTER to DATE.

Groupwise completion of missing industry values via self‑join on company.

How to run
Load layoffs.csv into world_layoffs.layoffs (MySQL or compatible).

Execute the SQL in order; requires CREATE/INSERT/UPDATE/DELETE/ALTER privileges.

Validate outputs using the included SELECT checks at each stage.

Output
Clean table: world_layoffs.layoffs_staging2 (deduped, standardized, typed, and pruned).

Notes
Keeps totals/funding NULLs to avoid biased imputations during EDA; only drops rows with neither total nor percentage available.

Uses ANSI‑style window functions and MySQL string/date functions for portability.

License
Dataset: Open Database; contents © original authors (see Kaggle page).

