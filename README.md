# 🎵 Music Store Data Analysis 🎶

**SQL Author**: Arjun Rajput  
**File name**: `music_store_analysis.sql`  
**Project**: Analyze music store sales and customer data to uncover valuable insights for enhancing business strategies. This project focuses on understanding **customer preferences**, **sales trends**, and optimizing **marketing efforts**.

---

## 📝 Project Overview

This project delves into:

- 🎤 **Artist Popularity**
- 💸 **Revenue Generation**
- 🌍 **Geographic Insights**
- 🎶 **Track and Genre Preferences**

---

## 🛠️ **Schema Diagram Overview**

Below is the schema diagram that links **customers**, **invoices**, **tracks**, **artists**, **albums**, and **genres**. This structure allows complex queries to analyze music store sales and customer behaviors.

<img width="600" alt="schema_diagram" src="https://github.com/user-attachments/assets/3d2cd16e-5c6d-48bf-bdc9-b70524a06370">

Key entities include:

- **Customer** 🧍‍♂️🧍‍♀️: Personal details, contact information, and invoices.
- **Employee** 🧑‍💼: Includes hierarchy with `reports_to` and job titles.
- **Track** 🎵: Music track information (album, media type, genre, length).
- **Album** 💿: Each track is associated with an album and artist.
- **Invoice** 🧾: Sales details including billing information and total cost.
- **Genre** 🎸: Categories of music tracks (e.g., Rock, Pop).

---

## 📊 SQL Queries

### 1️⃣ **Who holds the highest-ranking position based on job title?**
```sql
SELECT title, last_name, first_name 
FROM employee
ORDER BY levels DESC
LIMIT 1;
```

### 2️⃣ **Which countries generate the highest number of invoices?**
```sql
SELECT COUNT(*) AS c, billing_country 
FROM invoice
GROUP BY billing_country
ORDER BY c DESC;
```

### 3️⃣ **Top 3 total invoice values**
```sql
SELECT total 
FROM invoice
ORDER BY total DESC
LIMIT 3;
```

### 4️⃣ **Which city generates the highest revenue?**
We plan to host a **Music Festival** in this city 🎉.
```sql
SELECT billing_city, SUM(total) AS InvoiceTotal
FROM invoice
GROUP BY billing_city
ORDER BY InvoiceTotal DESC
LIMIT 1;
```

### 5️⃣ **Who is the best customer by total expenditure?**
```sql
SELECT customer.customer_id, first_name, last_name, SUM(total) AS total_spending
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
GROUP BY customer.customer_id
ORDER BY total_spending DESC
LIMIT 1;
```

### 6️⃣ **Who are the Rock music lovers?**
```sql
SELECT DISTINCT email, first_name, last_name
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
JOIN invoice_line ON invoice.invoice_id = invoice_line.invoice_id
WHERE track_id IN (
    SELECT track_id FROM track
    JOIN genre ON track.genre_id = genre.genre_id
    WHERE genre.name LIKE 'Rock'
)
ORDER BY email;
```

### 7️⃣ **Top 10 rock artists by number of tracks**
Let's invite them to our Music Festival! 🎸🤘
```sql
SELECT artist.artist_id, artist.name, COUNT(artist.artist_id) AS number_of_songs
FROM track
JOIN album ON album.album_id = track.album_id
JOIN artist ON artist.artist_id = album.artist_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id
ORDER BY number_of_songs DESC
LIMIT 10;
```

### 8️⃣ **Tracks longer than the average duration**
```sql
SELECT name, milliseconds
FROM track
WHERE milliseconds > (
    SELECT AVG(milliseconds) AS avg_track_length
    FROM track
)
ORDER BY milliseconds DESC;
```

### 9️⃣ **Total amount spent by each customer on the best-selling artist**
```sql
WITH best_selling_artist AS (
    SELECT artist.artist_id AS artist_id, artist.name AS artist_name, SUM(invoice_line.unit_price * invoice_line.quantity) AS total_sales
    FROM invoice_line
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN album ON album.album_id = track.album_id
    JOIN artist ON artist.artist_id = album.artist_id
    GROUP BY artist.artist_id
    ORDER BY total_sales DESC
    LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY c.customer_id, c.first_name, c.last_name, bsa.artist_name
ORDER BY amount_spent DESC;
```

### 🔟 **Most popular music genre by country**
```sql
WITH popular_genre AS (
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
    ROW_NUMBER() OVER (PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line
    JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
    JOIN customer ON customer.customer_id = invoice.customer_id
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN genre ON genre.genre_id = track.genre_id
    GROUP BY customer.country, genre.name, genre.genre_id
)
SELECT * FROM popular_genre WHERE RowNo <= 1;
```

### 1️⃣1️⃣ **Top customer by total spending in each country**
```sql
WITH Customer_with_country AS (
    SELECT customer.customer_id, first_name, last_name, billing_country, SUM(total) AS total_spending,
    ROW_NUMBER() OVER (PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
    FROM invoice
    JOIN customer ON customer.customer_id = invoice.customer_id
    GROUP BY customer.customer_id, first_name, last_name, billing_country
)
SELECT * FROM Customer_with_country WHERE RowNo <= 1;
```

---

