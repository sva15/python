# Phase 1 · Lesson 5 — Dictionaries
### *"The Rolodex Reborn"*

> **O'Reilly Skill:** Create, modify, and retrieve data from dictionaries.
> **DevOps connection:** JSON API responses, YAML configs, K8s manifests, environment variables — all dictionaries. This is the #1 most-used data structure in DevOps.
> **10-min sprint:** Create a dict for one server: name, IP, status, CPU%. Read each field. Update the status. Delete the CPU field.

---

## Why This Matters (Harvey's take)

Lists use numbers to find data. But in real life you say "give me **Wayne Enterprises'** file" — not "give me client number 47." Dictionaries let you look up data by a **meaningful label** (key) instead of an arbitrary number. JSON is a dictionary. Every API response is a dictionary. Kubernetes configs are dictionaries. Master this and you understand data on the internet.

---

## The Core Concept (Mike's breakdown)

### Creating Dictionaries

```python
# Empty dict
empty = {}

# Server config — real DevOps use
server = {
    "name": "api-node-1",
    "ip": "10.0.1.45",
    "status": "running",
    "cpu": 78.3,
    "port": 8080
}

# From keyword arguments
settings = dict(theme="dark", timeout=30, retries=3)

# From a list of tuples
pairs = [("host", "localhost"), ("port", 5432)]
db_config = dict(pairs)
```

### Accessing Values

```python
server = {"name": "api-01", "status": "running", "cpu": 78.3}

# Method 1: brackets — crashes if key missing
print(server["name"])      # api-01
# print(server["region"])  # KeyError!

# Method 2: .get() — safe, returns None or your default
print(server.get("region"))           # None
print(server.get("region", "us-east-1"))  # us-east-1
```

| Method | Key exists | Key missing |
|--------|-----------|-------------|
| `dict["key"]` | Returns value | 💥 KeyError |
| `dict.get("key")` | Returns value | None |
| `dict.get("key", default)` | Returns value | default |

**Rule:** Use `[]` when you're sure the key exists. Use `.get()` when it might not.

### Adding and Modifying

```python
server = {"name": "api-01"}

server["status"] = "running"     # add new key
server["name"] = "api-node-1"    # modify existing key — no error

# Merge another dict in
updates = {"cpu": 45.2, "port": 8080}
server.update(updates)
```

### Removing

```python
del server["cpu"]                     # remove by key
value = server.pop("status")          # remove and return value
value = server.pop("region", None)    # safe remove — no error if missing
```

### Iterating

```python
config = {"host": "localhost", "port": 5432, "db": "prod"}

for key in config:                    # keys only
    print(key)

for value in config.values():         # values only
    print(value)

for key, value in config.items():    # BOTH — use this pattern constantly
    print(f"{key}: {value}")
```

> **Memorize:** `for key, value in dict.items():` — you'll write this hundreds of times.

### Nested Dictionaries

```python
infrastructure = {
    "web": {
        "servers": ["web-01", "web-02"],
        "load_balancer": "nginx",
        "port": 80
    },
    "database": {
        "host": "db-primary.internal",
        "port": 5432,
        "replicas": 2
    }
}

# Access nested data — chain the keys
print(infrastructure["web"]["port"])           # 80
print(infrastructure["database"]["host"])      # db-primary.internal
print(infrastructure["web"]["servers"][0])     # web-01

# Safe nested access
region = infrastructure.get("web", {}).get("region", "us-east-1")
```

---

## DevOps Example — API Response Handler

```python
# This is what json.loads() gives you from any REST API
api_response = {
    "status": "success",
    "data": {
        "servers": [
            {"id": "i-001", "type": "t3.medium", "state": "running"},
            {"id": "i-002", "type": "t3.large", "state": "stopped"},
        ],
        "total": 2
    },
    "meta": {"request_id": "abc-123"}
}

if api_response["status"] == "success":
    servers = api_response["data"]["servers"]
    for s in servers:
        print(f"{s['id']} ({s['type']}): {s['state']}")

print(f"Request: {api_response['meta']['request_id']}")
```

---

## Gotchas (Louis's warnings)

**1. KeyError — the silent killer:**
```python
config = {"host": "localhost"}
port = config["port"]    # KeyError! Always use .get() when unsure
```

**2. Duplicate keys — last one wins silently:**
```python
d = {"host": "prod", "port": 5432, "host": "staging"}
print(d["host"])   # "staging" — "prod" is GONE, no warning!
```

**3. `in` checks KEYS, not values:**
```python
config = {"host": "localhost", "port": 5432}
print("localhost" in config)          # False — "localhost" is a VALUE
print("host" in config)               # True — "host" is a KEY
print("localhost" in config.values()) # True ✅
```

**4. Never modify while iterating:**
```python
# ❌ RuntimeError
for key in config:
    if key == "temp":
        del config[key]

# ✅ Iterate over a copy of keys
for key in list(config.keys()):
    if key == "temp":
        del config[key]
```

**5. Shallow copy trap:**
```python
original = {"host": "prod", "ports": [80, 443]}
copy = original.copy()
copy["ports"].append(8080)         # Modifies the ORIGINAL list too!
# Fix: import copy; deep = copy.deepcopy(original)
```

---

## Mental Model (Donna's analogy)

Dictionaries are a **Rolodex**:
- Each tab label = the **key** (must be unique)
- The card behind it = the **value**
- `dict["Wayne Enterprises"]` = flip to that tab and read the card
- `del dict["Luthor Corp"]` = pull the card out and shred it
- `"email" in dict` = does that tab label exist?

| Structure | Access by | Mutable? |
|-----------|----------|----------|
| String | Position [i] | ❌ |
| List | Position [i] | ✅ |
| Dictionary | Key ["name"] | ✅ |

---

## Practice (pick one, 10 minutes)

**Beginner:** Create a server config dict with host, port, environment, and debug fields. Print each field with `items()`. Update the environment. Use `.get()` to safely read a field that doesn't exist.

**Intermediate:** Write a word-counter. Given a sentence, build a dict where each key is a word and the value is how many times it appears.

**DevOps-specific:** You have a list of log lines. Parse them into a dict counting how many `INFO`, `WARNING`, and `ERROR` messages there are. Use `.get(level, 0) + 1` pattern.

---

## Quick Debug — find all 7 bugs

```python
client = {
    name: "Wayne Enterprises",        # Bug 1
    "contact": "Bruce Wayne",
}
email = client["email"]               # Bug 2
client.append({"phone": "555-0101"})  # Bug 3

rates = {"Harvey": 450, "Mike": 375}
for name in rates:
    if rates[name] < 400:
        del rates[name]               # Bug 4

combined = base + extra               # Bug 5

if "Harvey" in data:                  # Bug 6 — are you sure?
    print("Found Harvey")

print(f"Entries: {scores.count()}")   # Bug 7
```

---

## Knowledge Check

1. What's the difference between `dict["key"]` and `dict.get("key")`?
2. What does `{"a":1, "b":2, "a":3}` produce?
3. How do you iterate over both keys AND values simultaneously?
4. Why can tuples be dict keys but lists cannot?
5. Write one line that safely increments `counts["error"]` even if the key doesn't exist yet.

**Answers:** (1) `[]` raises KeyError if missing; `.get()` returns None or a default. (2) `{"a":3,"b":2}` — last value wins silently. (3) `for k, v in d.items():`. (4) Keys must be hashable (immutable); lists are mutable. (5) `counts["error"] = counts.get("error", 0) + 1`.

---

> **Next:** Lesson 6 — User Input → Harvey's contract generator needs to accept live input from associates.
