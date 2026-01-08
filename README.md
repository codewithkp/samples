```python
import json
import sys

def load_json_array(file_path):
    with open(file_path, "r", encoding="utf-8") as f:
        data = json.load(f)
        if not isinstance(data, list):
            raise ValueError(f"{file_path} does not contain a JSON array")
        return data

def index_by_key(records, key):
    index = {}
    for record in records:
        if key not in record:
            raise KeyError(f"Missing unique key '{key}' in record: {record}")
        index[record[key]] = record
    return index

def compare_files(file1, file2, unique_key):
    data1 = index_by_key(load_json_array(file1), unique_key)
    data2 = index_by_key(load_json_array(file2), unique_key)

    all_ids = set(data1.keys()) | set(data2.keys())

    print("uniqueid,fieldname,file1Value,file2Value")

    for uid in sorted(all_ids):
        rec1 = data1.get(uid, {})
        rec2 = data2.get(uid, {})

        all_fields = set(rec1.keys()) | set(rec2.keys())

        for field in all_fields:
            if field == unique_key:
                continue

            val1 = rec1.get(field)
            val2 = rec2.get(field)

            if val1 != val2:
                print(f"{uid},{field},{json.dumps(val1)},{json.dumps(val2)}")

if __name__ == "__main__":
    if len(sys.argv) != 4:
        print("Usage: python compare_json.py file1.json file2.json unique_key")
        sys.exit(1)

    file1_path = sys.argv[1]
    file2_path = sys.argv[2]
    unique_key = sys.argv[3]

    compare_files(file1_path, file2_path, unique_key)
```
