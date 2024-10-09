# SQLProject
SQL PROJECT- MUSIC STORE DATA ANALYSIS
--1. Who is the senior most employee based on job title?
SELECT employee_id, first_name, last_name, title, levels
FROM employee
ORDER BY levels desc

--2. Which countries have the most Invoices?
SELECT COUNT(*) as total_count, billing_country
from invoice
group by billing_country
order by total_count desc

--What are the top 3 values of total invoice?
select total 
from invoice
order by total desc
limit 3

/*Which city has the best customers? We would like to throw a promotional Music
Festivals in the city made the most money. Write a query that returns one city that
has the highest sum of invoice totals. Return both the city name & sum of all invoice
totals */

select * from invoice

select billing_city, sum(total) as highest_totals
from invoice
group by billing_city
order by highest_totals desc

/*Who is the best customer? The customer who has spent the most money will be
declared the best customer. Write a query that returns the person who has spent the
most money */

select *from invoice
select *from customer

select customer.customer_id, customer.first_name, customer.last_name, SUM(invoice.total) as total_spent
from customer
join invoice ON customer.customer_id = invoice.customer_id
group by customer.customer_id
order by total_spent desc
limit 1


/*Write query to return the email, first name, last name, & Genre of all Rock Music
listeners. Return your list ordered alphabetically by email starting with A */


select * from genre
select * from customer

select customer.first_name as fn, customer.last_name as EN,  customer.email as em
from customer
join invoice ON customer.customer_id = invoice.customer_id
join invoice_line ON invoice.invoice_id = invoice_line.invoice_id
join track ON invoice_line.track_id = track.track_id
join genre ON genre.genre_id = track.genre_id
WHERE genre.name = 'Rock'
ORDER BY em asc

/*Let's invite the artists who have written the most rock music in our dataset. Write a
query that returns the Artist name and total track count of the top 10 rock bands */

select *from artist

select *from album
select *from media_type

select artist.artist_id, artist.name 
from artist
join album ON artist.artist_id = album.artist_id
join track ON album.album_id = track.album_id
join genre ON track.genre_id = genre.genre_id
where genre.name = 'Rock'
group by artist.artist_id
order by artist.artist_id desc
limit 10


/*Return all the track names that have a song length longer than the average song length.
Return the Name and Milliseconds for each track. Order by the song length with the
longest songs listed first */

select * from track

select milliseconds, name
from track
where milliseconds > (select avg(milliseconds) from track)
order by milliseconds desc


/*Find how much amount is spent by each customer on artists? Write a query to return
customer name, artist name and total spent */

select * from customer
select * from artist
select * from invoice_line

select customer.customer_id, customer.first_name, customer.last_name, artist.name , Sum(invoice_line.unit_price * invoice_line.quantity) as total_spent

from invoice_line
join invoice ON invoice.invoice_id = invoice_line.invoice_id
join customer ON customer.customer_id = invoice.customer_id
join track ON track.track_id = invoice_line.track_id
join album ON   album.album_id = track.album_id
join artist ON artist.artist_id = album.artist_id
Group by customer.customer_id, customer.first_name, customer.last_name, artist.name
order by total_spent desc


-- cte function

WITH best_selling_artist AS (
     select artist.artist_id AS artist_id, artist.name as artist_name,
	 sum(invoice_line.unit_price*invoice_line.quantity) as total_sales
	 FROM invoice_line
	 join track ON track.track_id = invoice_line.track_id
	 join album ON album.album_id = track.album_id
	 join artist ON artist.artist_id = album.artist_id
	 GROUP BY 1
	 ORDER BY 3 DESC
	
	 	 
)

select c.customer_id, c.first_name, c.last_name, bsa.artist_name,
sum (il.unit_price*il.quantity) as amount_spent
from invoice i
join customer c ON c.customer_id = i.customer_id
join invoice_line il ON il.invoice_id = i.invoice_id
join track t ON t.track_id = il.track_id
join album alb ON alb.album_id = t.album_id
join best_selling_artist bsa ON bsa.artist_id = alb.artist_id
group by 1,2,3,4
order by 5 desc

/*We want to find out the most popular music Genre for each country. We determine the
most popular genre as the genre with the highest amount of purchases. Write a query
that returns each country along with the top Genre. For countries where the maximum
number of purchases is shared return all Genres */

WITH popular_genre AS
(

   select count(invoice_line.quantity) as purchases, customer.country, genre.name, genre.genre_id,
   ROW_NUMBER () OVER(PARTITION BY customer.country order by count(invoice_line.quantity) desc) as RowNo
   FROM invoice_line
   join invoice on invoice.invoice_id = invoice_line.invoice_id
   join customer on customer.customer_id = invoice.customer_id
   join track on track.track_id = invoice_line.track_id
   join genre on genre.genre_id = track.genre_id
   group by 2,3,4
   order by 2 asc, 1 desc
   

) 
select *from popular_genre where RowNo <= 1


/*Write a query that determines the customer that has spent the most on music for each
country. Write a query that returns the country along with the top customer and how
much they spent. For countries where the top amount spent is shared, provide all
customers who spent this amount */

with recursive 
     customer_with_country as (
         select customer.customer_id, first_name, last_name, billing_country, SUM(total) as total_spending
		 from invoice
		 join customer on customer.customer_id = invoice.customer_id
		 group by 1,2,3,4
		 order by 2,3 desc),

		 country_max_spending as(
                 select billing_country, max(total_spending) as max_spending
				 from customer_with_country
				 group by billing_country
				 )

select cc.billing_country, cc.total_spending, cc.first_name, cc.last_name, cc.customer_id
from customer_with_country cc
join country_max_spending ms
on cc.billing_country = ms.billing_country
where cc.total_spending = ms.max_spending
order by 1;


	 


