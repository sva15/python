# 🏛️ Level 2 – Applying | Lesson 10: Store and Retrieve Data with Python Objects

## *"The Vault"*

> **O'Reilly Skill Objective:** *Encode Python objects as JSON and decode JSON strings into Python objects; understand serialization patterns.*

> **Previously Learned:** Variables, data types, lists, tuples, dictionaries, file I/O, classes, objects, attributes, `@classmethod`, copy/deepcopy, mutability

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

*Wednesday. The firm's billing system crashed overnight. Mike runs in.*

**Mike:** "The server restarted and we lost all client data. Every `Client` object, every `Attorney` object — gone."

**Harvey:** "That's because they only exist in RAM. When the process dies, the objects die with it."

**Jessica:** "We need every client record, every case, every billing entry to survive restarts. Store it. Retrieve it. Intact."

**Mike:** "Serialization. Converting Python objects into a format that can be saved to disk and rebuilt later. JSON for data exchange with other systems. Pickle for Python-specific objects. CSV for spreadsheets. And `dataclasses` to make it all clean."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Think of it as archiving case files."

| Concept | Analogy |
|---------|---------|
| **Serialization** | Photographing a case file so it can be stored on microfilm |
| **Deserialization** | Printing the microfilm back into a readable file |
| **JSON** | A universal format any law firm can read |
| **Pickle** | A proprietary format only YOUR firm understands |
| **CSV** | A simple spreadsheet anyone can open |

| Format | Human Readable? | Python Only? | Supports Custom Objects? | Speed |
|--------|:-:|:-:|:-:|:-:|
| **JSON** | ✅ Yes | ❌ Universal | ❌ Not directly | Medium |
| **pickle** | ❌ No (binary) | ✅ Python only | ✅ Yes (native) | Fast |
| **CSV** | ✅ Yes | ❌ Universal | ❌ Flat data only | Fast |
| **YAML** | ✅ Yes | ❌ Universal | ❌ Not directly | Slow |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 JSON — The Universal Format ⭐⭐

```python
import json

# === Python → JSON string (serialization) ===
client_data = {
    "name": "Wayne Enterprises",
    "revenue": 2500000,
    "active": True,
    "cases": ["Merger", "Patent"],
    "contact": None
}

json_string = json.dumps(client_data)
print(json_string)
# {"name": "Wayne Enterprises", "revenue": 2500000, "active": true, "cases": ["Merger", "Patent"], "contact": null}

# Pretty print for readability
json_pretty = json.dumps(client_data, indent=2)
print(json_pretty)
# {
#   "name": "Wayne Enterprises",
#   "revenue": 2500000,
#   "active": true,
#   "cases": [
#     "Merger",
#     "Patent"
#   ],
#   "contact": null
# }

# === JSON string → Python (deserialization) ===
restored = json.loads(json_string)
print(restored["name"])      # Wayne Enterprises
print(type(restored))         # <class 'dict'>
```

**Python ↔ JSON type mapping:**

| Python | JSON |
|--------|------|
| `dict` | `object {}` |
| `list`, `tuple` | `array []` |
| `str` | `string ""` |
| `int`, `float` | `number` |
| `True` / `False` | `true` / `false` |
| `None` | `null` |

---

### 3.2 JSON Files — Save and Load ⭐

```python
import json

clients = [
    {"name": "Wayne", "revenue": 2500000, "type": "Merger"},
    {"name": "Stark", "revenue": 800000, "type": "Patent"},
    {"name": "Oscorp", "revenue": 600000, "type": "Litigation"},
]

# === Write to file ===
with open("clients.json", "w") as f:
    json.dump(clients, f, indent=2)

# === Read from file ===
with open("clients.json", "r") as f:
    loaded = json.load(f)

print(loaded[0]["name"])   # Wayne
print(len(loaded))          # 3
```

**Mike:** "Four functions to remember:"

| Function | What it does |
|----------|-------------|
| `json.dumps(obj)` | Python → JSON **string** |
| `json.loads(string)` | JSON string → **Python** |
| `json.dump(obj, file)` | Python → JSON **file** |
| `json.load(file)` | JSON file → **Python** |

