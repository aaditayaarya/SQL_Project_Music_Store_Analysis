# 🎵 SQL Project: Music Store Analysis

This project focuses on analyzing a digital music store database using SQL. The database includes customers, invoices, artists, genres, tracks, and invoice line items. The goal is to extract insights about customers, genres, and sales.

---

## 📁 Database Tables Used

- `customer`
- `invoice`
- `invoice_line`
- `track`
- `genre`
- `album`
- `artist`
- `album`
- `artist`
- `employee`
- `media_type`
- `playlist`
- `playlist_track`
- `track`

---

## 🧠 Question Set 1 - Easy

### 1. Who is the senior-most employee based on job title?
```sql
SELECT title, last_name, first_name 
FROM employee
ORDER BY levels DESC
LIMIT 1;
```

### 2. Which countries have the most invoices?
```sql
SELECT COUNT(*) AS c, billing_country 
FROM invoice
GROUP BY billing_country
ORDER BY c DESC;
```

### 3. What are the top 3 values of total invoice?
```sql
SELECT total 
FROM invoice
ORDER BY total DESC
LIMIT 3;
```

### 4. Which city has the best customers (highest total invoice amount)?
```sql
SELECT billing_city, SUM(total) AS InvoiceTotal
FROM invoice
GROUP BY billing_city
ORDER BY InvoiceTotal DESC
LIMIT 1;
```

### 5. Who is the best customer (highest spending)?
```sql
SELECT customer.customer_id, first_name, last_name, SUM(total) AS total_spending
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
GROUP BY customer.customer_id
ORDER BY total_spending DESC
LIMIT 1;
```

---

## 🧠 Question Set 2 - Moderate

### 1. Find email, first name, last name & genre of all Rock music listeners (ordered by email).
```sql
SELECT DISTINCT email, first_name, last_name
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
JOIN invoiceline ON invoice.invoice_id = invoiceline.invoice_id
WHERE track_id IN (
	SELECT track_id FROM track
	JOIN genre ON track.genre_id = genre.genre_id
	WHERE genre.name LIKE 'Rock'
)
ORDER BY email;
```

### 2. Top 10 Rock music artists by track count.
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

### 3. Tracks longer than average duration.
```sql
SELECT name, miliseconds
FROM track
WHERE miliseconds > (
	SELECT AVG(miliseconds) FROM track
)
ORDER BY miliseconds DESC;
```

---

## 🧠 Question Set 3 - Advanced

### 1. How much has each customer spent on the top-selling artist?
```sql
WITH best_selling_artist AS (
	SELECT artist.artist_id, artist.name AS artist_name,
	       SUM(invoice_line.unit_price * invoice_line.quantity) AS total_sales
	FROM invoice_line
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN album ON album.album_id = track.album_id
	JOIN artist ON artist.artist_id = album.artist_id
	GROUP BY artist.artist_id
	ORDER BY total_sales DESC
	LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name,
       SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY c.customer_id, c.first_name, c.last_name, bsa.artist_name
ORDER BY amount_spent DESC;
```

### 2. Most popular music genre by country.
```sql
WITH popular_genre AS (
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name,
           ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line 
	JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
	JOIN customer ON customer.customer_id = invoice.customer_id
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN genre ON genre.genre_id = track.genre_id
	GROUP BY customer.country, genre.name
)
SELECT * FROM popular_genre WHERE RowNo = 1;
```

### 3. Top spending customer by country.
```sql
WITH Customer_with_country AS (
	SELECT customer.customer_id, first_name, last_name, billing_country,
	       SUM(total) AS total_spending,
	       ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
	FROM invoice
	JOIN customer ON customer.customer_id = invoice.customer_id
	GROUP BY customer.customer_id, first_name, last_name, billing_country
)
SELECT * FROM Customer_with_country WHERE RowNo = 1;
```

---

## 🏁 Conclusion

This project uses SQL to analyze a music store database to:

- Identify customer behavior
- Find revenue insights
- Understand genre popularity across countries

It provides a foundation for business decisions like customer rewards, marketing promotions, and content targeting.
