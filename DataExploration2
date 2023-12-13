SELECT * 
FROM CovidPortfolioA..CovidDeaths
Order by 3,4

SELECT * 
FROM CovidPortfolioA..CovidVaccinations
Order by 3,4


-- let me select data that we are gonna work with 
SELECT Location , date , total_cases , new_cases , total_deaths , population  
FROM CovidPortfolioA..CovidDeaths
Order by 1,2

--- type of data
SELECT COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'CovidDeaths' 
    AND COLUMN_NAME IN ('total_deaths', 'total_cases');

	
	-- Total cases vs total deaths in USA 
	
SELECT Location , date , total_cases , total_deaths , (total_deaths/total_cases)*100 as DeathPercentage 
FROM CovidPortfolioA..CovidDeaths
Where Location like '%states%'
Order by 1,2

----- totaldeaths DESC ( max number of deaths ) 
SELECT Location , date , total_cases , total_deaths , (total_deaths/total_cases)*100 as DeathPercentage 
FROM CovidPortfolioA..CovidDeaths
Where Location like '%states%'
Order by CAST(total_deaths AS DECIMAL(18, 2)) DESC, Location, date;

--- Looking at the total cases vs Population 
--- i always try to select desc to see the maximum affected population
	
SELECT Location , date , total_cases ,Population , (total_cases/population)*100 as AffectedPopulation 
FROM CovidPortfolioA..CovidDeaths
Where Location like '%states%'
Order by AffectedPopulation DESC ,1,2

--- let me see all countries 
SELECT Location , date , total_cases ,Population , (total_cases/population)*100 as AffectedPopulation 
FROM CovidPortfolioA..CovidDeaths
Order by AffectedPopulation DESC ,1,2

-- Countries with highest infection rate compared to population 
	SELECT Location,Population,MAX(total_cases) AS HighestInfectioncount, MAX(total_cases/population)*100 as AffectedPopulationWRLDW
FROM CovidPortfolioA..CovidDeaths
Group By Location , Population
Order by AffectedPopulationWRLDW DESC 

--showing countries with highest death count per population 
	SELECT Location,Population,MAX(cast(total_deaths as int))AS Highest_Death_count, MAX(total_deaths/population)*100 as DeathPopulationWRLDW
FROM CovidPortfolioA..CovidDeaths
Group By Location , Population
Order by Highest_Death_count DESC 

----showing location with highest percentage death/population  
	SELECT Location,Population,MAX(cast(total_deaths as int))AS Highest_Death_count, MAX(total_deaths/population)*100 as highest_percentage_deathP
FROM CovidPortfolioA..CovidDeaths
Group By Location , Population
Order by highest_percentage_deathP DESC 

---highets percentage of death/cases across the world each day
SELECT
    Date,
    SUM(new_cases) AS total_cases,
    SUM(CAST(new_deaths AS INT)) AS total_deaths,
	    (SUM(CAST(new_deaths AS INT))/SUM(new_cases)) * 100 AS percentage_of_deaths
FROM
    CovidPortfolioA..CovidDeaths
				where continent is not null 

GROUP BY
    Date
	Order by 
	percentage_of_deaths DESC


	-- let's go now to COVID VACCINATIONS
SELECT * 
   FROM CovidPortfolioA..CovidDeaths dea
   Join   CovidPortfolioA..CovidVaccinations vac
   On dea.location = vac.location
   and dea.date = vac.date

   -- Highest new vacciation per day 
   SELECT dea.continent, dea.location,dea.date,dea.population,CAST(vac.new_vaccinations AS int )  new_vaccination
   FROM CovidPortfolioA..CovidDeaths dea 
   Join   CovidPortfolioA..CovidVaccinations vac
   On dea.location = vac.location
   AND  dea.date = vac.date
      where dea.continent is not null 
Order by 5 DESC

-- CTE METHOD 1 
  WITH PopsvsVac (continent,location,date,population,new_vaccinations,RollingPeopleVaccinated) AS 
  (
  SELECT dea.continent, dea.location,dea.date,dea.population, vac.new_vaccinations,
   SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.location order by dea.location , dea.date)AS RollingPeopleVaccinated
  FROM CovidPortfolioA..CovidDeaths dea 
   Join   CovidPortfolioA..CovidVaccinations vac
   On dea.location = vac.location
   AND  dea.date = vac.date
      where dea.continent is not null 
	  AND  new_vaccinations is not null 
)
Select*,(RollingPeopleVaccinated/population)*100 as pourcentage_vacinnatedpeople
FROM PopsvsVac

--CTE METHOD 2 
WITH RollingVaccinations AS (
  SELECT
    dea.continent,
    dea.location,
    dea.date,
    dea.population,
    vac.new_vaccinations,
    SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.date) AS RollingPeopleVaccinated
  FROM
    CovidPortfolioA..CovidDeaths dea
    JOIN CovidPortfolioA..CovidVaccinations vac
    ON dea.location = vac.location
    AND dea.date = vac.date
  WHERE
    dea.continent IS NOT NULL
    AND new_vaccinations IS NOT NULL
)
SELECT
  continent,
  location,
  date,
  population,
  new_vaccinations,
  RollingPeopleVaccinated,
  (RollingPeopleVaccinated / population) * 100 AS pourcentage_vaccinated_people
