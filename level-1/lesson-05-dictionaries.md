# 🏛️ Level 1 – Exploring | Lesson 5: Work with Dictionaries in Python

## *"The Rolodex Reborn"*

> **O'Reilly Skill Objective:** *Create, modify, and retrieve data from dictionaries.*

> **Previously Learned:** Variables, strings, lists (mutable, ordered), tuples (immutable, ordered), indexing, slicing, `in`, f-strings, `type()`, named tuples

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

*It's 8:30 AM. You arrive to find Donna standing at her desk, staring at a thick, leather-bound book — an actual, physical Rolodex sitting next to it.*

**Donna:** "Don't say a word."

**You:** "Is that a—"

**Donna:** "I said don't. Yes, it's a Rolodex. It's how Jessica tracked client contacts before the firm went digital. Every client's name, company, phone number, billing tier — all on little cards, organized alphabetically."

*She spins the Rolodex.*

**Donna:** "The problem? Jessica wants this data digitized *today*. She has a meeting with the managing partners at 3 PM, and she needs to look up any client's information *instantly* — by name, not by some slot number."

*Harvey walks over.*

**Harvey:** "Lists and tuples won't cut it here. Why?"

**You:** "Because lists use numeric indices. If I want to find Harvey's billing rate, I'd need to know he's at index 0. That doesn't scale."

*Harvey raises an eyebrow.*

**Harvey:** "Good. You're learning. What you need is a data structure where you look up information by **name**, not by **position**. The client's name is the **key**, and their information is the **value**."

**Mike:** *(sliding in on his chair)* "That's a **dictionary**. The single most important data structure in Python."

**Harvey:** "Don't oversell it."

**Mike:** "I'm not. JSON, APIs, databases, configuration files, caching — it's all dictionaries under the hood."

**Harvey:** "Fine. Teach them."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Let me frame this."

**Harvey:** "Everything we've learned so far stores data by **position**:"

| Data Structure | How You Access Data |
|---------------|-------------------|
| String | `name[0]` — by character position |
| List | `clients[3]` — by index number |
| Tuple | `record[2]` — by index number |

**Harvey:** "But in the real world, you don't say 'Give me client number 47.' You say 'Give me **Wayne Enterprises'** file.' You look up data by a **meaningful label**, not an arbitrary number."

**Harvey:** "That's what a dictionary does. It maps **keys** to **values**:"

```
"Harvey Specter"    →  Senior Partner, $450/hr
"Mike Ross"         →  Senior Associate, $375/hr
"Louis Litt"        →  Named Partner, $425/hr
```

**Harvey:** "Three facts about dictionaries:"

1. **Fast lookups** — Finding a value by key is nearly instantaneous, no matter how many entries you have (O(1) time)
2. **Flexible values** — Values can be any type: strings, numbers, lists, even other dictionaries
3. **Unique keys** — Each key appears only once. No duplicates

**Harvey:** "In Lessons 3 and 4, you learned to manage *collections*. Today, you learn to manage *relationships*. This changes everything."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "Dictionaries are the Swiss Army knife of Python. Let's master them."

---

### 3.1 Creating Dictionaries

```python
# Empty dictionary
empty = {}
also_empty = dict()

# Dictionary with data — key: value pairs
attorney = {
    "name": "Harvey Specter",
    "title": "Senior Partner",
    "hourly_rate": 450.00,
    "active": True,
    "cases_won": 147
}

# Single-line (for small dicts)
color = {"red": 255, "green": 128, "blue": 0}

# Using dict() constructor with keyword arguments
settings = dict(theme="dark", font_size=14, language="en")
print(settings)  # {'theme': 'dark', 'font_size': 14, 'language': 'en'}

# From a list of tuples
pairs = [("name", "Mike Ross"), ("title", "Associate"), ("rate", 375)]
attorney_2 = dict(pairs)
print(attorney_2)  # {'name': 'Mike Ross', 'title': 'Associate', 'rate': 375}
```

**Mike:** "Dictionaries use **curly braces** `{}` with **key: value** pairs separated by commas. Keys are usually strings, but they can be any **immutable** type."

---

### 3.2 Accessing Values

```python
attorney = {
    "name": "Harvey Specter",
    "title": "Senior Partner",
    "hourly_rate": 450.00,
    "cases_won": 147
}

# Method 1: Square bracket notation
print(attorney["name"])         # Harvey Specter
print(attorney["hourly_rate"])  # 450.0

# ❌ KeyError if key doesn't exist
# print(attorney["email"])     # KeyError: 'email'

# Method 2: .get() — safe access, returns None if not found
print(attorney.get("name"))      # Harvey Specter
print(attorney.get("email"))     # None (no error!)
print(attorney.get("email", "N/A"))  # N/A (custom default)
```

| Method | Key Exists | Key Missing |
|--------|-----------|-------------|
| `dict["key"]` | Returns value | 💥 `KeyError` |
| `dict.get("key")` | Returns value | Returns `None` |
| `dict.get("key", default)` | Returns value | Returns `default` |

