# 🏛️ Level 1 – Exploring | Lesson 14: Read and Write Text Files in Python

## *"The Filing Cabinet"*

> **O'Reilly Skill Objective:** *Read and write text files in Python.*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, `input()`, conditionals, boolean operators, loops, functions, classes, objects, `__init__`, `self`, methods, `__str__`, encapsulation, properties, inheritance basics

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

*It's Friday afternoon. Two weeks since you joined the firm. Harvey walks into your office — yes, you have an office now — and drops a USB drive on your desk.*

**Harvey:** "This drive has three years of client records in text files. Names, billing history, case notes. Right now, all your programs lose everything the second you close the terminal. That changes today."

**You:** "So instead of typing in client data every time, the program reads it from files?"

**Harvey:** "Reads AND writes. Your classification engine should load clients from a file, process them, and write the results to another file. Your billing system should log every transaction to a permanent record."

*Mike leans in from the doorway.*

**Mike:** "File I/O — input and output. It's the bridge between your program and the **outside world**. Without it, your program is a goldfish — no memory beyond the current session. With it, your program has a filing cabinet."

**Harvey:** "And not just reading our data. Think bigger. Every log file, every CSV report, every configuration file — they're all text files. Learn to read and write them, and you can interact with any system."

**Donna:** *(over the intercom)* "It's also the last lesson in Level 1. Make it count."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Here's why file I/O is the final lesson. It completes the picture."

| Without Files | With Files |
|---------------|-----------|
| Data dies when program stops | Data persists forever |
| Must re-enter data each run | Load data automatically |
| Can't share data between programs | Files are universal exchange format |
| Can't generate reports | Export results to files others can read |

**Harvey:** "Every real system does four things with files:"

1. **Read** — Load data from existing files
2. **Write** — Create new files with output
3. **Append** — Add to existing files without overwriting
4. **Process** — Read, transform, write — the ETL pipeline

**Harvey:** "Configuration files, log files, CSV data, JSON APIs, HTML pages — they're all text. Master file I/O, and you can work with any of them."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "Files. The permanent memory of your programs."

---

### 3.1 Opening and Closing Files — The Basics

```python
# ❌ OLD WAY — manual open/close (error-prone)
file = open("clients.txt", "r")
content = file.read()
file.close()    # Easy to forget! If an error happens above, file stays open!

# ✅ BEST WAY — context manager (with statement)
with open("clients.txt", "r") as file:
    content = file.read()
# File is automatically closed when the block ends — even if an error occurs!
```

**Mike:** "`with` is the Pythonic way. It guarantees the file is closed. Always use it."

#### File Modes

| Mode | Meaning | If File Exists | If File Doesn't Exist |
|------|---------|---------------|----------------------|
| `"r"` | **Read** (default) | Opens for reading | `FileNotFoundError` |
| `"w"` | **Write** | Overwrites completely | Creates new file |
| `"a"` | **Append** | Adds to the end | Creates new file |
| `"x"` | **Exclusive create** | `FileExistsError` | Creates new file |
| `"r+"` | **Read + write** | Opens for both | `FileNotFoundError` |

> ⚠️ **Critical:** `"w"` mode **destroys** existing content! If you open a file with `"w"`, everything in it is erased immediately.

---

### 3.2 Reading Files

#### `read()` — Read the Entire File as One String
```python
with open("clients.txt", "r") as f:
    content = f.read()
    print(content)
    print(type(content))    # <class 'str'>
    print(len(content))     # Total characters including newlines
```

#### `readline()` — Read One Line at a Time
```python
with open("clients.txt", "r") as f:
    line1 = f.readline()    # First line (includes \n)
    line2 = f.readline()    # Second line
    line3 = f.readline()    # Third line
    print(line1.strip())    # .strip() removes the trailing newline
```

#### `readlines()` — Read All Lines into a List
```python
with open("clients.txt", "r") as f:
    lines = f.readlines()
    print(lines)
    # ['Wayne Enterprises,2500000,Merger\n', 'Stark Industries,800000,Patent\n', ...]
    
    for line in lines:
        print(line.strip())    # Remove trailing newlines
```

#### Iterating Directly (Most Pythonic) ⭐
```python
# The file object IS iterable — you can loop over it directly
with open("clients.txt", "r") as f:
    for line in f:
        print(line.strip())

# This is MEMORY EFFICIENT — reads one line at a time
# For huge files (GB+), this is the ONLY safe approach
```

**Mike:** "Four ways to read. Use **direct iteration** for line-by-line processing. Use `read()` only for small files where you need the whole content at once."

---

### 3.3 Writing Files

#### `write()` — Write a String to a File
```python
# Create a new file (or overwrite existing!)
with open("report.txt", "w") as f:
    f.write("Pearson Specter Litt\n")
    f.write("Quarterly Report\n")
    f.write("=" * 30 + "\n")
    f.write(f"Date: 2026-03-13\n")
```

