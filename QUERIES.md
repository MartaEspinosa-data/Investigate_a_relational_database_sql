# Set #1

## Q1 - What is the number of movies in each family-friendly category?

```sql
SELECT
  f.title AS film_title,
  c.name AS category_name,
  COUNT(rental_duration) AS rental_count
FROM category c
JOIN film_category fc
  ON c.category_id = fc.category_id
JOIN film f
  ON f.film_id = fc.film_id
JOIN inventory i
  ON i.film_id = f.film_id
JOIN rental r
  ON i.inventory_id = r.inventory_id
WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
GROUP BY 1,
         2
ORDER BY 2;
```

## Q2 - how the length of rental duration of these family-friendly movies compares to the duration that all movies are rented for? 

``` sql
SELECT
  f.title,
  c.name AS name,
  f.rental_duration,
  NTILE(4) OVER (ORDER BY f.rental_duration) AS standard_quartile
FROM category c
JOIN film_category fc
  ON c.category_id = fc.category_id
JOIN film f
  ON f.film_id = fc.film_id
WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music');
```

## Q3 - Provide a table with the family-friendly film category, each of the quartiles, and the corresponding count of movies within each combination of film category for each corresponding rental duration category 

``` sql
SELECT
  name,
  standard_quartile,
  COUNT(*)
FROM (SELECT
  c.name AS name,
  f.rental_duration,
  NTILE(4) OVER (ORDER BY f.rental_duration) AS standard_quartile
FROM category c
JOIN film_category fc
  ON fc.category_id = c.category_id
JOIN film f
  ON fc.film_id = f.film_id
WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')) t1
GROUP BY 1,
         2
ORDER BY 1, 2;
```

## Q4 how the two stores compare in their count of rental orders during every month for all the years we have data for */

```sql
SELECT
  DATE_PART('month', rental.rental_date) AS Rental_month,
  DATE_PART('year', rental.rental_date) AS Rental_year,
  staff.store_id,
  COUNT(rental.rental_id) AS rental_counts
FROM rental
JOIN staff
  ON rental.staff_id = staff.staff_id
GROUP BY staff.store_id,
         Rental_month,
         Rental_year
ORDER BY rental_counts DESC;
```



## Q5 who were our top 10 paying customers, how many payments they made on a monthly basis during 2007, and what was the amount of the monthly payments 

```sql
WITH top_10_customers AS 
     (SELECT c.customer_id, SUM(p.amount) AS payments
FROM customer c
JOIN payment p
  ON p.customer_id = c.customer_id
GROUP BY c.customer_id
ORDER BY payments DESC
LIMIT 10)

SELECT
  DATE_TRUNC('month', payment_date) AS pay_month,
  CONCAT(c.first_name, ' ', c.last_name) AS full_name,
  COUNT(p.amount) AS pay_countpermon,
  SUM(p.amount) AS pay_amount
FROM top_10_customers
JOIN customer c
  ON top_10_customers.customer_id = c.customer_id
JOIN payment p
  ON p.customer_id = top_10_customers.customer_id
WHERE payment_date >= '2007-01-01'
AND payment_date <= '2008-01-01'
GROUP BY pay_month,
         full_name
ORDER BY full_name, pay_month;
```

##  Q6 For each of these top 10 paying customers, find out the difference across their monthly payments during 2007. Compare the payment amounts in each successive month 

```sql 
WITH top_10_customers AS 
     (SELECT c.customer_id, 
      SUM(p.amount) AS payments
FROM customer c
JOIN payment p
ON p.customer_id = c.customer_id
GROUP BY c.customer_id
ORDER BY payments DESC
LIMIT 10)

SELECT
  DATE_TRUNC('month', p.payment_date) pay_month,
  CONCAT(c.first_name, ' ', c.last_name) AS full_name,
  COUNT(DATE_TRUNC('month', p.payment_date)) AS pay_count_per_mon,
  SUM(p.amount),
  SUM(p.amount) - LAG(SUM(p.amount)) OVER (PARTITION BY CONCAT(c.first_name, ' ', c.last_name)
  ORDER BY DATE_TRUNC('month', p.payment_date)) AS Difference
FROM payment p
JOIN top_10_customers t
  ON p.customer_id = t.customer_id
JOIN customer c
  ON c.customer_id = t.customer_id
GROUP BY pay_month,
         full_name; 
```




