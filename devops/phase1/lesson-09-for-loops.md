# Phase 1 · Lesson 9 — For Loops
### *"The Automation Assembly Line"*

> **O'Reilly Skill:** Control program flow with for loops.
> **DevOps connection:** Loop over servers, pods, log lines, config entries. This is where you go from "script" to "automation".
> **10-min sprint:** Write a loop that checks 5 servers. Print "✅ running" or "🔴 down" for each. Count how many are down.

---

## Why This Matters (Harvey's take)

Writing the same code 237 times is not engineering — that's stenography. One `for` loop handles 237 clients, 500 servers, or a million log entries with the same 3 lines. Combined with `if` statements, it becomes powerful: process every server, make a decision about each, take action automatically.

---

## The Core Concept (Mike's breakdown)

### Basic `for` Loop

```python
servers = ["web-01", "web-02", "api-01", "db-01"]

for server in servers:
    print(f"Checking: {server}")

print("All servers checked")    # NOT indented — runs ONCE after loop
```

**Anatomy:**
```python
for server in servers:      # loop variable | iterable
    print(server)           # body — runs once per item
```

### Loop Over Different Types

```python
# List
for pod in ["pod-1", "pod-2", "pod-3"]:
    print(pod)

# String — character by character
for char in "DEVOPS":
    print(char)

# Dictionary — gives keys by default
config = {"host": "localhost", "port": 5432}
for key in config:
    print(f"{key}: {config[key]}")

# Dictionary .items() — MOST USEFUL pattern
for key, value in config.items():
    print(f"{key}: {value}")
```

### `range()` — Loop a Specific Number of Times

```python
for i in range(5):          # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 6):       # 1, 2, 3, 4, 5
    print(i)

for i in range(0, 20, 5):   # 0, 5, 10, 15
    print(i)

for i in range(5, 0, -1):   # 5, 4, 3, 2, 1 (countdown)
    print(i)
```

> ⚠️ `range()` stop is **exclusive** — same as slicing. `range(5)` = 0,1,2,3,4.

### `enumerate()` — Index + Value Together ⭐

```python
servers = ["web-01", "web-02", "api-01"]

# ❌ Clunky way
for i in range(len(servers)):
    print(f"{i}: {servers[i]}")

# ✅ Pythonic way
for i, server in enumerate(servers):
    print(f"{i}: {server}")

# Start counting from 1
for rank, server in enumerate(servers, start=1):
    print(f"{rank}. {server}")
```

### `zip()` — Parallel Lists Together ⭐

```python
servers = ["web-01", "api-01", "db-01"]
statuses = ["running", "stopped", "running"]
cpu = [45.2, 0.0, 23.1]

for server, status, cpu_pct in zip(servers, statuses, cpu):
    icon = "✅" if status == "running" else "🔴"
    print(f"{icon} {server}: {status} ({cpu_pct}% CPU)")
```

### Loop Control: `break`, `continue`, `else`

```python
# break — exit the loop immediately
for server in servers:
    if server == "CRITICAL":
        print(f"Found critical server: {server}")
        break     # stop searching

# continue — skip this iteration, go to next
for server in servers:
    if "inactive" in server:
        continue  # skip inactive servers
    print(f"Processing: {server}")

# for-else — else runs ONLY if loop finished without break
for server in servers:
    if server == "db-primary":
        print("Primary DB found!")
        break
else:
    print("⚠️ Primary DB not found in cluster!")
```

### Common Patterns

**Accumulator:**
```python
cpu_readings = [45.2, 78.3, 23.1, 91.0, 65.7]
total = 0
count_high = 0
for cpu in cpu_readings:
    total += cpu
    if cpu > 80:
        count_high += 1
print(f"Avg: {total/len(cpu_readings):.1f}%  High: {count_high}")
```

**Build a new collection (filter):**
```python
all_servers = ["web-01", "api-01", "web-02", "db-01"]
web_servers = []
for s in all_servers:
    if s.startswith("web"):
        web_servers.append(s)
print(web_servers)   # ['web-01', 'web-02']
```

**Grouping into a dictionary:**
```python
logs = [("ERROR", "timeout"), ("INFO", "started"), ("ERROR", "crash"), ("INFO", "ping")]

by_level = {}
for level, message in logs:
    if level not in by_level:
        by_level[level] = []
    by_level[level].append(message)

# {'ERROR': ['timeout', 'crash'], 'INFO': ['started', 'ping']}
```

---

## DevOps Example — Log File Analyzer

