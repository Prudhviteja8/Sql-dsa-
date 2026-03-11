# LeetCode SQL Solutions

---

## 1148. Article Views I

**Table:** `Views` (article_id, author_id, viewer_id, view_date)

**Task:** Find all authors that viewed at least one of their own articles. Return as `id`, sorted ascending.

**Solution:**
```sql
SELECT DISTINCT author_id AS id
FROM Views
WHERE author_id = viewer_id
ORDER BY id ASC;
```

**Key Takeaway:** Not every problem needs a JOIN. When all data is in one table, a simple WHERE clause is enough.

---

## 595. Big Countries

**Table:** `World` (name, continent, area, population, gdp)

**Task:** Find name, population, and area of countries with area >= 3,000,000 OR population >= 25,000,000.

**Solution:**
```sql
SELECT name, population, area
FROM World
WHERE area >= 3000000 OR population >= 25000000;
```

**Key Takeaway:**
- `=` means exactly equal, `>=` means at least
- `AND` = both conditions must be true, `OR` = either condition can be true

---

## 1683. Invalid Tweets

**Table:** `Tweets` (tweet_id, content)

**Task:** Find IDs of tweets where content length is strictly greater than 15.

**Solution:**
```sql
SELECT tweet_id
FROM Tweets
WHERE LENGTH(content) > 15;
```

**Key Takeaway:** "Strictly greater than" means `>`, not `>=`.

---

## 584. Find Customer Referee

**Table:** `Customer` (id, name, referee_id)

**Task:** Find names of customers NOT referred by customer with id = 2.

**Solution:**
```sql
SELECT name
FROM Customer
WHERE referee_id IS NULL OR referee_id != 2;
```

**Alternative:**
```sql
SELECT name
FROM Customer
WHERE COALESCE(referee_id, 0) != 2;
```

**Key Takeaway:** `NULL != 2` returns `NULL`, not `TRUE`. Always handle NULLs explicitly with `IS NULL` or `COALESCE`.

---

## 1587. Bank Account Summary II

**Tables:** `Users` (account, name), `Transactions` (trans_id, account, amount, transacted_on)

**Task:** Find name and balance of users with balance > 10,000.

**Solution:**
```sql
SELECT u.name, SUM(t.amount) AS balance
FROM Users u
JOIN Transactions t ON u.account = t.account
GROUP BY u.account, u.name
HAVING SUM(t.amount) > 10000;
```

**Key Takeaway — WHERE vs HAVING:**
| | WHERE | HAVING |
|---|-------|--------|
| When | Before GROUP BY | After GROUP BY |
| Filters | Individual rows | Grouped results |
| Aggregates? | No | Yes |

**SQL Clause Order:** `SELECT -> FROM -> JOIN -> ON -> WHERE -> GROUP BY -> HAVING -> ORDER BY`

---

## 1527. Patients With a Condition

**Table:** `Patients` (patient_id, patient_name, conditions)

**Task:** Find patients who have Type I Diabetes (prefix `DIAB1`).

**Solution:**
```sql
SELECT patient_id, patient_name, conditions
FROM Patients
WHERE conditions LIKE 'DIAB1%' OR conditions LIKE '% DIAB1%';
```

**Key Takeaway:** When matching a prefix in a space-separated string:
- `'DIAB1%'` — matches at the start
- `'% DIAB1%'` — matches after a space
- `'%DIAB1%'` — too broad, could match `XDIAB1`

---

## 620. Not Boring Movies

**Table:** `Cinema` (id, movie, description, rating)

**Task:** Find movies with odd id AND description is not "boring". Sort by rating descending.

**Solution:**
```sql
SELECT id, movie, description, rating
FROM Cinema
WHERE id % 2 = 1 AND description != 'boring'
ORDER BY rating DESC;
```

**Key Takeaway:**
- `id % 2 = 1` → odd numbers, `id % 2 = 0` → even numbers
- `IS NOT` is for NULLs, use `!=` for value comparison
- SQL uses single quotes `'text'` for strings

---

## 1141. User Activity for the Past 30 Days I

**Table:** `Activity` (user_id, session_id, activity_date, activity_type)

