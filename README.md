# Car_Resale_data_analysis
I have completed this project using Postgre SQL. I have written the SQL queries to find out the answer of the question that helped me to analyze the data. In this project, i have answered 15 questions  that really helped me to findout the result that i suppoesed to find out.
# SQL Queries 

## To Create a Table into the database
```sql
DROP TABLE IF EXISTS Cars;
CREATE TABLE Cars (
city VARCHAR(50),
maker VARCHAR(50),
model VARCHAR(50),
variant VARCHAR(100),
mileage INT,
make_year INT,
price FLOAT,
fuel_type VARCHAR(50),
no_of_owners INT,
color VARCHAR(50),
body_type VARCHAR(50),
transmission VARCHAR(50),
registration_year INT,
latest_publish_date DATE
);
```sql
SELECT * FROM Cars;
SELECT COUNT(*) FROM Cars;

/*
# Exploratory Data Analysis

## 1.	Which city has the highest number of listed cars? 
```sql
SELECT City, Count(*) as Quantity,
		RANK() OVER (ORDER BY Count(*) DESC) AS Rank
FROM Cars
	Group by City
	Limit 1;

## 2.	What are the top 10 car makers by number of listings?
```sql
SELECT maker, COUNT(*) AS Quantity,
		RANK() OVER(ORDER BY COUNT(*) DESC) AS RANK
FROM Cars
	GROUP BY maker
	LIMIT 10;

## 3.	Which models are most frequently sold in each city?
```sql
SELECT city,model, Quantity
FROM
		(
		SELECT city,model, count(*) as Quantity,
			DENSE_RANK() OVER(
			PARTITION BY city
			ORDER BY Count(*) DESC
			) AS Rank
		FROM Cars
		GROUP BY city,model
		)
	WHERE Rank=1;
	;

## 4.	What is the distribution of body types (SUV, Sedan, Hatchback)?
```sql
SELECT body_type, COUNT(*) as total_sales FROM Cars
GROUP BY 1
ORDER BY 2 DESC;

 # Pricing Analysis

## 5.	What is the average, median, and range of car prices overall?
```sql
SELECT 
	ROUND(AVG(price)::NUMERIC, 2) AS avg_price,
	MAX(price)-MIN(price) as range_price,
	 PERCENTILE_CONT(0.5) 
        WITHIN GROUP (ORDER BY price) AS median_price
FROM Cars;

## 6.	How does price vary by maker (Hyundai vs Maruti vs Honda)?
```sql
SELECT maker, 
		ROUND(AVG(price)::NUMERIC,2) AS AVG_price
FROM Cars
WHERE maker IN ('Hyundai','Maruti Suzuki','Honda')
GROUP BY maker
ORDER BY 2 DESC;

## 7.	Which models give the best value (lowest price for newer make_year)?
```sql
WITH ranked_cars AS (
SELECT 	maker,
		model,
		price,
		make_year,
		ROW_NUMBER() OVER (
		PARTITION BY maker
		ORDER BY make_year DESC, price ASC) AS value_rank
FROM Cars
)
SELECT maker,
		model,
		price,
		make_year
FROM ranked_cars
WHERE value_rank=1;

## 8.	How does price differ by body type (SUV vs Sedan vs Hatchback)?
```sql
SELECT * FROM Cars;
SELECT * FROM (
SELECT	body_type,
		price,
		ROW_NUMBER() OVER(
		PARTITION BY body_type
		ORDER BY price DESC) AS rank
FROM Cars
)
where rank=1
;
# Age & Depreciation
## 9. How does car price decrease with age (make_year vs price)?
```sql 
select * FROM (
select make_year,
		body_type,
		price,
		ROW_NUMBER() OVER (
	PARTITION BY make_year
	ORDER BY price DESC	) AS rank
from cars
)
WHERE rank=1;

## 10. What is the average price drop per year for each maker?
```sql
select * from cars;

select maker,
		make_year,
		Avg(price) as Avg_price
FROM cars
GROUP BY 1,2
ORDER BY 2 ASC
; 

## 11.	Are newer cars (2020+) priced significantly higher than older ones?
```sql
SELECT 
	CASE
		WHEN make_year>2020 THEN 'Newer Cars'
		ELSE 'Older Cars'
		END,
		COUNT(*) as total_cars,
		ROUND(avg(price):: numeric,2) as Avg_price
FROM cars
	GROUP BY 
			CASE
		WHEN make_year>2020 THEN 'Newer Cars'
		ELSE 'Older Cars'
		END;
# Mileage Impact
## 12.How does mileage affect car price?
```sql
SELECT * FROM cars; 
SELECT 
	CASE
		WHEN mileage<20000 THEN 'Below 20k Kms'
		WHEN mileage BETWEEN 20000 AND 50000 THEN 'Below 50k Kms'
		WHEN mileage BETWEEN 50001 AND 100000 THEN  'Below 100k Kms'
		ELSE 'Above 100k Kms'
	END AS mileage_group,
	COUNT(*) As total_cars,
	ROUND(avg(price)::numeric,2) as avg_price
FROM cars
GROUP BY 
		CASE
		WHEN mileage<20000 THEN 'Below 20k Kms'
		WHEN mileage BETWEEN 20000 AND 50000 THEN 'Below 50k Kms'
		WHEN mileage BETWEEN 50001 AND 100000 THEN  'Below 100k Kms'
		ELSE 'Above 100k Kms'
	END
ORDER BY 3 DESC;
## 13.	At what mileage range does price drop sharply?
```sql
WITH mileage_buckets AS (
    SELECT
        CASE
            WHEN mileage < 20000 THEN '0–20k'
            WHEN mileage BETWEEN 20000 AND 40000 THEN '20k–40k'
            WHEN mileage BETWEEN 40001 AND 60000 THEN '40k–60k'
            WHEN mileage BETWEEN 60001 AND 80000 THEN '60k–80k'
            WHEN mileage BETWEEN 80001 AND 100000 THEN '80k–1L'
            ELSE 'Above 1L'
        END AS mileage_range,
        AVG(CAST(price AS FLOAT)) AS avg_price
    FROM cars
    GROUP BY
        CASE
            WHEN mileage < 20000 THEN '0–20k'
            WHEN mileage BETWEEN 20000 AND 40000 THEN '20k–40k'
            WHEN mileage BETWEEN 40001 AND 60000 THEN '40k–60k'
            WHEN mileage BETWEEN 60001 AND 80000 THEN '60k–80k'
            WHEN mileage BETWEEN 80001 AND 100000 THEN '80k–1L'
            ELSE 'Above 1L'
        END
)
SELECT *
FROM mileage_buckets
ORDER BY avg_price DESC;

## 14.	Which makers retain higher prices even at high mileage?
```sql
SELECT * FROM (
SELECT maker,
		mileage,
		Avg(price) as Avg_price,
		RANK() OVER(ORDER BY Avg(price) DESC) as rnk
FROM cars
	WHERE mileage>80000
	GROUP BY maker,
			mileage
) 
where rnk<=5;
# Ownership & Condition
## 15.	How does number of owners impact price?
```sql
SELECT no_of_owners,
		ROUND(AVG(price)::numeric,2) as avg_price,
		Count(*) as total_cars
FROM cars
GROUP BY no_of_owners
ORDER BY no_of_owners;
