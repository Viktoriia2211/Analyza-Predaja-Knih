
# Analýza Predaja Kníh – SQL Dotazy

Tento dokument obsahuje príklady SQL dotazov na analýzu trendov cien, predajnosti kníh a úspešnosti autorov.

## 1. Trendy v cenách kníh podľa žánru

### Pozrieme sa na priemerné ceny a kategorizáciu do cenových pásiem

```sql
SELECT
    year,
    genre,
    ROUND(AVG(price), 2) AS avg_price,
    CASE
        WHEN AVG(price) >= 20 THEN 'Premium Segment'
        WHEN AVG(price) BETWEEN 10 AND 19.99 THEN 'Mid Range'
        ELSE 'Budget Segment'
    END AS price_category
FROM books
GROUP BY year, genre
ORDER BY year, genre;
```
### Pozrieme sa, ako sa mení cena oproti predchádzajúcemu roku
```sql
WITH cte AS (
    SELECT
        year,
        genre,
        AVG(price) AS avg_price
    FROM books
    GROUP BY year, genre
)
SELECT
    genre,
    year,
    ROUND(avg_price, 2) AS avg_price,
    ROUND(
        avg_price - LAG(avg_price) OVER (PARTITION BY genre ORDER BY year), 
        2
    ) AS diff_from_previous_year
FROM cte
ORDER BY genre, year;
```
# 2. Predajnosť kníh a ich ceny
### Pozrieme sa, či má cena vplyv na hodnotenie (ratings)
```sql
SELECT
    CASE 
        WHEN price < 10 THEN 'Low Price'
        WHEN price BETWEEN 10 AND 20 THEN 'Medium Price'
        ELSE 'High Price'
    END AS price_segment,
    ROUND(AVG(ratings), 2) AS avg_ratings,
    COUNT(*) AS total_books
FROM books
GROUP BY
    CASE 
        WHEN price < 10 THEN 'Low Price'
        WHEN price BETWEEN 10 AND 20 THEN 'Medium Price'
        ELSE 'High Price'
    END
ORDER BY avg_ratings DESC;
```
### Segmentácia kníh podľa počtu recenzií
```sql
SELECT
    CASE
        WHEN no_of_reviews < 1000 THEN 'Low Review Count'
        WHEN no_of_reviews BETWEEN 1000 AND 5000 THEN 'Medium Review Count'
        ELSE 'High Review Count'
    END AS review_segment,
    ROUND(AVG(price), 2) AS avg_price,
    COUNT(*) AS total_books
FROM books
GROUP BY
    CASE
        WHEN no_of_reviews < 1000 THEN 'Low Review Count'
        WHEN no_of_reviews BETWEEN 1000 AND 5000 THEN 'Medium Review Count'
        ELSE 'High Review Count'
    END
ORDER BY avg_price DESC;
```
### Ktoré knihy boli najpredávanejšie v jednotlivých rokoch?
```sql
WITH cte AS (
    SELECT
        year,
        title,
        no_of_reviews,
        ROW_NUMBER() OVER (PARTITION BY year ORDER BY no_of_reviews DESC) AS rn
    FROM books
)
SELECT
    year,
    title,
    no_of_reviews AS review_count
FROM cte
WHERE rn = 1
ORDER BY year;
```
# 3. Autor a úspešnosť
### Pozrieme sa, ktorí autori majú najväčší počet kníh
```sql
SELECT
    author,
    COUNT(*) AS total_books,
    ROUND(AVG(price), 2) AS avg_price
FROM books
GROUP BY author
ORDER BY total_books DESC
LIMIT 10;
```
### Ktorí autori majú nadpriemernú cenu kníh
```sql
SELECT
    author,
    ROUND(AVG(price), 2) AS avg_author_price
FROM books
GROUP BY author
HAVING AVG(price) > (SELECT AVG(price) FROM books)
ORDER BY avg_author_price DESC;
```
### Pozrieme sa na trend cien pre konkrétnych autorov
```sql
Copy
WITH cte AS (
    SELECT
        author,
        year,
        AVG(price) AS avg_price
    FROM books
    GROUP BY author, year
)
SELECT
    author,
    year,
    ROUND(avg_price, 2) AS avg_price,
    ROUND(
        avg_price - LAG(avg_price) OVER (PARTITION BY author ORDER BY year), 
        2
    ) AS diff_from_previous_year
FROM cte
ORDER BY author, year;
```
# Power BI DASHBOARD
![image](https://github.com/user-attachments/assets/a451112b-78f6-41cb-8ada-eaa11ed7a290)
