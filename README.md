# 🗂️ Tech Layoffs — End-to-End SQL Data Cleaning Pipeline

**Source:** [Layoffs.fyi](https://www.kaggle.com) (2020–2025)  
**Purpose:** Create reliable, analysis-ready data by staging, deduping, standardizing, typing, and pruning unusable records.

---

## 📌 Overview
This project builds a **reproducible SQL pipeline** that:
- ✅ Preserves raw data with **staging tables** (audit-friendly).
- ✅ Detects & removes **true duplicates** via composite keys + window functions.
- ✅ Standardizes **industry, country, and date** fields.
- ✅ Retains meaningful **NULLs**, pruning rows that are analytically unusable.
- ✅ Produces a **clean, analysis-ready dataset** for downstream dashboards and EDA.

---

## 📊 Architecture (Data Flow)

```mermaid
flowchart TD
    A[Raw CSV (layoffs.csv)] -->|Load| B[(world_layoffs.layoffs)]
    B -->|Copy| C[(layoffs_staging)]
    C -->|ROW_NUMBER (composite key)| D[(layoffs_staging2)]
    D -->|Standardize + Normalize| E[(Cleaned Table)]
    E -->|Analysis| F[Dashboards / EDA]

⚡ Pipeline Steps
🔹 0) Inspect Raw Data
SELECT * FROM world_layoffs.layoffs;
🔹 1) Staging

Preserve the raw table, work in a copy.

CREATE TABLE world_layoffs.layoffs_staging LIKE world_layoffs.layoffs;
INSERT INTO world_layoffs.layoffs_staging
SELECT * FROM world_layoffs.layoffs;

🔹 2) Duplicate Handling

Detect duplicates using a wide composite key and ROW_NUMBER().

WITH dups AS (
    SELECT company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions,
           ROW_NUMBER() OVER (
               PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions
               ORDER BY company
           ) AS rn
    FROM world_layoffs.layoffs_staging
)
SELECT * FROM dups WHERE rn > 1; -- review before deletion


Create a safe staging table and deduplicate:

CREATE TABLE world_layoffs.layoffs_staging2 (..., row_num INT);

INSERT INTO world_layoffs.layoffs_staging2
SELECT ..., ROW_NUMBER() OVER (PARTITION BY ... ORDER BY company) AS row_num
FROM world_layoffs.layoffs_staging;

DELETE FROM world_layoffs.layoffs_staging2 WHERE row_num >= 2;

🔹 3) Standardization & Typing

Convert empty strings → NULL

Backfill missing industries within same company

Harmonize industry vocabulary (Crypto)

Trim trailing punctuation from country names

Normalize dates into DATE

-- Normalize industry empty → NULL
UPDATE world_layoffs.layoffs_staging2 SET industry = NULL WHERE industry = '';

-- Backfill missing industry values
UPDATE t1
JOIN world_layoffs.layoffs_staging2 t2 ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL AND t2.industry IS NOT NULL;

-- Standardize industry name
UPDATE world_layoffs.layoffs_staging2
SET industry = 'Crypto'
WHERE industry IN ('Crypto Currency','CryptoCurrency');

-- Clean country field
UPDATE world_layoffs.layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);

-- Normalize dates
UPDATE world_layoffs.layoffs_staging2
SET date = STR_TO_DATE(date, '%m/%d/%Y');
ALTER TABLE world_layoffs.layoffs_staging2 MODIFY COLUMN date DATE;

🔹 4) Prune Unusable Records

Remove rows with no layoff count and no percentage.

DELETE FROM world_layoffs.layoffs_staging2
WHERE total_laid_off IS NULL
  AND percentage_laid_off IS NULL;

🔹 5) Final Verification
SELECT * FROM world_layoffs.layoffs_staging2;

📈 Example EDA Queries
-- Layoffs by year & industry
SELECT YEAR(date) AS yr, industry, SUM(total_laid_off) AS total
FROM world_layoffs.layoffs_staging2
GROUP BY 1,2
ORDER BY 1,3 DESC;

-- Top 10 countries
SELECT country, SUM(total_laid_off) AS total
FROM world_layoffs.layoffs_staging2
GROUP BY country
ORDER BY total DESC
LIMIT 10;

📂 Project Structure
sql/cleaning.sql   → full SQL cleaning pipeline
data/layoffs.csv   → raw dataset (not included)
notebooks/         → optional Jupyter/EDA
dashboards/        → BI/PowerBI/Streamlit dashboards

📝 Notes

Uses ROW_NUMBER() for safe deduplication across a composite key:
(company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions)

Retains NULL values to avoid misleading imputations.

Uses MySQL functions: ROW_NUMBER, TRIM, STR_TO_DATE. Adapt for other SQL dialects as needed.

📜 License

Dataset © original authors (Kaggle).
This pipeline is MIT-licensed — use freely with attribution.


---

This README is **parallel** to your script — so readers can open `README.md` for explanation and `sql/cleaning.sql` for the exact runnable pipeline.  

👉 Do you want me to also **embed your full `cleaning.sql` script inside the README** (so it’s all in one file), or keep it separate like this?

You said:
in a single markdown cell
ChatGPT said:

Perfect ✅ — here’s the entire README in one single Markdown block so you can copy-paste directly into README.md:

# 🗂️ Tech Layoffs — End-to-End SQL Data Cleaning Pipeline

**Source:** [Layoffs.fyi](https://www.kaggle.com) (2020–2025)  
**Purpose:** Create reliable, analysis-ready data by staging, deduping, standardizing, typing, and pruning unusable records.

---

## 📌 Overview
This project builds a **reproducible SQL pipeline** that:
- ✅ Preserves raw data with **staging tables** (audit-friendly).
- ✅ Detects & removes **true duplicates** via composite keys + window functions.
- ✅ Standardizes **industry, country, and date** fields.
- ✅ Retains meaningful **NULLs**, pruning rows that are analytically unusable.
- ✅ Produces a **clean, analysis-ready dataset** for downstream dashboards and EDA.

---

## 📊 Architecture (Data Flow)

```mermaid
flowchart TD
    A[Raw CSV (layoffs.csv)] -->|Load| B[(world_layoffs.layoffs)]
    B -->|Copy| C[(layoffs_staging)]
    C -->|ROW_NUMBER (composite key)| D[(layoffs_staging2)]
    D -->|Standardize + Normalize| E[(Cleaned Table)]
    E -->|Analysis| F[Dashboards / EDA]

⚡ Pipeline Steps
🔹 0) Inspect Raw Data
SELECT * FROM world_layoffs.layoffs;