#### `writelines()` — Write a List of Strings
```python
lines = [
    "Client,Revenue,Type\n",
    "Wayne Enterprises,2500000,Merger\n",
    "Stark Industries,800000,Patent\n",
    "Oscorp,600000,Litigation\n",
]

with open("clients.csv", "w") as f:
    f.writelines(lines)

# NOTE: writelines does NOT add newlines — you must include \n yourself!
```

#### Using `print()` to Write to Files
```python
with open("report.txt", "w") as f:
    print("Client Report", file=f)              # Adds newline automatically!
    print("=" * 30, file=f)
    print(f"Total clients: {237}", file=f)
    print(f"Revenue: ${5200000:,.2f}", file=f)
```

**Mike:** "`print(file=f)` is often cleaner than `f.write()` because it adds newlines automatically and supports all the formatting you already know."

---

### 3.4 Appending to Files

```python
# Append mode — adds to the END of the file
with open("activity_log.txt", "a") as f:
    f.write("2026-03-13 09:00 — System startup\n")
    f.write("2026-03-13 09:15 — Client Wayne loaded\n")

# Later, append more:
with open("activity_log.txt", "a") as f:
    f.write("2026-03-13 10:30 — Invoice generated\n")

# The file now has ALL three entries — nothing was overwritten
```

---

### 3.5 Parsing Structured Text Files ⭐

**Mike:** "Real files contain structured data. You need to parse them."

#### Parsing CSV-Style Data
```python
# clients.txt contains:
# name,revenue,case_type
# Wayne Enterprises,2500000,Merger
# Stark Industries,800000,Patent
# Oscorp,600000,Litigation

clients = []

with open("clients.txt", "r") as f:
    header = f.readline().strip()    # Skip (or parse) the header
    
    for line in f:
        line = line.strip()
        if not line:                  # Skip empty lines
            continue
        
        parts = line.split(",")       # Split by comma
        client = {
            "name": parts[0],
            "revenue": float(parts[1]),
            "case_type": parts[2]
        }
        clients.append(client)

# Now 'clients' is a list of dictionaries — data is in memory!
for c in clients:
    print(f"  {c['name']}: ${c['revenue']:,.2f} ({c['case_type']})")
```

#### Parsing Key-Value Configuration
```python
# config.txt contains:
# firm_name = Pearson Specter Litt
# tax_rate = 0.09
# max_retries = 5
# debug_mode = True

config = {}

with open("config.txt", "r") as f:
    for line in f:
        line = line.strip()
        if not line or line.startswith("#"):   # Skip empty lines and comments
            continue
        
        key, value = line.split("=", 1)    # Split on FIRST = only
        key = key.strip()
        value = value.strip()
        
        # Auto-convert types
        if value.lower() in ("true", "false"):
            value = value.lower() == "true"
        elif value.replace(".", "", 1).isdigit():
            value = float(value) if "." in value else int(value)
        
        config[key] = value

print(config)
# {'firm_name': 'Pearson Specter Litt', 'tax_rate': 0.09, 'max_retries': 5, 'debug_mode': True}
```

---

### 3.6 Working with File Paths

```python
import os

# Current working directory
print(os.getcwd())

# Join paths (cross-platform)
filepath = os.path.join("data", "clients", "wayne.txt")
print(filepath)    # data/clients/wayne.txt (or data\clients\wayne.txt on Windows)

# Check if file exists
if os.path.exists("clients.txt"):
    print("File exists!")
else:
    print("File not found!")

# Check if it's a file or directory
print(os.path.isfile("clients.txt"))    # True
print(os.path.isdir("data"))            # True/False

# Get file size
size = os.path.getsize("clients.txt")
print(f"File size: {size} bytes")

# List files in a directory
files = os.listdir(".")    # Current directory
for f in files:
    print(f"  {f}")

# Get filename and extension
name, ext = os.path.splitext("report_2026.csv")
print(f"Name: {name}, Extension: {ext}")   # report_2026, .csv
```

---

### 3.7 Error Handling with Files

```python
# Handle missing files gracefully
try:
    with open("missing_file.txt", "r") as f:
        content = f.read()
except FileNotFoundError:
    print("❌ File not found!")
except PermissionError:
    print("🔒 Permission denied!")
except Exception as e:
    print(f"⚠️ Error: {e}")

# Complete pattern with multiple error types
def read_file_safe(filepath):
    """Safely read a file with error handling."""
    try:
        with open(filepath, "r") as f:
            return f.read()
    except FileNotFoundError:
        print(f"  ❌ File '{filepath}' not found")
        return None
    except PermissionError:
        print(f"  🔒 No permission to read '{filepath}'")
        return None
    except UnicodeDecodeError:
        print(f"  ⚠️ File '{filepath}' contains non-text data")
        return None

content = read_file_safe("clients.txt")
if content:
    print(f"Read {len(content)} characters")
```

---

### 3.8 Reading and Writing CSV Data