**Mike:** "Use `[]` when you're **sure** the key exists. Use `.get()` when it **might not**."

---

### 3.3 Adding and Modifying Entries

**Mike:** "Dictionaries are **mutable** — like lists, unlike tuples."

```python
attorney = {"name": "Harvey Specter", "title": "Senior Partner"}

# Add a new key-value pair
attorney["hourly_rate"] = 450.00
attorney["email"] = "h.specter@psl.com"
print(attorney)
# {'name': 'Harvey Specter', 'title': 'Senior Partner', 
#  'hourly_rate': 450.0, 'email': 'h.specter@psl.com'}

# Modify an existing value
attorney["title"] = "Managing Partner"
print(attorney["title"])  # Managing Partner

# update() — merge another dictionary in
new_data = {"phone": "555-0100", "floor": 50, "title": "Name Partner"}
attorney.update(new_data)
print(attorney["title"])  # Name Partner (overwritten!)
print(attorney["phone"])  # 555-0100 (added)

# update() with keyword arguments
attorney.update(cases_won=150, wins_streak=True)
```

> 💡 **Key Point:** If you assign to a key that **exists**, it **overwrites**. If the key **doesn't exist**, it **creates** a new entry. Same syntax, different behavior.

---

### 3.4 Removing Entries

```python
attorney = {
    "name": "Harvey Specter",
    "title": "Senior Partner",
    "rate": 450.00,
    "email": "h.specter@psl.com",
    "temp_note": "Delete this"
}

# del — removes by key (KeyError if not found)
del attorney["temp_note"]
print(attorney)  # temp_note is gone

# pop() — removes AND returns the value
rate = attorney.pop("rate")
print(rate)     # 450.0
print(attorney) # 'rate' is gone

# pop() with default — no error if key missing
result = attorney.pop("fax", "No fax on file")
print(result)   # No fax on file (key didn't exist, no error)

# popitem() — removes and returns the LAST inserted pair
last_pair = attorney.popitem()
print(last_pair)  # ('email', 'h.specter@psl.com')

# clear() — removes everything
attorney.clear()
print(attorney)   # {}
```

| Method | Removes by | Returns | Error if missing? |
|--------|-----------|---------|-------------------|
| `del dict[key]` | Key | Nothing | Yes — `KeyError` |
| `dict.pop(key)` | Key | Value | Yes — `KeyError` |
| `dict.pop(key, default)` | Key | Value or default | No |
| `dict.popitem()` | Last pair | `(key, value)` tuple | Yes (if empty) |
| `dict.clear()` | Everything | Nothing | No |

---

### 3.5 Checking for Keys

```python
attorney = {"name": "Harvey", "title": "Partner", "rate": 450}

# Check if a key EXISTS
print("name" in attorney)       # True
print("email" in attorney)      # False
print("email" not in attorney)  # True

# NOTE: 'in' checks KEYS, not values!
print("Harvey" in attorney)     # False — "Harvey" is a VALUE, not a key
print(450 in attorney)           # False — 450 is a VALUE, not a key
```

---

### 3.6 Dictionary Views — keys(), values(), items()

**Mike:** "These three methods are your tools for inspecting and iterating over dictionaries."

```python
attorney = {"name": "Harvey", "title": "Partner", "rate": 450}

# All keys
print(attorney.keys())    # dict_keys(['name', 'title', 'rate'])

# All values
print(attorney.values())  # dict_values(['Harvey', 'Partner', 450])

# All key-value pairs (as tuples!)
print(attorney.items())   # dict_items([('name', 'Harvey'), ('title', 'Partner'), ('rate', 450)])
```

> 💡 **Connection to Lesson 4:** Notice that `items()` returns tuples! Each `(key, value)` pair is a tuple — which you already know how to unpack.

#### Iterating Over a Dictionary

```python
attorney = {"name": "Harvey Specter", "title": "Senior Partner", "rate": 450.00}

# Iterate over KEYS (default behavior)
for key in attorney:
    print(key)
# name
# title
# rate

# Iterate over VALUES
for value in attorney.values():
    print(value)
# Harvey Specter
# Senior Partner
# 450.0

# Iterate over KEY-VALUE PAIRS (most common!)
for key, value in attorney.items():
    print(f"{key}: {value}")
# name: Harvey Specter
# title: Senior Partner
# rate: 450.0
```

**Mike:** "The `for key, value in dict.items()` pattern is something you'll write hundreds of times in your career. Memorize it."

---

### 3.7 Nested Dictionaries

**Mike:** "Dictionaries can contain other dictionaries. This is how real-world data is structured."

