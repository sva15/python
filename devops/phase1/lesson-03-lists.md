# Phase 1 · Lesson 3 — Lists
### *"The Client Database Crisis"*

> **O'Reilly Skill:** Create, modify, and retrieve data from lists.
> **DevOps connection:** Lists of servers, container IDs, pod names, log entries — you loop over these constantly.
> **10-min sprint:** Create a list of 5 server names. Sort them. Find the longest name. Remove one. Add two new ones.

---

## Why This Matters (Harvey's take)

Without lists, you'd need `client_1`, `client_2`... `client_500`. That's not engineering — that's insanity. Lists let you store any number of related items and process them with loops. They're the first true **data structure** you'll learn.

---

## The Core Concept (Mike's breakdown)

### Creating Lists

```python
# Empty list
servers = []

# List of strings
pods = ["api-pod-1", "api-pod-2", "api-pod-3"]

# List of numbers
ports = [8080, 8081, 8082, 9090]

# Mixed types (Python allows this)
record = ["api-node-1", "running", 87.3, True]

# Nested lists (list of lists)
cluster = [
    ["web-01", "web-02"],    # web tier
    ["api-01", "api-02"],    # api tier
    ["db-01"],               # db tier
]
```

### Indexing & Slicing (same as strings)

```python
servers = ["web-01", "web-02", "api-01", "db-01"]
#           0         1         2         3       ← positive
#          -4        -3        -2        -1       ← negative

print(servers[0])      # web-01
print(servers[-1])     # db-01 (last)
print(servers[1:3])    # ['web-02', 'api-01']
print(servers[::-1])   # reversed
print(len(servers))    # 4
```

### Lists ARE Mutable (unlike strings)

```python
servers[2] = "api-02"       # ✅ works — replace by index
print(servers)               # ['web-01', 'web-02', 'api-02', 'db-01']
```

### Adding Elements

```python
pods = ["pod-1", "pod-2"]

pods.append("pod-3")          # add ONE item to end
pods.insert(1, "pod-0")       # insert at specific index
pods.extend(["pod-4", "pod-5"])  # add ALL items from another list
pods += ["pod-6"]             # shorthand for extend
```

> ⚠️ **append vs extend:** `append([4,5])` adds the LIST as one item → `[..., [4,5]]`. `extend([4,5])` adds each item individually → `[..., 4, 5]`.

### Removing Elements

```python
servers = ["web-01", "web-02", "api-01", "db-01"]

servers.remove("web-02")    # remove by VALUE (ValueError if not found)
fired = servers.pop(1)      # remove by INDEX, returns removed item
last = servers.pop()        # remove LAST item, returns it
del servers[0]              # remove by index, no return
servers.clear()             # remove everything
```

### Searching

```python
pods = ["api-01", "api-02", "web-01"]

print("api-01" in pods)         # True
print(pods.index("api-02"))     # 1
print(pods.count("web-01"))     # 1
```

### Sorting

```python
ports = [9090, 8080, 3000, 8081]

ports.sort()                    # sorts IN-PLACE, returns None
ports.sort(reverse=True)        # descending

ordered = sorted(ports)         # returns NEW sorted list, original unchanged
```

> ⚠️ **Critical trap:** `result = [3,1,2].sort()` → result is `None`! Use `sorted()` if you want to assign.

### Useful Built-ins

```python
hours = [120, 95, 140, 88, 110]
print(sum(hours))          # 553
print(min(hours))          # 88
print(max(hours))          # 140
print(sum(hours)/len(hours))  # 110.6 (average)
```

---

## DevOps Example — Server Health Report

```python
servers = ["web-01", "web-02", "api-01", "db-01", "api-02"]
statuses = ["running", "stopped", "running", "running", "stopped"]
cpu_usage = [45.2, 0.0, 78.9, 23.1, 0.0]

down_servers = []
for i in range(len(servers)):
    if statuses[i] == "stopped":
        down_servers.append(servers[i])

print(f"Total servers: {len(servers)}")
print(f"Down servers: {down_servers}")
print(f"Avg CPU (running): {sum(cpu_usage)/len(cpu_usage):.1f}%")
```

---

## Gotchas (Louis's warnings)

**1. The alias trap — the #1 list bug:**
```python
original = ["web-01", "web-02"]
copy = original          # NOT a copy — both point to the same list!
copy.append("api-01")
print(original)          # ['web-01', 'web-02', 'api-01'] — CHANGED!

# Fix:
safe_copy = original.copy()   # or original[:]  or list(original)
```

**2. Never mutate a list while iterating:**
```python
# DANGEROUS
for server in servers:
    if "down" in server:
        servers.remove(server)   # skips elements silently!

# SAFE — iterate over a copy
for server in servers.copy():
    if "down" in server:
        servers.remove(server)
```

**3. `IndexError` — access beyond list length:**
```python
items = ["a", "b", "c"]
items[3]       # IndexError! Valid: 0, 1, 2
items[5:10]    # [] — slicing never errors, just returns what's available
```

**4. Nested list repetition trap:**
```python
# ❌ All rows share the same inner list!
grid = [[0] * 3] * 3
grid[0][0] = 99
print(grid)   # [[99,0,0], [99,0,0], [99,0,0]] — all changed!

# ✅ Use a loop
grid = [[0]*3 for _ in range(3)]
```

---

## Mental Model (Donna's analogy)

Lists are a **filing cabinet drawer** with numbered slots:
- `[0]` = first slot, `[-1]` = last slot
- `append()` = add a file at the end
- `insert(i, x)` = squeeze a file in at position i
- `remove(x)` = pull a specific file out
- `sort()` = rearrange all files alphabetically

Key difference from strings: the files **slide in and out** (mutable). With strings, the charms are welded on.

---

## Practice (pick one, 10 minutes)

**Beginner:** Given `scores = [85, 92, 78, 95, 88, 71, 90]` — find sum, average, highest, lowest, and count of scores above 85.

**Intermediate:** Two server lists from two regions. Combine them. Find servers in both (intersection). Remove duplicates. Sort.

**DevOps-specific:** Parse this CSV string `"web-01,running,45.2\nweb-02,stopped,0.0\napi-01,running,78.9"` into a list of lists. Print only the running servers.

---

## Quick Debug — find all 7 bugs

```python
team = ("Harvey", "Mike", "Donna")    # Bug 1
team.append["Louis"]                   # Bug 2
team.remove("Jessica")                # Bug 3
backup = team                          # Bug 4
backup.append("Rachel")
sorted_team = team.sort()             # Bug 5
last = team[len(team)]                # Bug 6
combined = team_a.extend(team_b)      # Bug 7
print(combined)
```

---

## Knowledge Check

1. What is the difference between `append()` and `extend()`?
2. What is the alias trap? How do you safely copy a list?
3. What is the difference between `sort()` and `sorted()`?
4. What does `[1,2,3,4,5][-2:]` return?
5. What's the output? `a = [1,2,3]; b = a; a.append(4); print(b)`

**Answers:** (1) `append` adds one item (even if it's a list); `extend` adds each item from an iterable. (2) `b = a` makes both point to the same object. Use `.copy()`. (3) `sort()` modifies in-place, returns None. `sorted()` returns a new list. (4) `[4, 5]`. (5) `[1, 2, 3, 4]` — b is an alias.

---

> **Next:** Lesson 5 — Dictionaries → The #1 most-used data structure in DevOps and AI agents.
