# Data Cleaning Project in SQL 🧼

### Project Overview

This project presents a comprehensive data cleaning process for a global dataset on corporate layoffs spanning from 2020 to 2023.

### Data Source

https://github.com/AlexTheAnalyst/MySQL-YouTube-Series/blob/main/layoffs.csv

### Tools

- MySQL Workbench

### References

AlexTheAnalyst


### Code

```
SELECT *
FROM layoffs;


--> Created a staging table to work in.


CREATE table layoffs_staging
LIKE layoffs;

INSERT layoffs_staging
SELECT *
FROM layoffs;


--> 1.) Step: Removed duplicates


SELECT *
FROM layoffs_staging;

SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, total_laid_off, percentage_laid_off, date) AS row_num
FROM layoffs_staging;

WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)

SELECT *
FROM duplicate_cte
WHERE row_num > 1;

WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)

DELETE 
FROM duplicate_cte
WHERE row_num > 1;

--> Create a new column and added row numbers in, then deleted where row numbers are over 2, then later deleted that column.


CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

--> Deleted duplicates.


DELETE
FROM layoffs_staging2
WHERE row_num > 1;

SELECT *
FROM layoffs_staging2
WHERE row_num > 1;

























```
