# Data Cleaning Project in SQL - Global Layoffs

#### Project Overview

This project presents a comprehensive data cleaning process for a global dataset on corporate layoffs spanning from 2020 to 2023.

#### Data Cleaning Steps

1. Removed duplicates
2. Standardized the data
3. Populated NULL values / BLANK values
4. Removed any unnecessary columns and rows

#### Data Source

https://github.com/AlexTheAnalyst/MySQL-YouTube-Series/blob/main/layoffs.csv

#### Tools

- MySQL Workbench

#### References

AlexTheAnalyst


### Code

```
SELECT *
FROM layoffs;

--> 1.) Step: Removed duplicates


--> Created a staging table to work in.


CREATE table layoffs_staging
LIKE layoffs;

INSERT layoffs_staging
SELECT *
FROM layoffs;


-- Checked for duplicates by looking at every column, to see if any duplicate rows. As this table has no unique column (for example unique ID)


SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, total_laid_off, percentage_laid_off, date) AS row_num
FROM layoffs_staging;


-- Checked to see where the duplicates are (row_num = > 2)


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


-- Copied the layoffs.staging table again with the added column "row_num" and filtered for the ones which = 2 , as these are the duplicates, and then deleted them

-- Created the table with "Copy Clipboard - Create Statemment" and added: `row_num` INT


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

SELECT *
FROM layoffs_staging2
WHERE row_num > 1;


-- Inserted column and rows from layoffs_staging table into layoffs_staging2


INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

SELECT *
FROM layoffs_staging2;


-- Deleted duplicates.


DELETE
FROM layoffs_staging2
WHERE row_num > 1;

SELECT *
FROM layoffs_staging2
WHERE row_num > 1;


--> 2.) Step: Standardized the Data

-- Removed the space characters.


SELECT company, TRIM(company)
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET company = TRIM(company);


-- Looked at spelling mistakes in the industry column and corrected them.


SELECT DISTINCT industry
FROM layoffs_staging2
ORDER BY 1;

SELECT *
FROM layoffs_staging2
WHERE industry LIKE 'Crypto%';

UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';

SELECT DISTINCT industry
FROM layoffs_staging2;


-- Scanned the country column for errors and found a dot after Untited States. Deleted it and set the trimmed column as the new country column.


SELECT DISTINCT country
FROM layoffs_staging2
ORDER BY 1;

SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_staging2
ORDER BY 1;

UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';


-- Changed the date column from text to date.


SELECT date 
FROM layoffs_staging2;

SELECT date,
STR_TO_DATE(date, '%m/%d/%Y')
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET date = STR_TO_DATE(date, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN date DATE;

SELECT *
FROM layoffs_staging2;


--> 3.) Step: Populated NULL values / BLANK values

-- Set blanks to NULL's, as it's easier to work with them


SELECT *
FROM layoffs_staging2
WHERE industry IS NULL 
OR industry = '';

UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';

SELECT *
FROM layoffs_staging2
WHERE company = "Airbnb";


-- One row of AirBnb's is not populated in the column "industry" but from the second row it is visible that the industry is "travel".
-- This is the case for some other compies as well. Which is why I joined the table on itself to populate the NULL's where possible.


SELECT *
FROM layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;


-- Bally's is the only company where the industry could not be populated, as there was only one row and it was NULL


SELECT *
FROM layoffs_staging2
WHERE company LIKE "Bally%";

SELECT *
FROM layoffs_staging2;


-- 4.) Step: Removed any Columns and Rows

-- Cheked where there is a NULL in both columns "total_laid_off" and "percentage_laid_off" because then the information/ row is not useful for the context of further analysis so it can be removed.


SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

DELETE 
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

SELECT *
FROM layoffs_staging2;


-- In the end I just deleted the additional column "row_num" which I used earlier to identify duplicates, as it is unnecessary 


ALTER TABLE layoffs_staging2
DROP COLUMN row_num;

SELECT *
FROM layoffs_staging2;


-- > DATA IS CLEAN NOW ! :-)

```
