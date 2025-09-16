## 🎵 Music Store Database Analysis (SQL Project)

<!-- Table of Contents with HTML anchors -->
## 📑 Table of Contents
- <a href="#project-overview">Project Overview</a>  
- <a href="#tools--skills-used">Tools & Skills Used</a>  
- <a href="#set-1-sales--customers">Set 1: Sales & Customers</a>  
  - <a href="#set1-q1">Q1 - Senior-most Employee</a>  
  - <a href="#set1-q2">Q2 - Countries with Most Invoices</a>  
  - <a href="#set1-q3">Q3 - Top 3 Invoice Values</a>  
  - <a href="#set1-q4">Q4 - City with Highest Invoice Total</a>  
  - <a href="#set1-q5">Q5 - Best Customer (Highest Spender)</a>  
- <a href="#set-2-music-trends">Set 2: Music Trends</a>  
  - <a href="#set2-q1">Q1 - Rock Music Listeners</a>  
  - <a href="#set2-q2">Q2 - Top 10 Rock Artists</a>  
  - <a href="#set2-q3">Q3 - Tracks Longer than Average</a>  
- <a href="#set-3-advanced-insights">Set 3: Advanced Insights</a>  
  - <a href="#set3-q1">Q1 - Amount Spent by Each Customer on Best-selling Artist</a>  
  - <a href="#set3-q2">Q2 - Most Popular Genre per Country</a>  
  - <a href="#set3-q3">Q3 - Highest-spending Customer per Country</a>  
- <a href="#key-insights">Key Insights</a>  
- <a href="#conclusion">Conclusion</a>  
- <a href="#project-structure">Suggested Project Folder Structure</a>  
- <a href="Author--Contact">Set 3:Author & Contact</a>

---

<a id="project-overview"></a>
## 📌 Project Overview
This project analyzes a **Music Store Database** using **PostgreSQL** (PgAdmin4).  
Goal: answer real-world business questions related to customers, sales, invoices, artists, and genres using SQL queries and present actionable insights.

---

<a id="tools--skills-used"></a>
## 🛠 Tools & Skills Used
- **Database**: PostgreSQL (PgAdmin4)  
- **SQL Concepts**:
  - Joins (INNER, LEFT, etc.)  
  - Aggregate Functions: `SUM`, `COUNT`, `MAX`, `AVG`  
  - `GROUP BY` & `HAVING`  
  - Subqueries (scalar & IN-lists)  
  - Sorting (`ORDER BY`)  
  - Window Functions (`ROW_NUMBER()`, `PARTITION BY`)  
  - CTEs (`WITH`, including recursive when needed)

---

<a id="set-1-sales--customers"></a>
**✅ Set 1: Sales & Customers**

<a id="set1-q1"></a>
**Q1. Who is the senior-most employee based on job title?**  
```sql
SELECT *
FROM employee
ORDER BY levels DESC
LIMIT 1;

<a id="set1-q2"></a>
**Q2. Which countries have the most invoices?**
```
SELECT COUNT(*) AS total_invoices, billing_country
FROM invoice
GROUP BY billing_country
ORDER BY total_invoices DESC;
```

<a id="set1-q3"></a>
**Q3. What are the top 3 values of total invoice?**
```
SELECT total
FROM invoice
ORDER BY total DESC
LIMIT 3;
```

<a id="set1-q4"></a>
**Q4. Which city has the best customers (highest invoice total)?**
```
SELECT billing_city, SUM(total) AS invoice_total
FROM invoice
GROUP BY billing_city
ORDER BY invoice_total DESC;
```

**✅ Example Answer: Prague**

<a id="set1-q5"></a>
**Q5. Who is the best customer (highest spender)?**
```
SELECT c.customer_id, c.first_name, c.last_name,
       SUM(i.total) AS total_spent