> 💡 The `s` in `dumps`/`loads` stands for **string**.

---

### 3.3 Serializing Custom Objects ⭐

```python
import json

class Client:
    def __init__(self, name, revenue, case_type, active=True):
        self.name = name
        self.revenue = revenue
        self.case_type = case_type
        self.active = active
    
    def to_dict(self):
        """Convert object to JSON-serializable dict."""
        return {
            "name": self.name,
            "revenue": self.revenue,
            "case_type": self.case_type,
            "active": self.active,
        }
    
    @classmethod
    def from_dict(cls, data):
        """Create object from dict (deserialization)."""
        return cls(
            name=data["name"],
            revenue=data["revenue"],
            case_type=data.get("case_type", "General"),
            active=data.get("active", True),
        )
    
    def __repr__(self):
        return f"Client('{self.name}', ${self.revenue:,})"

# Serialize
wayne = Client("Wayne Enterprises", 2500000, "Merger")
json_str = json.dumps(wayne.to_dict(), indent=2)
print(json_str)

# Deserialize
data = json.loads(json_str)
restored = Client.from_dict(data)
print(restored)   # Client('Wayne Enterprises', $2,500,000)
```

---

### 3.4 Custom JSON Encoder/Decoder

```python
import json
from datetime import datetime, date

class FirmEncoder(json.JSONEncoder):
    """Custom encoder for firm-specific types."""
    
    def default(self, obj):
        if isinstance(obj, (datetime, date)):
            return {"__datetime__": obj.isoformat()}
        if isinstance(obj, set):
            return {"__set__": list(obj)}
        if hasattr(obj, "to_dict"):
            return {"__class__": type(obj).__name__, **obj.to_dict()}
        return super().default(obj)

def firm_decoder(obj):
    """Custom decoder — object_hook for json.loads."""
    if "__datetime__" in obj:
        return datetime.fromisoformat(obj["__datetime__"])
    if "__set__" in obj:
        return set(obj["__set__"])
    if "__class__" in obj:
        class_name = obj.pop("__class__")
        if class_name == "Client":
            return Client.from_dict(obj)
    return obj

# Use custom encoder
data = {
    "firm": "PSL",
    "created": datetime.now(),
    "clients": [
        Client("Wayne", 2500000, "Merger"),
        Client("Stark", 800000, "Patent"),
    ],
    "departments": {"Legal", "IT", "Finance"},
}

json_str = json.dumps(data, cls=FirmEncoder, indent=2)
print(json_str)

# Use custom decoder
restored = json.loads(json_str, object_hook=firm_decoder)
print(type(restored["created"]))     # <class 'datetime.datetime'>
print(type(restored["departments"]))  # <class 'set'>
```

---

### 3.5 `pickle` — Python-Native Serialization

```python
import pickle

class Attorney:
    def __init__(self, name, rate, cases=None):
        self.name = name
        self.rate = rate
        self.cases = cases or []
    
    def __repr__(self):
        return f"Attorney('{self.name}', ${self.rate}/hr, {len(self.cases)} cases)"

harvey = Attorney("Harvey", 450, ["Wayne v. Stark", "Dalton Merger"])

# === Serialize to bytes ===
pickled = pickle.dumps(harvey)
print(type(pickled))       # <class 'bytes'>
print(len(pickled))         # ~200 bytes

# === Deserialize from bytes ===
restored = pickle.loads(pickled)
print(restored)             # Attorney('Harvey', $450/hr, 2 cases)
print(restored.cases)       # ['Wayne v. Stark', 'Dalton Merger']

# === Save to / Load from file ===
with open("attorney.pkl", "wb") as f:    # Binary mode!
    pickle.dump(harvey, f)

with open("attorney.pkl", "rb") as f:
    loaded = pickle.load(f)
print(loaded)   # Attorney('Harvey', $450/hr, 2 cases)
```

**JSON vs Pickle:**

| Feature | JSON | Pickle |
|---------|------|--------|
| Human readable | ✅ | ❌ (binary) |
| Language support | Universal | Python only |
| Custom objects | Manual (`to_dict`/`from_dict`) | ✅ Automatic |
| Security | ✅ Safe | ⚠️ **NEVER unpickle untrusted data** |
| Speed | Medium | Fast |
| File size | Larger | Smaller |

