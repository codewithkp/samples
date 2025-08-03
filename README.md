# Sample Queries

## non-opt query
```sql
SELECT 
    e1.lifecycle_id,
    e1.event_type as first_event,
    e1.sender_transaction_amount as first_amount,
    e2.event_type as second_event,
    e2.sender_transaction_amount as second_amount,
    ABS(e1.sender_transaction_amount - e2.sender_transaction_amount) as amount_diff,
    CASE 
        WHEN e1.sender_transaction_amount > e2.sender_transaction_amount THEN 'Increasing'
        WHEN e1.sender_transaction_amount < e2.sender_transaction_amount THEN 'Decreasing'
        ELSE 'Same'
    END as trend,
    CONCAT(UPPER(LEFT(e1.event_type, 3)), '-', UPPER(LEFT(e2.event_type, 3))) as event_pair_code
FROM event_store e1
JOIN event_store e2 ON e1.lifecycle_id = e2.lifecycle_id 
    AND e1.created_at < e2.created_at
    AND CAST(e1.sender_transaction_amount AS STRING) != CAST(e2.sender_transaction_amount AS STRING)
WHERE e1.sender_transaction_amount IS NOT NULL
    AND e2.sender_transaction_amount IS NOT NULL
    AND TRIM(e1.event_type) != TRIM(e2.event_type)
ORDER BY amount_diff DESC;
```

## opt query

```sql
WITH ranked_events AS (
  SELECT
    lifecycle_id, 
    event_type, 
    sender_transaction_amount,
    created_at,
    ROW_NUMBER() OVER(PARTITION BY lifecycle_id ORDER BY created_at ASC) AS rn
  FROM 
    event_store
  WHERE 
    sender_transaction_amount IS NOT NULL
    AND TRIM(event_type) != ''
)
SELECT 
  e1.lifecycle_id, 
  e1.event_type AS first_event, 
  e1.sender_transaction_amount AS first_amount, 
  e2.event_type AS second_event, 
  e2.sender_transaction_amount AS second_amount, 
  ABS(e1.sender_transaction_amount - e2.sender_transaction_amount) AS amount_diff,
  CASE 
    WHEN e1.sender_transaction_amount > e2.sender_transaction_amount THEN 'Increasing'
    WHEN e1.sender_transaction_amount < e2.sender_transaction_amount THEN 'Decreasing'
    ELSE 'Same' 
  END AS trend,
  CONCAT(UPPER(LEFT(e1.event_type, 3)), '-', UPPER(LEFT(e2.event_type, 3))) AS event_pair_code
FROM 
  ranked_events e1
JOIN 
  ranked_events e2 ON e1.lifecycle_id = e2.lifecycle_id AND e1.rn = e2.rn - 1
ORDER BY 
  amount_diff DESC;
```

## non opt
 
```sql 
SELECT 
    lifecycle_id,
    event_type,
    sender_transaction_amount,
    CASE 
        WHEN REGEXP_CONTAINS(LOWER(event_type), r'.*payment.*') THEN 'Payment Related'
        WHEN REGEXP_CONTAINS(LOWER(event_type), r'.*transfer.*') THEN 'Transfer Related'
        WHEN REGEXP_CONTAINS(LOWER(event_type), r'.*verification.*') THEN 'Verification Related'
        WHEN REGEXP_CONTAINS(LOWER(event_type), r'.*review.*') THEN 'Review Related'
        ELSE 'Other'
    END as event_category,
    CASE 
        WHEN LENGTH(lifecycle_id) > 20 THEN 'Long ID'
        WHEN LENGTH(lifecycle_id) > 10 THEN 'Medium ID'
        ELSE 'Short ID'
    END as id_length_category,
    CONCAT(SUBSTR(lifecycle_id, 1, 3), '...', SUBSTR(lifecycle_id, -3)) as masked_id
FROM event_store
WHERE lifecycle_id IS NOT NULL
    AND event_type IS NOT NULL
    AND REGEXP_CONTAINS(lifecycle_id, r'^[A-Za-z0-9]+$')
ORDER BY sender_transaction_amount DESC;
```

## opt query

```sql
 WITH filtered_events AS (
  SELECT 
    lifecycle_id,
    event_type,
    sender_transaction_amount,
    CASE 
      WHEN LOWER(event_type) LIKE '%payment%' THEN 'Payment Related'
      WHEN LOWER(event_type) LIKE '%transfer%' THEN 'Transfer Related'
      WHEN LOWER(event_type) LIKE '%verification%' THEN 'Verification Related'
      WHEN LOWER(event_type) LIKE '%review%' THEN 'Review Related'
      ELSE 'Other'
    END as event_category,
    CASE 
      WHEN LENGTH(lifecycle_id) > 20 THEN 'Long ID'
      WHEN LENGTH(lifecycle_id) > 10 THEN 'Medium ID'
      ELSE 'Short ID'
    END as id_length_category,
    CONCAT(SUBSTR(lifecycle_id, 1, 3), '...', SUBSTR(lifecycle_id, -3)) as masked_id
  FROM event_store
  WHERE lifecycle_id IS NOT NULL
    AND event_type IS NOT NULL
    AND REGEXP_CONTAINS(lifecycle_id, r'^[A-Za-z0-9]+$')
)
SELECT *
FROM filtered_events
ORDER BY sender_transaction_amount DESC;
```

## non opt query

```sql
SELECT 
    UPPER(TRIM(sender_transaction_currency)) as clean_currency,
    LOWER(TRIM(event_type)) as clean_event_type,
    COUNT(*) as transaction_count,
    SUM(CAST(sender_transaction_amount AS FLOAT64)) as total_amount,
    AVG(CAST(sender_transaction_amount AS FLOAT64)) as avg_amount,
    CASE 
        WHEN UPPER(TRIM(sender_transaction_currency)) = 'USD' THEN 'US Dollar'
        WHEN UPPER(TRIM(sender_transaction_currency)) = 'EUR' THEN 'Euro'
        WHEN UPPER(TRIM(sender_transaction_currency)) = 'INR' THEN 'Indian Rupee'
        ELSE CONCAT('Other: ', UPPER(TRIM(sender_transaction_currency)))
    END as currency_name
FROM event_store
WHERE sender_transaction_amount IS NOT NULL
    AND TRIM(sender_transaction_currency) != ''
    AND TRIM(event_type) != ''
GROUP BY UPPER(TRIM(sender_transaction_currency)), LOWER(TRIM(event_type))
ORDER BY total_amount DESC;
```


-- opt query

WITH cleaned_data AS (
  SELECT 
    UPPER(TRIM(sender_transaction_currency)) AS clean_currency,
    LOWER(TRIM(event_type)) AS clean_event_type,
    sender_transaction_amount
  FROM event_store
  WHERE sender_transaction_amount IS NOT NULL
    AND TRIM(sender_transaction_currency) != ''
    AND TRIM(event_type) != ''
)

SELECT 
  clean_currency,
  clean_event_type,
  COUNT(*) AS transaction_count,
  SUM(CAST(sender_transaction_amount AS FLOAT64)) AS total_amount,
  AVG(CAST(sender_transaction_amount AS FLOAT64)) AS avg_amount,
  CASE 
    WHEN clean_currency = 'USD' THEN 'US Dollar'
    WHEN clean_currency = 'EUR' THEN 'Euro'
    WHEN clean_currency = 'INR' THEN 'Indian Rupee'
    ELSE CONCAT('Other: ', clean_currency)
  END AS currency_name
FROM cleaned_data
GROUP BY clean_currency, clean_event_type
ORDER BY total_amount DESC;
