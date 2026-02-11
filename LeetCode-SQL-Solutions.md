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
