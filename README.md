# Sample api extractor

```
import requests
import csv

# List of endpoints and their payloads
endpoints = [
    {
        "url": "http://127.0.0.1:8000/get-recommendations",
        "payload": {"market": "some_market"}
    },
    {
        "url": "http://127.0.0.1:8000/rule",
        "payload": {"market": "some_market", "rule_id": "4", "page": 1, "page_size": 4}
    }
]

headers = {
    "accept": "application/json",
    "Content-Type": "application/json"
}

def fetch_and_save_to_csv(url, payload, filename):
    try:
        response = requests.post(url, json=payload, headers=headers)
        response.raise_for_status()
        data = response.json()

        if not isinstance(data, list):
            print(f"Response from {url} is not a JSON array.")
            return

        if len(data) == 0:
            print(f"No data returned from {url}")
            return

        # Write to CSV
        with open(filename, mode="w", newline="", encoding="utf-8") as f:
            writer = csv.DictWriter(f, fieldnames=data[0].keys())
            writer.writeheader()
            writer.writerows(data)

        print(f"Data from {url} saved to {filename}")

    except Exception as e:
        print(f"Error fetching from {url}: {e}")

# Loop through endpoints
for idx, ep in enumerate(endpoints, start=1):
    fetch_and_save_to_csv(ep["url"], ep["payload"], f"output_{idx}.csv")
```


# BigQuery Optimization Playbook
Practical guide for fixing performance, cost, and storage inefficiencies detected from BigQuery query logs.

1. Truncate Tables
Recommendation:
Use TRUNCATE TABLE instead of DELETE without a filter condition.

Reason:
TRUNCATE is faster, cheaper, and doesn’t scan the table.

Step-by-Step Implementation:

Identify queries with DELETE and no WHERE clause.

Replace them with TRUNCATE TABLE.

Ensure no business logic relies on delete logs for auditing.

Example SQL:

sql
Copy
Edit
-- ❌ Inefficient
DELETE FROM `project.dataset.my_table`;

-- ✅ Optimized
TRUNCATE TABLE `project.dataset.my_table`;
GCP Console Steps:

Go to BigQuery → Select Dataset → Select Table.

Click Query Table → Enter TRUNCATE TABLE statement → Run.

2. IN with Constants
Recommendation:
Use UNNEST with arrays or join with lookup tables instead of long IN lists.

Reason:
Long IN clauses increase parsing overhead and scan time.

Step-by-Step Implementation:

Replace IN ('A','B','C') with UNNEST arrays.

For large lists, create a lookup table and JOIN.

Example SQL:

sql
Copy
Edit
-- ❌ Avoid
SELECT * 
FROM `project.dataset.my_table`
WHERE country IN ('US', 'UK', 'IN', 'CA', 'AU');

-- ✅ Optimized with UNNEST
SELECT * 
FROM `project.dataset.my_table`
WHERE country IN UNNEST(['US', 'UK', 'IN', 'CA', 'AU']);

-- ✅ Optimized with lookup table
SELECT t.*
FROM `project.dataset.my_table` t
JOIN `project.dataset.country_lookup` c
ON t.country = c.country_code;
GCP Console Steps:

Create a Lookup Table:

Go to BigQuery Console.

Click Create Table → Enter values manually or upload CSV.

Use it in a JOIN.

3. IN Clause with Subquery
Recommendation:
Use JOIN or EXISTS instead of IN with subqueries.

Reason:
JOIN and EXISTS often generate more efficient execution plans.

Step-by-Step Implementation:

Identify subqueries in IN.

Rewrite with JOIN or EXISTS.

Example SQL:

sql
Copy
Edit
-- ❌ Avoid
SELECT *
FROM orders
WHERE customer_id IN (
  SELECT customer_id
  FROM high_value_customers
);

-- ✅ Use JOIN
SELECT o.*
FROM orders o
JOIN high_value_customers h
ON o.customer_id = h.customer_id;

-- ✅ Use EXISTS
SELECT *
FROM orders o
WHERE EXISTS (
  SELECT 1
  FROM high_value_customers h
  WHERE o.customer_id = h.customer_id
);
GCP Console Steps:

Run the query in BigQuery Query Editor.

Check the Execution Details tab to compare performance.

4. Frequent Failures
Recommendation:
Investigate root cause, refactor logic, and add monitoring.

Reason:
Repeated failures waste resources and increase costs.

Step-by-Step Implementation:

Use Cloud Logging to find failed jobs.

Add error handling and query validation.

Break complex queries into smaller steps.

Use dry run to validate before execution:

bash
Copy
Edit
bq query --dry_run --use_legacy_sql=false 'SELECT * FROM dataset.table'
GCP Console Steps:

Go to BigQuery → Job History.

Filter by Failed.

Review Execution Details and error messages.

5. Jobs Failing Due to Resource Errors
Recommendation:
Fix data skew, repartition, and refactor queries.

Reason:
Uneven data distribution and poor partitioning can cause slot exhaustion.

Step-by-Step Implementation:

Check partition distribution:

sql
Copy
Edit
SELECT partition_field, COUNT(*)
FROM `project.dataset.table`
GROUP BY partition_field;
Use PARTITION BY and CLUSTER BY.

Avoid ORDER BY without LIMIT.

Example SQL:

sql
Copy
Edit
CREATE TABLE `project.dataset.optimized_table`
PARTITION BY DATE(order_date)
CLUSTER BY customer_id AS
SELECT * FROM source_table;
GCP Console Steps:

In Table Details, check Partitioning and Clustering.

Use Edit Schema to add partition fields.

6. High Volume Scan Jobs
Recommendation:
Apply column pruning, filters, and table partitioning.

Reason:
Scanning unnecessary data increases cost and execution time.

Step-by-Step Implementation:

Select only needed columns.

Apply filters early.

Partition & cluster large tables.

Example SQL:

sql
Copy
Edit
-- ❌ Avoid
SELECT * FROM `project.dataset.big_table`;

-- ✅ Optimized
SELECT col1, col2
FROM `project.dataset.big_table`
WHERE date >= '2024-01-01';
GCP Console Steps:

In the Query Editor, click Query settings → Enable Query validator to check scanned bytes.

7. Schema Duplication
Recommendation:
Use views instead of duplicating schema.

Reason:
Views avoid storage duplication and ensure a single source of truth.

Step-by-Step Implementation:

Identify duplicate schemas.

Drop duplicates, create a VIEW.

Example SQL:

sql
Copy
Edit
CREATE OR REPLACE VIEW `project.dataset.sales_view` AS
SELECT *
FROM `project.dataset.sales_table`;
GCP Console Steps:

Go to Dataset → Create View.

Paste SQL and save.

8. Table Cloning
Recommendation:
Use table clones for dev/test instead of full copies.

Reason:
Clones have zero initial storage cost.

Step-by-Step Implementation:

Identify CREATE TABLE AS SELECT used for exact copies.

Replace with CLONE.

Example SQL:

sql
Copy
Edit
CREATE TABLE `project.dataset.clone_table`
CLONE `project.dataset.original_table`;
GCP Console Steps:

Go to Table → Copy Table → Select Clone option.