> ⚠️ **SECURITY WARNING:** `pickle.loads()` can execute arbitrary code. NEVER unpickle data from untrusted sources!

---

### 3.6 CSV — Tabular Data

```python
import csv

# === Writing CSV ===
clients = [
    {"name": "Wayne", "revenue": 2500000, "type": "Merger"},
    {"name": "Stark", "revenue": 800000, "type": "Patent"},
    {"name": "Oscorp", "revenue": 600000, "type": "Litigation"},
]

with open("clients.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "revenue", "type"])
    writer.writeheader()
    writer.writerows(clients)

# === Reading CSV ===
with open("clients.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        # All values come as STRINGS!
        print(f"  {row['name']}: ${int(row['revenue']):,} ({row['type']})")

# === CSV to objects ===
def load_clients(filepath):
    with open(filepath, "r") as f:
        reader = csv.DictReader(f)
        return [
            Client(
                name=row["name"],
                revenue=int(row["revenue"]),
                case_type=row["type"]
            )
            for row in reader
        ]
```

---

### 3.7 `dataclasses` — The Modern Approach ⭐⭐

```python
from dataclasses import dataclass, field, asdict, astuple
import json

@dataclass
class Client:
    name: str
    revenue: float
    case_type: str = "General"
    active: bool = True
    cases: list = field(default_factory=list)   # Mutable default — safe!
    
    @property
    def tier(self):
        if self.revenue >= 1000000: return "Platinum"
        elif self.revenue >= 500000: return "Gold"
        return "Silver"

# Auto-generated __init__, __repr__, __eq__
wayne = Client("Wayne Enterprises", 2500000, "Merger")
print(wayne)
# Client(name='Wayne Enterprises', revenue=2500000, case_type='Merger', active=True, cases=[])

# Built-in serialization helpers
print(asdict(wayne))
# {'name': 'Wayne Enterprises', 'revenue': 2500000, 'case_type': 'Merger', 'active': True, 'cases': []}

print(astuple(wayne))
# ('Wayne Enterprises', 2500000, 'Merger', True, [])

# JSON serialization — one line!
json_str = json.dumps(asdict(wayne), indent=2)
print(json_str)

# Deserialization
data = json.loads(json_str)
restored = Client(**data)
print(restored)   # Same as original

# Equality comparison (auto-generated)
wayne2 = Client("Wayne Enterprises", 2500000, "Merger")
print(wayne == wayne2)   # True! (__eq__ auto-generated)
```

**Key `dataclass` features:**

| Feature | Regular Class | `@dataclass` |
|---------|:---:|:---:|
| `__init__` | Write manually | ✅ Auto-generated |
| `__repr__` | Write manually | ✅ Auto-generated |
| `__eq__` | Write manually | ✅ Auto-generated |
| Type hints | Optional | Required |
| `asdict()` / `astuple()` | Manual | ✅ Built-in |
| Default values | Manual | ✅ Clean syntax |
| Mutable defaults | BUG-PRONE | ✅ `field(default_factory=...)` |

---

### 3.8 Frozen Dataclasses — Immutable Objects

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Invoice:
    """Immutable invoice — can't be modified after creation."""
    invoice_id: str
    client: str
    amount: float
    paid: bool = False

inv = Invoice("INV-001", "Wayne", 38250.00)
# inv.amount = 0        # ❌ FrozenInstanceError!
# inv.paid = True       # ❌ FrozenInstanceError!

# Frozen dataclasses are hashable — can be used in sets and as dict keys!
invoices = {inv}
invoice_lookup = {inv: "processed"}
```

---

### 3.9 Complete Persistence Layer

```python
# ============================================
# Pearson Specter Litt — Data Persistence Layer
# JSON-based storage with dataclasses
# ============================================

import json
import os
from dataclasses import dataclass, field, asdict
from datetime import datetime
from typing import Optional