```python
# A dictionary of dictionaries
firm = {
    "harvey": {
        "full_name": "Harvey Specter",
        "title": "Senior Partner",
        "rate": 450.00,
        "cases": ["Hessington Oil", "Ava Hessington", "Liberty Rail"]
    },
    "mike": {
        "full_name": "Mike Ross",
        "title": "Senior Associate",
        "rate": 375.00,
        "cases": ["Gillis Industries", "Lola Jensen"]
    },
    "louis": {
        "full_name": "Louis Litt",
        "title": "Named Partner",
        "rate": 425.00,
        "cases": ["Forstman", "Cahill Investigation"]
    }
}

# Access nested data — chain the keys
print(firm["harvey"]["full_name"])    # Harvey Specter
print(firm["mike"]["rate"])          # 375.0
print(firm["louis"]["cases"][0])     # Forstman (list inside dict!)

# Modify nested data
firm["harvey"]["rate"] = 500.00
firm["mike"]["cases"].append("Sidwell Investment")

# Safely access nested data with .get()
email = firm["harvey"].get("email", "No email on file")
print(email)  # No email on file

# Iterate over the nested structure
for attorney_id, info in firm.items():
    print(f"\n{info['full_name']} ({info['title']})")
    print(f"  Rate: ${info['rate']:.2f}/hr")
    print(f"  Cases: {', '.join(info['cases'])}")
```

---

### 3.8 Dictionary Comprehension (Preview)

**Mike:** "Like list comprehensions (Level 2), you can build dictionaries in one line:"

```python
# Create a dictionary from two lists
names = ["Harvey", "Mike", "Louis"]
rates = [450, 375, 425]

rate_card = {name: rate for name, rate in zip(names, rates)}
print(rate_card)  # {'Harvey': 450, 'Mike': 375, 'Louis': 425}

# Transform values
inflated = {name: rate * 1.10 for name, rate in rate_card.items()}
print(inflated)  # {'Harvey': 495.0, 'Mike': 412.5, 'Louis': 467.5}

# Filter entries
expensive = {name: rate for name, rate in rate_card.items() if rate > 400}
print(expensive)  # {'Harvey': 450, 'Louis': 425}
```

**Mike:** "We'll go much deeper on comprehensions in Level 2. For now, just know they exist."

---

### 3.9 Useful Dictionary Methods — Complete Reference

```python
d = {"a": 1, "b": 2, "c": 3}

# setdefault() — get value if key exists, SET and return default if not
val = d.setdefault("d", 4)   # Key 'd' doesn't exist → creates it with value 4
print(val)    # 4
print(d)      # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

val = d.setdefault("a", 99)  # Key 'a' exists → returns existing value, doesn't overwrite
print(val)    # 1 (not 99!)

# fromkeys() — create a dictionary with the same value for all keys
template = dict.fromkeys(["name", "email", "phone"], "N/A")
print(template)  # {'name': 'N/A', 'email': 'N/A', 'phone': 'N/A'}

# copy() — shallow copy
original = {"a": 1, "b": [2, 3]}
copied = original.copy()
copied["a"] = 99
print(original["a"])  # 1 (safe — simple values are independent)
# BUT: nested mutable objects are NOT deep-copied!

# len() — number of key-value pairs
print(len(d))  # 4
```

---

### 3.10 Valid Key Types

**Mike:** "Not everything can be a dictionary key."

```python
# ✅ Strings (most common)
d = {"name": "Harvey"}

# ✅ Numbers
d = {1: "first", 2: "second", 3.14: "pi"}

# ✅ Tuples (immutable → hashable)
d = {(40.71, -74.00): "New York", (34.05, -118.24): "Los Angeles"}

# ✅ Booleans (but be careful — True == 1 and False == 0)
d = {True: "yes", False: "no"}

# ❌ Lists (mutable → NOT hashable)
# d = {[1, 2]: "nope"}  # TypeError: unhashable type: 'list'

# ❌ Dictionaries (mutable → NOT hashable)
# d = {{"a": 1}: "nope"}  # TypeError: unhashable type: 'dict'
```

> 💡 **Connection to Lesson 4:** This is why tuples can be dict keys but lists cannot — immutability enables **hashability**, which dictionaries require for fast lookups.

---

### 3.11 Merging Dictionaries (Python 3.9+)

```python
defaults = {"theme": "light", "font_size": 12, "language": "en"}
user_prefs = {"theme": "dark", "font_size": 16}

# Method 1: update() — modifies in place
merged = defaults.copy()
merged.update(user_prefs)
print(merged)  # {'theme': 'dark', 'font_size': 16, 'language': 'en'}

# Method 2: {**d1, **d2} — unpacking (Python 3.5+)
merged = {**defaults, **user_prefs}
print(merged)  # {'theme': 'dark', 'font_size': 16, 'language': 'en'}

# Method 3: | operator (Python 3.9+)
merged = defaults | user_prefs
print(merged)  # {'theme': 'dark', 'font_size': 16, 'language': 'en'}

# In all cases, later values OVERRIDE earlier ones for duplicate keys
```

---

### 3.12 Solving Donna's Problem — The Digital Rolodex

**Mike:** "Let's digitize that Rolodex."

