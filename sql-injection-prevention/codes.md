# SQL Injection Examples

SQL injection is a code injection technique that exploits vulnerabilities in database-driven applications. Here are examples of different types of SQL injection attacks and how to prevent them.

## Basic SQL Injection Examples

### 1. Classic SQL Injection (Bypassing Authentication)

```sql
-- Vulnerable SQL query in application
SELECT * FROM users WHERE username = '[user_input]' AND password = '[user_input]';

-- Attack input (username field)
admin' --
-- Resulting query (comments out password check)
SELECT * FROM users WHERE username = 'admin' --' AND password = '';
```

### 2. Union-Based SQL Injection

```sql
-- Vulnerable query
SELECT product_name, description FROM products WHERE id = [user_input];

-- Attack input
1 UNION SELECT username, password FROM users --
-- Resulting query
SELECT product_name, description FROM products WHERE id = 1 UNION SELECT username, password FROM users --';
```

### 3. Boolean-Based Blind SQL Injection

```sql
-- Vulnerable query
SELECT * FROM products WHERE id = [user_input];

-- Attack input to test for vulnerability
1 AND 1=1 -- Returns normal results if vulnerable
1 AND 1=2 -- Returns empty results if vulnerable
```

### 4. Time-Based Blind SQL Injection

```sql
-- Vulnerable query
SELECT * FROM users WHERE id = [user_input];

-- Attack input to extract data slowly
1; IF (SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin')='a' WAITFOR DELAY '0:0:5' --
```

## Prevention Examples

### Parameterized Queries (Best Defense)

#### Python (with psycopg2)
```python
# Safe
cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", (username, password))
```

#### PHP (with PDO)
```php
// Safe
$stmt = $pdo->prepare('SELECT * FROM users WHERE username = :username AND password = :password');
$stmt->execute(['username' => $username, 'password' => $password]);
```

#### Java (with PreparedStatement)
```java
// Safe
String sql = "SELECT * FROM users WHERE username = ? AND password = ?";
PreparedStatement stmt = connection.prepareStatement(sql);
stmt.setString(1, username);
stmt.setString(2, password);
ResultSet rs = stmt.executeQuery();
```

### Input Validation and Sanitization

#### Node.js (with mysql2)
```javascript
// Safe with placeholders
connection.execute(
  'SELECT * FROM users WHERE username = ? AND password = ?',
  [username, password],
  function(err, results) {
    // handle results
  }
);
```

### ORM Examples (Even Safer)

#### Python (with SQLAlchemy)
```python
# Safe ORM usage
result = session.query(User).filter_by(username=username, password=password).first()
```

#### PHP (with Laravel Eloquent)
```php
// Safe ORM usage
$user = User::where('username', $username)
            ->where('password', $password)
            ->first();
```

**Important Security Note:** These examples are provided for educational purposes to help developers understand and prevent SQL injection vulnerabilities. Never use these techniques maliciously or against systems without explicit permission.
