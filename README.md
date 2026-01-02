# WRIR | World Research Integrity Ranking

This repository contains the source code and documentation for the **World Research Integrity Ranking (WRIR)** analysis pipeline. This Jupyter Notebook serves as the computational supplementary material for the study *"Dissecting AI-related Paper Retraction Across Countries and Institutions"*.

The pipeline ingests retraction data, enriches it with institutional metadata via the OpenAlex API, and performs a fractional attribution analysis to quantify the distribution of research misconduct across countries and institutions.

## Prerequisites

The notebook is designed to run in **Google Colab** or a local Jupyter environment.

### Dependencies
The following Python packages are required:
* `pandas` (Data manipulation)
* `requests` (API querying)
* `tqdm` (Progress bars)
* `itables` (Interactive tables)
* `folium` (Geospatial visualization)
* `geopandas` (Geospatial data handling)
* `pycountry` (ISO country code normalization)
* `shapely` (Geometry operations)
* `plotly` (Interactive charting)

```bash
pip install pandas requests tqdm itables folium geopandas pycountry shapely plotly
```

## 📂 Input Data

The pipeline requires the **Retraction Watch Database** dataset.

1. Obtain the dataset (CSV format) from Crossref or the official Retraction Watch source, download URL: https://gitlab.com/crossref/retraction-watch-data.
2. Rename the file to `retraction_watch.csv` (or update the `RW_PATH` variable in the notebook).
3. Upload the CSV to the runtime environment.

## 🚀 Pipeline Workflow

The notebook executes the following analytical stages:

### 1. Data Ingestion & Cleaning
* **Loads** the raw Retraction Watch CSV.
* **Normalizes** Digital Object Identifiers (DOIs) to ensure consistency.
* **Parses** dates (`OriginalPaperDate`, `RetractionDate`) and computes the **Time-to-Retraction (TTR)**.
* **Deduplicates** records to ensure one unique entry per DOI (prioritizing the earliest retraction event).

### 2. Metadata Enrichment (OpenAlex)
* Queries the **OpenAlex API** (https://api.openalex.org/works/) for every unique DOI.
* Retrieves disambiguated **author affiliations** (Institutions and Country Codes).
* *Note: This step includes a rate limiter (`SLEEP_SEC`) to respect API etiquette.*

### 3. Fractional Attribution Modeling
* Implements a **Fractional Counting** model to handle multi-author papers.
* **Formula**: If a paper has $N$ distinct countries, each country receives a weight of $1/N$. This prevents inflation of scores for large international collaborations.
* Generates two attribution tables:
  * `doi_country`: Attribution weights by country.
  * `doi_institution`: Attribution weights by institution.

### 4. Metric Calculation
Calculates the following key indicators for every country and institution:
* **Retraction Score ($S$)**: Sum of fractional weights.
* **Median TTR**: Median days from publication to retraction.
* **Reason Entropy ($H$)**: Shannon entropy of retraction reasons (measuring the diversity of misconduct types).
* **Top Reason**: The most frequent retraction cause (e.g., "Computer-Aided Content", "Image Manipulation").

### 5. AI-Specific Analysis (Post-ChatGPT)
* Filters retractions occurring on or after **November 1, 2022**.
* Isolates papers with the specific retraction reason: *"Computer-Aided Content or Computer-Generated Content"*.
* Re-calculates rankings to identify hotspots of AI-generated fraud.

### 6. Reporting & Visualization
* Generates interactive tables using `itables`.
* Produces static HTML reports:
  * `country_ranking.html`: Full leaderboard of countries.
  * `institution_ranking.html`: Full leaderboard of institutions.
  * `WRIR_Annual_Report.html`: A comprehensive dashboard including TTR distributions and morphology clusters.

## 📊 Output Files

The execution of the notebook yields the following artifacts:

| File Name | Description |
| :--- | :--- |
| `country_ranking.html` | Interactive table of country rankings with Score, TTR, and Entropy. |
| `institution_ranking.html` | Interactive table of institutional rankings. |
| `WRIR_Annual_Report.html` | The final aggregated HTML report cited in the paper. |
| `rw_country_heatmap.html` | Geospatial visualization of retraction intensity. |

## 📜 Citation

If you use this code or data in your research, please cite the accompanying paper:

> Khalid Saqr, "Impact of generative AI on research integrity: Retraction behavior across countries and institutions." (2026).