FROM
  RollingVaccinations
ORDER BY
  location,
  date;

----temptable version 2 
DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From CovidPortfolioA..CovidDeaths dea
Join CovidPortfolioA..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
--where dea.continent is not null 
--order by 2,3

Select *, (RollingPeopleVaccinated/Population)*100 AS pourcentage
From #PercentPopulationVaccinated


----% of VAC PER COUNTRY 
WITH RankedVaccination AS (
    SELECT
      Location,
      Population,
        RollingPeopleVaccinated,
        (RollingPeopleVaccinated / Population) * 100 AS Percentage,
        ROW_NUMBER() OVER (PARTITION BY Location ORDER BY (RollingPeopleVaccinated / Population) DESC) AS RowNum
    FROM #PercentPopulationVaccinated
)

SELECT
    Location,
   Population,
    RollingPeopleVaccinated,
    Percentage
FROM RankedVaccination
WHERE RowNum = 1  

---% of VAC IN AFRICA 
WITH RankedVaccination AS (
    SELECT
	continent,
      Location,
      Population,
        RollingPeopleVaccinated,
        (RollingPeopleVaccinated / Population) * 100 AS Percentage,
        ROW_NUMBER() OVER (PARTITION BY Location ORDER BY (RollingPeopleVaccinated / Population) DESC) AS RowNum
    FROM #PercentPopulationVaccinated
)

SELECT
	continent,
    Location,
   Population,
    RollingPeopleVaccinated,
    Percentage
FROM RankedVaccination
WHERE RowNum = 1  
AND continent='Africa'
ORDER BY Percentage DESC


---% of VAC IN EUROPE 
WITH RankedVaccination AS (
    SELECT
	continent,
      Location,
      Population,
        RollingPeopleVaccinated,
        (RollingPeopleVaccinated / Population) * 100 AS Percentage,
        ROW_NUMBER() OVER (PARTITION BY Location ORDER BY (RollingPeopleVaccinated / Population) DESC) AS RowNum
    FROM #PercentPopulationVaccinated
)

SELECT
	continent,
    Location,
   Population,
    RollingPeopleVaccinated,
    Percentage
FROM RankedVaccination
WHERE RowNum = 1  
AND continent='Europe'
ORDER BY Percentage DESC

-- Creating View to store data for later visualizations

Create View PercentPopulationVaccinated2 as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From CovidPortfolioA..CovidDeaths dea
Join CovidPortfolioA..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 

SELECT* 
FROM PercentPopulationVaccinated



/*

Queries used for Tableau Project

*/



-- 1. 

Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From PortfolioProject..CovidDeaths
--Where location like '%states%'
where continent is not null 
--Group By date
order by 1,2

-- Just a double check based off the data provided
-- numbers are extremely close so we will keep them - The Second includes "International"  Location


--Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
--From PortfolioProject..CovidDeaths
----Where location like '%states%'
--where location = 'World'
----Group By date
--order by 1,2


-- 2. 

-- We take these out as they are not inluded in the above queries and want to stay consistent
-- European Union is part of Europe

Select location, SUM(cast(new_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
--Where location like '%states%'
Where continent is null 
and location not in ('World', 'European Union', 'International')
Group by location
order by TotalDeathCount desc


-- 3.

Select Location, Population, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
From PortfolioProject..CovidDeaths
--Where location like '%states%'
Group by Location, Population
order by PercentPopulationInfected desc


-- 4.


Select Location, Population,date, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
From PortfolioProject..CovidDeaths
--Where location like '%states%'
Group by Location, Population, date
order by PercentPopulationInfected desc












-- 1. 

Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From CovidPortfolioA..CovidDeaths
--Where location like '%states%'
where continent is not null 
--Group By date
order by 1,2

-- Just a double check based off the data provided
-- numbers are extremely close so we will keep them - The Second includes "International"  Location


--Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
--From PortfolioProject..CovidDeaths
----Where location like '%states%'
--where location = 'World'
----Group By date
--order by 1,2


-- 2. 

-- We take these out as they are not inluded in the above queries and want to stay consistent
-- European Union is part of Europe

Select location, SUM(cast(new_deaths as int)) as TotalDeathCount
From CovidPortfolioA..CovidDeaths
--Where location like '%states%'
Where continent is null 
and location not in ('World', 'European Union', 'International')
Group by location
order by TotalDeathCount desc


-- 3.

Select Location, Population, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
From CovidPortfolioA..CovidDeaths
--Where location like '%states%'
Group by Location, Population
order by PercentPopulationInfected desc


-- 4.


Select Location, Population,date, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
From CovidPortfolioA..CovidDeaths
--Where location like '%states%'
Group by Location, Population, date
order by PercentPopulationInfected desc


