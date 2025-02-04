# Sql-exploratory-data-analysis

-- Exploratory Data Analysis

-- Here we are jsut going to explore the data and find trends or patterns or anything interesting like outliers

-- normally when you start the EDA process you have some idea of what you're looking for

-- with this info we are just going to look around and see what we find!

SELECT * 
FROM world_layoffs.layoffs1;


SELECT MAX(total_laid_off)
FROM world_layoffs.layoffs1;


-- Which companies had 1 which is basically 100 percent of they company laid off

SELECT *
FROM world_layoffs.layoffs1
where percentage_laid_off = 1;

-- if we order by funcs_raised_millions we can see how big some of these companies were

SELECT *
FROM world_layoffs.layoffs1
WHERE  percentage_laid_off = 1
order by funds_raised desc;


-- Looking at Percentage to see how big these layoffs were
SELECT MAX(percentage_laid_off),  MIN(percentage_laid_off)
FROM world_layoffs.layoffs1
WHERE  percentage_laid_off IS NOT NULL;


-- Companies with the most Total Layoffs
SELECT company, SUM(total_laid_off)
FROM world_layoffs.layoffs1
GROUP BY company
order by 2 desc;

-- by location
SELECT location, SUM(total_laid_off)
FROM world_layoffs.layoffs
GROUP BY location
ORDER BY 2 DESC
LIMIT 10;

-- by country

SELECT country, SUM(total_laid_off)
FROM world_layoffs.layoffs1
GROUP BY country
ORDER BY 2 DESC
LIMIT 10;


-- Companies with the biggest single Layoff
SELECT company, total_laid_off
FROM world_layoffs.layoffs1
ORDER BY 2 DESC
LIMIT 5;

-- looking at date range
select min('date'), max('date') from layoffs1;

-- this it total in the past 4 years or in the dataset

-- by year 

SELECT YEAR(date), SUM(total_laid_off)
FROM world_layoffs.layoffs1
GROUP BY YEAR(date)
ORDER BY 1 ASC;

  -- by industry
SELECT industry, SUM(total_laid_off)
FROM world_layoffs.layoffs1
GROUP BY industry
ORDER BY 2 DESC;

-- by stage

SELECT stage, SUM(total_laid_off)
FROM world_layoffs.layoffs
GROUP BY stage
ORDER BY 1 DESC;

-- Rolling Total of Layoffs Per Month

SELECT SUBSTRING(date,1,7) as month, SUM(total_laid_off) AS total_laid_off
FROM world_layoffs.layoffs1
where substring(date, 1,7) is not null
GROUP BY month
ORDER BY 1 ASC;

 

-- Earlier we looked at Companies with the most Layoffs. Now let's look at that per year. It's a little more difficult.

-- I want to look at 

WITH Company_Year AS 
(
  SELECT company , YEAR(date) AS years, SUM(total_laid_off) AS total_laid_off
  FROM layoffs
  GROUP BY company, YEAR(date)
)
, Company_Year_Rank AS (
  SELECT company, years, total_laid_off, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
  FROM Company_Year
)
SELECT company, years, total_laid_off, ranking
FROM Company_Year_Rank
WHERE ranking <= 3
AND years IS NOT NULL
ORDER BY years ASC, total_laid_off DESC;


-- now use it in a CTE so we can query off of it
WITH DATE_CTE AS 
(
SELECT SUBSTRING(date,1,7) as dates, SUM(total_laid_off) AS total_laid_off
FROM world layoffs
GROUP BY dates
ORDER BY dates ASC
)
SELECT dates, SUM(total_laid_off) OVER (ORDER BY dates ASC) as rolling_total_layoffs
FROM DATE_CTE
ORDER BY dates ASC;