@dataclass
class Client:
    name: str
    revenue: float
    case_type: str = "General"
    active: bool = True
    attorney: Optional[str] = None
    created: str = field(default_factory=lambda: datetime.now().isoformat())
    notes: list = field(default_factory=list)
    
    @property
    def tier(self):
        if self.revenue >= 1000000: return "💎 Platinum"
        elif self.revenue >= 500000: return "🥇 Gold"
        elif self.revenue >= 100000: return "🥈 Silver"
        return "🥉 Bronze"
    
    @classmethod
    def from_dict(cls, data):
        return cls(**{k: v for k, v in data.items() 
                      if k in cls.__dataclass_fields__})


class DataStore:
    """JSON-based persistence layer for the firm."""
    
    def __init__(self, filepath="firm_data.json"):
        self.filepath = filepath
        self.data = {"clients": [], "metadata": {}}
        self._load()
    
    def _load(self):
        """Load data from JSON file."""
        if os.path.exists(self.filepath):
            with open(self.filepath, "r") as f:
                self.data = json.load(f)
            print(f"  📂 Loaded {len(self.data['clients'])} clients from {self.filepath}")
        else:
            print(f"  📂 New database: {self.filepath}")
    
    def _save(self):
        """Save data to JSON file."""
        self.data["metadata"]["last_saved"] = datetime.now().isoformat()
        self.data["metadata"]["count"] = len(self.data["clients"])
        
        with open(self.filepath, "w") as f:
            json.dump(self.data, f, indent=2)
    
    def add_client(self, client: Client):
        """Add a client and persist."""
        self.data["clients"].append(asdict(client))
        self._save()
        print(f"  ✅ Added: {client.name}")
    
    def get_client(self, name: str) -> Optional[Client]:
        """Retrieve a client by name."""
        for c_data in self.data["clients"]:
            if c_data["name"] == name:
                return Client.from_dict(c_data)
        return None
    
    def update_client(self, name: str, **updates):
        """Update a client's fields."""
        for c_data in self.data["clients"]:
            if c_data["name"] == name:
                c_data.update(updates)
                self._save()
                print(f"  ✅ Updated: {name}")
                return True
        print(f"  ❌ Not found: {name}")
        return False
    
    def delete_client(self, name: str):
        """Delete a client."""
        before = len(self.data["clients"])
        self.data["clients"] = [c for c in self.data["clients"] if c["name"] != name]
        if len(self.data["clients"]) < before:
            self._save()
            print(f"  ✅ Deleted: {name}")
            return True
        return False
    
    def search(self, **criteria) -> list:
        """Search clients by any field."""
        results = self.data["clients"]
        for key, value in criteria.items():
            results = [c for c in results if c.get(key) == value]
        return [Client.from_dict(c) for c in results]
    
    def all_clients(self) -> list:
        """Get all clients as objects."""
        return [Client.from_dict(c) for c in self.data["clients"]]
    
    def export_csv(self, filepath="clients.csv"):
        """Export to CSV."""
        import csv
        if not self.data["clients"]:
            return
        
        fields = list(self.data["clients"][0].keys())
        with open(filepath, "w", newline="") as f:
            writer = csv.DictWriter(f, fieldnames=fields)
            writer.writeheader()
            writer.writerows(self.data["clients"])
        print(f"  📤 Exported {len(self.data['clients'])} clients to {filepath}")
    
    def dashboard(self):
        """Display database summary."""
        clients = self.all_clients()
        
        print(f"\n{'='*55}")
        print(f"{'🏛️  FIRM DATA DASHBOARD':^55}")
        print(f"{'='*55}")
        
        if not clients:
            print("  No clients in database.")
            return
        
        print(f"\n  Total Clients: {len(clients)}")
        print(f"  Total Revenue: ${sum(c.revenue for c in clients):,.2f}")
        print(f"  Active: {sum(1 for c in clients if c.active)}")
        
        print(f"\n  {'Name':<22} {'Revenue':>12}  {'Type':<12} {'Tier'}")
        print(f"  {'─'*55}")
        for c in sorted(clients, key=lambda x: x.revenue, reverse=True):
            print(f"  {c.name:<22} ${c.revenue:>11,.2f}  {c.case_type:<12} {c.tier}")
        
        if "last_saved" in self.data.get("metadata", {}):
            print(f"\n  Last saved: {self.data['metadata']['last_saved'][:19]}")
        print(f"{'='*55}")


