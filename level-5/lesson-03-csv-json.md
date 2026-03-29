# 🏛️ Level 5 – Innovating | Lesson 3: Handle CSV or JSON Files with Python Modules

## *"The Paper Trail"*

> **O'Reilly Skill Objective:** *Parse and manipulate structured data files like CSV or JSON using modules. Serialize and deserialize data using JSON.*

> **Previously Learned:** File I/O, `open()`, context managers, dictionaries, lists, exception handling, `__repr__`, classes

---

## 📖 Table of Contents

1. [Opening Scene](#1--opening-scene)
2. [Concept Introduction – Harvey Specter](#2--concept-introduction--harvey-specter)
3. [Technical Breakdown – Mike Ross](#3--technical-breakdown--mike-ross)
4. [Edge Cases & Pitfalls – Louis Litt](#4--edge-cases--pitfalls--louis-litt)
5. [Mental Model – Donna Paulsen](#5--mental-model--donna-paulsen)
6. [Practice Exercises – Rachel Zane](#6--practice-exercises--rachel-zane)
7. [Debugging Scenario](#7--debugging-scenario)
8. [Real-World Application](#8--real-world-application)
9. [Knowledge Check](#9--knowledge-check)

---

## 1. 🎬 Opening Scene

*Opposing counsel delivers evidence as 5,000 CSV rows and the API returns billing data as JSON. Louis needs both processed by morning.*

```python
# ❌ Parsing CSV by hand
with open("evidence.csv") as f:
    for line in f:
        parts = line.strip().split(",")    # Breaks on commas inside quotes!
        name = parts[0]                     # Index-based — fragile!

# ❌ Building JSON by hand
json_string = '{"name": "' + name + '"}'   # Injection risk, escaping nightmare!
```

**Mike:** "Python has `csv` and `json` modules built in. They handle every edge case — quoted fields, nested structures, encoding, escaping — all of it."

```python
import csv, json

# ✅ CSV: headers become dict keys
with open("evidence.csv") as f:
    for row in csv.DictReader(f):
        print(row["client_name"], row["amount"])

# ✅ JSON: Python dicts ↔ JSON strings
data = json.loads('{"name": "Wayne", "revenue": 2500000}')
print(data["name"])    # Wayne
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "CSV is a spreadsheet as text. JSON is a data structure as text. Python reads and writes both natively."

| Format | Best For | Python Module |
|--------|----------|:-------------:|
| **CSV** | Tabular data (rows & columns) | `csv` |
| **JSON** | Nested/hierarchical data | `json` |

| `csv` Function | Purpose |
|----------------|---------|
| `csv.reader()` | Read rows as lists |
| `csv.DictReader()` | Read rows as dictionaries |
| `csv.writer()` | Write rows from lists |
| `csv.DictWriter()` | Write rows from dictionaries |

| `json` Function | Purpose |
|-----------------|---------|
| `json.loads(s)` | Parse JSON string → Python object |
| `json.dumps(obj)` | Python object → JSON string |
| `json.load(f)` | Parse JSON file → Python object |
| `json.dump(obj, f)` | Python object → JSON file |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Reading CSV Files ⭐

```python
import csv

# === csv.reader — rows as lists ===
# Sample CSV content (clients.csv):
# name,revenue,tier,active
# Wayne Enterprises,2500000,Platinum,true
# Stark Industries,1800000,Gold,true
# Oscorp,600000,Silver,false

with open("clients.csv", newline='') as f:
    reader = csv.reader(f)
    
    header = next(reader)    # First row is the header
    print(f"Columns: {header}")
    # Columns: ['name', 'revenue', 'tier', 'active']
    
    for row in reader:
        name, revenue, tier, active = row
        print(f"  {name}: ${int(revenue):,} [{tier}]")

# === csv.DictReader — rows as dicts (preferred!) ===
with open("clients.csv", newline='') as f:
    reader = csv.DictReader(f)
    
    for row in reader:
        # Each row is an OrderedDict — access by column name!
        print(f"  {row['name']}: ${int(row['revenue']):,} [{row['tier']}]")

# DictReader advantages:
# ✅ Access by column NAME, not index
# ✅ Handles header row automatically
# ✅ Survives column reordering
# ✅ Self-documenting code
```

---

### 3.2 Writing CSV Files ⭐

```python
import csv

# === csv.writer — from lists ===
clients = [
    ["Wayne Enterprises", 2500000, "Platinum", True],
    ["Stark Industries", 1800000, "Gold", True],
    ["Oscorp", 600000, "Silver", False],
]

with open("output.csv", "w", newline='') as f:
    writer = csv.writer(f)
    writer.writerow(["name", "revenue", "tier", "active"])    # Header
    writer.writerows(clients)                                  # All data rows

# === csv.DictWriter — from dicts (preferred!) ===
clients = [
    {"name": "Wayne Enterprises", "revenue": 2500000, "tier": "Platinum"},
    {"name": "Stark Industries", "revenue": 1800000, "tier": "Gold"},
    {"name": "Oscorp", "revenue": 600000, "tier": "Silver"},
]

with open("output.csv", "w", newline='') as f:
    fieldnames = ["name", "revenue", "tier"]
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    
    writer.writeheader()      # Write column names
    writer.writerows(clients) # Write all rows

# Result (output.csv):
# name,revenue,tier
# Wayne Enterprises,2500000,Platinum
# Stark Industries,1800000,Gold
# Oscorp,600000,Silver
```

---

### 3.3 CSV Dialects and Options ⭐

```python
import csv

# === Custom delimiters ===
# Tab-separated (TSV):
with open("data.tsv", newline='') as f:
    reader = csv.reader(f, delimiter='\t')
    for row in reader:
        print(row)

# Semicolon-separated (European):
with open("data.csv", newline='') as f:
    reader = csv.reader(f, delimiter=';')

# Pipe-separated:
with open("data.csv", newline='') as f:
    reader = csv.reader(f, delimiter='|')

# === Quoting options ===
with open("output.csv", "w", newline='') as f:
    writer = csv.writer(f, quoting=csv.QUOTE_ALL)
    writer.writerow(["name", "note"])
    writer.writerow(["Harvey", "Said, \"win it\""])
    # Output: "Harvey","Said, ""win it"""

# Quoting constants:
# csv.QUOTE_MINIMAL   — quote only when needed (default)
# csv.QUOTE_ALL       — quote everything
# csv.QUOTE_NONNUMERIC — quote non-numeric fields
# csv.QUOTE_NONE      — never quote (requires escapechar)

# === Detect dialect automatically ===
with open("unknown.csv", newline='') as f:
    sample = f.read(2048)
    dialect = csv.Sniffer().sniff(sample)
    f.seek(0)
    reader = csv.reader(f, dialect)
```

---

### 3.4 Parsing JSON — `json.loads()` and `json.load()` ⭐⭐

```python
import json

# === json.loads() — parse a JSON STRING ===
json_string = '''
{
    "firm": "Pearson Specter Litt",
    "partners": ["Harvey", "Louis", "Jessica"],
    "revenue": 45000000,
    "active": true,
    "address": null
}
'''

data = json.loads(json_string)
print(type(data))           # <class 'dict'>
print(data["firm"])          # Pearson Specter Litt
print(data["partners"][0])   # Harvey
print(data["revenue"])       # 45000000
print(data["active"])        # True (JSON true → Python True)
print(data["address"])       # None (JSON null → Python None)

# === json.load() — parse a JSON FILE ===
with open("firm_data.json") as f:
    data = json.load(f)

# === JSON → Python type mapping ===
# JSON           Python
# object {}      dict
# array []       list
# string ""      str
# number (int)   int
# number (float) float
# true/false     True/False
# null           None
```

---

### 3.5 Creating JSON — `json.dumps()` and `json.dump()` ⭐⭐

```python
import json

# === json.dumps() — Python → JSON STRING ===
data = {
    "client": "Wayne Enterprises",
    "revenue": 2500000,
    "active": True,
    "contacts": ["Bruce", "Lucius"],
    "notes": None,
}

# Basic
json_string = json.dumps(data)
print(json_string)
# {"client": "Wayne Enterprises", "revenue": 2500000, "active": true, "contacts": ["Bruce", "Lucius"], "notes": null}

# Pretty-printed
json_pretty = json.dumps(data, indent=2)
print(json_pretty)
# {
#   "client": "Wayne Enterprises",
#   "revenue": 2500000,
#   "active": true,
#   "contacts": [
#     "Bruce",
#     "Lucius"
#   ],
#   "notes": null
# }

# Sorted keys (useful for diffing/testing)
json_sorted = json.dumps(data, indent=2, sort_keys=True)

# === json.dump() — Python → JSON FILE ===
with open("output.json", "w") as f:
    json.dump(data, f, indent=2)

# === Useful parameters ===
# indent=2         → pretty-print with 2-space indentation
# sort_keys=True   → alphabetical key order
# ensure_ascii=False → allow Unicode characters (not \uXXXX)
# default=str      → fallback serializer for unsupported types
# separators=(',', ':')  → compact output (no spaces)
```

---

### 3.6 Custom JSON Serialization ⭐⭐

```python
import json
from datetime import datetime, date
from decimal import Decimal

# ❌ These types aren't JSON-serializable by default!
data = {
    "date": datetime.now(),
    "amount": Decimal("38250.50"),
}
# json.dumps(data)    # TypeError: Object of type datetime is not JSON serializable!

# === Solution 1: default function ===
def json_serializer(obj):
    """Custom serializer for types json doesn't handle."""
    if isinstance(obj, (datetime, date)):
        return obj.isoformat()
    if isinstance(obj, Decimal):
        return float(obj)
    if isinstance(obj, set):
        return list(obj)
    raise TypeError(f"Type {type(obj)} not serializable")

json_string = json.dumps(data, default=json_serializer, indent=2)
print(json_string)
# {
#   "date": "2026-03-29T16:08:42.123456",
#   "amount": 38250.5
# }

# === Solution 2: Custom JSONEncoder class ===
class FirmEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, (datetime, date)):
            return obj.isoformat()
        if isinstance(obj, Decimal):
            return str(obj)
        if isinstance(obj, set):
            return sorted(list(obj))
        if hasattr(obj, 'to_dict'):
            return obj.to_dict()
        return super().default(obj)

json_string = json.dumps(data, cls=FirmEncoder, indent=2)

# === Solution 3: Make your classes JSON-friendly ===
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue
    
    def to_dict(self):
        return {"name": self.name, "revenue": self.revenue}

wayne = Client("Wayne", 2500000)
json.dumps(wayne.to_dict())    # ✅
json.dumps(wayne, cls=FirmEncoder)    # ✅ Uses to_dict via encoder
```

---

### 3.7 Custom JSON Deserialization ⭐

```python
import json
from datetime import datetime

# === object_hook — transform dicts during parsing ===
json_string = '''
{
    "name": "Wayne Enterprises",
    "revenue": 2500000,
    "created_at": "2026-01-15T10:30:00"
}
'''

def date_parser(dct):
    """Convert ISO date strings back to datetime objects."""
    for key, value in dct.items():
        if isinstance(value, str):
            try:
                dct[key] = datetime.fromisoformat(value)
            except (ValueError, TypeError):
                pass
    return dct

data = json.loads(json_string, object_hook=date_parser)
print(type(data["created_at"]))    # <class 'datetime.datetime'>

# === object_pairs_hook — get items in order ===
from collections import OrderedDict

data = json.loads(json_string, object_pairs_hook=OrderedDict)

# === Deserialize to custom class ===
class Client:
    def __init__(self, name, revenue, created_at=None):
        self.name = name
        self.revenue = revenue
        self.created_at = created_at
    
    @classmethod
    def from_json(cls, json_string):
        data = json.loads(json_string)
        if 'created_at' in data:
            data['created_at'] = datetime.fromisoformat(data['created_at'])
        return cls(**data)

wayne = Client.from_json(json_string)
print(f"{wayne.name}: ${wayne.revenue:,}")
```

---

### 3.8 Working with Nested JSON ⭐

```python
import json

# Real-world JSON is often deeply nested
firm_data = {
    "firm": "Pearson Specter Litt",
    "offices": [
        {
            "city": "New York",
            "attorneys": [
                {"name": "Harvey", "role": "Partner", "cases": 15},
                {"name": "Mike", "role": "Associate", "cases": 22},
            ]
        },
        {
            "city": "Boston",
            "attorneys": [
                {"name": "Jessica", "role": "Managing Partner", "cases": 8},
            ]
        }
    ],
    "financials": {
        "2026": {"Q1": 12000000, "Q2": 14000000},
        "2025": {"Q1": 10000000, "Q2": 11000000}
    }
}

# Navigate nested structure
for office in firm_data["offices"]:
    print(f"\n📍 {office['city']}:")
    for attorney in office["attorneys"]:
        print(f"  {attorney['name']} ({attorney['role']}) — {attorney['cases']} cases")

# Access deeply nested data safely
def safe_get(data, *keys, default=None):
    """Safely navigate nested dicts/lists."""
    current = data
    for key in keys:
        try:
            current = current[key]
        except (KeyError, IndexError, TypeError):
            return default
    return current

q1_2026 = safe_get(firm_data, "financials", "2026", "Q1")
print(f"\nQ1 2026 revenue: ${q1_2026:,}")

missing = safe_get(firm_data, "financials", "2027", "Q1", default=0)
print(f"Q1 2027 revenue: ${missing}")    # $0 (not found)
```

---

### 3.9 CSV ↔ JSON Conversion

```python
import csv
import json

# === CSV → JSON ===
def csv_to_json(csv_path, json_path):
    """Convert a CSV file to JSON."""
    records = []
    with open(csv_path, newline='') as f:
        reader = csv.DictReader(f)
        for row in reader:
            # Auto-convert numeric fields
            for key, value in row.items():
                try:
                    row[key] = int(value)
                except ValueError:
                    try:
                        row[key] = float(value)
                    except ValueError:
                        if value.lower() in ('true', 'false'):
                            row[key] = value.lower() == 'true'
            records.append(row)
    
    with open(json_path, 'w') as f:
        json.dump(records, f, indent=2)
    
    return len(records)

# === JSON → CSV ===
def json_to_csv(json_path, csv_path):
    """Convert a JSON array to CSV."""
    with open(json_path) as f:
        records = json.load(f)
    
    if not records:
        return 0
    
    fieldnames = list(records[0].keys())
    
    with open(csv_path, 'w', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(records)
    
    return len(records)

# count = csv_to_json("clients.csv", "clients.json")
# count = json_to_csv("clients.json", "clients_copy.csv")
```

---

### 3.10 Solving the Firm's Problem — Evidence and Billing Pipeline

```python
# ============================================
# Pearson Specter Litt — Data Pipeline
# CSV evidence + JSON billing → Combined report
# ============================================

import csv
import json
import io
from datetime import datetime
from collections import defaultdict

# === Simulate CSV evidence data ===
EVIDENCE_CSV = """case_id,exhibit_num,description,date_filed,relevance_score
WAYNE-2026-001,EX-001,Financial records 2024-2025,2026-01-15,0.95
WAYNE-2026-001,EX-002,Email correspondence re: merger,2026-01-20,0.88
WAYNE-2026-001,EX-003,Board meeting minutes,2026-02-01,0.72
STARK-2026-002,EX-001,Patent filing documentation,2026-01-10,0.91
STARK-2026-002,EX-002,Prior art comparison,2026-01-25,0.85
STARK-2026-002,EX-003,Expert witness report,2026-02-15,0.94
OSCORP-2026-003,EX-001,Restructuring proposal,2026-02-01,0.78
OSCORP-2026-003,EX-002,Creditor negotiations log,2026-02-20,0.82
"""

# === Simulate JSON billing data ===
BILLING_JSON = '''
{
    "billing_period": "2026-Q1",
    "cases": [
        {
            "case_id": "WAYNE-2026-001",
            "client": "Wayne Enterprises",
            "attorney": "Harvey Specter",
            "entries": [
                {"date": "2026-01-15", "hours": 8.5, "rate": 450, "description": "Document review"},
                {"date": "2026-01-20", "hours": 6.0, "rate": 450, "description": "Email analysis"},
                {"date": "2026-02-01", "hours": 4.0, "rate": 450, "description": "Board minutes review"}
            ]
        },
        {
            "case_id": "STARK-2026-002",
            "client": "Stark Industries",
            "attorney": "Harvey Specter",
            "entries": [
                {"date": "2026-01-10", "hours": 12.0, "rate": 450, "description": "Patent analysis"},
                {"date": "2026-01-25", "hours": 5.5, "rate": 450, "description": "Prior art research"}
            ]
        },
        {
            "case_id": "OSCORP-2026-003",
            "client": "Oscorp",
            "attorney": "Mike Ross",
            "entries": [
                {"date": "2026-02-01", "hours": 10.0, "rate": 300, "description": "Restructuring plan"},
                {"date": "2026-02-20", "hours": 7.5, "rate": 300, "description": "Creditor meetings"}
            ]
        }
    ]
}
'''


def process_evidence(csv_data):
    """Parse evidence CSV into structured data."""
    evidence = defaultdict(list)
    reader = csv.DictReader(io.StringIO(csv_data))
    
    for row in reader:
        row['relevance_score'] = float(row['relevance_score'])
        evidence[row['case_id']].append(row)
    
    return dict(evidence)


def process_billing(json_data):
    """Parse billing JSON into structured data."""
    billing = json.loads(json_data)
    
    case_billing = {}
    for case in billing["cases"]:
        case_id = case["case_id"]
        total_hours = sum(e["hours"] for e in case["entries"])
        total_amount = sum(e["hours"] * e["rate"] for e in case["entries"])
        
        case_billing[case_id] = {
            "client": case["client"],
            "attorney": case["attorney"],
            "entries": case["entries"],
            "total_hours": total_hours,
            "total_amount": total_amount,
        }
    
    return case_billing


def generate_report(evidence, billing):
    """Combine evidence and billing into a unified report."""
    report = []
    
    all_cases = set(list(evidence.keys()) + list(billing.keys()))
    
    for case_id in sorted(all_cases):
        case_evidence = evidence.get(case_id, [])
        case_billing = billing.get(case_id, {})
        
        report.append({
            "case_id": case_id,
            "client": case_billing.get("client", "Unknown"),
            "attorney": case_billing.get("attorney", "Unassigned"),
            "exhibits": len(case_evidence),
            "avg_relevance": (
                sum(e["relevance_score"] for e in case_evidence) / len(case_evidence)
                if case_evidence else 0
            ),
            "total_hours": case_billing.get("total_hours", 0),
            "total_billed": case_billing.get("total_amount", 0),
        })
    
    return report


# === Run pipeline ===
print("=" * 70)
print(f"{'🏛️  EVIDENCE & BILLING PIPELINE':^70}")
print("=" * 70)

evidence = process_evidence(EVIDENCE_CSV)
billing = process_billing(BILLING_JSON)
report = generate_report(evidence, billing)

# Display
print(f"\n📋 CASE SUMMARY REPORT")
print(f"  {'Case ID':<20} {'Client':<20} {'Exhibits':>8} {'Relevance':>10} {'Hours':>6} {'Billed':>12}")
print(f"  {'─'*20} {'─'*20} {'─'*8} {'─'*10} {'─'*6} {'─'*12}")

for r in report:
    print(f"  {r['case_id']:<20} {r['client']:<20} {r['exhibits']:>8} "
          f"{r['avg_relevance']:>9.2f} {r['total_hours']:>6.1f} ${r['total_billed']:>10,.2f}")

# Totals
total_exhibits = sum(r['exhibits'] for r in report)
total_hours = sum(r['total_hours'] for r in report)
total_billed = sum(r['total_billed'] for r in report)
print(f"\n  {'TOTALS':<42} {total_exhibits:>8} {'':>10} {total_hours:>6.1f} ${total_billed:>10,.2f}")

# Export combined report as JSON
report_output = {
    "generated_at": datetime.now().isoformat(),
    "period": "2026-Q1",
    "cases": report,
    "summary": {
        "total_cases": len(report),
        "total_exhibits": total_exhibits,
        "total_hours": total_hours,
        "total_billed": total_billed,
    }
}

json_output = json.dumps(report_output, indent=2)
print(f"\n📄 JSON REPORT (first 500 chars):")
print(json_output[:500] + "...")

# Export as CSV
csv_buffer = io.StringIO()
writer = csv.DictWriter(csv_buffer, fieldnames=report[0].keys())
writer.writeheader()
writer.writerows(report)
print(f"\n📊 CSV REPORT:")
print(csv_buffer.getvalue())

print(f"{'='*70}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Missing `newline=''` in CSV 🔥

```python
# ❌ Without newline='', you get extra blank lines on Windows!
with open("data.csv", "w") as f:    # ❌
    writer = csv.writer(f)
    writer.writerow(["a", "b"])

# ✅ Always use newline=''
with open("data.csv", "w", newline='') as f:    # ✅
    writer = csv.writer(f)
    writer.writerow(["a", "b"])

# For reading too:
with open("data.csv", newline='') as f:
    reader = csv.reader(f)
```

---

### Pitfall 2: CSV Values Are Always Strings

```python
import csv

# ❌ CSV doesn't preserve types!
with open("data.csv", newline='') as f:
    for row in csv.DictReader(f):
        print(type(row["revenue"]))    # <class 'str'> NOT int!
        # row["revenue"] + 1000        # TypeError!

# ✅ Cast manually
with open("data.csv", newline='') as f:
    for row in csv.DictReader(f):
        revenue = int(row["revenue"])    # Cast to int
        active = row["active"].lower() == "true"    # Cast to bool
```

---

### Pitfall 3: JSON `dumps` vs `dump` (String vs File) 🔥

```python
import json

data = {"name": "Harvey"}

# dumps → returns a STRING (s = string)
json_string = json.dumps(data)

# dump → writes to a FILE (no s)
with open("output.json", "w") as f:
    json.dump(data, f)

# Same for loading:
# loads → parses a STRING
data = json.loads('{"name": "Harvey"}')

# load → reads from a FILE
with open("data.json") as f:
    data = json.load(f)
```

---

### Pitfall 4: Non-Serializable Types

```python
import json
from datetime import datetime

# ❌ datetime, set, Decimal, custom classes are NOT serializable!
data = {"time": datetime.now(), "tags": {1, 2, 3}}
# json.dumps(data)    # TypeError!

# ✅ Use default parameter
json.dumps(data, default=str)    # Fallback: convert to string

# ✅ Better: explicit conversion before serializing
data = {
    "time": datetime.now().isoformat(),
    "tags": sorted(list({1, 2, 3})),
}
json.dumps(data)    # ✅
```

---

### Pitfall 5: Encoding Issues

```python
import csv
import json

# ❌ UnicodeDecodeError on non-UTF-8 files
# with open("data.csv") as f:    # Default encoding may fail

# ✅ Specify encoding
with open("data.csv", encoding="utf-8", newline='') as f:
    reader = csv.reader(f)

# ✅ For legacy files
with open("legacy.csv", encoding="latin-1", newline='') as f:
    reader = csv.reader(f)

# ✅ JSON with non-ASCII characters
data = {"name": "Müller", "city": "São Paulo"}
json.dumps(data, ensure_ascii=False)    # Preserves Unicode: {"name": "Müller"}
json.dumps(data)                         # Escapes: {"name": "M\\u00fcller"}
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📁 Analogy: CSV = Spreadsheet, JSON = Filing Cabinet

```
CSV = A SPREADSHEET:
  ┌──────────┬──────────┬──────┐
  │ name     │ revenue  │ tier │
  ├──────────┼──────────┼──────┤
  │ Wayne    │ 2500000  │ Plat │
  │ Stark    │ 1800000  │ Gold │
  └──────────┴──────────┴──────┘
  Flat. Tabular. Rows and columns.
  Great for: data export, bulk processing, Excel.

JSON = A FILING CABINET:
  📁 Wayne Enterprises
     ├── revenue: $2,500,000
     ├── tier: "Platinum"
     ├── 📁 contacts
     │   ├── Bruce Wayne
     │   └── Lucius Fox
     └── 📁 cases
         ├── 📄 WAYNE-001
         └── 📄 WAYNE-002
  Nested. Hierarchical. Key-value.
  Great for: APIs, configs, complex data.

DONNA'S RULE:
  "CSV for spreadsheets. JSON for everything else.
   If it's flat → CSV. If it's nested → JSON."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: CSV Reader**
```
Read a CSV of 10 clients. Print each as:
  "Client: Wayne Enterprises | Revenue: $2,500,000 | Tier: Platinum"
```

**Exercise 2: JSON Parser**
```
Parse this JSON and print a formatted report:
  {"firm": "PSL", "attorneys": [{"name": "Harvey", "rate": 450}, ...]}
```

**Exercise 3: CSV Writer**
```
Given a list of dicts, write them to a CSV.
Read the CSV back and verify the data matches.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: API Response Handler**
```
Fetch JSON from a public API (or use sample data).
Parse it, filter records, and export as CSV.
```

**Exercise 5: CSV ↔ JSON Converter**
```
Build a CLI tool:
  python convert.py input.csv output.json
  python convert.py input.json output.csv
Auto-detect input format and convert.
```

**Exercise 6: JSON Config Manager**
```
Build a config manager:
  config = Config("settings.json")
  config.get("database.host")
  config.set("database.port", 5432)
  config.save()
Support nested keys with dot notation.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Streaming CSV Processor**
```
Process a 1GB CSV without loading it all in memory:
  - Read row by row
  - Filter, transform, aggregate
  - Write filtered results to new CSV
Track: memory usage, rows processed, time.
```

**Exercise 8: JSON Schema Validator**
```
Build a validator that checks JSON against a schema:
  schema = {"name": str, "age": int, "email": str}
  validate(data, schema)  → True/False + errors
Handle: nested schemas, optional fields, type coercion.
```

**Exercise 9: Data Pipeline Manager**
```
Build a pipeline:
  Pipeline("evidence.csv")
  .read_csv()
  .filter(lambda r: float(r['relevance']) > 0.8)
  .transform(lambda r: {**r, 'flagged': True})
  .to_json("flagged_evidence.json")
  .to_csv("flagged_evidence.csv")
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Extra blank lines in CSV (Windows)
with open("data.csv", "w") as f:
    csv.writer(f).writerow(["a", "b"])    # Extra \r\n on Windows!

# Bug 2: Integer treated as string from CSV
row = {"revenue": "2500000"}
total = row["revenue"] + 1000    # TypeError: can't add str + int!

# Bug 3: datetime not JSON serializable
json.dumps({"time": datetime.now()})    # TypeError!

# Bug 4: Wrong function (dumps vs dump)
with open("data.json", "w") as f:
    f.write(json.dump(data, f))    # json.dump writes to file AND returns None!
    # f.write(None) → TypeError!

# Bug 5: Key error on missing CSV column
for row in csv.DictReader(f):
    print(row["nonexistent"])    # KeyError!
```

### Fixed Code ✅

```python
# Fix 1: Add newline=''
with open("data.csv", "w", newline='') as f:
    csv.writer(f).writerow(["a", "b"])

# Fix 2: Cast to int
total = int(row["revenue"]) + 1000

# Fix 3: Use default or convert
json.dumps({"time": datetime.now().isoformat()})

# Fix 4: Don't wrap dump in write
with open("data.json", "w") as f:
    json.dump(data, f)    # Writes directly to f — don't call f.write()!

# Fix 5: Use .get() with default
print(row.get("nonexistent", "N/A"))
```

---

## 8. 🌍 Real-World Application

### CSV and JSON Everywhere

| Use Case | Format | Why |
|----------|:------:|-----|
| Spreadsheet export | CSV | Universal compatibility |
| API responses | JSON | Standard for web APIs |
| Configuration files | JSON | Human-readable, structured |
| Database import/export | CSV | Bulk data transfer |
| Logging (structured) | JSON | Machine-parseable logs |
| Data science | CSV | Pandas `read_csv()` |
| Package manifests | JSON | `package.json`, `composer.json` |
| Geospatial data | JSON | GeoJSON format |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is the difference between `csv.reader` and `csv.DictReader`?

**Q2.** Why should you pass `newline=''` when opening CSV files?

**Q3.** What is the difference between `json.load` and `json.loads`?

**Q4.** How do you serialize a `datetime` object to JSON?

**Q5.** What Python types map to JSON's `true`, `false`, and `null`?

**Q6.** How do you pretty-print JSON?

**Q7.** What does `csv.DictWriter` require that `csv.writer` doesn't?

**Q8.** How do you handle non-UTF-8 CSV files?

**Q9.** What is `json.dumps(data, default=str)` for?

**Q10.** When should you choose CSV over JSON?

---

### 📋 Answer Key

> **Q1.** `csv.reader` returns each row as a list of strings. `csv.DictReader` returns each row as a dictionary keyed by column headers.

> **Q2.** To prevent the CSV module and Python's newline handling from conflicting, which causes extra blank lines on Windows.

> **Q3.** `json.load(file)` reads from a file object. `json.loads(string)` parses a JSON string. The `s` stands for "string."

> **Q4.** Convert to ISO format string first: `datetime.now().isoformat()`, or use a custom `default` function or `JSONEncoder` subclass.

> **Q5.** `True`, `False`, and `None` respectively.

> **Q6.** `json.dumps(data, indent=2)` — the `indent` parameter adds newlines and indentation.

> **Q7.** A `fieldnames` list — specifying the column names and their order: `DictWriter(f, fieldnames=["name", "revenue"])`.

> **Q8.** Specify the encoding explicitly: `open("file.csv", encoding="latin-1", newline='')`.

> **Q9.** Fallback serializer — converts any type not natively supported by JSON (datetime, set, etc.) to a string using `str()`.

> **Q10.** When data is flat/tabular with uniform columns, for spreadsheet compatibility, for bulk data processing, or when human readability of raw data matters.

---

## 🎬 End of Lesson Scene

**Louis:** "The evidence CSV and billing JSON — are they combined?"

**Mike:** "Processed. 8 exhibits across 3 cases, cross-referenced with $24,075 in billing. Combined report exported as both JSON and CSV. The pipeline runs in under a second."

**Harvey:** "Next: binary files. Not everything is text."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `csv.reader` and `csv.DictReader` | ✅ |
| 2 | `csv.writer` and `csv.DictWriter` | ✅ |
| 3 | CSV dialects and custom delimiters | ✅ |
| 4 | `json.loads` / `json.load` (parsing) | ✅ |
| 5 | `json.dumps` / `json.dump` (creating) | ✅ |
| 6 | Custom JSON serialization | ✅ |
| 7 | Custom JSON deserialization | ✅ |
| 8 | Nested JSON navigation | ✅ |
| 9 | CSV ↔ JSON conversion | ✅ |
| 10 | Combined data pipeline | ✅ |

---

> **Next Lesson →** Level 5, Lesson 4: *Work with Binary Files and Encoding in Python*
