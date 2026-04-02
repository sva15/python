# Phase 1 · Lesson 1 — Variables & Numerical Data
### *"Your First Day at Pearson Specter Litt"*

> **O'Reilly Skill:** Write simple scripts that use variables to store, reassign, and perform arithmetic operations on integer and floating-point numbers.
> **DevOps connection:** Server counts, port numbers, timeouts, CPU thresholds — all numbers stored in variables.
> **10-min sprint:** Read the concept → copy one code block → run it → change one value → run again.

---

## Why This Matters (Harvey's take)

Every program ever written does one thing at its core — it **stores information** and **manipulates it**. Variables are the foundation. Numbers are how you measure winning: uptime percentages, response times, error rates.

---

## The Core Concept (Mike's breakdown)

A **variable** is a name that refers to a value stored in memory. You create one with `=`:

```python
server_count = 47          # integer (whole number)
cpu_threshold = 85.5       # float (decimal)
firm_name = "PSL"          # string (text)
```

### Integer vs Float

```python
# Integers — exact whole numbers
port_number = 8080
active_pods = 12
year = 2026

# Floats — decimal numbers
cpu_usage = 87.3
disk_threshold = 0.85
```

### Arithmetic Operators

```python
a = 20
b = 6

print(a + b)    # 26  — addition
print(a - b)    # 14  — subtraction
print(a * b)    # 120 — multiplication
print(a / b)    # 3.333... — division (ALWAYS returns float)
print(a // b)   # 3  — floor division (integer result)
print(a % b)    # 2  — modulus (remainder)
print(a ** b)   # 64000000 — exponentiation
```

### Augmented Assignment (shorthand you'll use constantly)

```python
cpu = 50
cpu += 10    # same as: cpu = cpu + 10  → 60
cpu -= 5     # → 55
cpu *= 2     # → 110
```

### Type Conversion

```python
price = 49.99
print(int(price))      # 49  — truncates, does NOT round
print(round(price))    # 50  — rounds

count = 10
print(float(count))    # 10.0

user_input = "42"
print(int(user_input) + 8)  # 50
```

---

## DevOps Example — API Rate Limiter

```python
max_requests = 100
current_requests = 73
time_remaining_seconds = 22

remaining = max_requests - current_requests
rate = remaining / time_remaining_seconds

print(f"Remaining requests: {remaining}")
print(f"Safe rate: {round(rate, 2)} req/sec")
# Remaining requests: 27
# Safe rate: 1.23 req/sec
```

---

## Gotchas (Louis's warnings)

**1. Floating-point is not exact:**
```python
print(0.1 + 0.2)   # 0.30000000000000004 — not 0.3!
# For money: use the decimal module
from decimal import Decimal
print(Decimal('0.1') + Decimal('0.2'))   # 0.3 (exact)
```

**2. `/` always returns float:**
```python
print(10 / 2)    # 5.0 — not 5!
print(type(10/2))  # <class 'float'>
```

**3. Division by zero crashes:**
```python
# Guard it:
divisor = 0
result = 100 / divisor if divisor != 0 else 0
```

**4. Large numbers — use underscores for readability:**
```python
firm_revenue = 1_000_000_000   # same as 1000000000
```

---

## Mental Model (Donna's analogy)

Variables are **labeled boxes**:
- Creating a variable = putting something in a box and labelling it
- Reassigning = replacing what's in the box
- Reading a variable = looking inside the box

Integers = counting on fingers (exact, whole)
Floats = reading a thermometer (precise, with decimals)

---

## Practice (pick one, 10 minutes)

**Beginner:** A client was billed for 85 hours at $375/hr. Calculate: total, tax (8.5%), and total-with-tax. Print all three.

**Intermediate:** Write a compound interest calculator. Formula: `A = P * (1 + r) ** t`. Calculate $500,000 at 6.5% after 1, 5, 10, and 20 years.

**DevOps-specific:** You have 3 servers. Each handles 1,200 requests/minute. You expect 10% growth next quarter. How many servers will you need at 80% capacity?

---

## Quick Debug — find all 7 bugs

```python
hourly_rate = 375
hours_worked = "40"         # Bug 1
tax rate = 8.5              # Bug 2

total = hourly_rate * hours_worked    # Bug 3

discount_percent = 10
discounted_total = total - total * discount_percent   # Bug 4

tax_amount = discounted_total * tax rate / 100        # Bug 5

print("Rate: $" hourly_rate)          # Bug 6
print("Total: $" + total)             # Bug 7
```

---

## Knowledge Check (Harvey's quiz)

1. What's the difference between `/` and `//`? Give an example where they differ.
2. What does `print(type(10/2))` output? Why?
3. Why does `0.1 + 0.2 != 0.3`? How do you fix it for financial calculations?
4. What's wrong with `2nd_client = "Acme Corp"`?
5. What is `x` after this runs: `x = 10; x += 5; x *= 2; x //= 3`?

**Answers:** (1) `/` returns float always; `//` floors to int. `7/2=3.5`, `7//2=3`. (2) `<class 'float'>`. (3) Binary can't represent 0.1 exactly; use `Decimal`. (4) Can't start with a digit. (5) `10`.

---

> **Next:** Lesson 2 — Strings & Output → Harvey needs you to generate formatted legal documents.
