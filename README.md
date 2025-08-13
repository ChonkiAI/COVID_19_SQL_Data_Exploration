# 🦠 COVID-19 Data Exploration in SQL

## 📌 Overview
This project involves **exploratory data analysis (EDA)** on a global COVID-19 dataset using SQL.  
The aim was to discover insights into **infection rates, death rates, and vaccination trends** across countries and continents.

I leveraged **advanced SQL techniques** to transform raw COVID-19 data into actionable insights—demonstrating my ability to work with real-world datasets, perform complex queries, and prepare data for visualizations.

---

## 🎯 Key Objectives
- Analyze **COVID-19 cases, deaths, and vaccination progress** globally.
- Compare **infection and mortality rates** between countries and continents.
- Track **vaccination rollouts** and **population coverage** over time.
- Create **views** for future dashboard visualizations.

---

## 🛠 Skills & SQL Concepts Used
- **Data Filtering & Ordering**
- **Aggregations** (`SUM`, `MAX`, `GROUP BY`)
- **Joins** (Deaths + Vaccination datasets)
- **Common Table Expressions (CTEs)**
- **Window Functions** (`OVER`, `PARTITION BY`)
- **Temporary Tables**
- **Views Creation** (visualization-ready datasets)
- **Data Type Conversions** (`CAST`, `CONVERT`)

---

## 📂 Dataset
- COVID-19 Deaths: [Data](https://github.com/ChonkiAI/COVID_19_SQL_Data_Exploration/blob/main/CovidDeaths.xlsx)
- COVID-19 Vaccination: [Data](http://github.com/ChonkiAI/COVID_19_SQL_Data_Exploration/blob/main/CovidVaccinations.xlsx)
- Source: [Our World in Data – COVID-19 Dataset](https://ourworldindata.org/covid-deaths)  
- Format: CSV files imported into SQL Server

---

## 📊 Queries & Insights

Below are all the SQL queries used in the project, along with a **brief description** of what they accomplish and **how**.

---

### 1. Previewing the Dataset
**Purpose:** View raw data and filter out entries with missing continent information.  
**How:** Select all columns where `continent IS NOT NULL` and sort by location and date.
```
SELECT *
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 3, 4;
```
## 2. Selecting Key Columns
**Purpose:** Focus on relevant metrics for analysis.  
**How:** Retrieve `location`, `date`, `total_cases`, `new_cases`, `total_deaths`, and `population`.
```
SELECT Location, date, total_cases, new_cases, total_deaths, population
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1, 2;
```
## 3. Total Cases vs Total Deaths
**Purpose:** Calculate the death percentage for each date in a specific country.
**How:** Divide total deaths by total cases, multiply by 100.
```
SELECT Location, date, total_cases, total_deaths, 
       (total_deaths/total_cases)*100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE location LIKE '%states%'
  AND continent IS NOT NULL
ORDER BY 1, 2;
```
## 4. Total Cases vs Population
**Purpose:** Find the percentage of population infected over time.
**How:** Divide total cases by population, multiply by 100.
```
SELECT Location, date, Population, total_cases,  
       (total_cases/population)*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
ORDER BY 1, 2;
```
## 5. Countries with Highest Infection Rate
**Purpose:** Identify the countries most affected proportionally.
**How:** Find the max infection count and calculate % of population infected.
```
SELECT Location, Population, 
       MAX(total_cases) AS HighestInfectionCount,  
       MAX((total_cases/population))*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
GROUP BY Location, Population
ORDER BY PercentPopulationInfected DESC;
```
## 6. Countries with Highest Death Count
**Purpose:** Find which countries had the most deaths.
**How:** Convert deaths to INT and take the maximum per country.
```
SELECT Location, MAX(CAST(Total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY Location
ORDER BY TotalDeathCount DESC;
```
## 7. Continents with Highest Death Count
**Purpose:** Compare deaths at the continent level.
**How:** Aggregate by continent.
```
SELECT continent, MAX(CAST(Total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY TotalDeathCount DESC;
```
## 8. Global Numbers
**Purpose:** View total cases, total deaths, and death percentage globally.
**How:** Sum new cases and deaths, then calculate death percentage.
```
SELECT SUM(new_cases) AS total_cases, 
       SUM(CAST(new_deaths AS int)) AS total_deaths, 
       SUM(CAST(new_deaths AS int))/SUM(new_cases)*100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1, 2;
```
## 9. Total Population vs Vaccinations
**Purpose:** Calculate cumulative vaccinations per location.
**How:** Use a JOIN between deaths and vaccinations datasets, with a rolling sum.
```
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(CONVERT(int,vac.new_vaccinations)) 
       OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
  ON dea.location = vac.location
 AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
ORDER BY 2, 3;
```
## 10. CTE for Vaccination Percentage
**Purpose:** Calculate vaccination percentage using a CTE for cleaner logic.
**How:** Store rolling vaccination totals in CTE, then compute percentage.
```
WITH PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated) AS (
    SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
           SUM(CONVERT(int,vac.new_vaccinations)) 
           OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
    FROM PortfolioProject..CovidDeaths dea
    JOIN PortfolioProject..CovidVaccinations vac
      ON dea.location = vac.location
     AND dea.date = vac.date
    WHERE dea.continent IS NOT NULL
)
SELECT *, (RollingPeopleVaccinated/Population)*100 AS VaccinatedPercentage
FROM PopvsVac;
```
## 11. Temporary Table for Vaccination Percentage
**Purpose:** Store intermediate vaccination calculations for later queries.
**How:** Create and populate a temp table with rolling vaccination totals.
```
DROP TABLE IF EXISTS #PercentPopulationVaccinated;

CREATE TABLE #PercentPopulationVaccinated (
    Continent NVARCHAR(255),
    Location NVARCHAR(255),
    Date DATETIME,
    Population NUMERIC,
    New_vaccinations NUMERIC,
    RollingPeopleVaccinated NUMERIC
);

INSERT INTO #PercentPopulationVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(CONVERT(int,vac.new_vaccinations)) 
       OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
  ON dea.location = vac.location
 AND dea.date = vac.date;

SELECT *, (RollingPeopleVaccinated/Population)*100 AS VaccinatedPercentage
FROM #PercentPopulationVaccinated;
```
## 12. Creating a View for Visualization
**Purpose:** Create a reusable dataset for BI dashboards.
**How:** Store vaccination calculations in a view for easy access.
```
CREATE VIEW PercentPopulationVaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(CONVERT(int,vac.new_vaccinations)) 
       OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
  ON dea.location = vac.location
 AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;
```