FROM customer c
JOIN invoice i ON c.customer_id = i.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC
LIMIT 1;
```

---

<a id="set-2-music-trends"></a>
**✅ Set 2: Music Trends**

<a id="set2-q1"></a>
**Q1. Rock Music listeners (email, first name, last name) ordered by email.**
```
SELECT DISTINCT c.email, c.first_name, c.last_name
FROM customer c
JOIN invoice i ON c.customer_id = i.customer_id
JOIN invoice_line il ON i.invoice_id = il.invoice_id
WHERE il.track_id IN (
    SELECT t.track_id
    FROM track t
    JOIN genre g ON t.genre_id = g.genre_id
    WHERE g.name LIKE 'Rock'
)
ORDER BY c.email;
```

<a id="set2-q2"></a>
**Q2. Top 10 artists who wrote the most Rock tracks.**
```
SELECT a.artist_id, a.name, COUNT(t.track_id) AS number_of_songs
FROM artist a
JOIN album al ON a.artist_id = al.artist_id
JOIN track t ON al.album_id = t.album_id
JOIN genre g ON t.genre_id = g.genre_id
WHERE g.name LIKE 'Rock'
GROUP BY a.artist_id, a.name
ORDER BY number_of_songs DESC
LIMIT 10;
```

<a id="set2-q3"></a>
**Q3. Tracks longer than the average song length.**
```
SELECT name, milliseconds
FROM track
WHERE milliseconds > (
    SELECT AVG(milliseconds)
    FROM track
)
ORDER BY milliseconds DESC
LIMIT 10;
```

---

<a id="set-3-advanced-insights"></a>
**✅ Set 3: Advanced Insights**

<a id="set3-q1"></a>
**Q1. Amount spent by each customer on the best-selling artist.**
```
WITH best_selling_artist AS (
    SELECT a.artist_id, a.name AS artist_name,
           SUM(il.unit_price * il.quantity) AS total_sales
    FROM invoice_line il
    JOIN track t ON il.track_id = t.track_id
    JOIN album al ON t.album_id = al.album_id
    JOIN artist a ON a.artist_id = al.artist_id
    GROUP BY a.artist_id, a.name
    ORDER BY total_sales DESC
    LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name,
       SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album al ON al.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = al.artist_id
GROUP BY c.customer_id, c.first_name, c.last_name, bsa.artist_name
ORDER BY amount_spent DESC;
```

<a id="set3-q2"></a>
**Q2. Most popular music genre in each country.**
```
WITH genre_country_counts AS (
    SELECT c.country, g.genre_id, g.name,
           COUNT(il.invoice_line_id) AS purchases,
           ROW_NUMBER() OVER (PARTITION BY c.country ORDER BY COUNT(il.invoice_line_id) DESC) AS rn
    FROM invoice_line il
    JOIN invoice i ON il.invoice_id = i.invoice_id
    JOIN customer c ON i.customer_id = c.customer_id
    JOIN track t ON il.track_id = t.track_id
    JOIN genre g ON t.genre_id = g.genre_id
    GROUP BY c.country, g.genre_id, g.name
)
SELECT country, genre_id, name AS top_genre, purchases
FROM genre_country_counts
WHERE rn = 1
ORDER BY country;
```

<a id="set3-q3"></a>
**Q3. Highest-spending customer per country.**

Method 1: Using MAX aggregate
```
WITH customer_spend AS (
    SELECT c.customer_id, c.first_name, c.last_name, i.billing_country,
           SUM(i.total) AS total_spending
    FROM invoice i
    JOIN customer c ON c.customer_id = i.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name, i.billing_country
),
country_max AS (
    SELECT billing_country, MAX(total_spending) AS max_spending
    FROM customer_spend
    GROUP BY billing_country
)
SELECT cs.billing_country, cs.customer_id, cs.first_name, cs.last_name, cs.total_spending
FROM customer_spend cs
JOIN country_max cm ON cs.billing_country = cm.billing_country
WHERE cs.total_spending = cm.max_spending
ORDER BY cs.billing_country;
```

Method 2: Using ROW_NUMBER()
```
WITH customer_spend_ranked AS (
    SELECT c.customer_id, c.first_name, c.last_name, i.billing_country,
           SUM(i.total) AS total_spending,
           ROW_NUMBER() OVER (PARTITION BY i.billing_country ORDER BY SUM(i.total) DESC) AS rn
    FROM invoice i
    JOIN customer c ON c.customer_id = i.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name, i.billing_country
)
SELECT billing_country, customer_id, first_name, last_name, total_spending
FROM customer_spend_ranked
WHERE rn = 1
ORDER BY billing_country;
```

---

<a id="key-insights"></a>
##📊 Key Insights

- Identified the best customer by total spend.

- Found the most profitable city for promotions.

- Determined most popular genres per country.

- Ranked artists and tracks by sales and engagement.

---

<a id="conclusion"></a>
##🚀 Conclusion
This project demonstrates how SQL can be used to extract actionable insights from a relational database.
It highlights advanced querying skills like joins, aggregates, CTEs, and window functions to solve real-world business problems.

---

<a id="project-structure"></a>
## 📂 Project Structure

The project is organized as follows:

```
Music_Store_Database/
├── Outputs/ # Contains query results (CSV files, screenshots, etc.)
│ ├── Set1/ # Results related to Sales & Customers queries
│ ├── Set2/ # Results related to Music Trends queries
│ └── Set3/ # Results related to Advanced Insights queries
│
├── Questions/ # Contains SQL query files for each set
│ ├── set1.sql # Queries for Sales & Customers
│ ├── set2.sql # Queries for Music Trends
│ └── set3.sql # Queries for Advanced Insights
│
├── .gitignore # Git ignore file to exclude unnecessary files
├── README.md # Documentation file (project overview, queries, insights)
```

---

<a id="Author--Contact"></a>
## Author & Contact
**Chirag Tomar**

Data Analyst

📧 Email: tomarchirag431@gmail.com

🔗 [LinkedIn](https://www.linkedin.com/in/chirag-tomar-28a522376/)

🔗 [LeetCode](https://leetcode.com/u/codewithchirag18/)