**Task:** Find daily active user count for 30-day period ending 2019-07-27.

**Solution:**
```sql
SELECT activity_date AS day, COUNT(DISTINCT user_id) AS active_users
FROM Activity
WHERE activity_date BETWEEN '2019-06-28' AND '2019-07-27'
GROUP BY activity_date;
```

**Alternative using DATEDIFF:**
```sql
SELECT activity_date AS day, COUNT(DISTINCT user_id) AS active_users
FROM Activity
WHERE DATEDIFF('2019-07-27', activity_date) < 30
  AND activity_date <= '2019-07-27'
GROUP BY activity_date;
```

**Key Takeaway:**
- `COUNT(DISTINCT col)` — not `DISTINCT(col)`
- Dates must be in single quotes: `'2019-07-27'`
- `BETWEEN` is inclusive on both ends

---

## 1693. Daily Leads and Partners

**Table:** `DailySales` (date_id, make_name, lead_id, partner_id)

**Task:** For each date_id and make_name, find the number of distinct lead_ids and distinct partner_ids.

**Solution:**
```sql
SELECT date_id, make_name,
       COUNT(DISTINCT lead_id) AS unique_leads,
       COUNT(DISTINCT partner_id) AS unique_partners
FROM DailySales
GROUP BY date_id, make_name;
```

**Key Takeaway:**
- `DISTINCT(col)` is NOT a function — use `COUNT(DISTINCT col)`
- Group by multiple columns when needed

---

## 1729. Find Followers Count

**Table:** `Followers` (user_id, follower_id)

**Task:** For each user, find the number of followers. Sort by user_id ascending.

**Solution:**
```sql
SELECT user_id, COUNT(follower_id) AS followers_count
FROM Followers
GROUP BY user_id
ORDER BY user_id ASC;
```

**Key Takeaway:**
- `GROUP BY user_id` → "for each user"
- `COUNT(follower_id)` → counts followers per user
- No `DISTINCT` needed since `(user_id, follower_id)` is already a primary key

---

## 182. Duplicate Emails

**Table:** `Person` (id, email)

**Task:** Find all duplicate emails (appear more than once).

**Solution:**
```sql
SELECT email
FROM Person
GROUP BY email
HAVING COUNT(email) > 1;
```

**Key Takeaway:**
- Use `HAVING` (not `WHERE`) to filter with aggregate functions
- If you see `COUNT()`, `SUM()`, `AVG()` in a filter → always `HAVING`

---

## 1050. Actors and Directors Who Cooperated At Least Three Times

**Table:** `ActorDirector` (actor_id, director_id, timestamp)

**Task:** Find (actor_id, director_id) pairs that cooperated at least 3 times.

**Solution:**
```sql
SELECT actor_id, director_id
FROM ActorDirector
GROUP BY actor_id, director_id
HAVING COUNT(*) >= 3;
```

**Key Takeaway:**
- Use commas to separate columns in SELECT, not `AND`
- `COUNT(*)` counts all rows — takes only one argument
- `>= 3` means "at least 3", `> 3` means "more than 3"

---

## 1378. Replace Employee ID With The Unique Identifier

**Tables:** `Employees` (id, name), `EmployeeUNI` (id, unique_id)

**Task:** Show unique_id and name for each employee. If no unique_id, show NULL.

**Solution:**
```sql
SELECT e2.unique_id, e1.name
FROM Employees e1
LEFT JOIN EmployeeUNI e2 ON e1.id = e2.id;
```

**Key Takeaway:**
- `LEFT JOIN` keeps all rows from the left table even if no match (fills NULL)
- `JOIN` (INNER) only returns rows that match in both tables
- "If no match, show NULL" → always use `LEFT JOIN`

---

## 1068. Product Sales Analysis I

**Tables:** `Sales` (sale_id, product_id, year, quantity, price), `Product` (product_id, product_name)

**Task:** Find product_name, year, and price for each sale.

**Solution:**
```sql
SELECT p.product_name, s.year, s.price
FROM Sales s
JOIN Product p ON s.product_id = p.product_id;
```

