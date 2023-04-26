### 1. How many total crimes were reported in 2021?

```sql
SELECT 
COUNT(*) AS total_crimes
FROM ChicagoCrime..chicago_crimes_2021;
```
![image](https://user-images.githubusercontent.com/104872221/230933472-e9705ffb-c235-42f5-b823-4edd3bd29e62.png)

### 2. What is the count of Homicides, Battery and Assaults reported?
```sql
SELECT
 crime_type,
 COUNT(*) AS crime_count
FROM ChicagoCrime..chicago_crimes_2021
WHERE crime_type IN ('homicide', 'battery', 'assault')
GROUP BY crime_type
ORDER BY crime_count DESC;
```
![image](https://user-images.githubusercontent.com/104872221/230935499-923571f5-cc1c-4ce0-a318-dfbf3532fb59.png)

### 3. What are the top ten communities that had the most crimes reported?
```sql
SELECT
 top 10
 a.name,
 a.population,
 a.density,
 COUNT(*) AS crimes_reported
FROM ChicagoCrime..chicago_areas$ AS a
LEFT JOIN ChicagoCrime..chicago_crimes_2021 AS c
ON a.community_area_id = c.community_id
GROUP BY a.name, a.population, a.density
ORDER BY crimes_reported DESC;
```
![image](https://user-images.githubusercontent.com/104872221/230938961-ae6123ae-4b50-4133-ab2f-ce9677865df9.png)

### 4. What are the top ten communities that had the least amount of crimes reported?
```sql
SELECT
 top 10
 a.name,
 a.population,
 a.density,
 COUNT(*) AS crimes_reported
FROM ChicagoCrime..chicago_areas$ AS a
LEFT JOIN ChicagoCrime..chicago_crimes_2021 AS c
ON a.community_area_id = c.community_id
GROUP BY a.name, a.population, a.density
ORDER BY crimes_reported ASC;
```
![image](https://user-images.githubusercontent.com/104872221/230955478-af9d4acf-ec24-4c98-8c16-ecbedfb8bf4f.png)


### 5. What month had the most crimes reported?
```sql
SELECT 
 DATENAME(month, CAST(crime_date AS DATETIME)) AS crime_date,
 COUNT(*) AS total_crimes_reported
FROM ChicagoCrime..chicago_crimes_2021
GROUP BY DATENAME(month, CAST(crime_date AS DATETIME))
ORDER BY total_crimes_reported DESC;
```
![image](https://user-images.githubusercontent.com/104872221/230961524-dc4fe51f-f31c-47c8-8c1c-fecc214bb443.png)

### 6. What month had the most homicides and what was the average and median temperature?

```sql
WITH cte_crimes AS (
SELECT 
 DATENAME(month, CAST(c.crime_date AS DATE)) AS crime_date,
 c.crime_type,
 t.temp_high
FROM ChicagoCrime..chicago_crimes_2021 AS c
INNER JOIN ChicagoCrime..chicago_temp_2021$ AS t
ON t.date = CAST(c.crime_date AS DATE)
)
SELECT 
 crime_date,
 COUNT(*) AS crimes_reported,
 ROUND(AVG(temp_high),1) AS avg_temp
 --PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY temp_high) OVER() AS median_temp
FROM cte_crimes
WHERE crime_type = 'homicide'
GROUP BY crime_date
ORDER BY crime_date;
```
![image](https://user-images.githubusercontent.com/104872221/231579251-d67ab5ae-aac3-4968-84eb-4653fd153add.png)

### 7. What weekday were most crimes committed?

```sql
SELECT 
 DATENAME(weekday, CONVERT(DATE, crime_date)) AS weekdays,
 COUNT(*) AS n_crimes
FROM ChicagoCrime..chicago_crimes_2021
GROUP BY DATENAME(weekday, CONVERT(DATE, crime_date))
ORDER BY n_crimes DESC;
```

![image](https://user-images.githubusercontent.com/104872221/231587306-5e6fc32e-5298-467d-b13a-975b3735b980.png)


### 8. What are the top ten city streets that have had the most reported crimes?

```sql
WITH top10crime_cities AS (
SELECT 
 c.city_block AS street_name,
 COUNT(*) AS n_crimes,
 DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS rnk
FROM ChicagoCrime..chicago_crimes_2021 AS c
GROUP BY city_block)
SELECT 
 *
FROM top10crime_cities
WHERE rnk <=10
ORDER BY rnk;
```

![image](https://user-images.githubusercontent.com/104872221/231833097-6214cd37-177b-4011-88a0-edab52c89f81.png)

### 9. What are the top ten city streets that have had the most homicides including ties?

```sql
WITH most_homicides AS (
SELECT 
city_block,
COUNT(*) n_homicide,
RANK() OVER (ORDER BY COUNT(*) DESC) AS rnk
FROM ChicagoCrime..chicago_crimes_2021
WHERE crime_type = 'homicide'
GROUP BY city_block
)
SELECT 
 *
FROM most_homicides
WHERE rnk <=10;
```

![image](https://user-images.githubusercontent.com/104872221/232100071-029541be-1d5a-487e-87fc-7cd2c4aa49fe.png)

### 10. What are the top ten city streets that have had the most burglaries?

```sql
WITH most_burglaries AS (
SELECT 
city_block,
COUNT(*) n_homicide,
RANK() OVER (ORDER BY COUNT(*) DESC) AS rnk
FROM ChicagoCrime..chicago_crimes_2021
WHERE crime_type = 'burglary'
GROUP BY city_block
)
SELECT 
 *
FROM most_burglaries
WHERE rnk <=10;
```

![image](https://user-images.githubusercontent.com/104872221/232101031-2de0378a-2f08-42b4-ae07-be49a77c0256.png)


### 11. What was the number of reported crimes on the hottest day of the year vs the coldest?

```sql
WITH cold AS (
SELECT 
 t.temp_low AS temperature,
 COUNT(*) n_of_crimes
FROM ChicagoCrime..chicago_crimes_2021 AS c
INNER JOIN ChicagoCrime..chicago_temp_2021$ AS t
ON CONVERT(DATE, t.date) = CONVERT(DATE, c.crime_date)
WHERE t.temp_low = 
	(SELECT MIN(temp_low)
	FROM ChicagoCrime..chicago_temp_2021$)
GROUP BY t.temp_low
),
hot AS (
SELECT 
 t.temp_high AS temperature,
 COUNT(*) n_of_crimes
FROM ChicagoCrime..chicago_crimes_2021 AS c
INNER JOIN ChicagoCrime..chicago_temp_2021$ AS t
ON CONVERT(DATE, t.date) = CONVERT(DATE, c.crime_date)
WHERE t.temp_high IN 
	(SELECT MAX(temp_high)
	FROM ChicagoCrime..chicago_temp_2021$)
GROUP BY t.temp_high
)

SELECT 
 temperature,
 n_of_crimes
FROM hot
UNION
SELECT 
 temperature,
 n_of_crimes
FROM cold;
```

![image](https://user-images.githubusercontent.com/104872221/232149447-5a5650c9-bea3-4cdf-bbc5-700e6286941d.png)

### 12. What is the number and types of reported crimes on Michigan Ave (The Rodeo Drive of the Midwest)?

```sql
SELECT 
crime_type,
city_block,
COUNT(*) AS n_of_crimes
--name,
FROM ChicagoCrime..chicago_crimes_2021 AS c
--INNER JOIN ChicagoCrime..chicago_areas$ AS a
--ON a.community_area_id = c.community_id
WHERE city_block = ' michigan ave'
GROUP BY crime_type, city_block
ORDER BY n_of_crimes DESC;
```

![image](https://user-images.githubusercontent.com/104872221/232227507-da5f1ae1-504d-44fc-9715-b4f33d99f176.png)


### 13. What are the top 5 least reported crime, how many arrests were made and the percentage of arrests made?
```sql
SELECT 
 crime_type,
 n_crimes,
 n_arrests,
 ROUND(CAST(n_arrests AS float)/ n_crimes * 100,2) AS arrest_percent
FROM (
SELECT 
 crime_type,
 COUNT(*) AS n_crimes,
 SUM(
		CASE WHEN arrest = 'true' THEN 1
		ELSE 0
		END) AS n_arrests,
 DENSE_RANK() OVER (ORDER BY COUNT(*) ASC) AS rnk
FROM ChicagoCrime..chicago_crimes_2021
GROUP BY crime_type) as temp
WHERE temp.rnk <5
ORDER BY n_crimes ASC
```

![image](https://user-images.githubusercontent.com/104872221/232237315-cda7eca1-1e48-4eeb-b3a1-6c29cdc79065.png)

### 14. What is the percentage of domestic violence crimes?

```sql
WITH violences AS (
SELECT 
 crime_type,
 SUM(CASE 
	WHEN domestic = 'true' THEN 1 
	ELSE 0
 END) AS domestic_violence,
 SUM(CASE 
	WHEN domestic = 'false' THEN 1 
	ELSE 0
 END) AS non_domestic_violence,
 COUNT(*) AS total_violence
FROM ChicagoCrime..chicago_crimes_2021
GROUP BY crime_type
)
SELECT 
 crime_type,
 domestic_violence,
 non_domestic_violence,
 total_violence,
 ROUND(domestic_violence / CAST(total_violence AS float) * 100,2) AS percent_domestic,
 ROUND(non_domestic_violence / CAST(total_violence AS float) * 100,2) AS percent_non_domestic
FROM violences
ORDER BY total_violence DESC;
```

![image](https://user-images.githubusercontent.com/104872221/232314892-6dc7ef03-34f8-46e7-bd4b-ce8b46c6ac93.png)

### 15. Display how many crimes were reported on a monthly basis in chronological order. What is the month to month percentage change of crimes reported?

```sql
WITH CTE AS (
 SELECT
        DATEPART(MONTH, crime_date) AS crime_month,
        COUNT(*) AS n_crimes
    FROM 
       ChicagoCrime..chicago_crimes_2021
    GROUP BY 
        DATEPART(MONTH, crime_date)
)

SELECT
    crime_month,
    n_crimes,
    ROUND(100 * (n_crimes - 
	LAG(n_crimes) OVER (ORDER BY crime_month)) / CAST(LAG(n_crimes) OVER (ORDER BY crime_month) AS FLOAT), 2) AS month_to_month
FROM
    CTE
ORDER BY
    crime_month;
```

![image](https://user-images.githubusercontent.com/104872221/232320978-641245ff-8832-4fec-9a66-bf28159de7d2.png)


### 16. Display the most consecutive days where a homicide occured and the timeframe.

```sql
WITH CTE AS (
    SELECT CAST(crime_date AS date) AS crime_date, 
           DATEADD(DAY, - ROW_NUMBER() OVER (ORDER BY crime_date), CAST(crime_date AS date)) AS grp,
		   ROW_NUMBER() OVER (ORDER BY crime_date) AS row_numb
    FROM ChicagoCrime..chicago_crimes_2021
	WHERE crime_type = 'homicide'
)
SELECT TOP 5
       MIN(crime_date) AS start_date, 
       MAX(crime_date) AS end_date,
	   DATEDIFF(DAY, MIN(crime_date), MAX(crime_date)) + 1 AS consecutive_days
FROM CTE
GROUP BY grp
ORDER BY consecutive_days DESC
--HAVING DATEDIFF(DAY, MIN(crime_date), MAX(crime_date)) + 1;
```

![image](https://user-images.githubusercontent.com/104872221/232332092-0b1a91e2-36f0-4c30-9e7a-5eac4108cdc7.png)

 

### 17. What are the top 10 most common locations for reported crimes and their frequency depending on the season?

```sql
WITH weather_crimes AS (
SELECT 
crime_location,
SUM(CASE WHEN temp_high >= 41  AND temp_high <= 85 THEN 1 ELSE 0 END) AS 'Spring',
SUM(CASE WHEN temp_high >= 4 AND temp_high <= 40 THEN 1 ELSE 0 END) AS 'Winter',
SUM(CASE WHEN temp_high >=86 THEN 1 ELSE 0 END) AS 'Summer',
COUNT(*) AS total_crimes,
DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) AS rnk
FROM ChicagoCrime..chicago_crimes_2021 AS c
INNER JOIN ChicagoCrime..chicago_temp_2021$ AS t
ON CAST(c.crime_date AS DATE) = CAST(t.date AS DATE)
GROUP BY crime_location
)
SELECT 
 *
FROM weather_crimes
WHERE rnk <= 10;
```

![image](https://user-images.githubusercontent.com/104872221/232335527-e39c738a-29c2-4038-8660-5472465a918a.png)

### 18. What is the Month, day of the week and the number of homicides that occured in a babershop or beauty salon?

```sql
WITH crimes_cte AS (
SELECT 
CAST(crime_date AS DATE) AS crime_date,
crime_type,
crime_location,
city_block
FROM ChicagoCrime..chicago_crimes_2021
WHERE (crime_location LIKE '%shop%' AND crime_location != 'pawn shop')
AND crime_type = 'homicide'
)
SELECT
 DATENAME(month, crime_date) AS crime_month,
 DATENAME(WEEKDAY, crime_date) AS crime_day,
 crime_type,
 crime_location,
 COUNT(*) AS n_homicides
FROM crimes_cte
GROUP BY 
DATENAME(month, crime_date),  
DATENAME(WEEKDAY, crime_date),
crime_type,
crime_location;
```

![image](https://user-images.githubusercontent.com/104872221/232336985-02a1a288-65fd-41c2-8554-a226421bd013.png)

