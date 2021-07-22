## My Notes on Postgres

### PSQL basics (Linux)

To enter `psql` run `sudo -i -u postgres`. This connects you to the `postgres` account on your server. Then type `psql` to enter... psql.

- `\q` takes you to postgres command prompt
- `exit` exits you from psql
- `\l` list databses
- `\c <db name>` enter a database

```sql
CREATE TABLE [IF NOT EXISTS] table_name (
  column_1 datatype(length) column_constraints,
  table_constraints
);
```

Column constraints:

- `NOT NULL` returns selected columns that are not NULL
- `UNIQUE` returns unique values
- `PRIMARY KEY` unique identifier of a row
- `CHECK` data must satisfy a boolean expression
- `FOREIGN KEY` ensures values in a column or group of columns exist in another column

Primary Keys can be more than one value.
Foreign Keys use `REFERENCE` to indicate the table the Foreign Key relates to.

```sql
CREATE TABLE account_roles (
  user_id INT NOT NULL,
  role_id INT NOT NULL,
  PRIMARY KEY (user_id, role_id)
  FOREIGN KEY (role_id) REFERENCES roles (role_id),
  FOREIGN KEY (user_id) REFERENCES accounts (user_id)
);
```

Databases

```sql
# create new database
CREATE DATABASE database_name;

# drop database
DROP DATABASE [ IF EXISTS ] database_name;
```

### Querying Data

- `SELECT` retrieves data from a single table.

  ```sql
  SELECT item_we_want, another_item FROM table_name;
  ```

- `ALIAS` changes the name of the column that's returned with keyword `AS`.

  ```sql
  SELECT colmn_name alias_name FROM table_name;

  SELECT first_name || ' ' || last_name AS full_name FROM table_name;
  ```

- `ORDER BY` allows you to sort rows in ascending (ASC) or descending (DESC) order.

  ```sql
  SELECT first_name FROM customer ORDER BY first_name DESC;

  # You can select this for multipule fields.
  SELECT first_name, last_name FROM customers ORDER BY last_name ASC, first_name ASC;

  # ORDER BY can also be used on number fields
  SELECT first_name FROM customers ORDER BY LENGTH(first_name);

  # NULLS this keyword lets you select how null values will be used
  SELECT item FROM table ORDER BY with NULL;

  SELECT item FROM table ORDER BY item NULLS FIRST;

  SELECT item FROM table ORDER BY item NULLS LAST;
  ```

- `DISTINCT` removes duplicate rows from a result set.

  ```sql
  SELECT DISTINCT column_1 FROM table_name;
  # `DISTINCT` can be combined with multiple columns with `ON`. It's best practice to include `ORDER BY` with `DISTINCT ON`.

  SELECT DISTINCT ON column_1, column_2 FROM table_name ORDER BY column_1;
  ```

### Filtering Data