```python
# ============================================
# Pearson Specter Litt — Digital Client Rolodex
# ============================================

# Client database — dictionary of dictionaries
clients = {
    "wayne_enterprises": {
        "company": "Wayne Enterprises",
        "contact": "Bruce Wayne",
        "phone": "555-0101",
        "email": "b.wayne@wayne.com",
        "billing_tier": "Platinum",
        "annual_revenue": 2500000.00,
        "attorney": "Harvey Specter",
        "active": True
    },
    "stark_industries": {
        "company": "Stark Industries",
        "contact": "Tony Stark",
        "phone": "555-0202",
        "email": "tony@stark.com",
        "billing_tier": "Platinum",
        "annual_revenue": 1800000.00,
        "attorney": "Harvey Specter",
        "active": True
    },
    "oscorp": {
        "company": "Oscorp",
        "contact": "Norman Osborn",
        "phone": "555-0303",
        "email": "n.osborn@oscorp.com",
        "billing_tier": "Gold",
        "annual_revenue": 950000.00,
        "attorney": "Louis Litt",
        "active": True
    },
    "luthor_corp": {
        "company": "Luthor Corp",
        "contact": "Lex Luthor",
        "phone": "555-0404",
        "email": "lex@luthor.com",
        "billing_tier": "Silver",
        "annual_revenue": 400000.00,
        "attorney": "Mike Ross",
        "active": False
    }
}

# === LOOKUP: Find a client by key ===
print("=" * 60)
print(f"{'CLIENT LOOKUP':^60}")
print("=" * 60)

lookup = "wayne_enterprises"
if lookup in clients:
    c = clients[lookup]
    print(f"\n  Company:  {c['company']}")
    print(f"  Contact:  {c['contact']}")
    print(f"  Phone:    {c['phone']}")
    print(f"  Email:    {c['email']}")
    print(f"  Tier:     {c['billing_tier']}")
    print(f"  Revenue:  ${c['annual_revenue']:,.2f}")
    print(f"  Attorney: {c['attorney']}")
    print(f"  Status:   {'Active ✅' if c['active'] else 'Inactive ❌'}")

# === REPORT: All clients, formatted ===
print("\n" + "=" * 60)
print(f"{'FULL CLIENT REPORT':^60}")
print("=" * 60)

print(f"\n{'Company':<22} {'Tier':<10} {'Revenue':>14} {'Attorney':<18} {'Status':>6}")
print("-" * 75)

total_revenue = 0
active_count = 0

for client_id, info in clients.items():
    status = "✅" if info["active"] else "❌"
    print(f"{info['company']:<22} {info['billing_tier']:<10} "
          f"${info['annual_revenue']:>13,.2f} {info['attorney']:<18} {status:>4}")
    total_revenue += info["annual_revenue"]
    if info["active"]:
        active_count += 1

print("-" * 75)
print(f"{'TOTAL':<22} {'':10} ${total_revenue:>13,.2f}")
print(f"\nActive Clients: {active_count}/{len(clients)}")

# === SEARCH: Find clients by attorney ===
print("\n" + "=" * 60)
search_attorney = "Harvey Specter"
print(f"Clients assigned to {search_attorney}:")
print("-" * 40)

for client_id, info in clients.items():
    if info["attorney"] == search_attorney:
        print(f"  • {info['company']} ({info['billing_tier']})")

# === ANALYTICS: Revenue by tier ===
print("\n" + "=" * 60)
print("Revenue by Billing Tier:")
print("-" * 40)

tier_totals = {}
for info in clients.values():
    tier = info["billing_tier"]
    tier_totals[tier] = tier_totals.get(tier, 0) + info["annual_revenue"]

for tier, total in sorted(tier_totals.items(), key=lambda x: x[1], reverse=True):
    print(f"  {tier:<12} ${total:>14,.2f}")

print("=" * 60)
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

*Louis has been sitting silently in the corner, taking notes. He stands up.*

**Louis:** "Dictionaries have some of the most *dangerous* gotchas in Python. Pay attention."

---

### Pitfall 1: KeyError — The Silent Killer

**Louis:** "Access a key that doesn't exist with brackets? Your program dies."

```python
data = {"name": "Harvey"}

# CRASH
# print(data["email"])  # KeyError: 'email'

# SAFE — use .get()
print(data.get("email"))             # None
print(data.get("email", "Unknown"))  # Unknown

# SAFE — check first
if "email" in data:
    print(data["email"])
else:
    print("No email on file")
```

---

### Pitfall 2: Keys Must Be Unique — Last One Wins

**Louis:** "What happens if you use the same key twice?"

```python
data = {
    "name": "Harvey",
    "title": "Senior Partner",
    "name": "Mike"          # Duplicate key!
}

print(data)  # {'name': 'Mike', 'title': 'Senior Partner'}
# "Harvey" is GONE — silently overwritten! No warning!
```

**Louis:** "Python doesn't warn you. The last value for a duplicate key **silently wins**."

---

### Pitfall 3: The Mutable Default Trap (Preview)

**Louis:** "This is a Level 2 topic, but know the danger now:"

```python
# DANGEROUS — shared mutable default
def add_client(name, clients={}):
    clients[name] = True
    return clients

