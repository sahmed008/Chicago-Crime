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