```python
# Writing CSV
clients = [
    {"name": "Wayne Enterprises", "revenue": 2500000, "type": "Merger"},
    {"name": "Stark Industries", "revenue": 800000, "type": "Patent"},
    {"name": "Oscorp", "revenue": 600000, "type": "Litigation"},
]

with open("clients_export.csv", "w") as f:
    # Header
    f.write("Name,Revenue,Type\n")
    # Data
    for client in clients:
        f.write(f"{client['name']},{client['revenue']},{client['type']}\n")

print("✅ CSV exported!")

# Reading CSV
loaded_clients = []
with open("clients_export.csv", "r") as f:
    headers = f.readline().strip().split(",")
    
    for line in f:
        values = line.strip().split(",")
        client = dict(zip(headers, values))   # Using zip from Lesson 9!
        client["Revenue"] = float(client["Revenue"])
        loaded_clients.append(client)

for c in loaded_clients:
    print(c)
```

#### Using the `csv` Module (Recommended for Complex CSV)
```python
import csv

# Writing with csv module
with open("clients_proper.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["Name", "Revenue", "Type"])
    writer.writerow(["Wayne Enterprises", 2500000, "Merger"])
    writer.writerow(["Stark Industries", 800000, "Patent"])

# Reading with csv module
with open("clients_proper.csv", "r") as f:
    reader = csv.reader(f)
    headers = next(reader)     # Get the header row
    
    for row in reader:
        print(f"  {row[0]}: ${float(row[1]):,.2f} ({row[2]})")

# DictReader — returns dictionaries automatically!
with open("clients_proper.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(f"  {row['Name']}: ${float(row['Revenue']):,.2f}")
```

---

### 3.9 Working with JSON Files

```python
import json

# Writing JSON
firm_data = {
    "firm_name": "Pearson Specter Litt",
    "year": 2026,
    "partners": ["Harvey", "Jessica", "Louis"],
    "clients": [
        {"name": "Wayne Enterprises", "revenue": 2500000, "active": True},
        {"name": "Stark Industries", "revenue": 800000, "active": True},
        {"name": "Oscorp", "revenue": 600000, "active": False},
    ],
    "settings": {
        "tax_rate": 0.09,
        "max_retries": 5
    }
}

# Write to JSON file
with open("firm_data.json", "w") as f:
    json.dump(firm_data, f, indent=2)     # indent=2 for pretty printing

print("✅ JSON exported!")

# Read from JSON file
with open("firm_data.json", "r") as f:
    loaded_data = json.load(f)

print(f"Firm: {loaded_data['firm_name']}")
print(f"Partners: {', '.join(loaded_data['partners'])}")
print(f"Active clients: {sum(1 for c in loaded_data['clients'] if c['active'])}")
```

> 💡 **Connection to Lesson 5:** JSON maps directly to Python dictionaries and lists. `json.dump()` writes a dict to a file. `json.load()` reads a file into a dict.

---

### 3.10 Solving Harvey's Problem — Client Data Pipeline

