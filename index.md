## My Notes on Postgres

### PSQL basics (Linux)
To enter `psql` run `sudo -i -u postgres`. This connects you to the `postgres` account on your server. Then type `psql` to enter... psql.

- `\q`  takes you to postgres command prompt
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



### SELECT
- `SELECT` retrieves data from a single table.
  ```sql
  SELECT item_we_want, another_item FROM table_name;
  ```

- `ALIAS` changes the name of the column that's returned with keyword `AS`.
  ```sql
  SELECT colmn_name alias_name FROM table_name;
  ```
  ```sql
  SELECT first_name || ' ' || last_name AS full_name FROM table_name;
  ```

- `ORDER BY` allows you to sort rows in ascending (ASC) or descending (DESC) order.
  ```sql
  SELECT first_name FROM customer ORDER BY first_name DESC;
  ```
  You can select this for multipule fields.
  ```sql
  SELECT first_name, last_name FROM customers ORDER BY last_name ASC, first_name ASC;
  ```
  ORDER BY can also be used on number fields
  ```sql
  SELECT first_name FROM customers ORDER BY LENGTH(first_name);
  ```
  NULLS this keyword lets you select how null values will be used
  ```sql
  SELECT item FROM table ORDER BY with NULL;
  ``` 
  ```sql
  SELECT item FROM table ORDER BY item NULLS FIRST;
  ```
  ```sql
  SELECT item FROM table ORDER BY item NULLS LAST;
  ```

- `DISTINCT` removes duplicate rows from a result set.
  ```sql
  SELECT DISTINCT column_1 FROM table_name;
  ```
  `DISTINCT` can be combined with multiple columns with `ON`. It's best practice to include `ORDER BY` with `DISTINCT ON`.
  ```sql
  SELECT DISTINCT ON column_1, column_2 FROM table_name ORDER BY column_1;
  ```
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
  `%` wildcard - at the start or end will do a fuzzy search
  ```sql
  SELECT first_name FROM customers WHERE first_name LIKE '%er%';
  ```
  `_` returns when something starts or ends with something
  ```sql
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

### JOINS

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

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

