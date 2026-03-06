```python
import re

def transform_query(query):

    if not query:
        return query

    # ---------------------------------
    # 1 Remove comments
    # ---------------------------------
    query = re.sub(r"--.*?$", "", query, flags=re.MULTILINE)
    query = re.sub(r"/\*.*?\*/", "", query, flags=re.DOTALL)

    # ---------------------------------
    # 2 DELETE -> SELECT
    # ---------------------------------
    delete_pattern = re.compile(r"^\s*DELETE\s+FROM", re.IGNORECASE)
    if delete_pattern.search(query):
        query = delete_pattern.sub("SELECT * FROM", query)

    # ---------------------------------
    # 3 INSERT INTO -> Extract SELECT
    # ---------------------------------
    insert_pattern = re.compile(
        r"INSERT\s+INTO\s+[^\(]+\([^\)]*\)\s*(SELECT.*)",
        re.IGNORECASE | re.DOTALL
    )

    match = insert_pattern.search(query)

    if match:
        query = match.group(1)

    # ---------------------------------
    # 4 MERGE -> Extract SELECT from USING
    # Pattern:
    # MERGE INTO table USING (SELECT ...) AS src
    # ---------------------------------
    merge_pattern = re.compile(
        r"MERGE\s+INTO\s+.*?USING\s*\((SELECT.*?)\)\s+AS",
        re.IGNORECASE | re.DOTALL
    )

    match = merge_pattern.search(query)

    if match:
        query = match.group(1)

    # ---------------------------------
    # 5 Environment replacements
    # ---------------------------------
    query = query.replace("-prod", "-dev")
    query = query.replace("_PROD", "_DEV_SIT")

    return query.strip()
```