# === DEMONSTRATE THE SYSTEM ===
print("=" * 55)
print(f"{'DATA PERSISTENCE DEMO':^55}")
print("=" * 55)

# Initialize
store = DataStore("psl_clients.json")

# Add clients
store.add_client(Client("Wayne Enterprises", 2500000, "Merger", attorney="Harvey"))
store.add_client(Client("Stark Industries", 800000, "Patent", attorney="Mike"))
store.add_client(Client("Oscorp", 600000, "Litigation", attorney="Louis"))
store.add_client(Client("Dalton Industries", 1200000, "Merger", attorney="Harvey"))
store.add_client(Client("Luthor Corp", 150000, "Corporate", attorney="Louis"))

# Retrieve
wayne = store.get_client("Wayne Enterprises")
print(f"\n🔍 Retrieved: {wayne}")
print(f"   Tier: {wayne.tier}")

# Update
store.update_client("Stark Industries", revenue=900000, attorney="Rachel")

# Search
mergers = store.search(case_type="Merger")
print(f"\n🔍 Merger clients: {[c.name for c in mergers]}")

# Dashboard
store.dashboard()

# Export
store.export_csv("psl_export.csv")

# Cleanup demo files
for f in ["psl_clients.json", "psl_export.csv", "clients.json", "clients.csv", "attorney.pkl"]:
    if os.path.exists(f):
        os.remove(f)
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: JSON Can't Serialize Everything

```python
import json
from datetime import datetime

# ❌ These fail with json.dumps()
# json.dumps(datetime.now())     # TypeError!
# json.dumps({1, 2, 3})          # TypeError! (sets)
# json.dumps(b"bytes")           # TypeError!
# json.dumps(some_object)        # TypeError!

# ✅ Convert first
data = {
    "created": datetime.now().isoformat(),    # str
    "tags": list({1, 2, 3}),                  # list
    "data": b"bytes".decode("utf-8"),         # str
}
json.dumps(data)   # ✅
```

---

### Pitfall 2: JSON Loses Type Information

```python
import json

# Tuples become lists
data = {"coords": (1, 2, 3)}
restored = json.loads(json.dumps(data))
print(type(restored["coords"]))   # <class 'list'> — NOT tuple!

# Integer keys become strings
data = {1: "one", 2: "two"}
restored = json.loads(json.dumps(data))
print(restored)                    # {"1": "one", "2": "two"} — string keys!

# Sets can't be serialized at all
# json.dumps({1, 2, 3})           # TypeError
```

---

### Pitfall 3: CSV All Values Are Strings

```python
import csv

# When reading CSV, EVERYTHING is a string
with open("data.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        # row["revenue"] is "2500000" (string), not 2500000 (int)!
        total = row["revenue"] + row["revenue"]   # "25000002500000" ← concatenation!

# ✅ Convert types explicitly
        total = int(row["revenue"]) + int(row["revenue"])   # 5000000 ✅
```

---

### Pitfall 4: Pickle Security Vulnerability 🔥

```python
import pickle

# ⚠️ NEVER unpickle data from the internet, emails, user uploads, etc.
# A malicious pickle file can:
# - Delete files on your computer
# - Download malware
# - Open a remote shell

# ❌ DANGEROUS
# data = pickle.loads(untrusted_data)

# ✅ Use JSON for untrusted data — it can only contain safe types
# ✅ Use pickle ONLY for internal, trusted data
```

---

### Pitfall 5: File Encoding Issues

```python
# ❌ Default encoding varies by OS
with open("data.json", "w") as f:
    json.dump({"name": "Müller"}, f)

# ✅ Always specify UTF-8
with open("data.json", "w", encoding="utf-8") as f:
    json.dump({"name": "Müller"}, f, ensure_ascii=False)

with open("data.json", "r", encoding="utf-8") as f:
    data = json.load(f)
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🗄️ Analogy: Archiving Case Files

```
PYTHON OBJECT (in memory):
  🧠 Harvey's brain holds the case details
  ⚡ Fast access, but disappears when he goes home

