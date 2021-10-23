### Covid 19 Data SQL Queries

/*

Covid 19 Data Exploration
Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions

*/

-- Select the main data I'm working with
``` sql
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM `covid-19-healthy-diet.COVID_datasets.Covid_deaths`
ORDER BY 1,2;
```

-- Total Cases vs Total Deaths; Shows the likelihood of dying if a person contracts Covid in respective country
```sql
SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases) * 100 as DeathPercentage
FROM `covid-19-healthy-diet.COVID_datasets.Covid_deaths`
ORDER BY 1,2;
```

-- Total Cases vs Population; Shows the likelihood of contracting Covid in respective country
```sql
SELECT location, date, total_cases, population, (total_cases/population) * 100 as PercentPopInfected
FROM `covid-19-healthy-diet.COVID_datasets.Covid_deaths`
WHERE location = 'United States'
ORDER BY 1,2;
```

-- What countries have the highest infection rate?
```sql
SELECT location, population, max(total_cases) as HighestInfectionCount, 
  max((total_cases/population)) * 100 as PercentPopInfected
FROM `covid-19-healthy-diet.COVID_datasets.Covid_deaths`
GROUP BY 1,2
ORDER BY 4 DESC;
```

-- Showing countries with highest death count per population
```sql
SELECT location, max(total_deaths) as HighestDeath
FROM `covid-19-healthy-diet.COVID_datasets.Covid_deaths`
WHERE continent is not null 
GROUP BY 1
ORDER BY 2 DESC;
```

-- Showing continents with highest death count per population. We have to GROUP BY location and put in WHERE statement due to the nature of the dataset
```sql
SELECT location, max(total_deaths) as HighestDeath
FROM `covid-19-healthy-diet.COVID_datasets.Covid_deaths`
WHERE continent is null 
GROUP BY 1
ORDER BY 2 DESC;
```

-- Global Numbers
```sql
SELECT date, sum(new_cases) as total_cases, sum(new_deaths) as total_deaths, 
  (sum(new_deaths)/sum(new_cases))*100 as DeathPercentage
FROM `covid-19-healthy-diet.COVID_datasets.Covid_deaths`
WHERE continent is not null
GROUP BY 1
ORDER BY 1,2;
```

-- Total Population vs Vaccinations
```sql
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
FROM `covid-19-healthy-diet.COVID_datasets.Covid_deaths` dea
JOIN `covid-19-healthy-diet.COVID_datasets.Covid_vaccinations` vac
ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent is not null 
ORDER BY 1,2,3;
```

-- Total Population vs Rolling Vaccinations
```sql
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, sum(vac.new_vaccinations) OVER 
    (PARTITION BY dea.location ORDER BY dea.location,dea.date) as RollingVaccinated
FROM `covid-19-healthy-diet.COVID_datasets.Covid_deaths` dea
JOIN `covid-19-healthy-diet.COVID_datasets.Covid_vaccinations` vac
ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent is not null 
ORDER BY 1,2,3;
```

-- Use CTE
```sql
WITH PopvsVac AS (SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
  sum(vac.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location,dea.date) as RollingVaccinated
FROM `covid-19-healthy-diet.COVID_datasets.Covid_deaths` dea
JOIN `covid-19-healthy-diet.COVID_datasets.Covid_vaccinations` vac
ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent is not null) 

SELECT *, (RollingVaccinated/population)*100 AS PercentOfRollingVacIndiv
FROM PopvsVac
WHERE PopvsVac.location = 'Vietnam';
```
