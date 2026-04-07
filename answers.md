# API Ethics Assignment

## Task 1 — Classify and Handle PII Fields

| Field Name        | PII Type        | Action           |
|------------------|----------------|------------------|
| full_name        | Direct PII     | Drop             |
| email            | Direct PII     | Drop             |
| date_of_birth    | Indirect PII   | Mask (convert to age) |
| zip_code         | Indirect PII   | Generalize (first 3 digits) |
| job_title        | Indirect PII   | Keep             |
| diagnosis_notes  | Sensitive Data | Pseudonymize     |

---

## Task 2 — Ethical Issues and Corrected Code

### Issue 1: No Rate Limit Handling

```python
import requests
import time

API_URL = "https://healthstats-api.example.com/records"
API_KEY = "free_tier_key_abc123"

records = []

for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    
    if response.status_code == 429:
        time.sleep(60)
        continue

    data = response.json()
    records.extend(data["results"])
    
    time.sleep(1)


# Issue 2: Storing Raw Sensitive Data
from datetime import datetime

def calculate_age(dob):
    birth_year = datetime.strptime(dob, "%Y-%m-%d").year
    return datetime.now().year - birth_year

def anonymize_text(text):
    return text.replace("patient", "individual")

def clean_record(record):
    return {
        "age": calculate_age(record["date_of_birth"]),
        "region": record["zip_code"][:3],
        "job_title": record["job_title"],
        "diagnosis_notes": anonymize_text(record["diagnosis_notes"])
    }

cleaned_records = [clean_record(r) for r in records]

save_to_database(cleaned_records)
