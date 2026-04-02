# Phase 1 · Lesson 7 — If Statements
### *"The Decision Engine"*

> **O'Reilly Skill:** Control program flow with if statements.
> **DevOps connection:** "If CPU > 90%, scale up. Elif < 30%, scale down." Decision logic is in every script you'll ever write.
> **10-min sprint:** Write a script that reads a CPU percentage and prints: "Critical" (>90), "High" (>75), "Normal" (>50), or "Idle".

---

## Why This Matters (Harvey's take)

Right now your code runs top-to-bottom like a train on tracks. `if` statements give it a steering wheel. Every system makes decisions: Is this client profitable? Is this server healthy? Has the deadline passed? Without conditionals, code can't adapt to any situation.

---

## The Core Concept (Mike's breakdown)

### Comparison Operators — How to Ask Questions

```python
a, b = 10, 20
print(a == b)    # False — equal to?
print(a != b)    # True  — not equal to?
print(a < b)     # True  — less than?
print(a > b)     # False — greater than?
print(a <= b)    # True  — less than or equal?
print(a >= b)    # False — greater than or equal?
```

> ⚠️ `=` is assignment. `==` is comparison. Mixing them up is the most common bug.

### The `if` Statement

```python
cpu = 87.3

if cpu > 90:
    print("CRITICAL — scale up now")
    print("Paging on-call engineer")

print("This always runs")    # NOT indented → always executes
```

**Rules:**
1. Condition followed by a **colon** `:`
2. Body is **indented** (4 spaces — Python standard)
3. First un-indented line is outside the `if`

### `if-else` — Two Branches

```python
status = "stopped"

if status == "running":
    print("✅ Server healthy")
else:
    print("🔴 Server is down!")
```

### `if-elif-else` — Multiple Branches

```python
cpu = 67.0

if cpu >= 90:
    action = "CRITICAL — scale up"
elif cpu >= 75:
    action = "HIGH — watch closely"
elif cpu >= 50:
    action = "NORMAL"
else:
    action = "IDLE — consider scaling down"

print(f"CPU {cpu}%: {action}")
```

**Only ONE block executes** — the first match wins. Order matters enormously.

### Nested `if`

```python
service = "api"
requests_per_sec = 1200

if service == "api":
    if requests_per_sec > 1000:
        print("API overloaded — trigger autoscale")
    else:
        print("API load normal")
elif service == "db":
    print("Database check needed")
```

### Conditional Expression (Ternary) — one-liner

```python
cpu = 78.5
status = "High" if cpu > 75 else "Normal"
print(status)   # High

# Common in f-strings (from Lesson 2)
print(f"Server is {'healthy' if cpu < 90 else 'critical'}")
```

### Truthiness and Falsiness

**Falsy** (treated as False): `False`, `None`, `0`, `0.0`, `""`, `[]`, `{}`, `()`
**Truthy** (treated as True): everything else

```python
# Pythonic empty-check
servers = []
if servers:
    print(f"Processing {len(servers)} servers")
else:
    print("No servers to process")   # prints this

# Check for None
result = None
if result is not None:
    print(f"Result: {result}")
```

> Use `is` ONLY for `None`, `True`, `False`. Use `==` for everything else.

---

## DevOps Example — Deployment Gate

```python
cpu_ok = 65.0
memory_ok = 78.0
deployment_env = "production"
is_peak_hours = True

# Production deployment check
if deployment_env == "production":
    if is_peak_hours:
        print("⛔ Cannot deploy during peak hours (9am-6pm)")
    elif cpu_ok > 80 or memory_ok > 85:
        print(f"⛔ Resources too high: CPU {cpu_ok}%, MEM {memory_ok}%")
    else:
        print("✅ Production deployment approved")
elif deployment_env == "staging":
    print("✅ Staging deploy — proceeding")
else:
    print("✅ Dev deploy — proceeding")
```

---

## Gotchas (Louis's warnings)

