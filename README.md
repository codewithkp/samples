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
