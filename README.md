```python
def categorize_sql_files():

    categories = {
        "SELECT": "SELECT",
        "INSERT": "INSERT",
        "MERGE": "MERGE",
        "UPDATE": "UPDATE",
        "DELETE": "DELETE"
    }

    # Create directories if not present
    for folder in categories.values():
        os.makedirs(os.path.join(OUTPUT_DEV_SQL_DIR, folder), exist_ok=True)

    os.makedirs(os.path.join(OUTPUT_DEV_SQL_DIR, "OTHER"), exist_ok=True)

    for filename in os.listdir(OUTPUT_DEV_SQL_DIR):

        file_path = os.path.join(OUTPUT_DEV_SQL_DIR, filename)

        # Skip directories
        if not filename.endswith(".sql"):
            continue

        try:
            with open(file_path, "r", encoding="utf-8") as f:
                content = f.read().strip()

            if not content:
                continue

            # Get first keyword
            first_word = content.split()[0].upper()

            if first_word in categories:
                target_folder = categories[first_word]
            else:
                target_folder = "OTHER"

            target_path = os.path.join(
                OUTPUT_DEV_SQL_DIR,
                target_folder,
                filename
            )

            shutil.move(file_path, target_path)

        except Exception as e:
            print(f"⚠ Error processing {filename}: {e}")
```