```python
# ============================================
# Pearson Specter Litt — Client Data Pipeline
# ============================================
import json
import csv
import os
from datetime import datetime

# ---- Step 1: Create sample data files ----

def create_sample_data():
    """Create sample client data files for demonstration."""
    
    # Raw client records (text file)
    raw_data = """# PSL Client Registry
# Format: name|revenue|rate|case_type|status
Wayne Enterprises|2500000|450.00|Merger|active
Stark Industries|800000|375.00|Patent|active
Oscorp|600000|400.00|Litigation|active
Dalton Industries|1200000|425.00|Merger|active
Luthor Corp|150000|325.00|Corporate|active
Gillis Industries|75000|280.00|Patent|inactive
Queen Industries|3000000|500.00|Merger|active
Palmer Tech|400000|350.00|Patent|active"""
    
    with open("raw_clients.txt", "w") as f:
        f.write(raw_data)
    
    print("📄 Created raw_clients.txt")


def load_clients(filepath):
    """Parse raw client data from a text file."""
    clients = []
    
    try:
        with open(filepath, "r") as f:
            for line in f:
                line = line.strip()
                
                # Skip comments and empty lines
                if not line or line.startswith("#"):
                    continue
                
                parts = line.split("|")
                if len(parts) != 5:
                    print(f"  ⚠️ Skipping malformed line: {line}")
                    continue
                
                client = {
                    "name": parts[0].strip(),
                    "revenue": float(parts[1].strip()),
                    "rate": float(parts[2].strip()),
                    "case_type": parts[3].strip(),
                    "status": parts[4].strip(),
                }
                
                # Classify
                rev = client["revenue"]
                if rev >= 1000000:
                    client["tier"] = "Platinum"
                elif rev >= 500000:
                    client["tier"] = "Gold"
                elif rev >= 100000:
                    client["tier"] = "Silver"
                else:
                    client["tier"] = "Bronze"
                
                clients.append(client)
    
    except FileNotFoundError:
        print(f"  ❌ File not found: {filepath}")
        return []
    
    print(f"  ✅ Loaded {len(clients)} clients from {filepath}")
    return clients


def export_csv(clients, filepath):
    """Export clients to a CSV file."""
    with open(filepath, "w", newline="") as f:
        if not clients:
            return
        
        writer = csv.DictWriter(f, fieldnames=clients[0].keys())
        writer.writeheader()
        writer.writerows(clients)
    
    print(f"  ✅ Exported {len(clients)} clients to {filepath}")


def export_json(data, filepath):
    """Export data to a JSON file."""
    with open(filepath, "w") as f:
        json.dump(data, f, indent=2)
    
    print(f"  ✅ Exported to {filepath}")


def generate_report(clients, filepath):
    """Generate a formatted text report."""
    active = [c for c in clients if c["status"] == "active"]
    
    # Calculate stats
    total_rev = sum(c["revenue"] for c in active)
    avg_rev = total_rev / len(active) if active else 0
    
    tiers = {}
    for c in active:
        tier = c["tier"]
        tiers[tier] = tiers.get(tier, 0) + 1
    
    by_type = {}
    for c in active:
        ct = c["case_type"]
        by_type[ct] = by_type.get(ct, 0) + c["revenue"]
    
    # Write report
    with open(filepath, "w") as f:
        print("=" * 60, file=f)
        print(f"{'PEARSON SPECTER LITT':^60}", file=f)
        print(f"{'CLIENT PORTFOLIO REPORT':^60}", file=f)
        print(f"{'Generated: ' + datetime.now().strftime('%Y-%m-%d %H:%M'):^60}", file=f)
        print("=" * 60, file=f)
        
        print(f"\n📊 PORTFOLIO SUMMARY", file=f)
        print(f"{'─'*60}", file=f)
        print(f"  Total Clients:  {len(clients)}", file=f)
        print(f"  Active Clients: {len(active)}", file=f)
        print(f"  Total Revenue:  ${total_rev:>14,.2f}", file=f)
        print(f"  Avg Revenue:    ${avg_rev:>14,.2f}", file=f)
        
        print(f"\n📋 TIER DISTRIBUTION", file=f)
        print(f"{'─'*60}", file=f)
        for tier in ["Platinum", "Gold", "Silver", "Bronze"]:
            count = tiers.get(tier, 0)
            bar = "█" * (count * 4) + "░" * ((5 - count) * 4)
            print(f"  {tier:<10} [{bar}] {count}", file=f)
        
        print(f"\n💼 REVENUE BY CASE TYPE", file=f)
        print(f"{'─'*60}", file=f)
        for ct in sorted(by_type, key=by_type.get, reverse=True):
            pct = by_type[ct] / total_rev * 100
            print(f"  {ct:<15} ${by_type[ct]:>14,.2f}  ({pct:.1f}%)", file=f)
        
        print(f"\n📝 CLIENT DETAILS", file=f)
        print(f"{'─'*60}", file=f)
        print(f"  {'Name':<22} {'Tier':<10} {'Type':<12} {'Revenue':>14}", file=f)
        print(f"  {'─'*58}", file=f)
        
        for c in sorted(active, key=lambda x: x["revenue"], reverse=True):
            icon = {"Platinum": "💎", "Gold": "🥇", "Silver": "🥈", "Bronze": "🥉"}.get(c["tier"], "")
            print(f"  {c['name']:<22} {icon}{c['tier']:<8} {c['case_type']:<12} ${c['revenue']:>13,.2f}", file=f)
        
        print(f"\n{'='*60}", file=f)
        print(f"{'End of Report':^60}", file=f)
        print(f"{'='*60}", file=f)
    
    print(f"  ✅ Report written to {filepath}")


def append_log(message, filepath="audit_log.txt"):
    """Append an entry to the audit log."""
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    with open(filepath, "a") as f:
        f.write(f"[{timestamp}] {message}\n")


# ===== RUN THE PIPELINE =====

print("=" * 60)
print(f"{'CLIENT DATA PIPELINE':^60}")
print("=" * 60)
print()

# Step 1: Create sample data
print("Step 1: Creating sample data...")
create_sample_data()
append_log("Pipeline started — sample data created")

# Step 2: Load and parse
print("\nStep 2: Loading client data...")
clients = load_clients("raw_clients.txt")
append_log(f"Loaded {len(clients)} clients from raw_clients.txt")

# Step 3: Export to CSV
print("\nStep 3: Exporting to CSV...")
active_clients = [c for c in clients if c["status"] == "active"]
export_csv(active_clients, "active_clients.csv")
append_log(f"Exported {len(active_clients)} active clients to CSV")

# Step 4: Export to JSON
print("\nStep 4: Exporting to JSON...")
export_data = {
    "firm": "Pearson Specter Litt",
    "generated": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
    "total_clients": len(clients),
    "active_clients": len(active_clients),
    "total_revenue": sum(c["revenue"] for c in active_clients),
    "clients": active_clients
}
export_json(export_data, "firm_data.json")
append_log("Exported firm data to JSON")

# Step 5: Generate report
print("\nStep 5: Generating text report...")
generate_report(clients, "portfolio_report.txt")
append_log("Generated portfolio report")

# Step 6: Read back and verify
print("\nStep 6: Verification...")
with open("portfolio_report.txt", "r") as f:
    report_lines = f.readlines()
    print(f"  Report: {len(report_lines)} lines")

with open("firm_data.json", "r") as f:
    json_data = json.load(f)
    print(f"  JSON: {json_data['total_clients']} clients, ${json_data['total_revenue']:,.2f}")

with open("audit_log.txt", "r") as f:
    log_entries = f.readlines()
    print(f"  Audit Log: {len(log_entries)} entries")

append_log("Pipeline completed — all verifications passed")

print(f"\n{'='*60}")
print(f"{'✅ PIPELINE COMPLETE':^60}")
print(f"{'='*60}")
print()
print("Files generated:")
for fname in ["raw_clients.txt", "active_clients.csv", "firm_data.json", 
              "portfolio_report.txt", "audit_log.txt"]:
    if os.path.exists(fname):
        size = os.path.getsize(fname)
        print(f"  📄 {fname:<25} ({size:,} bytes)")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

**Louis:** "File operations are where programs interact with the real world — and the real world is messy."

---

### Pitfall 1: `"w"` Mode Destroys Data 🔥🔥🔥

```python
# Someone's important file
with open("important_data.txt", "w") as f:
    f.write("New data")
