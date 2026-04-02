# Phase 1 · Lesson 2 — Strings & Output
### *"The Case of the Missing Contract Clause"*

> **O'Reilly Skill:** Construct dynamic text using f-strings, manipulate text with string methods, and display formatted results with `print()`.
> **DevOps connection:** Log messages, command strings, config values, API endpoint URLs — all strings.
> **10-min sprint:** Build one dynamic message using an f-string with a number formatted as currency or percentage.

---

## Why This Matters (Harvey's take)

Most real-world data is text. Names, addresses, log entries, API responses, config files — all strings. Python's string handling is the most powerful of any mainstream language. Master it and you'll never produce a badly formatted output again.

---

## The Core Concept (Mike's breakdown)

### Creating Strings

```python
firm = "Pearson Specter Litt"       # double quotes
client = 'Wayne Enterprises'        # single quotes — identical

# Multi-line strings
contract = """
This agreement is entered into on March 13, 2026
between Buyer and Seller.
"""

# Triple quotes preserve line breaks exactly as written
```

### Indexing & Slicing (same rules as lists)

```python
firm = "SPECTER"
#       S P E C T E R
#       0 1 2 3 4 5 6   ← positive index
#      -7-6-5-4-3-2-1   ← negative index

print(firm[0])       # S
print(firm[-1])      # R (last character)
print(firm[1:4])     # PEC (stop is exclusive)
print(firm[::-1])    # RETCEPS (reversed)
```

### Strings Are IMMUTABLE

```python
name = "Harvey"
# name[0] = "h"   # ❌ TypeError — can't change in-place

# Instead, create a new string:
name = "h" + name[1:]   # "harvey"
```

### f-Strings — your most-used tool

```python
client = "Wayne Enterprises"
amount = 2500000.50
date = "March 13, 2026"

# Basic f-string
print(f"Contract for {client}")

# Math inside braces
hours, rate = 85, 375.00
print(f"Total: ${hours * rate}")

# Number formatting
print(f"Revenue: ${amount:,.2f}")       # $2,500,000.50
print(f"Tax Rate: {0.085:.2%}")         # 8.50%
print(f"{'Name':<15} {'Amount':>10}")   # aligned columns
```

### Essential String Methods

```python
name = "harvey specter"

# Case
print(name.upper())       # HARVEY SPECTER
print(name.title())       # Harvey Specter
print(name.lower())       # harvey specter

# Search
text = "This is binding"
print("binding" in text)       # True
print(text.find("binding"))    # 8 (index, -1 if not found)
print(text.count("i"))         # 3

# Clean
messy = "   Harvey Specter   \n"
print(messy.strip())      # "Harvey Specter"

# Replace & Split
clause = "The Buyer shall pay the Seller"
print(clause.replace("Buyer", "Acquiring Party"))

csv = "Harvey,Mike,Louis"
names = csv.split(",")         # ['Harvey', 'Mike', 'Louis']
print(" & ".join(names))       # "Harvey & Mike & Louis"
```

### `print()` — the full picture

```python
# Multiple arguments — separated by space
print("Harvey", "Specter", "Attorney")    # Harvey Specter Attorney

# Custom separator
print("2026", "03", "13", sep="-")        # 2026-03-13

# Custom end (default is newline)
print("Loading", end="")
print("...done")                           # Loading...done

# Escape characters
print("Name:\tHarvey")    # tab
print("Line1\nLine2")     # newline
print(r"C:\Users\new")    # raw string — backslash is literal
```

---

## DevOps Example — Log Parser

```python
log = "[2026-03-13 14:22:05] ERROR: ConnectionTimeout | Server: api-node-3 | Duration: 30.5s"

timestamp = log[1:20]
level = log[log.find("] ")+2 : log.find(":")]
message = log[log.find(": ")+2 : log.find(" |")]

print(f"Time: {timestamp}")
print(f"Level: {level}")
print(f"Message: {message}")
# Time: 2026-03-13 14:22:05
# Level: ERROR
# Message: ConnectionTimeout
```

---

## Gotchas (Louis's warnings)

**1. String methods don't modify in-place — capture the return:**
```python
name = "harvey"
name.upper()           # ❌ result is LOST
name = name.upper()    # ✅
```

**2. `find()` vs `index()` — choose carefully:**
```python
text.find("missing")   # returns -1 (safe)
text.index("missing")  # raises ValueError (strict)
```

**3. Case-sensitive comparisons:**
```python
"Harvey" == "harvey"   # False!
# Always normalize: "Harvey".lower() == "harvey".lower()
```

**4. Hidden whitespace kills comparisons:**
```python
name = "Harvey\n"
name == "Harvey"       # False! Strip it: name.strip()
```

**5. `+` concatenation in loops is slow — use `join()`:**
```python
# Bad (O(n²)):
result = ""
for s in big_list:
    result = result + s

# Good (O(n)):
result = "".join(big_list)
```

---

## Mental Model (Donna's analogy)

Strings are **charm bracelets** — each character is a charm at a numbered position. They're **welded on** (immutable), so you can't swap one charm. You make a whole new bracelet with the change.

f-strings are **form letters** — `{variable}` is the blank to fill in. Python reads the template, fills in the blanks, and produces a perfect letter.

`print()` is the **printer** — takes what you give it and puts it on screen. `sep` is the space between items; `end` is what happens after the last one.

---

## Practice (pick one, 10 minutes)

**Beginner:** Given `raw_email = "  JESSICA.PEARSON@PSL.COM  "`, produce `"jessica.pearson@psl.com"` in one chained expression.

**Intermediate:** Build a formatted receipt. Three items with names and prices. Print aligned columns, subtotal, 8.5% tax, and total. Use f-strings for all formatting.

**DevOps-specific:** Parse this URL: `"https://api.example.com/v2/clients/4287?fields=name,revenue"`. Extract: protocol, domain, version, resource, client ID, and query params.

---

## Quick Debug — find all 7 bugs

```python
raw_name = "   JESSICA   PEARSON   "
clean_name = raw_name.strip           # Bug 1
proper_name = clean_name.Title()      # Bug 2
email = proper_name.lower.replace(" ", ".") + "@psl.com"  # Bug 3
first_initial = proper_name[0]
last_initial = proper_name[proper_name.find(" ")]  # Bug 4
separator = "-" * "40"                # Bug 5
```

---

## Knowledge Check

1. What does `"Hello"[1:4]` return? Why doesn't it include index 4?
2. Why can't you do `name[0] = "h"` in Python?
3. Write an f-string that formats `1234567.891` as `"$1,234,567.89"`.
4. What does `"  hello world  ".strip().split()` return?
5. Why is `join()` better than `+` in a loop?

**Answers:** (1) `"ell"` — stop is exclusive. (2) Strings are immutable. (3) `f"${1234567.891:,.2f}"`. (4) `['hello', 'world']`. (5) `join()` is O(n); `+` is O(n²) — creates a new string each iteration.

---

> **Next:** Lesson 3 — Lists → Harvey's managing 200 clients. He needs to store, sort, and filter them all.