**Key Takeaway:**
- `FROM` and `JOIN` take **table names**, not column names
- Always use `ON` keyword after JOIN to specify the condition
- JOIN syntax: `FROM TableA a JOIN TableB b ON a.col = b.col`

---

## 1965. Employees With Missing Information

**Tables:** `Employees` (employee_id, name), `Salaries` (employee_id, salary)

**Task:** Find IDs of employees with missing name OR missing salary.

**Solution:**
```sql
SELECT employee_id FROM Employees
WHERE employee_id NOT IN (SELECT employee_id FROM Salaries)
UNION
SELECT employee_id FROM Salaries
WHERE employee_id NOT IN (SELECT employee_id FROM Employees)
ORDER BY employee_id ASC;
```

**Key Takeaway:**
- `NOT IN (subquery)` finds rows that don't exist in other table
- `UNION` combines results from two queries

---

## 1795. Rearrange Products Table

**Table:** `Products` (product_id, store1, store2, store3)

**Task:** Turn columns into rows (unpivot). Exclude NULL prices.

**Solution:**
```sql
SELECT product_id, 'store1' AS store, store1 AS price FROM Products WHERE store1 IS NOT NULL
UNION ALL
SELECT product_id, 'store2' AS store, store2 AS price FROM Products WHERE store2 IS NOT NULL
UNION ALL
SELECT product_id, 'store3' AS store, store3 AS price FROM Products WHERE store3 IS NOT NULL;
```

**Key Takeaway:**
- `UNION ALL` combines multiple SELECTs into one result
- Use separate SELECT for each column to unpivot

---

# SQL Notes

## SQL Basics

### Query Template
```
SELECT columns      ← what do you want?
FROM table          ← which table?
WHERE condition     ← filter rows
GROUP BY column     ← make piles
HAVING condition    ← filter groups
ORDER BY column     ← sort results
LIMIT n             ← how many rows?
```