# EVERYTHING that was in important_data.txt is GONE!

# ✅ Always check before overwriting
import os

filepath = "important_data.txt"
if os.path.exists(filepath):
    confirm = input(f"'{filepath}' exists. Overwrite? (yes/no): ")
    if confirm.lower() != "yes":
        print("Aborted!")
    else:
        with open(filepath, "w") as f:
            f.write("New data")

# ✅ Or use "x" mode — fails if file exists
try:
    with open("important_data.txt", "x") as f:
        f.write("New data")
except FileExistsError:
    print("File already exists — use a different name or append mode")
```

---

### Pitfall 2: Forgetting to Close Files (Without `with`)

```python
# ❌ If an error occurs, file stays open!
f = open("data.txt", "r")
content = f.read()
raise ValueError("Something broke!")    # File never closed!
f.close()   # Never reached!

# ✅ 'with' handles this automatically
with open("data.txt", "r") as f:
    content = f.read()
    raise ValueError("Something broke!")
# File is STILL closed properly!
```

---

### Pitfall 3: Newline Characters

```python
# readline() and readlines() include the trailing \n
with open("data.txt", "r") as f:
    lines = f.readlines()
    
print(lines)
# ['Wayne Enterprises\n', 'Stark Industries\n', 'Oscorp\n']

# ❌ Comparing without stripping
if lines[0] == "Wayne Enterprises":
    print("Match!")    # Never matches! (has \n at the end)

# ✅ Always strip!
if lines[0].strip() == "Wayne Enterprises":
    print("Match!")    # ✅ Works
```

---

### Pitfall 4: Encoding Issues

```python
# ❌ Default encoding might not handle special characters
# with open("data.txt", "r") as f:  # Might fail on é, ñ, 中文, etc.

# ✅ Specify encoding explicitly
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()

# ✅ For writing too
with open("data.txt", "w", encoding="utf-8") as f:
    f.write("Café résumé naïve — special characters!")
```

---

### Pitfall 5: File Paths on Different Operating Systems

```python
# ❌ Hardcoded path separators
path = "data\\clients\\wayne.txt"    # Windows only!
path = "data/clients/wayne.txt"       # Linux/Mac only!

# ✅ Use os.path.join — works on any OS
import os
path = os.path.join("data", "clients", "wayne.txt")
# Windows: data\clients\wayne.txt
# Linux:   data/clients/wayne.txt
```

---

### Pitfall 6: Reading Huge Files into Memory

```python
# ❌ Loads ENTIRE file into memory — will crash on 5GB files!
with open("huge_log.txt", "r") as f:
    content = f.read()           # Loads everything!
    lines = f.readlines()        # Also loads everything!

# ✅ Process line by line — uses almost no memory
count = 0
with open("huge_log.txt", "r") as f:
    for line in f:               # Reads one line at a time
        if "ERROR" in line:
            count += 1
print(f"Found {count} errors")
```

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "The perfect finale analogy."

---

### 🗄️ Analogy: File I/O Is Like a Physical Filing Cabinet

**Donna:** "Your program is a person sitting at a desk. Files are the filing cabinet."

```
Without File I/O:
  📋 Person has a notepad (memory)
  📋 Everything written disappears when they leave
  📋 Must start from scratch every morning

With File I/O:
  🗄️ Person has a filing cabinet (disk)
  📂 Can save documents for tomorrow
  📂 Can read yesterday's documents
  📂 Can share documents with others
```

| Action | Analogy | Python |
|--------|---------|--------|
| `open("r")` | Open the filing cabinet drawer | `with open(file, "r") as f:` |
| `read()` | Take out the whole folder | `content = f.read()` |
| `readline()` | Read one page at a time | `line = f.readline()` |
| `write()` | Put a new folder in (replacing old) | `f.write(data)` |
| `append()` | Add pages to an existing folder | `open(file, "a")` |
| `close()` | Push the drawer shut | Automatic with `with` |

**Donna:** "And the `with` statement? Think of it as a librarian who always puts the book back on the shelf when you're done, even if you leave in a hurry."

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "Files. The final frontier of Level 1."

---

### 🟢 Beginner Exercises

**Exercise 1: Write and Read**
```
1. Write your name, age, and favorite color to a file called "profile.txt"
   (one per line)