```python
logs = [
    "[09:15] INFO: User login | user=harvey",
    "[09:16] ERROR: DB timeout | server=db-01",
    "[09:17] INFO: File uploaded | user=mike",
    "[09:18] WARNING: Disk low | server=web-01",
    "[09:19] ERROR: Auth failed | user=unknown",
]

counts = {"INFO": 0, "WARNING": 0, "ERROR": 0}
errors = []

for log in logs:
    for level in counts:
        if level in log:
            counts[level] += 1
            if level == "ERROR":
                errors.append(log)
            break

print("Log Summary:")
for level, count in counts.items():
    bar = "█" * (count * 5)
    print(f"  {level:<10} {count}  {bar}")

print(f"\n⚠️ {len(errors)} error(s):")
for err in errors:
    print(f"  {err}")
```

---

## Gotchas (Louis's warnings)

**1. Never modify a list while iterating:**
```python
# ❌ Skips elements silently
for server in servers:
    if "down" in server:
        servers.remove(server)

# ✅ Iterate over a copy
for server in servers[:]:
    if "down" in server:
        servers.remove(server)
```

**2. `range()` stop is exclusive:**
```python
for i in range(5):   # 0,1,2,3,4 — NOT 5!
for i in range(1,6): # 1,2,3,4,5 — to include 5, stop at 6
```

**3. Loop variable shadows outer variables:**
```python
server = "production-db"    # important variable!
for server in all_servers:  # 'server' now means something different!
    print(server)
print(server)               # now prints last item from all_servers!
# Fix: use distinct names (s, srv, current_server)
```

**4. Empty iterable silently does nothing:**
```python
servers = []       # empty — maybe data loading failed
for s in servers:
    process(s)     # never runs — no error, no warning
# Add a guard:
if not servers:
    print("Warning: no servers to process!")
```

**5. Use `_` when you don't need the loop variable:**
```python
for _ in range(3):
    print("retry attempt")    # just need it 3 times
```

---

## Mental Model (Donna's analogy)

`for` loops are an **assembly line**:

```
Conveyor Belt: [Server A] [Server B] [Server C] [Server D]
                   ↓           ↓           ↓           ↓
             [🔧 Check]  [🔧 Check]  [🔧 Check]  [🔧 Check]
                   ↓           ↓           ↓           ↓
             [📋 Report] [📋 Report] [📋 Report] [📋 Report]
```

- Conveyor belt = your list (the iterable)
- Each box = one item
- Work station = your loop body
- `break` = emergency stop button
- `continue` = "this box is defective — skip it"
- `enumerate()` = stamp each box with a number AND process it
- `zip()` = two belts side-by-side, process a pair at a time

---

## Practice (pick one, 10 minutes)

**Beginner:** Given `hours = [45, 52, 38, 61, 48, 55, 42]` — using a loop (no `sum()` or `max()`), calculate total, average, highest, and count of weeks above 50.

**Intermediate:** Parse this CSV: `"web-01,running,45.2\nweb-02,stopped,0.0\napi-01,running,78.9"`. Use loops to calculate average CPU of running servers and list which are stopped.

**DevOps-specific:** You have a list of log lines. Count by level (INFO/WARNING/ERROR). For each ERROR, extract the server name (it's after "server="). Print a summary report.

---

## Quick Debug — find all 6 bugs

```python
clients = ["Wayne", "Stark", "Oscorp"]
hours = [85, 62, 45]

for i in range(1, len(clients)):    # Bug 1
    print(clients[i])

for i in range(len(clients) + 1):   # Bug 2
    print(clients[i])

total = 0
for h in hours:
    total = h                        # Bug 3
print(f"Total: {total}")

for client in clients:
    result = client.upper()
print(f"All caps: {result}")        # Bug 4 — only last one

data = [("Wayne", 85), ("Stark", 62)]
for name in data:                    # Bug 5
    print(f"{name}: {hours} hours")

for i, name in enumerate(clients):
    print(f"Client {i}: {name}")    # Bug 6 — expected 1,2,3
```

---

## Knowledge Check

1. What does `range(3, 8)` produce?
2. What's the difference between `for i in range(len(list))` and `for item in list`?
3. What does `enumerate()` return on each iteration?
4. What happens with `zip()` when lists have different lengths?
5. When does the `else` block of a `for-else` run?

**Answers:** (1) 3,4,5,6,7. (2) First gives indices; second gives values. Use first only when you need the index position. (3) `(index, value)` tuples. (4) Stops at the shortest. (5) Only if the loop finished without hitting `break`.

---

> **Next:** Lesson 11 — Functions → The same billing logic lives in 3 different places. Time to write it once and reuse it everywhere.