- `WHERE` filters rows based on condition. You CANNOT reference aliases in the `WHERE` cluase.
  Comparison and Logic Operators Available

  - `=`, `>`, `=>`, `<`, `<=`
  - `<>` or `!=` (not equal)
    ```sql
    SELECT first_name FROM customers WHERE first_name <> 'Job';
    ```
  - `AND` `OR`
    ```sql
    SELECT first_name FROM customers WHERE first_name = 'Goon' AND last_name = 'Bean';
    ```
  - `IN` returns true if a value matches any value in a list
    ```sql
    SELECT first_name FROM customers WHERE first_name IN ('Tsuki', 'Tazeki', 'Skoogert');
    ```
  - `BETWEEN` returns true if a value is between a range of values
    ```sql
    SELECT first_name FROM customers WHERE LENGTH(first_name) BETWEEN 3 AND 5;
    ```
  - `LIKE` returns true if it matches a pattern
    ```sql
    SELECT first_name FROM customers WHERE first_name LIKE 'N8%';
    ```
  - `IS NULL` retruns true if a value is NULL

  ```sql
  SELECT * FROM table WHERE first_name IS NULL;
  ```

  - `NOT` negate the result of other operations

  ```sql
  SELECT * FROM table WHERE item NOT IN ('I', 'Me', 'Mine');
  SELECT * FROM table WHERE item IS NOT NULL;
  SELECT * FROM table WHERE item NOT LIKE '%Na%';
  SELECT * FROM table WHERE item NOT BETWEEN 32 AND 600;
  ```

  - `LIMIT` - constrains the number of rows returned

  ```sql
  SELECT users FROM customers LIMIT 1000;
  ```

  `OFFSET` can be used in congunction with `LIMIT` to 'page' or 'skip' result

  ```sql
  SELECT users FROM customers LIMIT 100 OFFSET 100;
  ```

  The above will return users from 200-300.

  - `FETCH` retrieves a portion of rows returned by a query. It is functionally equilivant to `LIMIT`, but `LIMIT` is NOT an SQL-standard. If you need to make your app compatible with other database systems use `FETCH` instead of `LIMIT`.

  ```sql
  SELECT title FROM film ORDER BY title FETCH FIRST 5 ROW ONLY;
  ```

  - `IN` you can use `IN` with `WHERE` to check if a value matches from a list or subquery.

  ```sql
  SELECT * FROM users WHERE first_name IN ('Tsuki', 'Taziki', 'BHB');
  SELECT customer_id FROM rentals WHERE custimer_id NOT IN (1,2);
  SELECT customer_id FROM customer
  WHERE customer_id IN (
    SELECT customer_id FROM rental WHERE return_date = 'today';
  );
  ```

  - `BETWEEN` this operator returns values within a range of 2 values;

  ```sql
  SELECT price FROM products BETWEEN 20 AND 30;
  ```

  - `LIKE` returns items that follow a pattern

  ```sql
  # `%` wildcard - at the start or end will do a fuzzy search
  SELECT first_name FROM customers WHERE first_name LIKE '%er%';

  # `_` returns when something starts or ends with something
  SELECT first_name FROM customers WHERE first_name LIKE '_s';
  ```

  `NOT LIKE` returns results that do not match pattern

  ```sql
  SELECT first_name FROM customers WHERE first_name NOT LIKE 'jen%';
  ```

  `ILIKE` returns results that follow a pattern and are case-sensitive

  ```sql
  SELECT first_name FROM custimers WHERE first_name ILIKE '%Mc%';
  ```

  - `NULL / IS NULL` checks if a value is null

  ```sql
  SELECT * FROM table WHERE first_name IS NULL;
  SELECT * FROM table WHERE first_name IS NOT NULL;
  ```

### Joining Data

`INNER JOIN` joins values in the columns of the first table with the values in the columns in the second table. The result will be a new row that contains columns from both tables. Values that do not match both tables will not be returned.

```sql
SELECT fruit_a, fruit_b FROM basket_a INNER JOIN basket_b ON fruit_a = fruit_b;
+- fruit_a -+- fruit_b -+
| Apple     | Apple     |
+-----------+-----------+
```

`LEFT JOIN` or `LEFT OUTER JOIN` joins values from the first table, and if there are not matchs in the second table, the second tables columns will return null.

```sql
SELECT fruit_a, fruit_b FROM basket_a LEFT JOIN basket_b ON fruit_a = fruit_b;
+- fruit_a -+- fruit_b -+
| Apple     | Apple     |
| Banana    | [null]    |
+-----------+-----------+
```

`RIGHT JOIN` this is the invers of `LEFT JOIN`.

```sql
SELECT fruit_a, fruit_b FROM basket_a RIGHT JOIN basket_b ON fruit_a = fruit_b;
+- fruit_a -+- fruit_b -+
| Apple     | Apple     |
| [null]    | Orange    |
+-----------+-----------+
```

`FULL OUTER JOIN` returns result set from both left and right tables, if there is no match the row will be null.

```sql
SELECT fruit_a, fruit_b FROM basket_a RIGHT JOIN basket_b ON fruit_a = fruit_b;
+- fruit_a -+- fruit_b -+
| Apple     | Apple     |
| [null]    | [null]    |
+-----------+-----------+
```

`CROSS JOIN` allows you to produce a Cartesian Product or rows.
Cartesian Product of A and B, A={7, 8} B={2, 4, 6} Product={7-2, 7-4, 7-6, 8-2, 8-4, 8-6}