2. Read the file back and print each line
3. Verify the file exists using os.path.exists
```

**Exercise 2: Line Counter**
```
Write a function count_lines(filepath) that:
1. Opens a file
2. Counts total lines, blank lines, and non-blank lines
3. Returns a dictionary with the counts
4. Handles FileNotFoundError gracefully

Test with any text file.
```

**Exercise 3: Append Logger**
```
Write a function log(message, filepath="app.log"):
1. Appends a timestamped message to the log file
2. Format: "[YYYY-MM-DD HH:MM:SS] message"

Call it 5 times with different messages.
Then read and display the full log.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Contact Manager**
```
Build a contact manager that stores contacts in a CSV file.

Functions:
1. add_contact(name, phone, email) → appends to contacts.csv
2. list_contacts() → reads and displays all contacts
3. search_contact(name) → finds and returns matching contact
4. delete_contact(name) → removes a contact (rewrite file without it)

Use csv module. Include a header row.
```

**Exercise 5: File Statistics**
```
Write a function analyze_file(filepath) that returns:
- Total characters
- Total words
- Total lines
- Average word length
- Most common word
- Longest line

Test with a paragraph of text you write to a file first.
```

**Exercise 6: Config File Manager**
```
Create functions to manage a config file:

1. create_default_config(filepath) → creates default key=value pairs
2. read_config(filepath) → returns a dictionary
3. update_config(filepath, key, value) → updates a specific setting
4. display_config(filepath) → formatted display

Support comments (#) and section headers [section].
Handle type conversion (int, float, bool, string).
```

---

### 🔴 Advanced Exercises

**Exercise 7: Student Grade Book (File-Based)**
```
Build a grade book that persists to a JSON file:

1. add_student(name, grades=[]) → adds to gradebook.json
2. add_grade(name, subject, score) → adds a grade
3. student_report(name) → prints all grades and average
4. class_report() → prints all students, averages, rankings
5. export_csv() → exports summary to grades.csv
6. import_csv(filepath) → imports students from CSV

All data should persist between program runs.
```

**Exercise 8: Log File Analyzer**
```
Given a log file with entries like:
  [2026-03-13 09:15:22] INFO: User login | user=harvey
  [2026-03-13 09:16:45] ERROR: DB timeout | server=db-01
  [2026-03-13 09:17:01] WARNING: Disk low | server=web-01

Write a function that:
1. Parses each log entry into a dictionary
2. Counts entries by severity (INFO, WARNING, ERROR)
3. Extracts all unique users and servers
4. Finds the time range (earliest to latest)
5. Exports a summary report to a new file
6. Exports errors only to an errors.txt file
```

**Exercise 9: Multi-File Data Merger**
```
Create 3 CSV files with client data (different columns in each):

  File 1: clients_basic.csv — name, industry
  File 2: clients_financial.csv — name, revenue, expenses
  File 3: clients_cases.csv — name, case_type, attorney

Write a function that:
1. Reads all three files
2. Merges them by client name into unified records
3. Calculates profit and margin for each
4. Exports the merged data to clients_complete.csv
5. Generates a formatted text report
6. Handles missing data gracefully (not all clients in all files)
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "File bugs."

### Buggy Code 🐛

```python
# Bug Hunt: File Operations
# Find and fix ALL the bugs!

# Bug 1: File mode error
with open("data.txt", "r") as f:
    f.write("Hello!")          # Writing to a read-only file!

# Bug 2: Forgetting newlines
with open("names.txt", "w") as f:
    f.write("Harvey")
    f.write("Mike")
    f.write("Louis")
# File contains: HarveyMikeLouis (all one line!)

# Bug 3: writelines doesn't add newlines
names = ["Harvey", "Mike", "Louis"]
with open("names.txt", "w") as f:
    f.writelines(names)
# File contains: HarveyMikeLouis

# Bug 4: Reading after writing (same file handle)
with open("data.txt", "w") as f:
    f.write("Test data\n")
    content = f.read()         # Can you read from a write-mode file?
print(content)

# Bug 5: Not stripping newlines when comparing
with open("names.txt", "r") as f:
    first_name = f.readline()
    
if first_name == "Harvey":
    print("Found Harvey!")     # Never prints!

# Bug 6: Overwriting when you meant to append
with open("log.txt", "w") as f:
    f.write("Entry 1\n")
    
with open("log.txt", "w") as f:    # Opens again with "w"!
    f.write("Entry 2\n")
# log.txt only contains: Entry 2
```

---

### Fixed Code ✅

```python
# Fixed: File Operations

# Fix 1: Use write mode to write
with open("data.txt", "w") as f:      # ✅ "w" mode for writing
    f.write("Hello!")

# Fix 2: Add \n for new lines
with open("names.txt", "w") as f:
    f.write("Harvey\n")                # ✅ Newline added
    f.write("Mike\n")
    f.write("Louis\n")

