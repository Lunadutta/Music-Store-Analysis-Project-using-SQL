# Music-Store-Analysis-Project-using-SQL


/*	Question Set 1 - Easy */

/* Q1: Who is the senior most employee based on job title? */
select * from employee order by levels desc limit 1;

/* Q2: Which countries have the most Invoices? */
select billing_country, count(Invoice_id) from invoice group by billing_country order by 
count(Invoice_id ) desc limit 1;

/* Q3: What are top 3 values of total invoice? */
select * from invoice order by total desc limit 3;

/* Q4: Which city has the best customers? We would like to throw a promotional Music Festival 
in the city we made the most money.*/
select billing_city , sum(total) as total_price from invoice group by billing_city order by 
sum(total) desc limit 1;

/* Q5: Who is the best customer? The customer who has spent the most money will be declared 
the best customer. 
Write a query that returns the person who has spent the most money.*/
select invoice.customer_id,first_name,last_name,sum(total) as total_price from invoice full outer join 
customer on invoice.customer_id = customer.customer_id group by invoice.customer_id ,first_name,last_name
order by sum(total) desc limit 1;


/* Question Set 2 - Moderate */

/* Q1: Write query to return the email, first name, last name, & Genre of all Rock Music listeners.
Return your list ordered alphabetically by email starting with A. */

select first_name ,last_name,email from customer join invoice on
customer.customer_id = invoice.customer_id
join invoice_line on invoice.invoice_Id =invoice_line.invoice_Id 
join Track on Track.Track_Id=invoice_line.Track_id join Genre on Track.genre_Id= Genre.genre_id
where Genre.name = 'Rock' order by customer.email ;

/* Q2: Let's invite the artists who have written the most rock music in our dataset. 
Write a query that returns the Artist name and total track count of the top 10 rock bands. */

select artist.artist_id,artist.name ,COUNT(artist.artist_id) AS number_of_songs 
from  artist
join  album on artist.artist_id = album.artist_id 
join Track on  album.album_id = Track.album_id 
join genre on Genre.genre_id =  Track.genre_id 
where Genre.name like 'Rock' 
group by artist.artist_id order by number_of_songs desc limit 10;

/* Q3: Return all the track names that have a song length longer than the average song length. 
Return the Name and Milliseconds for each track. Order by the song length with 
the longest songs listed first. */

select name , milliseconds from track where milliseconds > (select  (sum(milliseconds)/count(*) )as average_length 
from track ) order by  milliseconds desc ;

/* Question Set 3 - Advance */

/* Q1: Find how much amount spent by each customer on artists? Write a query to return customer name, artist name and total spent */

WITH best_selling_artist AS (
	SELECT artist.artist_id AS artist_id, artist.name AS artist_name, SUM(invoice_line.unit_price*invoice_line.quantity) AS total_sales
	FROM invoice_line
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN album ON album.album_id = track.album_id
	JOIN artist ON artist.artist_id = album.artist_id
	GROUP BY 1
	ORDER BY 3 DESC
	LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, SUM(il.unit_price*il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY 1,2,3,4
ORDER BY 5 DESC;


/* Q2: We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre 
with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where 
the maximum number of purchases is shared return all Genres. */

WITH popular_genre AS 
(
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
	ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line 
	JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
	JOIN customer ON customer.customer_id = invoice.customer_id
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN genre ON genre.genre_id = track.genre_id
	GROUP BY 2,3,4
	ORDER BY 2 ASC, 1 DESC
)
SELECT * FROM popular_genre WHERE RowNo <= 1

/* Q3: Write a query that determines the customer that has spent the most on music for each country. 
Write a query that returns the country along with the top customer and how much they spent. 
For countries where the top amount spent is shared, provide all customers who spent this amount. */


WITH Customter_with_country AS (
		SELECT customer.customer_id,first_name,last_name,billing_country,SUM(total) AS total_spending,
	    ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
		FROM invoice
		JOIN customer ON customer.customer_id = invoice.customer_id
		GROUP BY 1,2,3,4
		ORDER BY 4 ASC,5 DESC)
SELECT * FROM Customter_with_country WHERE RowNo <= 1