🔹 1) Staging

Preserve the raw table, work in a copy.

CREATE TABLE world_layoffs.layoffs_staging LIKE world_layoffs.layoffs;
INSERT INTO world_layoffs.layoffs_staging
SELECT * FROM world_layoffs.layoffs;

🔹 2) Duplicate Handling

Detect duplicates using a wide composite key and ROW_NUMBER().

WITH dups AS (
    SELECT company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions,
           ROW_NUMBER() OVER (
               PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions
               ORDER BY company
           ) AS rn
    FROM world_layoffs.layoffs_staging
)
SELECT * FROM dups WHERE rn > 1; -- review before deletion


Create a safe staging table and deduplicate:

CREATE TABLE world_layoffs.layoffs_staging2 (
    company TEXT,
    location TEXT,
    industry TEXT,
    total_laid_off INT,
    percentage_laid_off TEXT,
    date TEXT,
    stage TEXT,
    country TEXT,
    funds_raised_millions INT,
    row_num INT
);

INSERT INTO world_layoffs.layoffs_staging2
( company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions, row_num )
SELECT company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions,
       ROW_NUMBER() OVER (
           PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions
           ORDER BY company
       ) AS row_num
FROM world_layoffs.layoffs_staging;

DELETE FROM world_layoffs.layoffs_staging2 WHERE row_num >= 2;

🔹 3) Standardization & Typing

Convert empty strings → NULL

Backfill missing industries within same company

Harmonize industry vocabulary (Crypto)

Trim trailing punctuation from country names

Normalize dates into DATE

-- Normalize industry empty → NULL
UPDATE world_layoffs.layoffs_staging2 SET industry = NULL WHERE industry = '';

-- Backfill missing industry values
UPDATE world_layoffs.layoffs_staging2 t1
JOIN world_layoffs.layoffs_staging2 t2 ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL AND t2.industry IS NOT NULL;

-- Standardize industry name
UPDATE world_layoffs.layoffs_staging2
SET industry = 'Crypto'
WHERE industry IN ('Crypto Currency','CryptoCurrency');

-- Clean country field
UPDATE world_layoffs.layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);

-- Normalize dates
UPDATE world_layoffs.layoffs_staging2
SET date = STR_TO_DATE(date, '%m/%d/%Y');
ALTER TABLE world_layoffs.layoffs_staging2 MODIFY COLUMN date DATE;

🔹 4) Prune Unusable Records

Remove rows with no layoff count and no percentage.

DELETE FROM world_layoffs.layoffs_staging2
WHERE total_laid_off IS NULL
  AND percentage_laid_off IS NULL;

🔹 5) Final Verification
SELECT * FROM world_layoffs.layoffs_staging2;

📈 Example EDA Queries
-- Layoffs by year & industry
SELECT YEAR(date) AS yr, industry, SUM(total_laid_off) AS total
FROM world_layoffs.layoffs_staging2
GROUP BY 1,2
ORDER BY 1,3 DESC;

-- Top 10 countries
SELECT country, SUM(total_laid_off) AS total
FROM world_layoffs.layoffs_staging2
GROUP BY country
ORDER BY total DESC
LIMIT 10;

📂 Project Structure
sql/cleaning.sql   → full SQL cleaning pipeline
data/layoffs.csv   → raw dataset (not included)
notebooks/         → optional Jupyter/EDA
dashboards/        → BI/PowerBI/Streamlit dashboards

📝 Notes

Uses ROW_NUMBER() for safe deduplication across a composite key:
(company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions)

Retains NULL values to avoid misleading imputations.

Uses MySQL functions: ROW_NUMBER, TRIM, STR_TO_DATE. Adapt for other SQL dialects as needed.

