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

# SQL Notes

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
