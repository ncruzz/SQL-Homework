# Unit 10 Assignment - SQL

### Create these queries to develop greater fluency in SQL, an important database language.

-- 1a. Display the first and last names of all actors from the table actor.

USE sakila;

SELECT first_name, last_name
FROM actor;

-- 1b. Display the first and last name of each actor in a single column in upper case letters. Name the column Actor Name.

SELECT UPPER(CONCAT_WS(' ', first_name, last_name)) AS "Actor Name"
FROM actor;

-- 2a. You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." What is one query would you use to obtain this information?

SELECT actor_id, first_name, last_name
FROM actor
WHERE first_name = "JOE";

-- 2b. Find all actors whose last name contain the letters GEN:

SELECT * FROM actor
WHERE last_name LIKE "%GEN%";

-- 2c. Find all actors whose last names contain the letters LI. This time, order the rows by last name and first name, in that order:

SELECT * FROM actor
WHERE last_name LIKE "%LI%"
ORDER BY last_name, first_name;

-- 2d. Using IN, display the country_id and country columns of the following countries: Afghanistan, Bangladesh, and China:

SELECT country_id, country 
FROM country
WHERE country IN ("Afghanistan", "Bangladesh", "China");

-- 3a. You want to keep a description of each actor. You don't think you will be performing queries on a description, so create a column in the table actor named description and use the data type BLOB (Make sure to research the type BLOB, as the difference between it and VARCHAR are significant).

SELECT * FROM actor;

ALTER TABLE actor 
ADD description BLOB;

-- 3b. Very quickly you realize that entering descriptions for each actor is too much effort. Delete the description column.

ALTER TABLE actor 
DROP COLUMN description;

-- 4a. List the last names of actors, as well as how many actors have that last name.

SELECT last_name, COUNT(*) AS 'Last Name Count'
FROM actor 
GROUP BY last_name;

-- 4b. List last names of actors and the number of actors who have that last name, but only for names that are shared by at least two actors.

SELECT last_name, COUNT(*) AS 'Last Name Count'
FROM actor 
GROUP BY last_name
HAVING COUNT(*) > 1;

-- 4c. The actor HARPO WILLIAMS was accidentally entered in the actor table as GROUCHO WILLIAMS. Write a query to fix the record.

UPDATE actor
SET first_name = "Harpo"
WHERE first_name = "GROUCHO" AND last_name = "WILLIAMS";

-- 4d. Perhaps we were too hasty in changing GROUCHO to HARPO. It turns out that GROUCHO was the correct name after all! In a single query, if the first name of the actor is currently HARPO, change it to GROUCHO.

UPDATE actor
SET first_name = "GROUCHO"
WHERE first_name = "HARPO" AND last_name = "WILLIAMS";

-- 5a. You cannot locate the schema of the address table. Which query would you use to re-create it?

USE sakila;

CREATE TABLE re_address(
	address_id INT AUTO_INCREMENT PRIMARY KEY,
	address VARCHAR(50) NOT NULL,
	address2 VARCHAR(50),
	district VARCHAR(20) NOT NULL,
	city_id SMALLINT(5) UNSIGNED NOT NULL,
	postal_code VARCHAR(10),
	phone VARCHAR(20) NOT NULL,
	location GEOMETRY NOT NULL,
	last_update TIMESTAMP NOT NULL

);

-- 6a. Use JOIN to display the first and last names, as well as the address, of each staff member. Use the tables staff and address:

SELECT s.first_name, s.last_name, a.address
FROM staff AS s
INNER JOIN address AS a
ON a.address_id = s.address_id;

-- 6b. Use JOIN to display the total amount rung up by each staff member in August of 2005. Use tables staff and payment

SELECT CONCAT(s.first_name, ' ' , s.last_name) AS 'Employee Name', sum(p.amount) AS 'Total'
FROM payment AS p
JOIN staff AS s
ON p.staff_id = s.staff_id
WHERE payment_date like "%2005-08%"
GROUP BY p.staff_id;