SERIALIZATION (save to disk):
  🧠 → 📄 Write it down on paper (JSON/CSV)
  🧠 → 📦 Seal it in a proprietary container (pickle)
  
  JSON   = Typed report anyone can read
  Pickle = Sealed box only Python can open
  CSV    = Simple spreadsheet for managers

DESERIALIZATION (load from disk):
  📄 → 🧠 Read the paper back into memory
  📦 → 🧠 Open the sealed box

KEY INSIGHT:
  Serialization = "How do I turn a living object into a file?"
  Deserialization = "How do I bring a file back to life?"
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: JSON Basics**
```
a) Create a dict with: name, age, hobbies (list), address (nested dict)
b) Serialize to JSON string (pretty printed)
c) Deserialize back to Python
d) Save to a file, then load from the file
e) Verify original == loaded
```

**Exercise 2: Object Serialization**
```
Create a Book class with: title, author, pages, genre
a) Add to_dict() and from_dict() methods
b) Serialize a list of 5 books to JSON
c) Load them back and print each book
```

**Exercise 3: CSV Round-Trip**
```
a) Create 10 student records (name, grade, score)
b) Write to CSV using csv.DictWriter
c) Read back using csv.DictReader
d) Handle type conversion (score → int)
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Custom Encoder**
```
Build a custom JSON encoder that handles:
  - datetime objects
  - set objects
  - Decimal objects
  - Custom class instances (via to_dict())

Write matching decoder. Round-trip test.
```

**Exercise 5: Dataclass CRUD**
```
Build a complete CRUD system using dataclasses + JSON:

@dataclass
class Product:
    id: int
    name: str
    price: float
    quantity: int

Operations: create, read, update, delete, list_all, search
All data persists to a JSON file.
```

**Exercise 6: Format Converter**
```
Build a tool that converts between formats:
  json_to_csv(json_path, csv_path)
  csv_to_json(csv_path, json_path)
  json_to_pickle(json_path, pkl_path)
  pickle_to_json(pkl_path, json_path)

Handle edge cases: nested data, type conversion, missing fields.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Database Abstraction**
```
Build a JSONDatabase class that supports:
  - Multiple "tables" (collections)
  - Auto-incrementing IDs
  - Querying with operators (gt, lt, contains, etc.)
  - Indexes for fast lookup
  - Transaction support (commit/rollback)
  - Backup and restore
```

**Exercise 8: Schema Migration**
```
Build a system that handles schema changes:
  - Version 1: {name, revenue}
  - Version 2: Added "type" field (default: "General")
  - Version 3: Renamed "revenue" to "annual_revenue"

Each migration transforms old data to new format.
Apply migrations automatically on load.
```

**Exercise 9: Streaming JSON Processor**
```
Build a system that processes large JSON files:
  - Read records one at a time (not all at once)
  - Use generators for memory efficiency
  - Support filtering and transformation
  - Write results to a new file in a streaming fashion

Process a 100K+ record file using constant memory.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Trying to serialize a datetime
import json
from datetime import datetime

data = {"name": "Wayne", "created": datetime.now()}
json.dumps(data)    # TypeError!

# Bug 2: Losing tuple types
data = {"coords": (1, 2, 3)}
restored = json.loads(json.dumps(data))
assert isinstance(restored["coords"], tuple)   # AssertionError!

# Bug 3: CSV types are strings
import csv
# After reading CSV:
row = {"name": "Wayne", "revenue": "2500000"}
total = row["revenue"] * 2    # Wrong!

# Bug 4: Pickle on untrusted data
data = download_from_internet()
obj = pickle.loads(data)    # DANGEROUS!

# Bug 5: Missing newline in CSV
with open("data.csv", "w") as f:    # Missing newline=""
    writer = csv.writer(f)
    writer.writerows(data)    # Extra blank lines on Windows!
```

### Fixed Code ✅