result1 = add_client("Harvey")
print(result1)   # {'Harvey': True}

result2 = add_client("Mike")
print(result2)   # {'Harvey': True, 'Mike': True} — WHAT?!
# The same dictionary is reused across calls!

# FIX — use None as default
def add_client_safe(name, clients=None):
    if clients is None:
        clients = {}
    clients[name] = True
    return clients
```

---

### Pitfall 4: Dictionary Ordering

**Louis:** "Fun fact — dictionaries maintain **insertion order** as of Python 3.7. Before that, they were unordered."

```python
data = {"c": 3, "a": 1, "b": 2}
print(list(data.keys()))   # ['c', 'a', 'b'] — insertion order preserved

# But dictionaries are NOT sorted — they remember insertion order
# If you need sorted keys:
for key in sorted(data):
    print(f"{key}: {data[key]}")
# a: 1
# b: 2
# c: 3
```

---

### Pitfall 5: Shallow Copy Catches

**Louis:** "Just like lists, `.copy()` is shallow:"

```python
original = {
    "name": "Harvey",
    "cases": ["Case A", "Case B"]   # Mutable list as a value
}

shallow = original.copy()
shallow["name"] = "Mike"       # Safe — strings are immutable
shallow["cases"].append("Case C")  # DANGER — modifies the ORIGINAL list!

print(original["name"])    # Harvey (safe)
print(original["cases"])   # ['Case A', 'Case B', 'Case C'] ← CHANGED!
```

```python
# FIX — use deepcopy for nested structures
import copy
deep = copy.deepcopy(original)
deep["cases"].append("Case D")
print(original["cases"])   # ['Case A', 'Case B', 'Case C'] ← SAFE!
```

---

### Pitfall 6: Mutating Dictionary While Iterating

**Louis:** "Same rule as lists — **never modify a dictionary while iterating over it**:"

```python
scores = {"Harvey": 95, "Mike": 88, "Louis": 72, "Harold": 55}

# ❌ RuntimeError: dictionary changed size during iteration
# for name in scores:
#     if scores[name] < 75:
#         del scores[name]

# ✅ Iterate over a COPY of the keys
for name in list(scores.keys()):
    if scores[name] < 75:
        del scores[name]

print(scores)  # {'Harvey': 95, 'Mike': 88}

# ✅ Or build a new dictionary
passing = {name: score for name, score in scores.items() if score >= 75}
```

---

### Pitfall 7: `True == 1` and `False == 0` Key Collision

**Louis:** "This is an *obscure* but real trap:"

```python
d = {True: "yes", 1: "one"}
print(d)        # {True: 'one'} — only ONE entry!
print(len(d))   # 1

# Because True == 1 in Python, they hash to the same key
# The value 'one' overwrites 'yes'
```

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "Perfect analogy incoming."

---

### 📇 Analogy: Dictionaries Are Like a Rolodex

**Donna:** "You literally watched me use one this morning."

```
📇 Rolodex (Dictionary):

  Tab: "Wayne Enterprises"     →  Card: Bruce Wayne, 555-0101, Platinum
  Tab: "Stark Industries"     →  Card: Tony Stark, 555-0202, Platinum
  Tab: "Oscorp"               →  Card: Norman Osborn, 555-0303, Gold
  Tab: "Luthor Corp"          →  Card: Lex Luthor, 555-0404, Silver
```

- **Key** = The **tab label** you flip to (must be unique, can't have two identical tabs)
- **Value** = The **information** on the card behind that tab
- **Lookup** (`rolodex["Wayne Enterprises"]`) = Flipping to the "W" tab and pulling the card
- **Adding** (`rolodex["New Client"] = info`) = Inserting a new card with a new tab
- **Deleting** (`del rolodex["Luthor Corp"]`) = Pulling a card out and shredding it

**Donna:** "Now compare all four data structures we've learned:"

| Structure | Real-World Analogy | Access By | Mutable? | Ordered? |
|-----------|-------------------|-----------|----------|----------|
| **String** | Charm bracelet | Position `[i]` | ❌ | ✅ |
| **List** | Filing cabinet | Position `[i]` | ✅ | ✅ |
| **Tuple** | Signed contract | Position `[i]` | ❌ | ✅ |
| **Dictionary** | Rolodex | Key `["name"]` | ✅ | ✅* |

*_Insertion-ordered since Python 3.7_

**Donna:** "Lists are numbered slots. Dictionaries are labeled slots. That's the whole difference."

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "Dictionaries are the most practical data structure. Let's put them to work."

---

### 🟢 Beginner Exercises

**Exercise 1: Dictionary Basics**
```
Create a dictionary called 'case' with:
  - case_id: "PSL-2026-001"
  - client: "Wayne Enterprises"
  - attorney: "Harvey Specter"
  - status: "Active"
  - amount: 2500000.00

a) Print the entire dictionary
b) Print just the client name
c) Print all keys
d) Print all values
e) Check if "status" is a key
f) Check if "judge" is a key
g) Use .get() to safely access "judge" with default "Not assigned"
```

**Exercise 2: Modifying a Dictionary**
```
Start with: profile = {"name": "Mike Ross", "title": "Associate"}

