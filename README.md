# GoodThought NGO: Donations & Impact Analysis (SQL + DuckDB)
 
![GoodThought NGO](ngo_project_image.jpg)
 
## Project Overview
 
GoodThought NGO runs education, healthcare and sustainable-development assignments across four regions. This project uses **SQL** to answer two questions that help the NGO decide where to focus fundraising and which projects to learn from:
 
1. Which five assignments attracted the most donation money, and which type of donor gave it?
2. Which assignment delivered the highest impact in each region?
The analysis runs **locally on my own PC with DuckDB**: no database server, Docker or pgAdmin needed. DuckDB reads the CSV files directly and runs standard SQL.
 
### Why This Matters
 
NGOs have limited fundraising time and programme budget. Knowing who funds the biggest projects, and whether funding follows impact, shows where to build donor relationships and which successful approaches are worth repeating in other regions.
 
---
 
## Dataset
 
The data comes from the DataCamp project *Impact Analysis of GoodThought NGO Initiatives* (originally a PostgreSQL database covering 2010 to 2023). I exported the three tables to CSV and loaded them into DuckDB.
 
| Table | Rows | What it holds |
|---|---|---|
| `assignments` | 5,000 | Project name, start and end dates, budget, region, impact score |
| `donors` | 5,000 | Donor name and donor type (Individual, Organization, Corporate) |
| `donations` | 5,000 | Each donation: donor, amount, date and the assignment it funded |
 
### Entity Relationship Diagram
 
![ERD](erd.png)
 
`donations` links `donors` and `assignments` (one donor or assignment can have many donations).
 
---
 
## Tools Used
 
- **SQL**: joins, aggregation, CTEs, window functions
- **DuckDB**: runs SQL directly on CSV files
- **Python / pandas**: loading results into DataFrames
- **Jupyter Notebook**
---
 
## Task 1: Top 5 Assignments by Total Donations
 
```sql
WITH donation_details AS (
    SELECT
        d.assignment_id,
        ROUND(SUM(d.amount), 2) AS total_sum_donations,
        dn.donor_type
    FROM donations d
    JOIN donors dn ON d.donor_id = dn.donor_id
    GROUP BY d.assignment_id, dn.donor_type
)
SELECT
    a.assignment_name,
    a.region,
    dd.total_sum_donations AS rounded_total_donation_amount,
    dd.donor_type
FROM assignments a
JOIN donation_details dd ON a.assignment_id = dd.assignment_id
ORDER BY dd.total_sum_donations DESC
LIMIT 5;
```
 
**Result**
 
| Assignment | Region | Total Donated | Donor Type |
|---|---|---|---|
| Assignment_3033 | East | 3,840.66 | Individual |
| Assignment_300 | West | 3,133.98 | Organization |
| Assignment_4114 | North | 2,778.57 | Organization |
| Assignment_1765 | West | 2,626.98 | Organization |
| Assignment_268 | East | 2,488.69 | Individual |
 
## Task 2: Highest-Impact Assignment in Each Region
 
`ROW_NUMBER()` with a tiebreaker guarantees exactly one assignment per region, even when impact scores tie.
 
```sql
WITH donation_counts AS (
    SELECT assignment_id, COUNT(donation_id) AS total_donations
    FROM donations
    GROUP BY assignment_id
),
ranked_assignments AS (
    SELECT
        a.assignment_name,
        a.region,
        a.impact_score,
        dc.total_donations,
        ROW_NUMBER() OVER (
            PARTITION BY a.region
            ORDER BY a.impact_score DESC, a.assignment_name ASC
        ) AS rank_in_region
    FROM assignments a
    JOIN donation_counts dc ON a.assignment_id = dc.assignment_id
)
SELECT assignment_name, region, impact_score, total_donations
FROM ranked_assignments
WHERE rank_in_region = 1
ORDER BY region;
```
 
**Result**
 
| Assignment | Region | Impact Score | Number of Donations |
|---|---|---|---|
| Assignment_316 | East | 10.00 | 2 |
| Assignment_2253 | North | 9.99 | 1 |
| Assignment_3547 | South | 10.00 | 1 |
| Assignment_2794 | West | 9.99 | 2 |
 
---
 
## Key Insights
 
- **Funding does not follow impact.** None of the five most-funded assignments appear among the highest-impact assignment in each region, and the top-impact assignments received only one or two donations each. Donors may not be choosing projects by results.
- **Organizations and individuals fund the biggest projects.** Three of the five top-funded assignments were funded by Organizations and two by Individuals; no Corporate donor appears in the top five. This is a small sample, so it points to where to look next, not a final conclusion.
- **Impact is spread across all regions.** Every region has an assignment scoring about 10, so high impact is not limited to one area.
### What This Means for GoodThought
 
- Share impact scores with donors so funding can follow results.
- Look at what the top-impact assignments did and test it in other regions.
- Build a relationship programme for the donor types giving the largest amounts.
### Data Quality Findings
 
- Some assignments have **negative budgets** (for example Assignment_1 at -32,322.03).
- Some assignments have an **end date before the start date** (for example Assignment_9).
Neither task uses these fields, so the results are not affected, but both would need correcting before the data goes to leadership.
 
---
 
## How to Run This Project
 
1. Clone the repository.
2. Install the packages: `pip install duckdb pandas`
3. Make sure `assignments.csv`, `donors.csv` and `donations.csv` are in the same folder as the notebook.
4. Open `goodthought_duckdb.ipynb` in Jupyter and choose **Run All**.
The notebook checks that all three CSV files exist, loads them into DuckDB, prints row counts, and then runs both queries.
 
---
 
## Repository Structure
 
```
workspace/
├── goodthought_duckdb.ipynb     # Main notebook (DuckDB version)
├── notebook.ipynb               # Original DataCamp notebook
├── assignments.csv
├── donors.csv
├── donations.csv
├── erd.png                      # Entity relationship diagram
├── erd_flow.png
├── ngo_project_image.jpg
└── README.md
```
 
---
 
## Skills Demonstrated
 
SQL joins · Aggregation (`SUM`, `COUNT`, `ROUND`) · CTEs · Window functions (`ROW_NUMBER`, `PARTITION BY`) · Handling ties in rankings · Data quality checks · DuckDB · pandas · Turning query results into business recommendations
 
---
 
## Author
 
**Uti Wisdom**
ALX Data Engineering Graduate | Data Science Volunteer Mentor | Secondary School Teacher, Warri, Nigeria
[LinkedIn](https://linkedin.com/in/uti-wisdom-286602228/) · [GitHub](https://github.com/utiwisdom) · [Portfolio](https://datascienceportfol.io/wisdomuti8)
 