```sql
table_a
+- label -+
| A       |
| B       |
+---------+

table_b
+- score -+
| 1       |
| 2       |
| 3       |
+---------+

THESE 3 QUERIES ARE THE SAME:
SELECT * FROM table_a CROSS JOIN table_b; (this one is easiest to read IMO)
SELECT * FROM table_a, table_b;
SELECT * FROM table_a INNER JOING table_b ON true;

+- label -+- score -+
| A       | 1       |
| B       | 1       |
| A       | 2       |
| B       | 2       |
| A       | 3       |
| B       | 3       |
+---------+---------+
```

`NATURAL JOIN` creates an implicit join based onthe same conlumn names in the joined table.
It does this by automatically joining the table on `FOREIGN KEY`.
A `NATURAL JOIN` can use be `NATURAL INNER JOIN`, `NATURAL LEFT JOIN`, `NATURAL RIGHT JOIN`

```sql
table categories
+- categories -+
| 'Laptop'     |
+--------------+

table products
+- product_name -+- category_id -+
| 'Think Pad'    | 1             |
+----------------+---------------+

SELECT * FROM products NATURAL JOIN categories;
Result:
+- category_id -+- product_id -+- product_name -+- category_name -+
| 1             | 1            | 'Think Pad'    | 'Laptop'        |
+---------------+--------------+----------------+-----------------+
```

`Table aliases` temporarily assign tables new names during a query. These are needed when doing a self join.

```sql
a_very_long_table_name AS alias
```

### Grouping Data

- `GROUP BY` allows you to divide rows into groups. It will return distinct rows. It can be used with Aggrate Functions and math.

  ```sql
  SELECT column_1, column_2 FROM table_name GROUP BY column_1, column_2;

  # returns total amount each customer has paid
  SELECT customer_id SUM(amount) FROM payment GROUP BY customer_id;

  # returns total payments of each customer
  SELECT first_name, SUM (amount) AS amount
  FROM payment
  JOIN customer USING (customer_id)
  GROUP BY full_name
  ORDER BY amount;

  # returns number of payment transactions each staff has completed
  SELECT staff_id, COUNT (payment_id)
  FROM payment GROUP BY  staff_id;

  # returns payments by date
  SELECT DATE(payment_date) paid_date, SUM(amount) sum
  FROM payment GROUP BY DATE(payment_date);
  ```

  - `HAVING` specifies a search condition for a group or an aggregate. `HAVING` is to `GROUP BY` what `WHERE` is to `SELECT`.

  ```sql
  SELECT column_1 FROM table_name GROUP BY column_1 HAVING condition;

  # finds total amount for each customer
  SELECT customer_id, SUM (amount) FROM payment GROUP BY customer_id;

  # finds total amount for each customer, who has spent over 200
  SELECT customer_id, SUM (amount) FROM payment GROUP BY customer_id HAVING SUM (amount) > 200;
  ```

- `UNION` combines results of two or more `SELECT` statements

  - the numner and order of columns in the select must be the same
  - the data types must be compatible

  ```sql
  SELECT column_1 FROM table_1 UNION SELECT column_1 FROM table_2;

  # this UNION combines tables 'top_rated_films' and 'most_popupar_films'
  SELECT * FROM top_rated_films
  UNION
  SELECT * FROM most_popular_films;

  # UNION ALL combines result sets and INCLUDES duplicates, standard UNION ignores dups
  SELECT * FROM top_rated_films
  UNION ALL
  SELECT * FROM most_popular_films;

  # use GROUP BY to order results
  SELECT * FROM top_rated_films
  UNION ALL
  SELECT * FROM most_popular_films
  ORDER BY titls;
  ```

- `INTERSECT` combines result sets of two or more select statements into one result set.

  - the number of columns and their order in the `SELECT` clause must be the same
  - the data types must be compatible

  ```sql
  SELECT select_list FROM table_a
  INTERSECT
  SELECT select_list FROM table_b
  ORDER BY sort_expression;

  # get popular films that are also top rated films (films that are in BOTH tables)
  SELECT * FROM most_popular_films
  INTERSECT
  SELECT * FROM top_rated_films;
  ```

