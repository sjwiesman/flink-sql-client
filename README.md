
Start SQL Client

```
docker-compose exec sql-client ./sql-client.sh
```

```sql
CREATE TABLE transactions (
    account_id BIGINT,
    amount BIGINT,
    transaction_time TIMESTAMP(3),
    WATERMARK FOR transaction_time AS transaction_time - INTERVAL '5' SECOND
) WITH (
    'connector' = 'datagen',
    'rows-per-second' = '10'
);
```

```sql
CREATE TABLE spend_report (
    account_id BIGINT,
    log_ts TIMESTAMP(3),
    amount BIGINT,
    PRIMARY KEY (account_id, log_ts) NOT ENFORCED
) WITH (
    'connector' = 'jdbc',
    'url' = 'jdbc:mysql://mysql:3306/sql-demo',
    'driver' = 'com.mysql.jdbc.Driver',
    'table-name' = 'spend_report',
    'username' = 'sql-demo',
    'password' = 'demo-sql'
);
```

```sql
INSERT INTO spend_report
SELECT 
    account_id,
    TUMBLE_START(transaction_time, INTERVAL '1' SECOND) AS log_ts,
    SUM(amount) AS amount
FROM transactions
GROUP BY 
    TUMBLE(transaction_time, INTERVAL '1' SECOND),
    account_id;
```

```
$ docker-compose exec mysql mysql -Dsql-demo -usql-demo -pdemo-sql
```

