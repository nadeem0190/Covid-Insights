-- Exploring insights on Covid-19 with PostgreSQL

SELECT * 
FROM coviddeaths
WHERE continent IS NOT NULL
ORDER BY 3, 4


-- Getting started with data

SELECT location, date, total_cases, new_cases, total_deaths, population
FROM coviddeaths
WHERE continent IS NOT NULL
ORDER BY 1, 2;


/* Let's look at Total Cases vs Total Deaths

Likelihood of dying with Covid in your country */

SELECT location, date, total_cases, total_deaths,
ROUND(((total_deaths::decimal / total_cases)*100), 2) AS death_percentage
FROM coviddeaths
WHERE location LIKE 'India'
ORDER BY 1, 2;


-- Taking a look at Total Cases vs Population to find the Percentage of Population infected with Covid

SELECT location AS country, date, population, total_cases,
ROUND(((total_cases::decimal / population)*100), 5) AS population_infected_percentage
FROM coviddeaths
-- WHERE location LIKE 'India'
WHERE continent IS NOT NULL
ORDER BY 1, 2;


-- Looking at countries with Highest infection Rate compared to Population

SELECT location AS country, population, MAX(total_cases) AS highest_infection_count,
ROUND(MAX(total_cases::decimal / population)*100, 2) AS population_infected_percentage
FROM coviddeaths
-- WHERE location LIKE 'India'
WHERE continent IS NOT NULL AND total_cases IS NOT NULL
GROUP BY location, population
ORDER BY population_infected_percentage DESC;


-- List of Countries with Highest Death Count per Population

SELECT location AS country, MAX(total_deaths) AS total_death_count
FROM coviddeaths
WHERE continent IS NOT NULL AND total_deaths IS NOT NULL
GROUP BY location
ORDER BY total_death_count DESC;



-- BREAKING THINGS DOWN BY CONTINENT

-- List of Continents with Highest Death Count per Population

SELECT location, MAX(total_deaths) AS total_death_count
FROM coviddeaths
WHERE continent IS NULL AND total_deaths IS NOT NULL
AND location NOT IN 
('World', 'International', 'High income', 'Upper middle income', 'Lower middle income', 'Low income', 'European Union')
GROUP BY location
ORDER BY total_death_count DESC;



-- GLOBAL NUMBERS

-- Including Date
SELECT date, SUM(new_cases) AS total_cases, SUM(new_deaths) AS total_deaths,
ROUND(SUM(new_deaths :: decimal) / SUM(new_cases) * 100, 2) AS death_percentage
FROM coviddeaths
WHERE continent IS NOT NULL
GROUP BY date
ORDER BY date ASC;

-- Overall Numbers
SELECT SUM(new_cases) AS total_cases, SUM(new_deaths) AS total_deaths,
ROUND(SUM(new_deaths :: decimal) / SUM(new_cases) * 100, 2) AS death_percentage
FROM coviddeaths
WHERE continent IS NOT NULL;



-- Taking a look at Vaccinations Table
SELECT * 
FROM covidvaccinations
ORDER BY 3, 4;


-- Looking at Total Population vs Vaccinations to find Percentage of Population that has received atleast One Covid Dose

SELECT d.continent, d.location, d.date, d.population, v.new_vaccinations 
, SUM(v.new_vaccinations) OVER (PARTITION BY d.location ORDER BY d.location, d.date) AS rolling_vaccinated_people
-- , (rolling_vaccinated_people / d.population) * 100
FROM coviddeaths d
JOIN
covidvaccinations v
	ON d.location = v.location
	AND d.date = v.date
WHERE d.continent IS NOT NULL
ORDER BY 2, 3;



-- Using CTE to perform a calculation on Partition By in previous query

WITH PopvsVac(continent, location, date, population, new_vaccinations, rolling_vaccinated_people)
AS
(
SELECT d.continent, d.location, d.date, d.population, v.new_vaccinations 
, SUM(v.new_vaccinations) OVER (PARTITION BY d.location ORDER BY d.location, d.date) AS rolling_vaccinated_people
FROM coviddeaths d
JOIN
covidvaccinations v
	ON d.location = v.location
	AND d.date = v.date
WHERE d.continent IS NOT NULL
ORDER BY 2, 3
	)
SELECT *, ((rolling_vaccinated_people / population) * 100) AS percenatge_population_vaccinated
FROM PopvsVac


-- Temp Table to perform a calculation on Partition By in previous query

DROP TABLE IF EXISTS PercentPopulationVaccinated;
CREATE TEMP TABLE PercentPopulationVaccinated
(
	continent VARCHAR(255),
	location VARCHAR(255),
	date DATE,
	population BIGINT,
	new_vaccinations BIGINT,
	rolling_vaccinated_people NUMERIC
)
;

INSERT INTO PercentPopulationVaccinated(
SELECT d.continent, d.location, d.date, d.population, v.new_vaccinations 
, SUM(v.new_vaccinations) OVER (PARTITION BY d.location ORDER BY d.location, d.date) AS rolling_vaccinated_people
FROM coviddeaths d
JOIN
covidvaccinations v
	ON d.location = v.location
	AND d.date = v.date
WHERE d.continent IS NOT NULL
ORDER BY 2, 3)
;

SELECT *, ((rolling_vaccinated_people / population) * 100) AS percenatge_population_vaccinated
FROM PercentPopulationVaccinated


-- Creating View to store data for visualizations

Create View PercentPopulationVaccinated AS
SELECT d.continent, d.location, d.date, d.population, v.new_vaccinations 
, SUM(v.new_vaccinations) OVER (PARTITION BY d.location ORDER BY d.location, d.date) AS rolling_vaccinated_people
--, (rolling_vaccinated_people / d.population) * 100
FROM coviddeaths d
JOIN
covidvaccinations v
	ON d.location = v.location
	AND d.date = v.date
WHERE d.continent IS NOT NULL
ORDER BY 2, 3;


SELECT * FROM PercentPopulationVaccinated