- `EXCEPT` retruns distinct rows from the first query that are not in the second

  - the number of columns and their orders must be the same in the two queries
  - data types of the respective columns must be compatible

  ```sql
  # returns top-rated films that are not popular
  SELECT * FROM top_rated_films
  EXCEPT
  SELECT * FROM most_popular_films;
  ```

- `GROUPING SET` a set of comma-seperated columns in parentheses you group by using the `GROUP BY` clause

  ```sql
  SELECT c1, c2 FROM table_name
  GROUP BY GROUPING SETS ((c1, c2), (c1), (c2));
  ```

- `CUBE` a cube allows you to generate multiple grouping sets

- `ROLLUP` a shorthand for defining multiple groupling sets. It's different from a `CUBE` becaues it does not generate all posible grouping sets based on the specified columns, it just makes a subset of those.

### Subqueries

- `SUBQUERY`

  ```sql
  # get the average rental rate of films who have an above average rental rate
  SELECT film_id, title FROM film
  WHERE rental_rate > (
    SELECT AVG(rental_rate) FROM film
  );
  # the query that is in brackets is called a subquery or inner query. The query that contains the subquery is the outer query (clever I know).

  # subquery with the `IN` operator, to get films returned between '2005-05-29' and '2005-05-30'
  SELECT film_id, title FROM film
  WHERE film_id IN (
    SELECT inventory.film_id FROM rental
    INNER JOIN inventory ON inventory.inventory_id = rental.inventory_id
    WHERE return_data BETWEEN '2005-05-29' AND '2005-05-30'
  );

  # subquery with `EXISTS`, returns customer where patment is true
  SELECT first_name, last_name FROM customer
  WHERE EXISTS (
    SELECT 1 FROM payment WHERE payment.customer_id = customer.customer_id
  );
  ```

- `ANY` retrieve data by comparing a value, with a set of values returned by a subquery. The sub query may only return `1` column. `ANY` is equivalent to the `IN` operator.

  ```sql
  # find films whose title lengths are greater than the length of the film category.
  SELECT title FROM film
  WHERE length > ANY (
    SELECT MAX (length) FROM film
    INNER JOIN film_category USING (film_id)
    GROUP BY category_id
  );
  ```

- `ALL` query data by comparing a value with a list of values returned by a subquery. `ALL` must be used with a comparision operator.

  ```sql
  SELECT title, film FROM film
  WHERE length > ALL (
    SELECT ROUND(AVG (length), 2) FROM film
    GROUP BY rating
  )
  ORDER BY length;
  ```

- `EXISTS` / `NOT EXISTS` a boolean operator that check for the existance of rows returned by a subquery.

  ```sql
  # find customers who have at least one payment whose amount is greater than 11.
  SELECT first_name FROM customer
  WHERE EXISTS (
    SELECT 1 FROM payment
    WHERE payment.customer_id = customer.customer_id
    AND amount > 11)
  )
  ORDER BY first_name;

  # using NULL - the subquery returned `NULL` so the query returned all rows from the cuetomers table
  SELECT first_name FROM customer
  WHERE EXISTS( SELECT NULL )
  ORDER BY first_name;
  ```

### Common Table Expressions

`CTE`s are temporary and only exist during the execution of the query. You can use them to make recursive queries.

```sql
WITH cte_name (column_list) AS (
  CTE_query_definition
)
STATEMENT;
```

- Specify the CTE name and the columns you want.
- Inside the body specify a query that returns a result set. If you don't specify the column_list, the select list of the CTE_query_definition will be the column list.
- Use the CTE like a table or view in the statement, which can be `SELECT`/`INSERT`/`UPDATE`/`DELETE`.

```sql
# creating the CTE
WITH cte_rental AS (
  SELECT staff_id, COUNT(rental_id) AS rental_count FROM rental
  GROUP BY staff_id
)

# using the CTE
SELECT staff.staff_id, first_name, rental_count
FROM staff
INNER JOIN cte_rental USING (staff_id);
```

### Modifying Data

### Transactions

<!--
### DATA TYPES

### Markdown Reference

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/). -->