```python
# Fix 1: Convert datetime to string
data = {"name": "Wayne", "created": datetime.now().isoformat()}
json.dumps(data)    # ✅

# Fix 2: Convert back explicitly or accept the limitation
data = {"coords": (1, 2, 3)}
restored = json.loads(json.dumps(data))
restored["coords"] = tuple(restored["coords"])   # ✅ Manual conversion

# Fix 3: Convert CSV strings to appropriate types
row = {"name": "Wayne", "revenue": "2500000"}
total = int(row["revenue"]) * 2    # ✅ 5000000

# Fix 4: Use JSON for untrusted data
data = download_from_internet()
obj = json.loads(data)    # ✅ Safe — only creates Python primitives

# Fix 5: Always use newline="" for CSV on Windows
with open("data.csv", "w", newline="") as f:    # ✅
    writer = csv.writer(f)
    writer.writerows(data)
```

---

## 8. 🌍 Real-World Application

### Where Serialization Is Used

| Application | Format | Why |
|------------|--------|-----|
| **REST APIs** | JSON | Universal, human-readable |
| **Config files** | JSON/YAML | Easy to edit |
| **Data science** | CSV/Parquet | Tabular data |
| **Caching** | Pickle/JSON | Fast retrieval |
| **Message queues** | JSON | Cross-language |
| **Settings** | JSON | Persistent user prefs |
| **ML models** | Pickle/ONNX | Save trained models |
| **Databases** | JSON (MongoDB) | Document storage |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is serialization? What is deserialization?

**Q2.** What is the difference between `json.dump()` and `json.dumps()`?

**Q3.** Which Python types can JSON serialize natively?

**Q4.** How do you serialize a custom object to JSON?

**Q5.** Why should you NEVER use `pickle.loads()` on untrusted data?

**Q6.** What is a `@dataclass`? What does it auto-generate?

**Q7.** What does `field(default_factory=list)` do and why is it needed?

**Q8.** What happens to tuples when serialized/deserialized as JSON?

**Q9.** Why should you use `newline=""` when opening CSV files?

**Q10.** Write code to save a list of dictionaries to JSON and load it back.

---

### 📋 Answer Key

> **Q1.** Serialization: converting objects to a storable/transmittable format. Deserialization: converting that format back to objects.

> **Q2.** `dump()` writes to a file. `dumps()` returns a string.

> **Q3.** `dict`, `list`, `str`, `int`, `float`, `bool`, `None`.

> **Q4.** Add a `to_dict()` method that returns a dict, then `json.dumps(obj.to_dict())`. Add a `from_dict()` classmethod for deserialization.

> **Q5.** Pickle can execute arbitrary code — a malicious pickle can run any command on your system.

> **Q6.** A decorator that auto-generates `__init__`, `__repr__`, `__eq__` based on annotated fields.

> **Q7.** Safely handles mutable defaults. Without it, all instances would share the same list object.

> **Q8.** Tuples become lists. JSON has no tuple type.

> **Q9.** Prevents extra blank lines on Windows (the csv module handles newlines itself).

> **Q10.**
```python
import json
data = [{"name": "Wayne", "revenue": 2500000}]
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)
with open("data.json", "r") as f:
    loaded = json.load(f)
```

---

## 🎬 End of Lesson Scene

**Mike:** "The data survives restarts now. JSON for exchange, pickle for internal caching, CSV for reports. And `dataclasses` make the serialization clean."

**Harvey:** "The vault is built. Next: Python's built-in library — the tools that come with the language."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | JSON serialization/deserialization | ✅ |
| 2 | `dumps`/`loads` vs `dump`/`load` | ✅ |
| 3 | Custom object serialization (`to_dict`/`from_dict`) | ✅ |
| 4 | Custom JSON encoder/decoder | ✅ |
| 5 | `pickle` (with security warning) | ✅ |
| 6 | CSV read/write with `csv` module | ✅ |
| 7 | `@dataclass` with `asdict`/`astuple` | ✅ |
| 8 | Frozen dataclasses (immutable) | ✅ |
| 9 | Complete persistence layer | ✅ |
| 10 | Format comparison and decisions | ✅ |

---

> **Next Lesson →** Level 2, Lesson 11: *Import Python Built-In Libraries*