# Fix 3: Add newlines to each item
names = ["Harvey\n", "Mike\n", "Louis\n"]     # ✅ Include \n
with open("names.txt", "w") as f:
    f.writelines(names)
# Or use a loop: for name in names: f.write(name + "\n")

# Fix 4: Open separately for writing and reading
with open("data.txt", "w") as f:
    f.write("Test data\n")
    
with open("data.txt", "r") as f:      # ✅ Open again in read mode
    content = f.read()
print(content)

# Fix 5: Strip whitespace when comparing
with open("names.txt", "r") as f:
    first_name = f.readline().strip()  # ✅ Strip \n

if first_name == "Harvey":
    print("Found Harvey!")             # ✅ Now works!

# Fix 6: Use append mode for adding to existing file
with open("log.txt", "w") as f:
    f.write("Entry 1\n")

with open("log.txt", "a") as f:       # ✅ "a" for append
    f.write("Entry 2\n")
# log.txt contains both entries!
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | Writing to a file opened in `"r"` mode | File modes |
| 2 | `write()` doesn't add newlines automatically | Manual `\n` needed |
| 3 | `writelines()` doesn't add newlines either | Include `\n` in strings |
| 4 | Can't read from a `"w"` mode file | Separate read/write operations |
| 5 | `readline()` includes `\n` — comparison fails | Always `.strip()` |
| 6 | `"w"` mode overwrites — use `"a"` to append | Write vs append mode |

---

## 8. 🌍 Real-World Application

**Harvey:** "Files are how programs communicate with the world."

### Where File I/O Is Used:

| Domain | File I/O Use Case |
|--------|------------------|
| **Configuration** | Reading `config.ini`, `settings.yaml`, `.env` files |
| **Logging** | Writing application logs, error logs, audit trails |
| **Data Processing** | Reading/writing CSV, JSON, XML data files |
| **Web Development** | Generating HTML, reading templates |
| **DevOps** | Parsing log files, generating deployment scripts |
| **Data Science** | Loading datasets, exporting results |
| **Automation** | Reading input lists, writing output reports |
| **Testing** | Reading test fixtures, writing test results |

### Real Production Example: Log Rotation System

```python
import os
from datetime import datetime

class Logger:
    """A file-based logging system with rotation."""
    
    def __init__(self, base_path="logs", max_size_kb=100):
        self.base_path = base_path
        self.max_size_kb = max_size_kb
        self.current_file = None
        
        # Create logs directory if it doesn't exist
        if not os.path.exists(base_path):
            os.makedirs(base_path)
        
        self._set_current_file()
    
    def _set_current_file(self):
        """Set the current log file path."""
        date_str = datetime.now().strftime("%Y-%m-%d")
        self.current_file = os.path.join(self.base_path, f"app_{date_str}.log")
    
    def _check_rotation(self):
        """Rotate log if it exceeds max size."""
        if os.path.exists(self.current_file):
            size_kb = os.path.getsize(self.current_file) / 1024
            if size_kb >= self.max_size_kb:
                timestamp = datetime.now().strftime("%H%M%S")
                rotated = self.current_file.replace(".log", f"_{timestamp}.log")
                os.rename(self.current_file, rotated)
                print(f"  🔄 Log rotated: {rotated}")
    
    def log(self, level, message):
        """Write a log entry."""
        self._check_rotation()
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        entry = f"[{timestamp}] {level.upper():>7}: {message}\n"
        
        with open(self.current_file, "a", encoding="utf-8") as f:
            f.write(entry)
    
    def info(self, message):
        self.log("INFO", message)
    
    def warning(self, message):
        self.log("WARNING", message)
    
    def error(self, message):
        self.log("ERROR", message)
    
    def read_recent(self, n=10):
        """Read the last n log entries."""
        if not os.path.exists(self.current_file):
            return []
        with open(self.current_file, "r") as f:
            lines = f.readlines()
        return [line.strip() for line in lines[-n:]]

# Usage
logger = Logger()
logger.info("Application started")
logger.info("Loading client database")
logger.warning("Cache miss — rebuilding index")
logger.error("Failed to connect to backup server")
logger.info("Processing 237 client records")

print("\nRecent log entries:")
for entry in logger.read_recent():
    print(f"  {entry}")
```

---

## 9. ✅ Knowledge Check

**Harvey:** "Final quiz of Level 1."

---

**Q1.** What is the difference between `"r"`, `"w"`, and `"a"` file modes?

**Q2.** Why should you use `with open(...)` instead of manually calling `open()` and `close()`?

**Q3.** What is the difference between `read()`, `readline()`, and `readlines()`?

**Q4.** What is the most memory-efficient way to read a large file?

**Q5.** What does `write()` return? Does it add a newline?

**Q6.** Why might `readline()` comparisons fail? How do you fix it?

**Q7.** What Python module is used for CSV files? What class reads CSV into dictionaries?

**Q8.** How do you convert a Python dictionary to JSON and save it to a file?

**Q9.** Why is specifying `encoding="utf-8"` important?

