# 🦠 COVID-19 Data Exploration in SQL

## 📌 Overview
This project involves **exploratory data analysis (EDA)** on a global COVID-19 dataset using SQL.  
The goal is to uncover meaningful insights into **infection rates, death rates, and vaccination trends** across different countries and continents.

Through this project, I applied **advanced SQL techniques** to transform raw COVID-19 data into actionable insights—demonstrating my ability to work with **real-world datasets**, perform complex queries, and prepare data for visualization.

---

## 🎯 Key Objectives
- Analyze **COVID-19 cases, deaths, and vaccination progress** globally.
- Compare **infection and mortality rates** between countries and continents.
- Track **vaccination rollouts** and their impact on population coverage.
- Prepare **aggregated datasets** for future dashboard visualizations.

---

## 🛠 Skills & SQL Concepts Used
- **Data Filtering & Ordering**
- **Aggregations** (`SUM`, `MAX`, `GROUP BY`)
- **Joins** (combining deaths and vaccination datasets)
- **Common Table Expressions (CTEs)**
- **Window Functions** (`OVER`, `PARTITION BY`)
- **Temporary Tables**
- **Views Creation** (for visualization-ready datasets)
- **Data Type Conversions** (`CAST`, `CONVERT`)

---

## 📂 Dataset
The dataset used in this project includes:
- **COVID-19 Deaths Data**
- **COVID-19 Vaccination Data**

Data source: [Our World in Data – COVID-19 Dataset](https://ourworldindata.org/covid-deaths)  
Format: CSV files imported into SQL Server

---

## 📊 Key Insights Derived
1. **Likelihood of Death upon Infection**  
   Calculated death percentage for each country and compared across regions.
   
2. **Infection Rates by Population**  
   Identified countries with the **highest infection rates** relative to their population.
   
3. **Global Death Totals by Continent**  
   Aggregated deaths to reveal the **most affected continents**.
   
4. **Vaccination Progress Tracking**  
   Computed **rolling totals** of vaccinated individuals using window functions.
   
5. **Visualization-Ready Data**  
   Created views to feed into **Power BI/Tableau dashboards**.

---

## 🧮 Sample Queries
### Percentage of Population Infected
```sql
SELECT Location, date, Population, total_cases,
       (total_cases/population)*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
ORDER BY 1, 2;
