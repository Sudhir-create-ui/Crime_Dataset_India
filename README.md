# Crime_Dataset_India
Analysis of Crime_Dataset
CREATE TABLE crime_dataset_india (
    report_number INT PRIMARY KEY,
    date_reported DATE,
    date_of_occurrence DATE,
    time_of_occurrence TIME,
    city VARCHAR(100),
    crime_code INT,
    crime_description VARCHAR(255),
    victim_age INT,
    victim_gender CHAR(1),
    weapon_used VARCHAR(100),
    crime_domain VARCHAR(100),
    police_deployed INT,
    case_closed BOOLEAN,
    date_case_closed DATE
);
select * from crime_dataset_india;

ALTER TABLE crime_dataset_india
ALTER COLUMN report_number TYPE VARCHAR(20);

ALTER TABLE crime_dataset_india
ALTER COLUMN date_of_occurrence TYPE TEXT;


copy public.crime_dataset_india(
    report_number, date_reported, date_of_occurrence, time_of_occurrence,
    city, crime_code, crime_description, victim_age, victim_gender,
    weapon_used, crime_domain, police_deployed, case_closed, date_case_closed
)
FROM 'E:/Crime_Data/crime_dataset_india1.csv'
WITH (
    FORMAT csv,
    HEADER,  -- ✅ This skips the first row
    DELIMITER ',',
    ENCODING 'UTF8',
    QUOTE '"',
    ESCAPE '"');


SELECT COUNT(*) FROM crime_dataset_india;


SELECT *
FROM crime_dataset_india
LIMIT 10;

SELECT
  COUNT(*) FILTER (WHERE date_of_occurrence IS NULL) AS null_dates,
  COUNT(*) FILTER (WHERE report_number IS NULL) AS null_firs
FROM crime_dataset_india;

ALTER TABLE crime_dataset_india
ALTER COLUMN date_of_occurrence
TYPE TIMESTAMP
USING to_timestamp(date_of_occurrence, 'MM-DD-YYYY HH24:MI');

SELECT date_of_occurrence
FROM crime_dataset_india
WHERE date_of_occurrence IS NOT NULL
AND date_of_occurrence !~ '^\d{2}-\d{2}-\d{4}';


UPDATE crime_dataset_india
SET date_of_occurrence = NULL
WHERE date_of_occurrence !~ '^\d{2}-\d{2}-\d{4}';

SELECT COUNT(*) 
FROM crime_dataset_india
WHERE date_of_occurrence IS NULL;


SELECT DISTINCT date_of_occurrence
FROM crime_dataset_india
WHERE date_of_occurrence IS NOT NULL
LIMIT 20;


SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'crime_dataset_india'
AND column_name = 'date_of_occurrence';

ALTER TABLE crime_dataset_india
ADD COLUMN date_clean TIMESTAMP;

UPDATE crime_dataset_india
SET date_clean = CASE
  WHEN date_of_occurrence IS NULL THEN NULL
  ELSE date_of_occurrence::TIMESTAMP
END;

ALTER TABLE crime_dataset_india
DROP COLUMN date_of_occurrence;

ALTER TABLE crime_dataset_india
RENAME COLUMN date_clean TO date_of_occurrence;

SELECT 
  MIN(date_of_occurrence) AS earliest_date,
  MAX(date_of_occurrence) AS latest_date,
  COUNT(*) AS total_rows
FROM crime_dataset_india;

CREATE OR REPLACE VIEW crime_cleaned AS
SELECT
    crime_code,
    time_of_occurrence,
    police_deployed,
    case_closed,
    EXTRACT(YEAR FROM time_of_occurrence) AS crime_year,
    EXTRACT(MONTH FROM time_of_occurrence) AS crime_month,
    EXTRACT(DAY FROM time_of_occurrence) AS crime_day
FROM crime_dataset_india
WHERE time_of_occurrence IS NOT NULL;



--Total cases by Police deployed
SELECT police_deployed, COUNT(*) AS total_cases
FROM crime_cleaned
GROUP BY police_deployed
ORDER BY total_cases DESC;

--Cases closed vs open
SELECT case_closed, COUNT(*) AS total_cases
FROM crime_cleaned
GROUP BY case_closed;





CREATE INDEX idx_time_occurrence ON crime_dataset_india(time_of_occurrence);
CREATE INDEX idx_police_deployed ON crime_dataset_india(police_deployed);
CREATE INDEX idx_case_closed ON crime_dataset_india(case_closed);

--Total cases per police deployed

SELECT police_deployed, COUNT(*) AS total_cases
FROM crime_dataset_india
GROUP BY police_deployed
ORDER BY total_cases DESC;

--Average number of cases closed per police unit

SELECT police_deployed, 
       COUNT(*) FILTER (WHERE case_closed = 'Yes') AS closed_cases,
       COUNT(*) AS total_cases,
       ROUND(100.0 * COUNT(*) FILTER (WHERE case_closed = 'Yes') / COUNT(*), 2) AS closure_percentage
FROM crime_dataset_india
GROUP BY police_deployed
ORDER BY closure_percentage DESC;


























