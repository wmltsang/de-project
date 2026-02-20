# DE Project Outline

# data
[Zillow Housing Research Data portal showing real estate market analytics and datasets](https://www.zillow.com/research/data/)

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![DuckDB](https://img.shields.io/badge/DuckDB-Latest-yellow.svg)](https://duckdb.org/)
[![dbt](https://img.shields.io/badge/dbt-1.0+-orange.svg)](https://www.getdbt.com/)

An end-to-end data pipeline that processes 6M+ healthcare provider records from the NPPES public dataset. Built to learn and demonstrate modern data engineering practices with DuckDB, dbt, and Polars.

## Table of Contents
- [Overview](#overview)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Data Models](#data-models)
- [Sample Queries](#sample-queries)
- [What I Learned](#what-i-learned)

---

## Overview

**Why This Project?**

When learning data engineering, I wanted to work with real, messy data instead of clean tutorial datasets. The NPPES (National Plan and Provider Enumeration System) is a publicly available dataset with 6M+ records of US healthcare providers - it's large enough to require thoughtful data modeling, has real data quality issues, and answers actual business questions.

I built this project to practice the full ELT workflow: extracting raw CSV files, loading them to S3, transforming with dbt, and serving analytics-ready tables. Along the way I learned a lot about data cleaning, dimensional modeling, and modern data tools.

**Questions This Data Answers:**
- How many healthcare providers operate in each state?
- What are the most common medical specialties by geography?
- What's the split between individual providers vs. organizations?
- Which counties have the highest provider density?

---

## Key Features

- **ELT Pipeline**: Full Extract-Load-Transform workflow from raw CSVs to analytics tables
- **Modern Data Stack**: DuckDB (OLAP database) + dbt (transformations) + Polars (data cleaning)
- **Cloud Storage**: Automated S3 uploads with boto3
- **Data Modeling**: Staging and mart layers following dimensional modeling principles
- **Data Quality**: Cleaned 500K+ rows with missing/invalid data
- **Columnar Storage**: Using Parquet format for 5x faster loads vs CSV

---

## Tech Stack

**Core:**
- **Python 3.9+** - Main language for scripting and orchestration
- **DuckDB** - Embedded OLAP database (surprisingly fast for analytical queries on my laptop)
- **dbt** - SQL-based transformation framework with built-in testing
- **Polars** - Data cleaning library (chose this over Pandas for better performance)
- **AWS S3 + boto3** - Cloud storage for raw and processed data

**Supporting:**
- Matplotlib for basic visualizations
- PyArrow for Parquet file handling
- python-dotenv for environment variables

---

## Architecture

```
┌─────────────────┐
│   Raw Data      │
│  (NPPES CSVs)   │
└────────┬────────┘
         │
         │ Extract & Load
         ▼
┌─────────────────┐
│   AWS S3        │
│  Data Lake      │
└────────┬────────┘
         │
         │ Download
         ▼
┌─────────────────┐      ┌──────────────────┐
│  DuckDB Local   │◄─────┤  Polars Engine   │
│  Database       │      │  (Data Cleaning) │
└────────┬────────┘      └──────────────────┘
         │
         │ dbt Transform
         ▼
┌─────────────────────────────────────┐
│        Analytics Layer              │
│  ┌─────────────────────────────┐   │
│  │   Staging Models            │   │
│  │  - stg_nppes_providers      │   │
│  │  - stg_taxonomy             │   │
│  │  - stg_fips (geography)     │   │
│  └─────────────────────────────┘   │
│                                     │
│  ┌─────────────────────────────┐   │
│  │   Mart Models               │   │
│  │  - mart_provider_summary    │   │
│  │  - mart_providers_by_state  │   │
│  │  - mart_providers_by_county │   │
│  │  - mart_provider_directory  │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
         │
         ▼
┌─────────────────┐
│  Visualizations │
│  & Analytics    │
└─────────────────┘
```

### Data Flow
1. **Extract**: Download raw NPPES data from CMS public data sources
2. **Load**: Upload raw data to AWS S3 for cloud storage
3. **Transform**: Use dbt to create cleaned staging tables and analytical marts
4. **Analyze**: Query DuckDB for insights and generate visualizations

---

## Quick Start

### Prerequisites
- Python 3.9+
- AWS account (optional, only needed for S3 features)
- ~2GB free disk space

### Setup

1. Clone the repo
```bash
git clone https://github.com/YOUR_USERNAME/nppes-healthcare-analytics.git
cd nppes-healthcare-analytics
```

2. Install dependencies
```bash
pip install -r requirements.txt
```

3. Set up environment variables (for S3 features)
```bash
cp .env.example .env
# Edit .env with your AWS credentials
```

4. Run data cleaning
```bash
python scripts/clean_nppes.py
```

5. Run dbt transformations
```bash
cd nnpes
dbt run
dbt test
```

6. Query the data
```bash
duckdb nnpes.duckdb
# Or use query_examples.sql in your SQL client
```

---

## Project Structure

```
.
├── data/
│   ├── raw/              # Raw CSV files from NPPES
│   └── cleaned/          # Cleaned Parquet files
├── nnpes/                # dbt project
│   ├── models/
│   │   ├── staging/      # 1:1 source transformations
│   │   └── marts/        # Business-ready analytics tables
│   ├── dbt_project.yml   # dbt configuration
│   └── dev.duckdb        # DuckDB database file
├── scripts/
│   ├── clean_nppes.py           # Data cleaning with Polars
│   ├── upload_to_s3.py          # S3 upload automation
│   ├── visualize_data.py        # Chart generation
│   └── explore_raw_data.py      # Data exploration
├── src/
│   ├── s3_services.py           # AWS S3 utilities
│   └── my_logger.py             # Custom logging
├── visualizations/              # Generated charts
├── query_examples.sql           # Sample SQL queries
├── requirements.txt             # Python dependencies
└── README.md                    # This file
```

---

## Data Models

### Staging Layer (1:1 with sources)
- **stg_nppes_providers**: Cleaned provider data with standardized column names
- **stg_taxonomy**: Healthcare specialty classification codes
- **stg_fips**: Geographic reference data (state/county codes)

### Mart Layer (Business Logic)
- **mart_provider_summary**: Overall statistics (total providers, entity types, geographic coverage)
- **mart_providers_by_state**: Provider counts and metrics aggregated by state
- **mart_providers_by_county**: County-level provider density analysis
- **mart_provider_directory**: Searchable provider directory with specialty information
- **mart_specialty_distribution**: Analysis of medical specialties across regions

---

## Sample Queries

### Top 10 States by Provider Count
```sql
SELECT
    state,
    total_providers,
    individual_providers,
    organization_providers
FROM mart_providers_by_state
ORDER BY total_providers DESC
LIMIT 10;
```

### Provider Directory Search (California)
```sql
SELECT
    provider_name,
    provider_type,
    specialty_name,
    city,
    zip_code
FROM mart_provider_directory
WHERE state = 'CA'
  AND specialty_classification = 'Physician'
LIMIT 20;
```

### Overall Dataset Summary
```sql
SELECT * FROM mart_provider_summary;
```

**Sample Output:**
| total_providers | total_states | total_specialties | pct_individuals |
|----------------|--------------|-------------------|-----------------|
| 6,234,567      | 54           | 842               | 78.3%           |

More examples available in [query_examples.sql](query_examples.sql).

---

## Visualizations

Some basic charts I generated from the census data:

### Geographic Distribution
![State Distribution](visualizations/state_distribution.png)

*Top 10 states by county count showing Texas, Georgia, and Virginia leading in geographic coverage.*

### County Analysis
![Counties by State](visualizations/counties_by_state.png)

*Bar chart showing the distribution of counties across the top 15 states.*

### Metropolitan Coverage
![CBSA Coverage](visualizations/cbsa_coverage.png)

*Analysis of counties within Core-Based Statistical Areas (metropolitan regions).*

---

## What I Learned

### The Good Stuff

**DuckDB is seriously fast** - I was skeptical about an embedded database handling 6M rows, but DuckDB proved me wrong. Analytical queries that would take minutes in SQLite run in seconds. The `read_csv()` function in SQL is also really convenient.

**Polars > Pandas for ETL** - After testing both, Polars was 5-10x faster for data cleaning operations. The syntax took some getting used to (`.with_columns()` instead of `.assign()`), but the performance gains were worth it.

**dbt's testing saved me** - The schema tests caught several issues I would've missed: duplicate NPIs, NULL states in rows I thought were clean, and mismatched taxonomy codes. The `dbt test` command became part of my workflow.

**Parquet is magic** - Converting from CSV to Parquet reduced file sizes by ~80% and made DuckDB loads way faster. Should've done this from the start.

### The Challenges

**Data quality was worse than expected** - The NPPES dataset had ~500K rows missing state codes, weird ZIP formats (some 5-digit, some 9-digit with dashes), and inconsistent naming. Spent a lot of time on data cleaning before I could even start the fun transformations.

**dbt model structure took iterations** - My first attempt had everything in one huge SQL file. After learning about staging vs mart layers, I refactored into a proper layered structure. Breaking things is the best way to learn.

**S3 permissions confusion** - Got stuck for a bit figuring out boto3 credentials and IAM roles. Eventually got it working with environment variables, but AWS IAM is still not my favorite thing.

### Key Takeaways

- Always explore your data first (I now run `SUMMARIZE` and `SELECT * LIMIT 100` before any transformations)
- Version control your SQL transformations - dbt's version control saved me multiple times when I broke a model
- Parquet >> CSV for analytical workloads
- Test your data transformations (dbt tests caught bugs I definitely would've missed)

---

## Known Limitations

Being honest about what this project doesn't do (yet):

- **No automated data refresh** - Currently requires manual download of NPPES data from CMS
- **Local processing only** - Everything runs on my laptop; would need optimization for cloud deployment
- **Basic visualizations** - Using matplotlib for static charts; an interactive dashboard would be better
- **Limited data validation** - Basic dbt tests exist, but could be more comprehensive with Great Expectations
- **No API layer** - Data is query-only through DuckDB; building a REST API would make it more accessible

---

## Future Improvements

If I continue working on this project, here's what I'd add:

1. **Streamlit dashboard** - Interactive visualizations would be way better than static PNG charts
2. **Incremental dbt models** - Currently re-processing all 6M rows each time; incremental models would be more efficient
3. **Geographic mapping** - Plotly or Folium maps showing provider density by county
4. **CI/CD with GitHub Actions** - Automated testing when I push changes
5. **Docker container** - Make it easier for others to run locally

---

## Data Sources

All data is publicly available:
- **NPPES Data**: [CMS National Provider Identifier Registry](https://npiregistry.cms.hhs.gov/)
- **Taxonomy Codes**: [NUCC Healthcare Provider Taxonomy](https://www.nucc.org/)
- **FIPS Codes**: [US Census Bureau](https://www.census.gov/)

---

## License

MIT License - see [LICENSE](LICENSE) file for details.

---

## Connect

**Jing You**
📧 jingliuyou@gmail.com

Questions or feedback? Feel free to open an issue or reach out!

---

*Portfolio project built while learning modern data engineering*