a) Add "email": "m.ross@psl.com"
b) Add "rate": 375.00
c) Change "title" to "Senior Associate"
d) Remove "email" using pop() and print the removed value
e) Print the final dictionary
```

**Exercise 3: Word Counter**
```
Given: sentence = "the quick brown fox jumps over the lazy dog the fox"

Create a dictionary that counts how many times each word appears.
Expected output:
  {'the': 3, 'quick': 1, 'brown': 1, 'fox': 2, 'jumps': 1, 
   'over': 1, 'lazy': 1, 'dog': 1}

Hint: Use split() to get words, then loop through them.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Contact Book**
```
Build a contact book dictionary where:
  - Keys are names (strings)
  - Values are dictionaries with: phone, email, department

Add at least 4 contacts. Then:
a) Print a formatted contact card for a specific person
b) Update someone's phone number
c) Add a new contact
d) Delete a contact
e) List all contacts in a specific department
f) Count contacts per department
```

**Exercise 5: Grade Book**
```
Create a grade book dictionary:

grades = {
    "Mike": [85, 92, 78, 95],
    "Rachel": [88, 91, 85, 90],
    "Harold": [72, 68, 75, 70],
    "Katrina": [90, 93, 97, 95]
}

a) Calculate and print each student's average
b) Find the student with the highest average
c) Find the student with the lowest single grade
d) Add a new grade of 88 to Mike's list
e) Add a new student "Kyle" with grades [80, 85, 82, 79]
f) Remove Harold from the grade book
g) Print a formatted report card for all students
```

**Exercise 6: Inventory System**
```
Create a product inventory:

inventory = {
    "laptop": {"price": 1200.00, "quantity": 15, "category": "electronics"},
    "keyboard": {"price": 75.00, "quantity": 50, "category": "electronics"},
    "desk_lamp": {"price": 45.00, "quantity": 30, "category": "furniture"},
    "notebook": {"price": 12.00, "quantity": 100, "category": "supplies"}
}

a) Calculate total inventory value (price × quantity for each item)
b) Find the most expensive item
c) Find items with quantity below 20 (low stock)
d) Calculate total value per category
e) Apply a 10% price increase to all electronics
f) Add a new product
```

---

### 🔴 Advanced Exercises

**Exercise 7: Two-Way Translation Dictionary**
```
Create a bidirectional English-Spanish dictionary:

Given:
  translations = {"hello": "hola", "goodbye": "adiós", "cat": "gato", 
                   "dog": "perro", "house": "casa"}

a) Create a reverse dictionary (Spanish → English)
b) Write a function that can translate in either direction
c) Handle the case where a word is not found
d) Add the ability to add new translations (updating both directions)
```

**Exercise 8: Log Analyzer**
```
Parse these log entries into structured dictionaries:

logs = [
    "[2026-03-13 09:15:22] INFO: User login | user=harvey | ip=192.168.1.10",
    "[2026-03-13 09:16:45] ERROR: Connection timeout | server=db-primary | retry=3",
    "[2026-03-13 09:17:01] INFO: File uploaded | user=mike | size=2.4MB",
    "[2026-03-13 09:18:30] WARNING: Disk space low | server=web-01 | usage=89%",
    "[2026-03-13 09:19:15] ERROR: Auth failed | user=unknown | ip=10.0.0.5",
]

a) Parse each log line into a dictionary with: timestamp, level, message, and metadata
b) Count logs by level (how many INFO, ERROR, WARNING)
c) Find all ERROR logs
d) Group logs by level into a dictionary of lists
e) Find unique users mentioned in the logs
```

**Exercise 9: Mini Database**
```
Build a simple in-memory key-value database with these operations:

Commands:
  SET key value     — stores a value
  GET key           — retrieves a value  
  DELETE key        — removes a key
  EXISTS key        — checks if key exists
  KEYS              — lists all keys
  COUNT             — counts all entries
  CLEAR             — removes everything

Create a function for each command. Use a dictionary as the underlying storage.
Test with at least 10 operations and print the result of each.
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "Dictionary bugs. These are sneaky."

### Buggy Code 🐛

```python
# Bug Hunt: Client Database
# Find and fix ALL the bugs!

# Bug 1: Create a client record
client = {
    name: "Wayne Enterprises",         # Something missing...
    "contact": "Bruce Wayne",
    "revenue": 2500000
}

# Bug 2: Access a key safely
email = client["email"]               # Will this work?
print(f"Email: {email}")

# Bug 3: Add and update
client.append({"phone": "555-0101"})  # Wrong method?

# Bug 4: Iterate and modify
rates = {"Harvey": 450, "Mike": 375, "Louis": 425, "Harold": 280}
for name in rates:
    if rates[name] < 400:
        del rates[name]               # Danger!

# Bug 5: Merge dictionaries
base = {"a": 1, "b": 2}
extra = {"c": 3, "d": 4}
combined = base + extra               # Can you add dicts?

