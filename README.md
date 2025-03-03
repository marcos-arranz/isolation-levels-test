# Isolation levels test

## Dirty reads

It does not happen in PostgreSQL

### Testing

Terminal 1

```sql
BEGIN TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
UPDATE accounts SET balance = 5000 where id = 1;
```

Terminal 2

```sql
BEGIN TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
select balance from accounts where id = 1;
```

Terminal 1

```sql
COMMIT;
```

Terminal 2

```sql
select balance from accounts where id = 1;
```

## Lost update

Terminal 1

```sql
BEGIN;
UPDATE accounts SET balance = 1000+1 where id = 1;
```

Terminal 2

```sql
BEGIN;
UPDATE accounts SET balance = 1000+1 where id = 1;
```

Terminal 1

```sql
COMMIT;
```

Terminal 2

```sql
COMMIT;
SELECT balance from account where id = 1;
```

Repeat with `BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ`

## Non-repeatable read

Terminal 1

```sql
COMMIT;
```

Terminal 2

```sql
COMMIT;
SELECT * FROM accounts;
```

Terminal 1

```sql
UPDATE accounts SET balance = 1000 + 5 WHERE id = 1;
COMMIT;
```

Terminal 2

```sql
SELECT * FROM accounts;
```

Repeat with `BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ`

## Phantom Reads

Terminal 1

```sql
BEGIN;
```

Terminal 2

```sql
BEGIN;
SELECT * FROM accounts;
```

Terminal 1

```sql
INSERT INTO accounts(name, balance) values ('John', 6000);
COMMIT;
```

Terminal 2

```sql
SELECT * FROM accounts;
```

Repeat with `BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ`

## Serialization anomaly

Terminal 1

```sql
BEGIN;
SELECT SUM(balance) from accounts;
```

Terminal 2

```sql
BEGIN;
SELECT SUM(balance) from accounts;
INSERT INTO accounts(name, balance) values ('Jeff', <Value of select>)
COMMIT;
```

Terminal 1

```sql
INSERT INTO accounts(name, balance) values ('Bill', <Value of select>)
COMMIT;


Repeat with `BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE`
```
