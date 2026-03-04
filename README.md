```python
import os
import requests
import sqlparse
import pandas as pd

# -----------------------------
# Configuration
# -----------------------------
API_URL = "https://my-api:8000/rule"

PAYLOAD = {
    "market": "val1",
    "page": 1,
    "page_size": 10650,
    "project_name": "val2",
    "rule_id": "7",
    "version_id": "30"
}

OUTPUT_SQL_DIR = "queries"
SUMMARY_CSV_FILE = "summary.csv"

# Create directory if not exists
os.makedirs(OUTPUT_SQL_DIR, exist_ok=True)


# -----------------------------
# Step 1: Call API
# -----------------------------
def fetch_records():
    try:
        response = requests.post(API_URL, json=PAYLOAD, verify=False)  # verify=False if internal SSL
        response.raise_for_status()
        return response.json().get("data", [])
    except requests.exceptions.RequestException as e:
        print(f"Error calling API: {e}")
        return []


# -----------------------------
# Step 2: Format SQL
# -----------------------------
def format_sql(query):
    return sqlparse.format(
        query,
        reindent=True,
        keyword_case="upper",
        strip_comments=False
    )


# -----------------------------
# Step 3: Process Records
# -----------------------------
def process_records(records):
    summary_data = []

    for record in records:
        log_id = record.get("log_id")
        query = record.get("query", "")

        if not log_id:
            print("Skipping record without log_id")
            continue

        # Format SQL
        formatted_query = format_sql(query)

        # Write SQL to file
        sql_filename = os.path.join(OUTPUT_SQL_DIR, f"{log_id}.sql")
        with open(sql_filename, "w", encoding="utf-8") as f:
            f.write(formatted_query)

        # Remove query field for summary
        record_copy = record.copy()
        record_copy.pop("query", None)

        summary_data.append(record_copy)

    return summary_data


# -----------------------------
# Step 4: Write Summary CSV
# -----------------------------
def write_summary_csv(summary_data):
    if summary_data:
        df = pd.DataFrame(summary_data)
        df.to_csv(SUMMARY_CSV_FILE, index=False)
        print(f"Summary CSV written to {SUMMARY_CSV_FILE}")
    else:
        print("No data to write to CSV")


# -----------------------------
# Main Execution
# -----------------------------
if __name__ == "__main__":
    records = fetch_records()
    print(f"Fetched {len(records)} records")

    summary_data = process_records(records)
    write_summary_csv(summary_data)

    print("Processing completed.")
```