**Q10.** What happens if you open a file with `"w"` mode that already has content?

---

### 📋 Answer Key

> **Q1.** `"r"` = read only (file must exist). `"w"` = write (creates or **overwrites**). `"a"` = append (creates or adds to end).

> **Q2.** `with` guarantees the file is closed even if an error occurs. Manual `close()` can be skipped if an exception happens.

> **Q3.** `read()` = entire file as one string. `readline()` = one line at a time. `readlines()` = all lines as a list.

> **Q4.** Iterate directly: `for line in f:` — reads one line at a time without loading the whole file.

> **Q5.** Returns the number of characters written. No, it does NOT add a newline — you must include `\n` yourself.

> **Q6.** `readline()` includes the trailing `\n`. Fix: `line.strip()` before comparing.

> **Q7.** The `csv` module. `csv.DictReader` reads rows as dictionaries with column headers as keys.

> **Q8.** `import json; json.dump(my_dict, file_object, indent=2)`

> **Q9.** Different systems use different default encodings. `utf-8` ensures special characters (é, ñ, 中文) are handled correctly across all platforms.

> **Q10.** All existing content is **immediately erased** — the file becomes empty before you write anything new.

---

## 🎬 End of Lesson Scene — Level 1 Finale

*It's 6:00 PM on a Friday. Two weeks since you joined the firm. Harvey, Mike, Louis, Donna, Rachel, and Jessica are standing around your desk, watching the pipeline run.*

```
Step 1: Creating sample data...     ✅
Step 2: Loading 8 clients...        ✅
Step 3: Exporting to CSV...         ✅
Step 4: Exporting to JSON...        ✅
Step 5: Generating report...        ✅
Step 6: Verification passed         ✅

✅ PIPELINE COMPLETE
```

**Jessica:** *(reading the portfolio report)* "Revenue breakdown by case type. Tier distribution. Sorted by profitability. This is exactly what the partners need."

**Harvey:** "And it persists. Shut down the machine, restart it tomorrow, and all the data is still there."

**Mike:** "Two weeks ago, you didn't know what a variable was. Now you've built a complete data pipeline — loading text files, parsing structured data, processing with classes and functions, exporting to CSV and JSON, generating reports, and logging everything."

**Louis:** "The error handling is… acceptable." *(This is high praise from Louis.)*

**Donna:** "Level 1 — complete."

**Harvey:** *(standing up)* "Level 2 starts Monday. You'll apply everything you've learned to build real projects — exception handling, modules, inheritance, file processing, testing. It's where beginners become professionals."

*He pauses at the door.*

**Harvey:** "Good first two weeks. Don't let it go to your head."

*The elevator doors close. Level 1 is done. Fourteen lessons. From variables to file I/O. From scripts to systems. The foundation is set.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | File modes — `"r"`, `"w"`, `"a"`, `"x"` | ✅ |
| 2 | Context manager — `with open() as f:` | ✅ |
| 3 | Reading — `read()`, `readline()`, `readlines()`, iteration | ✅ |
| 4 | Writing — `write()`, `writelines()`, `print(file=f)` | ✅ |
| 5 | Appending to files | ✅ |
| 6 | Parsing structured text (CSV, key-value, delimited) | ✅ |
| 7 | File paths with `os.path` | ✅ |
| 8 | Error handling with `try/except` for files | ✅ |
| 9 | `csv` module — `reader`, `writer`, `DictReader`, `DictWriter` | ✅ |
| 10 | `json` module — `dump`, `load` | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| Strings + `split()` (Lesson 2) | Parsing file lines |
| Lists (Lesson 3) | Storing parsed data |
| Dictionaries (Lesson 5) | CSV → dict, JSON → dict |
| `for` loops (Lesson 9) | Iterating over file lines |
| Functions (Lesson 11) | Modular file operations |
| Classes (Lesson 12) | Logger class |
| f-strings (Lesson 2) | Formatted file output |
| `zip()` (Lesson 9) | `dict(zip(headers, values))` |
| Boolean operators (Lesson 8) | File existence checks |
| `input()` (Lesson 6) | Interactive file operations |

---

## 🏆 Level 1 Complete — Summary

| Lesson | Topic | Key Skill |
|--------|-------|-----------|
| 1 | Variables & Numbers | Store and calculate data |
| 2 | Strings & Output | Text manipulation, f-strings |
| 3 | Lists | Ordered, mutable collections |
| 4 | Tuples | Immutable sequences, unpacking |
| 5 | Dictionaries | Key-value data structures |
| 6 | User Input | Interactive programs |
| 7 | If Statements | Decision-making |
| 8 | Boolean Operators | Complex logic |
| 9 | For Loops | Iteration & automation |
| 10 | While Loops | Indefinite repetition |
| 11 | Functions | Reusable code blocks |
| 12 | Classes | Blueprints for objects |
| 13 | Objects | Instantiation & management |
| 14 | File I/O | Persistent data storage |

> **Next → Level 2: Applying**  
> *Exception handling, modules, inheritance, advanced file processing, testing, and building real projects. You have the tools. Now learn to use them like a professional.*