-- 6c. List each film and the number of actors who are listed for that film. Use tables film_actor and film. Use inner join.

SELECT f.title AS 'Film Title', COUNT(a.actor_id) AS 'NUMBER OF ACTORS'
FROM film AS f
JOIN film_actor AS a
ON f.film_id = a.film_id
GROUP BY f.title;

-- 6d. How many copies of the film Hunchback Impossible exist in the inventory system?
   SELECT COUNT(*) AS 'Copies of Hunchback Impossible'
   FROM inventory
   WHERE film_id IN
      (
       SELECT film_id
       FROM film
       WHERE title = 'Hunchback Impossible'
      );
      
-- 6e. Using the tables payment and customer and the JOIN command, list the total paid by each customer. List the customers alphabetically by last name

SELECT first_name, last_name, SUM(amount)
FROM payment AS p
JOIN customer AS C
on p.customer_id = c.customer_id
GROUP BY p.customer_id
ORDER BY last_name;

-- 7a. The music of Queen and Kris Kristofferson have seen an unlikely resurgence. As an unintended consequence, films starting with the letters K and Q have also soared in popularity. Use subqueries to display the titles of movies starting with the letters K and Q whose language is English.

SELECT f.title
FROM film AS f
JOIN language AS l
ON f.language_id = l.language_id
WHERE f.title LIKE 'K%' OR 'Q%' AND l.name = 'English';

-- 7b. Use subqueries to display all actors who appear in the film Alone Trip.

SELECT last_name, first_name
FROM actor
WHERE actor_id IN
	(SELECT actor_id FROM film_actor
	WHERE film_id IN
		(SELECT film_id FROM film
		WHERE title = "Alone Trip"
		)
	);

-- 7c. You want to run an email marketing campaign in Canada, for which you will need the names and email addresses of all Canadian customers. Use joins to retrieve this information.

SELECT CONCAT_WS(' ', c.first_name, c.last_name) AS 'Name', c.email AS 'E-mail Address'
FROM customer AS c
JOIN address AS a 
ON c.address_id = a.address_id
JOIN city AS ci 
ON a.city_id = ci.city_id
JOIN country AS co 
ON co.country_id = ci.country_id
WHERE co.country = 'Canada';

-- 7d. Sales have been lagging among young families, and you wish to target all family movies for a promotion. Identify all movies categorized as family films.

SELECT f.title AS 'Movie Title', c.name AS Category
FROM film AS f
JOIN film_category AS fc
ON fc.film_id = f.film_id
JOIN category AS c
ON c.category_id = fc.category_id
WHERE c.name = 'Family';

-- 7e. Display the most frequently rented movies in descending order.

SELECT f.title AS 'Movie Title', count(r.rental_date) AS '# of times rented'
FROM film AS f
JOIN inventory AS i 
ON i.film_id = f.film_id
JOIN rental AS r 
ON r.inventory_id = i.inventory_id
GROUP BY f.title
ORDER BY COUNT(r.rental_date) DESC;

-- 8a. In your new role as an executive, you would like to have an easy way of viewing the Top five genres by gross revenue. Use the solution from the problem above to create a view. If you haven't solved 7h, you can substitute another query to create a view.

CREATE VIEW MOVIE_RENTALS AS
SELECT f.title AS 'Movie Title', count(r.rental_date) AS '# of times rented'
FROM film AS f
JOIN inventory AS i 
ON i.film_id = f.film_id
JOIN rental AS r 
ON r.inventory_id = i.inventory_id
GROUP BY f.title
ORDER BY COUNT(r.rental_date) DESC;


-- 8b. How would you display the view that you created in 8a?

SELECT *
FROM MOVIE_RENTALS;

-- 8c. You find that you no longer need the view top_five_genres. Write a query to delete it.

DROP VIEW MOVIE_RENTALS;