### SELECT Examples
- `SELECT *` → all columns
- `SELECT name, marks` → specific columns
- `SELECT name AS student_name` → rename column (alias can't have spaces, use underscores)

### WHERE Conditions
- `WHERE marks > 80` → greater than
- `WHERE city = 'Hyderabad'` → equals (single quotes for strings!)
- `WHERE marks BETWEEN 75 AND 90` → range (inclusive)
- `WHERE city IN ('Chennai', 'Mumbai')` → multiple values
- `WHERE city = 'Hyderabad' AND marks > 80` → both conditions
- `WHERE city = 'Chennai' OR city = 'Mumbai'` → either condition (must repeat column name)

### ORDER BY
- `ORDER BY marks ASC` → low to high (default)
- `ORDER BY marks DESC` → high to low
- Always put **column name** before ASC/DESC

### LIMIT
- `ORDER BY marks DESC LIMIT 3` → top 3
- `ORDER BY marks ASC LIMIT 3` → bottom 3

## GROUP BY — How to Decide

Find "per ____" or "each ____" in the question → that's your GROUP BY.

| Question says | GROUP BY |
|---|---|
| Total per **customer** | `customer_id` |
| Count per **city** | `city` |
| Average per **department** | `department` |
| Total per **channel** per **month** | `channel, month` |

## Aggregate Functions

| Function | Meaning |
|----------|---------|
| `COUNT(*)` | How many rows |
| `SUM(col)` | Add up values |
| `AVG(col)` | Average |
| `MAX(col)` | Highest |
| `MIN(col)` | Lowest |
| `COUNT(DISTINCT col)` | Count unique values |

## WHERE vs HAVING

**Simple rule:** See COUNT, SUM, AVG, MIN, MAX in the condition? → HAVING. Otherwise → WHERE.

| Condition | Use |
|---|---|
| `city = 'Hyderabad'` | WHERE |
| `marks > 80` | WHERE |
| `COUNT(*) > 5` | HAVING |
| `SUM(amount) > 10000` | HAVING |
| `AVG(marks) > 80` | HAVING |

**Analogy:** WHERE = choosing who enters the restaurant. HAVING = after they sit in groups, deciding which groups get served.

## Types of JOINs

| JOIN Type | What it does |
|---|---|
| `JOIN` (INNER) | Only rows that match in **both** tables |
| `LEFT JOIN` | **All** rows from left table + matches from right (NULL if no match) |
| `RIGHT JOIN` | All rows from right table + matches from left |
| `FULL OUTER JOIN` | All rows from both tables (NULL where no match) |

## WHERE vs HAVING

| | WHERE | HAVING |
|---|-------|--------|
| **When** | Before GROUP BY | After GROUP BY |
| **Filters** | Individual rows | Grouped results |
| **Can use aggregates?** | No (SUM, COUNT, etc.) | Yes |

**Analogy:** WHERE = choosing who enters the restaurant. HAVING = after they sit in groups, deciding which groups get served.

## NULL Handling

| Expression | Result |
|---|---|
| `NULL != 2` | `NULL` (not TRUE!) |
| `NULL = 2` | `NULL` |
| `NULL IS NULL` | `TRUE` |
| `COALESCE(NULL, 0)` | `0` |

## Comparison Operators

| Operator | Meaning |
|----------|---------|
| `=` | exactly equal |
| `>=` | at least (greater than or equal) |
| `>` | strictly greater than |
| `AND` | both conditions must be true |
| `OR` | either condition can be true |

## LIKE Patterns

| Pattern | Matches |
|---------|---------|
| `'abc%'` | Starts with abc |
| `'%abc'` | Ends with abc |
| `'%abc%'` | Contains abc anywhere |
| `'_abc'` | Any single char + abc |

## DATEDIFF()

`DATEDIFF(date1, date2)` returns the difference in days (date1 - date2).

| Expression | Result |
|---|---|
| `DATEDIFF('2019-07-27', '2019-07-27')` | 0 |
| `DATEDIFF('2019-07-27', '2019-07-20')` | 7 |
| `DATEDIFF('2019-07-27', '2019-06-27')` | 30 |

**Real-World Use Cases:**
- **E-Commerce:** Find orders in last 7 days → `WHERE DATEDIFF(CURDATE(), order_date) <= 7`
- **Apps:** Find inactive users → `WHERE DATEDIFF(CURDATE(), last_login) > 30`
- **Hospital:** Overdue follow-ups → `WHERE DATEDIFF(CURDATE(), surgery_date) > 14`
- **HR:** Employee tenure → `DATEDIFF(CURDATE(), joining_date) / 365 AS years`

**Note:** For time differences (hours, minutes), use `TIMESTAMPDIFF(MINUTE, start, end)` instead.

**Syntax varies by database:**
| Database | Syntax |
|---|---|
| MySQL | `DATEDIFF(date1, date2)` → returns days |
| SQL Server | `DATEDIFF(unit, date1, date2)` |
| PostgreSQL | `date1 - date2` → subtract directly |

## Odd/Even Check

| Expression | Result | Meaning |
|---|---|---|
| `5 % 2` | 1 | Odd |
| `4 % 2` | 0 | Even |

## JOIN Syntax Template

```sql
SELECT columns
FROM TableA a
JOIN TableB b ON a.common_col = b.common_col;
```

- `FROM` and `JOIN` take **table names**, not column names
- Always use `ON` keyword after JOIN
- Use aliases (a, b) to keep queries short

## NOT IN / UNION

**NOT IN:**
```sql
WHERE col NOT IN (SELECT col FROM other_table)   -- subquery
WHERE col NOT IN (1, 2, 3)                        -- list
```

**UNION vs UNION ALL:**
| | UNION | UNION ALL |
|---|---|---|
| Duplicates | Removes | Keeps |
| Speed | Slower | Faster |

## Common Mistakes to Avoid

| Mistake | Correct |
|---|---|
| `"text"` | `'text'` (single quotes) |
| `ORDER BY DESC` | `ORDER BY column DESC` |
| `WHERE COUNT(*) > 5` | `HAVING COUNT(*) > 5` |
| `DISTINCT(col)` | `COUNT(DISTINCT col)` |
| `FROM table.column` | `FROM table` (table names only) |
| `groupby` | `GROUP BY` (two words) |
| `AS total marks` | `AS total_marks` (no spaces in alias) |
| Missing `FROM` | Always include `FROM table_name` |
