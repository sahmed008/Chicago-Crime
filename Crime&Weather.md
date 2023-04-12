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