# Bug 6: Check for a value (not key!)
data = {"name": "Harvey", "title": "Partner"}
if "Harvey" in data:                  # Is this checking what you think?
    print("Found Harvey")

# Bug 7: Count items
scores = {"math": 95, "english": 88}
print(f"Total entries: {scores.count()}")  # Wrong method?
```

---

### Fixed Code ✅

```python
# Fixed: Client Database

# Fix 1: Keys must be STRINGS (in quotes) — 'name' without quotes is a variable
client = {
    "name": "Wayne Enterprises",       # Added quotes ✅
    "contact": "Bruce Wayne",
    "revenue": 2500000
}

# Fix 2: Use .get() for safe access — "email" key doesn't exist
email = client.get("email", "No email on file")  # ✅
print(f"Email: {email}")

# Fix 3: Dicts use subscript or update(), NOT append()
client["phone"] = "555-0101"           # ✅ Direct assignment
# OR: client.update({"phone": "555-0101"})

# Fix 4: Never modify a dict while iterating — iterate over a copy
rates = {"Harvey": 450, "Mike": 375, "Louis": 425, "Harold": 280}
for name in list(rates.keys()):        # ✅ Iterate over a snapshot
    if rates[name] < 400:
        del rates[name]
print(rates)  # {'Harvey': 450, 'Louis': 425}

# Fix 5: Can't use + with dicts — use update() or | (Python 3.9+)
base = {"a": 1, "b": 2}
extra = {"c": 3, "d": 4}
combined = {**base, **extra}           # ✅ Dict unpacking
print(combined)  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# Fix 6: 'in' checks KEYS, not values — check .values() instead
data = {"name": "Harvey", "title": "Partner"}
if "Harvey" in data.values():          # ✅ Check values explicitly
    print("Found Harvey")

# Fix 7: Dicts don't have count() — use len()
scores = {"math": 95, "english": 88}
print(f"Total entries: {len(scores)}")  # ✅
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | `name` without quotes is a variable, not a string key | Keys must be valid types |
| 2 | Accessing non-existent key with `[]` → KeyError | Use `.get()` for safety |
| 3 | `append()` is a list method, not dict | Use `dict[key] = value` |
| 4 | Deleting from dict during iteration → RuntimeError | Iterate over `list(dict.keys())` |
| 5 | `+` doesn't work for dicts | Use `{**d1, **d2}` or `.update()` |
| 6 | `in` checks keys, not values | Use `in dict.values()` |
| 7 | Dicts have `len()`, not `count()` | `len(dict)` for size |

---

## 8. 🌍 Real-World Application

**Harvey:** "Dictionaries power the modern internet. That's not an exaggeration."

### Where Dictionaries Are Used in Production Software:

| Domain | Dictionary Use Case |
|--------|-------------------|
| **Web APIs** | JSON data is Python dictionaries — every API request/response |
| **Databases** | ORM objects, query result rows, schema definitions |
| **Configuration** | YAML/TOML/JSON config files → dictionaries |
| **Caching** | In-memory key-value stores (Redis interface, memoization) |
| **Web Frameworks** | Request headers, cookies, session data, URL params |
| **Data Science** | Feature dictionaries, label maps, hyperparameters |
| **DevOps** | Environment variables, Docker labels, Terraform state |

### Real Production Example: API Response Handler

```python
# Every web API returns JSON — which Python loads as dictionaries

# Simulated API response (this is what json.loads() gives you)
api_response = {
    "status": "success",
    "data": {
        "clients": [
            {"id": 1, "name": "Wayne Enterprises", "revenue": 2500000},
            {"id": 2, "name": "Stark Industries", "revenue": 1800000},
            {"id": 3, "name": "Oscorp", "revenue": 950000}
        ],
        "total": 3,
        "page": 1
    },
    "meta": {
        "request_id": "abc-123",
        "timestamp": "2026-03-13T14:22:05Z"
    }
}

# Navigate the response
if api_response["status"] == "success":
    clients = api_response["data"]["clients"]
    for client in clients:
        print(f"  {client['name']}: ${client['revenue']:,.2f}")
    
    print(f"\nTotal clients: {api_response['data']['total']}")
    print(f"Request ID: {api_response['meta']['request_id']}")
```

### Real Production Example: Configuration Manager

```python
# Application configuration — dictionaries all the way down
config = {
    "app": {
        "name": "PSL Case Manager",
        "version": "2.1.0",
        "debug": False
    },
    "database": {
        "host": "db.pearsonspecter.com",
        "port": 5432,
        "name": "cases_prod"
    },
    "email": {
        "smtp_host": "mail.psl.com",
        "from": "noreply@psl.com",
        "timeout": 30
    }
}

# Access nested config with safe defaults
db_host = config.get("database", {}).get("host", "localhost")
debug_mode = config.get("app", {}).get("debug", True)

print(f"Connecting to {db_host}...")
print(f"Debug mode: {'ON' if debug_mode else 'OFF'}")
```

