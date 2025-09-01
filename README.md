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
