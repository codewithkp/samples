```python
def process_sql_files():

    for filename in os.listdir(OUTPUT_SQL_DIR):

        if not filename.endswith(".sql"):
            continue

        source_file = os.path.join(OUTPUT_SQL_DIR, filename)
        target_file = os.path.join(OUTPUT_DEV_SQL_DIR, filename)

        try:
            with open(source_file, "r", encoding="utf-8") as f:
                query = f.read()

            processed_query = transform_query(query)

            with open(target_file, "w", encoding="utf-8") as f:
                f.write(processed_query)

        except Exception as e:
            print(f"⚠ Error processing {filename}: {e}")
```