**Harvey:** "JSON *is* dictionaries. APIs *are* dictionaries. Config files *are* dictionaries. If you understand dictionaries, you understand data on the internet."

---

## 9. ✅ Knowledge Check

**Harvey:** "Final round."

---

**Q1.** What is the difference between `dict["key"]` and `dict.get("key")`?

**Q2.** What happens if you create a dictionary with duplicate keys? `{"a": 1, "b": 2, "a": 3}`?

**Q3.** Why can tuples be dictionary keys but lists cannot?

**Q4.** How do you iterate over both keys AND values of a dictionary at the same time?

**Q5.** What is the output?
```python
d = {"a": 1, "b": 2}
d["c"] = 3
d["a"] = 10
print(d)
```

**Q6.** How do you safely get a value from a nested dictionary: `data["user"]["address"]["city"]` — if any key might not exist?

**Q7.** What does `in` check for in a dictionary — keys or values?

**Q8.** What is the difference between `dict.pop("key")` and `del dict["key"]`?

**Q9.** Name three ways to merge two dictionaries.

**Q10.** Given `counts = {}`, write code that increments `counts["apple"]` — creating the key with value `1` if it doesn't exist, or adding `1` if it does.

---

### 📋 Answer Key

> **Q1.** `dict["key"]` raises `KeyError` if key is missing. `dict.get("key")` returns `None` (or a custom default) if missing — no crash.

> **Q2.** `{'a': 3, 'b': 2}` — the last value for duplicate key `'a'` silently wins.

> **Q3.** Dict keys must be **hashable** (immutable). Tuples are immutable → hashable. Lists are mutable → not hashable.

> **Q4.** `for key, value in dict.items():`

> **Q5.** `{'a': 10, 'b': 2, 'c': 3}` — `'c'` is added, `'a'` is overwritten.

> **Q6.** Chain `.get()`: `data.get("user", {}).get("address", {}).get("city", "Unknown")` — each `.get()` returns an empty dict if the key is missing, so the chain doesn't break.

> **Q7.** **Keys** only. To check values, use `value in dict.values()`.

> **Q8.** `pop()` removes AND returns the value. `del` removes without returning. Both raise errors if key is missing (unless `pop` has a default).

> **Q9.** (1) `d1.update(d2)`, (2) `{**d1, **d2}`, (3) `d1 | d2` (Python 3.9+).

> **Q10.** `counts["apple"] = counts.get("apple", 0) + 1` — `.get()` returns `0` if key doesn't exist, then adds 1.

---

## 🎬 End of Lesson Scene

*It's 2:45 PM. Donna's Rolodex data is fully digitized. The system can look up any client by name in milliseconds. Jessica walks over to Donna's desk with her tablet.*

**Jessica:** "This is impressive. I just searched 'Wayne' and got the full profile in a second."

**Donna:** *(glancing at you)* "The new associate built it."

*Jessica looks at you.*

**Jessica:** "Keep going. This firm needs more automation."

*She walks toward the partner meeting. Harvey catches your eye from his office and gives a single nod.*

**Mike:** *(quietly)* "That's the highest compliment Jessica gives. A nod from Harvey and a 'keep going' from Jessica? You're doing well."

**Louis:** *(muttering)* "I could have built that Rolodex in half the time."

**Donna:** "No, you couldn't have, Louis."

*Day five. You can store data, protect it, and now look it up by name. Tomorrow — you'll learn to listen. User input.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | Dictionary creation — `{}`, `dict()`, from tuples | ✅ |
| 2 | Accessing — `[]` vs `.get()` | ✅ |
| 3 | Adding and modifying — `dict[key] = value`, `.update()` | ✅ |
| 4 | Removing — `del`, `.pop()`, `.popitem()`, `.clear()` | ✅ |
| 5 | Membership — `in` checks keys, not values | ✅ |
| 6 | Views — `.keys()`, `.values()`, `.items()` | ✅ |
| 7 | Iteration — `for key, value in dict.items()` | ✅ |
| 8 | Nested dictionaries — chained access | ✅ |
| 9 | Dictionary comprehension (preview) | ✅ |
| 10 | Merging — `update()`, `{**d1, **d2}`, `\|` | ✅ |
| 11 | Valid key types — hashable requirement | ✅ |
| 12 | `.setdefault()`, `.fromkeys()`, `.copy()` | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| Tuples as dict keys (Lesson 4) | Immutability enables hashability |
| `in` operator (Lessons 2–4) | Now checks dict keys |
| List mutability (Lesson 3) | Dicts are also mutable |
| Tuple immutability (Lesson 4) | Values stored with structural safety |
| f-string formatting (Lesson 2) | Formatted dictionary output |
| `split()` (Lesson 2) | Parsing strings into dict keys/values |
| `.copy()` alias trap (Lesson 3) | Same trap with dict shallow copy |

---

> **Next Lesson →** Level 1, Lesson 6: *Get User Input With Python's `input()`*  
> *Harvey's contract generator needs to accept live input from associates — it's time to make your programs interactive…*