**1. Indentation is the syntax — get it wrong and it silently runs wrong code:**
```python
x = 10
if x > 5:
    print("Greater than 5")
print("This is NOT in the if-block")   # Always runs — not indented
```

**2. `elif` order matters critically:**
```python
# ❌ WRONG — score 95 hits the first condition (>=60) and gets "D"
if score >= 60:
    grade = "D"
elif score >= 80:
    grade = "B"
elif score >= 90:
    grade = "A"

# ✅ RIGHT — check from highest threshold first
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 60:
    grade = "D"
```

**3. Multiple `if` vs `if-elif` are NOT the same:**
```python
x = 15
# Multiple if — ALL checked independently, ALL three print
if x > 10: print("A")
if x > 5:  print("B")
if x > 0:  print("C")

# if-elif — only FIRST match runs, prints only "A"
if x > 10:  print("A")
elif x > 5: print("B")
elif x > 0: print("C")
```

**4. Always define variables in ALL branches:**
```python
# ❌ If temp is 25, 'message' is never defined
if temp > 30: message = "Hot"
elif temp < 10: message = "Cold"
print(message)   # NameError if temp is 25!

# ✅ Add an else
if temp > 30: message = "Hot"
elif temp < 10: message = "Cold"
else: message = "Normal"
```

**5. Float comparison:**
```python
result = 0.1 + 0.2
if result == 0.3:         # False! Float imprecision
    print("equal")

if abs(result - 0.3) < 1e-9:    # ✅
    print("close enough")
```

---

## Mental Model (Donna's analogy)

If statements are **traffic signals**. Without them, your code is a straight road. With them:

```
Start → Step 1 → 🚦 DECISION
                      ↙       ↘
                [if True]   [else]
                  Step A     Step B
                      ↘       ↙
                   Step 3 → End
```

- `if` = green light only when condition is met
- `if-else` = fork in road — you must go one way
- `if-elif-else` = roundabout — take the first exit that fits, ignore all others

---

## Practice (pick one, 10 minutes)

**Beginner:** Ask for a score (0-100). Print the letter grade with `A/B/C/D/F`. Add `+` and `-` variants.

**Intermediate:** Build a billing rate calculator. Ask for client type (corporate/individual) and case complexity (simple/moderate/complex). Each combination has a different rate. Print the final rate.

**DevOps-specific:** Ask for HTTP status code. Print what it means: 200=OK, 201=Created, 301=Redirect, 400=Bad Request, 401=Unauthorized, 403=Forbidden, 404=Not Found, 429=Rate Limited, 5xx=Server Error.

---

## Quick Debug — find all 6 bugs

```python
revenue = 450000
if revenue = 500000:           # Bug 1
    print("Exact match")

score = 85
if score >= 60: grade = "D"   # Bug 2 — wrong order
elif score >= 80: grade = "B"

status = "pending"
if status == "active" or "approved":  # Bug 3
    print("Good to go")

temperature = 25
if temperature > 30: message = "Hot"
elif temperature < 10: message = "Cold"
print(message)                 # Bug 4

if risk_score > 5:
    print("High risk")
    if active:
    print("Needs review")      # Bug 5

user_input = input("Enter 5: ")
if user_input == 5:            # Bug 6
    print("Match!")
```

---

## Knowledge Check

1. What's the difference between `=` and `==`?
2. Which values are "falsy" in Python? Name 5.
3. What's wrong with `if status == "yes" or "y":`?
4. Why does `elif` order matter? Give an example where wrong order breaks the logic.
5. What is the output? `x=10; if x>5: print("A"); elif x>8: print("B"); else: print("C")`

**Answers:** (1) `=` assigns; `==` compares. (2) `False`, `0`, `""`, `[]`, `{}`, `None`, `()`, `0.0`. (3) `"y"` is always truthy — condition is always True. (4) First match wins; if least-restrictive is first, nothing more specific can match. (5) `"A"` — `x>5` is True, executes and skips elif.

---

> **Next:** Lesson 9 — For Loops → 237 clients need billing statements. You'll process them all with 3 lines of code.
