```python
import os
import requests
import pandas as pd
import sqlglot
from sqlglot import parse_one
from sqlglot.errors import ParseError

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

os.makedirs(OUTPUT_SQL_DIR, exist_ok=True)


# -----------------------------
# Step 1: Call API
# -----------------------------
def fetch_records():
    try:
        response = requests.post(API_URL, json=PAYLOAD, verify=False)
        response.raise_for_status()
        return response.json().get("data", [])
    except requests.exceptions.RequestException as e:
        print(f"❌ API Error: {e}")
        return []


# -----------------------------
# Step 2: Format SQL using sqlglot
# -----------------------------
def format_sql(query):
    if not query:
        return query

    # Ignore Stored Procedure Calls
    if query.strip().upper().startswith("CALL"):
        print("ℹ Skipping formatting for stored procedure CALL")
        return query

    try:
        parsed = parse_one(query, read="bigquery")
        formatted = parsed.sql(
            pretty=True,
            dialect="bigquery"
        )
        return formatted

    except ParseError as pe:
        print(f"⚠ Parse error while formatting SQL: {pe}")
        return query  # fallback to raw query

    except Exception as e:
        print(f"⚠ Unexpected formatting error: {e}")
        return query


# -----------------------------
# Step 3: Process Records
# -----------------------------
def process_records(records):
    summary_data = []

    for record in records:
        log_id = record.get("log_id")
        query = record.get("query", "")

        if not log_id:
            print("⚠ Skipping record without log_id")
            continue

        # Format SQL safely
        formatted_query = format_sql(query)

        # Write SQL to file
        sql_filename = os.path.join(OUTPUT_SQL_DIR, f"{log_id}.sql")
        try:
            with open(sql_filename, "w", encoding="utf-8") as f:
                f.write(formatted_query)
        except Exception as e:
            print(f"⚠ Failed writing file {sql_filename}: {e}")

        # Prepare summary (exclude query)
        record_copy = record.copy()
        record_copy.pop("query", None)
        summary_data.append(record_copy)

    return summary_data


# -----------------------------
# Step 4: Write Summary CSV
# -----------------------------
def write_summary_csv(summary_data):
    if not summary_data:
        print("No data available for summary CSV")
        return

    try:
        df = pd.DataFrame(summary_data)
        df.to_csv(SUMMARY_CSV_FILE, index=False)
        print(f"✅ Summary CSV written to {SUMMARY_CSV_FILE}")
    except Exception as e:
        print(f"❌ Failed writing summary CSV: {e}")


# -----------------------------
# Main Execution
# -----------------------------
if __name__ == "__main__":
    records = fetch_records()
    print(f"Fetched {len(records)} records")

    summary_data = process_records(records)
    write_summary_csv(summary_data)

    print("✅ Processing completed.")
```
