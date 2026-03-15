# 🏛️ Level 1 – Exploring | Lesson 11: Write Functions with Conditionals or Loops

## *"The Toolbox"*

> **O'Reilly Skill Objective:** *Write functions with conditionals or loops.*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, `input()`, `if`/`elif`/`else`, boolean operators, `for` loops, `while` loops, `break`, `continue`, `range()`, `enumerate()`, `zip()`

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

*It's 7:30 AM. You arrive to find Mike staring at his screen, surrounded by three monitors, each showing hundreds of lines of code.*

**Mike:** "Something's wrong. We have the billing calculator in the client intake system. We have the billing calculator in the batch report generator. We have the billing calculator in the quarterly dashboard. They all calculate billing the same way — but they're all separate copies of the code."

**You:** "So?"

**Mike:** "So yesterday Louis changed the tax rate from 8.5% to 9.0%. I updated it in the intake system. Forgot to update the batch generator. The quarterly dashboard still has the OLD rate. Three systems, three different numbers. Louis is going to have a meltdown."

*Harvey walks over.*

**Harvey:** "This is exactly the problem I warned you about. You've been writing **scripts** — blocks of code that run top to bottom. But when the same logic exists in multiple places, you get inconsistency, bugs, and maintenance nightmares."

**Harvey:** "The solution? **Functions**. You write the billing calculation ONCE. Give it a name. Then every system calls that single function. Change the tax rate in one place — every system updates automatically."

**Mike:** "Functions are the single most important concept in programming. They turn code into **reusable tools**."

**Harvey:** "Today, you build a toolbox."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Let me explain why functions change everything."

**Harvey:** "Right now, you're writing code like a handwritten letter — unique, one-time, not reusable. Functions turn your code into a **stamp** — design it once, stamp it everywhere."

| Without Functions | With Functions |
|-------------------|---------------|
| Copy-paste the same code everywhere | Write once, call anywhere |
| Bug in one copy? Fix all copies manually | Fix once, fixed everywhere |
| Hard to read (huge blocks of code) | Readable (named, self-documenting) |
| Hard to test (everything interleaved) | Easy to test (isolated units) |

**Harvey:** "Functions give you four superpowers:"

1. **Reusability** — Write once, use unlimited times
2. **Abstraction** — Hide complex details behind a simple name
3. **Organization** — Break big problems into small, named pieces
4. **Testability** — Test each piece independently

**Harvey:** "In the real world, professional codebases are 90% functions. The main script is just a few lines that call functions."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "Functions are how you go from beginner to professional. Let's build from scratch."

---

### 3.1 Defining and Calling a Function

```python
# DEFINE a function
def greet():
    print("Welcome to Pearson Specter Litt!")

# CALL the function
greet()     # Welcome to Pearson Specter Litt!
greet()     # Welcome to Pearson Specter Litt!
greet()     # Welcome to Pearson Specter Litt!
```

**Mike:** "The anatomy:"

```python
def greet():                           # ← 'def' keyword + function name + parentheses + colon
    print("Welcome to PSL!")           # ← indented body (the code to run)
    print("How can we help you?")      # ← still part of the body

greet()                                # ← CALL: function_name()
```

| Component | Purpose |
|-----------|---------|
| `def` | Keyword that defines a function |
| `greet` | Function name (follow variable naming rules — snake_case) |
| `()` | Parentheses (will hold parameters later) |
| `:` | Colon ending the definition line |
| Indented block | The function body |
| `greet()` | Calling (executing) the function |

> ⚠️ **Defining vs Calling:** `def greet():` creates the function but does NOT run it. `greet()` actually runs it. Define once, call many times.

---

### 3.2 Parameters and Arguments

**Mike:** "Functions become powerful when they accept **input**."

```python
# One parameter
def greet(name):
    print(f"Welcome, {name}!")

greet("Harvey")     # Welcome, Harvey!
greet("Mike")       # Welcome, Mike!

# Multiple parameters
def calculate_billing(hours, rate):
    total = hours * rate
    print(f"Billing: {hours}h × ${rate:.2f} = ${total:,.2f}")

calculate_billing(85, 450.00)    # Billing: 85h × $450.00 = $38,250.00
calculate_billing(62, 375.00)    # Billing: 62h × $375.00 = $23,250.00
```

| Term | Meaning | Example |
|------|---------|---------|
| **Parameter** | Variable in the function definition | `def greet(name):` — `name` is a parameter |
| **Argument** | Value passed when calling the function | `greet("Harvey")` — `"Harvey"` is an argument |

---

### 3.3 Return Values

**Mike:** "Functions don't just DO things — they can **give back** results."

```python
# Without return — function does something but gives nothing back
def greet(name):
    print(f"Hello, {name}")    # Side effect: prints to screen

result = greet("Harvey")
print(result)                  # None — nothing was returned!

# With return — function calculates and gives back the result
def calculate_billing(hours, rate):
    total = hours * rate
    return total               # Gives back the value

bill = calculate_billing(85, 450.00)
print(f"Bill: ${bill:,.2f}")   # Bill: $38,250.00

# The returned value can be used in expressions
tax = calculate_billing(85, 450.00) * 0.09
print(f"Tax: ${tax:,.2f}")     # Tax: $3,442.50
```

> 💡 **Connection to Lesson 4:** Functions that return multiple values use **tuples**!

```python
def analyze_case(revenue, costs):
    profit = revenue - costs
    margin = (profit / revenue * 100) if revenue > 0 else 0
    is_profitable = profit > 0
    return profit, margin, is_profitable    # Returns a TUPLE

# Unpack the return value (tuple unpacking from Lesson 4!)
profit, margin, profitable = analyze_case(500000, 380000)
print(f"Profit: ${profit:,}")       # Profit: $120,000
print(f"Margin: {margin:.1f}%")      # Margin: 24.0%
print(f"Profitable: {profitable}")   # Profitable: True
```

---

### 3.4 Default Parameters

**Mike:** "You can give parameters default values — making them optional."

```python
def calculate_billing(hours, rate, tax_rate=0.09):
    subtotal = hours * rate
    tax = subtotal * tax_rate
    total = subtotal + tax
    return total

# Use default tax rate (0.09)
bill1 = calculate_billing(85, 450.00)
print(f"Bill: ${bill1:,.2f}")    # Bill: $41,647.50

# Override the default
bill2 = calculate_billing(85, 450.00, tax_rate=0.15)
print(f"Bill: ${bill2:,.2f}")    # Bill: $43,987.50

# Another example
def greet(name, greeting="Hello", firm="PSL"):
    print(f"{greeting}, {name}! Welcome to {firm}.")

greet("Harvey")                                   # Hello, Harvey! Welcome to PSL.
greet("Harvey", greeting="Good morning")          # Good morning, Harvey! Welcome to PSL.
greet("Harvey", firm="Zane Specter Litt")          # Hello, Harvey! Welcome to Zane Specter Litt.
```

> ⚠️ **Rule:** Default parameters must come AFTER non-default parameters.
```python
# ✅ Correct
def func(a, b, c=10):
    pass

# ❌ SyntaxError
# def func(a=10, b, c):
#     pass
```

---

### 3.5 Keyword Arguments vs Positional Arguments

```python
def create_case(client, attorney, case_type, value):
    print(f"Case: {client} | {attorney} | {case_type} | ${value:,.2f}")

# Positional — order matters!
create_case("Wayne", "Harvey", "Merger", 500000)

# Keyword — order doesn't matter!
create_case(attorney="Harvey", value=500000, client="Wayne", case_type="Merger")

# Mixed — positional first, then keyword
create_case("Wayne", "Harvey", value=500000, case_type="Merger")
```

---

### 3.6 Functions with Conditionals

**Mike:** "Now we combine functions with everything from Lessons 7–8."

```python
def classify_client(revenue, has_debt=False, is_referral=False):
    """Classify a client based on revenue and risk factors."""
    
    if has_debt:
        return "🔴 REJECTED", "Outstanding debt"
    
    if revenue >= 1000000:
        tier = "💎 Platinum"
    elif revenue >= 500000:
        tier = "🥇 Gold"
    elif revenue >= 100000:
        tier = "🥈 Silver"
    else:
        if is_referral:
            tier = "🥉 Bronze (Referral)"
        else:
            return "🔴 REJECTED", "Revenue below minimum"
    
    return tier, "Accepted"

# Test
tier, status = classify_client(750000)
print(f"Tier: {tier}, Status: {status}")
# Tier: 🥇 Gold, Status: Accepted

tier, status = classify_client(30000, is_referral=True)
print(f"Tier: {tier}, Status: {status}")
# Tier: 🥉 Bronze (Referral), Status: Accepted

tier, status = classify_client(200000, has_debt=True)
print(f"Tier: {tier}, Status: {status}")
# Tier: 🔴 REJECTED, Status: Outstanding debt
```

---

### 3.7 Functions with Loops

```python
def find_top_earner(associates):
    """Find the associate with the highest billing."""
    if not associates:
        return None, 0
    
    top_name = ""
    top_billing = 0
    
    for name, hours, rate in associates:
        billing = hours * rate
        if billing > top_billing:
            top_billing = billing
            top_name = name
    
    return top_name, top_billing

# Test
team = [
    ("Mike", 52, 375),
    ("Rachel", 48, 350),
    ("Katrina", 55, 400),
    ("Harold", 38, 280),
]

name, billing = find_top_earner(team)
print(f"Top earner: {name} (${billing:,.2f})")
# Top earner: Katrina ($22,000.00)
```

```python
def validate_input(prompt, validator, error_msg="Invalid input"):
    """Keep asking until valid input is received."""
    while True:
        value = input(prompt).strip()
        if validator(value):
            return value
        print(f"  ❌ {error_msg}")

# Usage with different validators
name = validate_input(
    "Enter name: ", 
    lambda x: len(x) > 0, 
    "Name cannot be empty"
)

age = validate_input(
    "Enter age: ", 
    lambda x: x.isdigit() and 0 < int(x) < 150, 
    "Enter a valid age (1-149)"
)
print(f"Name: {name}, Age: {age}")
```

---

### 3.8 Functions with Both Conditionals AND Loops

```python
def generate_report(clients, min_revenue=0, tier_filter=None):
    """Generate a filtered client report."""
    print(f"\n{'='*60}")
    print(f"{'CLIENT REPORT':^60}")
    print(f"{'='*60}")
    
    filtered = []
    for client in clients:
        # Apply filters (conditionals inside the loop)
        if client["revenue"] < min_revenue:
            continue
        if tier_filter and client["tier"] != tier_filter:
            continue
        filtered.append(client)
    
    if not filtered:
        print("  No clients match the criteria.")
        return 0, 0
    
    # Display results
    total = 0
    print(f"\n  {'Name':<22} {'Tier':<10} {'Revenue':>14}")
    print(f"  {'-'*46}")
    
    for client in filtered:
        print(f"  {client['name']:<22} {client['tier']:<10} ${client['revenue']:>13,.2f}")
        total += client["revenue"]
    
    avg = total / len(filtered)
    
    print(f"  {'-'*46}")
    print(f"  {'Total':<22} {'':10} ${total:>13,.2f}")
    print(f"  {'Average':<22} {'':10} ${avg:>13,.2f}")
    print(f"  Clients shown: {len(filtered)}/{len(clients)}")
    print(f"{'='*60}")
    
    return total, len(filtered)

# Usage
clients = [
    {"name": "Wayne Enterprises", "tier": "Platinum", "revenue": 2500000},
    {"name": "Stark Industries", "tier": "Gold", "revenue": 800000},
    {"name": "Oscorp", "tier": "Gold", "revenue": 600000},
    {"name": "Luthor Corp", "tier": "Silver", "revenue": 150000},
    {"name": "Gillis Industries", "tier": "Bronze", "revenue": 45000},
]

# Different calls, different filters — SAME function
generate_report(clients)                              # All clients
generate_report(clients, min_revenue=500000)          # High revenue only
generate_report(clients, tier_filter="Gold")          # Gold tier only
```

---

### 3.9 Docstrings

**Mike:** "Professional functions include documentation:"

```python
def calculate_billing(hours, rate, tax_rate=0.09):
    """
    Calculate total billing amount including tax.
    
    Args:
        hours (int/float): Number of billable hours
        rate (float): Hourly billing rate in dollars
        tax_rate (float): Tax rate as decimal (default: 0.09 = 9%)
    
    Returns:
        float: Total billing amount including tax
    
    Example:
        >>> calculate_billing(85, 450.00)
        41647.50
    """
    subtotal = hours * rate
    tax = subtotal * tax_rate
    return subtotal + tax

# Access the docstring
print(calculate_billing.__doc__)
help(calculate_billing)
```

---

### 3.10 Variable Scope — Local vs Global

**Mike:** "This is where most function bugs come from."

```python
# GLOBAL scope — defined outside any function
firm_name = "Pearson Specter Litt"

def greet(name):
    # LOCAL scope — defined inside this function
    greeting = f"Welcome to {firm_name}, {name}!"    # ✅ Can READ global
    print(greeting)

greet("Harvey")
# print(greeting)    # ❌ NameError — 'greeting' is LOCAL to greet()
print(firm_name)      # ✅ Global variable — accessible everywhere
```

```python
# GOTCHA: Can't modify a global without 'global' keyword
total = 0

def add_to_total(amount):
    # total += amount       # ❌ UnboundLocalError!
    # Python thinks 'total' is local because of the assignment
    
    global total             # ✅ Explicitly declare it's the global
    total += amount

add_to_total(100)
print(total)   # 100
```

**Mike:** "But — **avoid `global` whenever possible**. It makes code hard to reason about. Pass values in, return values out:"

```python
# ✅ BETTER — pure function, no globals
def add_to_total(current_total, amount):
    return current_total + amount

total = 0
total = add_to_total(total, 100)
total = add_to_total(total, 200)
print(total)   # 300
```

---

### 3.11 `*args` and `**kwargs`

**Mike:** "For functions that accept a variable number of arguments:"

```python
# *args — collects extra positional arguments into a TUPLE
def calculate_total(*amounts):
    """Calculate total of any number of amounts."""
    return sum(amounts)

print(calculate_total(100, 200, 300))       # 600
print(calculate_total(50))                  # 50
print(calculate_total(10, 20, 30, 40, 50))  # 150

# **kwargs — collects extra keyword arguments into a DICTIONARY
def create_profile(name, **details):
    """Create a profile with flexible fields."""
    profile = {"name": name}
    profile.update(details)
    return profile

p = create_profile("Harvey", title="Partner", rate=450, floor=50)
print(p)  # {'name': 'Harvey', 'title': 'Partner', 'rate': 450, 'floor': 50}
```

---

### 3.12 Solving Mike's Problem — The Universal Billing Library

```python
# ============================================
# Pearson Specter Litt — Billing Function Library
# ============================================
# Single source of truth — used by ALL systems

TAX_RATE = 0.09    # Change here → changes EVERYWHERE

def calculate_billing(hours, rate, tax_rate=TAX_RATE, discount=0):
    """
    Calculate total billing with tax and optional discount.
    
    Returns:
        dict: Breakdown of subtotal, discount, tax, and total
    """
    subtotal = hours * rate
    discount_amount = subtotal * discount
    taxable = subtotal - discount_amount
    tax = taxable * tax_rate
    total = taxable + tax
    
    return {
        "subtotal": subtotal,
        "discount": discount_amount,
        "taxable": taxable,
        "tax": tax,
        "total": total
    }


def classify_client(revenue, costs=0, is_referral=False, has_debt=False):
    """Classify a client into a billing tier."""
    if has_debt:
        return {"tier": "Rejected", "reason": "Outstanding debt", "icon": "🔴"}
    
    profit_margin = ((revenue - costs) / revenue * 100) if revenue > 0 else 0
    
    if revenue >= 1000000:
        tier, icon = "Platinum", "💎"
    elif revenue >= 500000:
        tier, icon = "Gold", "🥇"
    elif revenue >= 100000:
        tier, icon = "Silver", "🥈"
    elif is_referral:
        tier, icon = "Bronze", "🥉"
    else:
        return {"tier": "Rejected", "reason": "Below minimum", "icon": "🔴"}
    
    return {"tier": tier, "icon": icon, "margin": profit_margin, "reason": "Accepted"}


def assign_attorney(case_type, revenue, is_urgent=False):
    """Assign the appropriate attorney based on case type and value."""
    if is_urgent and revenue >= 500000:
        return "Harvey Specter", "Urgent high-value case"
    
    assignments = {
        "Merger": "Harvey Specter" if revenue >= 500000 else "Mike Ross",
        "Litigation": "Louis Litt",
        "Patent": "Mike Ross",
        "Corporate": "Harvey Specter" if revenue >= 1000000 else "Louis Litt",
    }
    
    attorney = assignments.get(case_type, "Associate Pool")
    reason = f"{case_type} case — ${revenue:,.0f}"
    return attorney, reason


def generate_invoice(client_name, hours, rate, **options):
    """Generate a complete invoice using all billing functions."""
    discount = options.get("discount", 0)
    tax_rate = options.get("tax_rate", TAX_RATE)
    case_type = options.get("case_type", "General")
    
    billing = calculate_billing(hours, rate, tax_rate, discount)
    
    print(f"\n{'='*50}")
    print(f"{'INVOICE':^50}")
    print(f"{'Pearson Specter Litt':^50}")
    print(f"{'='*50}")
    print(f"\n  Client:    {client_name}")
    print(f"  Case Type: {case_type}")
    print(f"  Hours:     {hours}")
    print(f"  Rate:      ${rate:,.2f}/hr")
    print(f"\n  {'—'*40}")
    print(f"  Subtotal:  ${billing['subtotal']:>12,.2f}")
    if discount > 0:
        print(f"  Discount:  -${billing['discount']:>11,.2f} ({discount*100:.0f}%)")
    print(f"  Taxable:   ${billing['taxable']:>12,.2f}")
    print(f"  Tax:       ${billing['tax']:>12,.2f} ({tax_rate*100:.1f}%)")
    print(f"  {'—'*40}")
    print(f"  TOTAL:     ${billing['total']:>12,.2f}")
    print(f"{'='*50}")
    
    return billing


# === Using the library ===
print("Testing the Billing Library:")
print("=" * 50)

# System 1: Client Intake
bill = calculate_billing(85, 450.00)
print(f"Intake billing: ${bill['total']:,.2f}")

# System 2: Batch Report
bill = calculate_billing(62, 375.00, discount=0.10)
print(f"Batch billing:  ${bill['total']:,.2f}")

# System 3: Invoice Generation
generate_invoice(
    "Wayne Enterprises", 85, 450.00,
    case_type="Merger", discount=0.05
)

# Classification
result = classify_client(750000, 580000)
print(f"\nClassification: {result['icon']} {result['tier']}")
print(f"Margin: {result['margin']:.1f}%")

# Assignment
attorney, reason = assign_attorney("Merger", 750000)
print(f"Attorney: {attorney} ({reason})")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

**Louis:** "Functions have some of the most insidious bugs in Python."

---

### Pitfall 1: Mutable Default Arguments 🔥🔥🔥

**Louis:** "THE number one function bug in Python. Memorize this."

```python
# ❌ DANGEROUS — mutable default is shared across ALL calls!
def add_client(name, clients=[]):
    clients.append(name)
    return clients

result1 = add_client("Harvey")
print(result1)     # ['Harvey']

result2 = add_client("Mike")
print(result2)     # ['Harvey', 'Mike'] ← WHAT?! Where did Harvey come from?!

# They're the SAME list object!
print(result1 is result2)   # True
```

**Louis:** "The default list `[]` is created ONCE when the function is defined — not each time it's called. Every call shares the same list."

```python
# ✅ FIX — use None as default, create a new list inside
def add_client(name, clients=None):
    if clients is None:
        clients = []         # New list each time
    clients.append(name)
    return clients

result1 = add_client("Harvey")
print(result1)     # ['Harvey']

result2 = add_client("Mike")
print(result2)     # ['Mike'] ✅ Independent!
```

---

### Pitfall 2: Forgetting to Return

```python
# ❌ Function does work but returns None
def calculate_total(hours, rate):
    total = hours * rate    # Calculated... but not returned!

result = calculate_total(85, 450)
print(result)              # None ← Forgot return!

# ✅ Return the result
def calculate_total(hours, rate):
    total = hours * rate
    return total             # ✅
```

---

### Pitfall 3: `return` Stops the Function Immediately

```python
def process():
    print("Step 1")
    return "done"
    print("Step 2")    # ⚠️ NEVER EXECUTES — dead code!

result = process()     # Only "Step 1" prints
```

```python
# Common bug: return inside a loop
def find_all_matches(items, target):
    results = []
    for item in items:
        if item == target:
            return item       # ❌ Returns on FIRST match only!
    return results

# ✅ Fix: collect all, then return
def find_all_matches(items, target):
    results = []
    for item in items:
        if item == target:
            results.append(item)
    return results             # ✅ Returns ALL matches
```

---

### Pitfall 4: Scope Confusion

```python
x = 10

def modify():
    x = 20        # This creates a LOCAL x — doesn't touch global x!
    print(f"Inside: {x}")    # 20

modify()
print(f"Outside: {x}")      # 10 — global x is unchanged!
```

---

### Pitfall 5: Calling vs Referencing

```python
def greet():
    return "Hello!"

# ❌ Missing parentheses — this is the function OBJECT, not the result
result = greet
print(result)          # <function greet at 0x...>

# ✅ Call with parentheses
result = greet()
print(result)          # Hello!
```

---

### Pitfall 6: Modifying Arguments In-Place

```python
def process_scores(scores):
    scores.sort()          # ❌ Modifies the ORIGINAL list!
    return scores

original = [88, 72, 95, 63]
result = process_scores(original)
print(original)   # [63, 72, 88, 95] ← CHANGED!

# ✅ Work on a copy
def process_scores(scores):
    sorted_scores = sorted(scores)    # Creates a new list
    return sorted_scores

original = [88, 72, 95, 63]
result = process_scores(original)
print(original)   # [88, 72, 95, 63] ← Unchanged!
```

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "Functions are easy to understand with the right analogy."

---

### 🧰 Analogy: Functions Are Like a Kitchen Appliance

**Donna:** "Think of a blender."

```
FUNCTION = Blender

PARAMETERS (what goes in):
  → Fruits, ice, milk

BODY (what happens inside):
  → Blend everything together

RETURN VALUE (what comes out):
  → A smoothie

You don't need to know HOW the blender works.
You just put stuff in and get a result out.
```

```python
def make_smoothie(fruit, liquid, ice=True):    # Parameters = ingredients
    result = f"{fruit} + {liquid}"              # Body = blending
    if ice:
        result += " + ice"
    return result + " = smoothie!"              # Return = the smoothie

drink = make_smoothie("banana", "milk")
print(drink)  # banana + milk + ice = smoothie!
```

**Donna:** "And **scope**? What happens in the blender stays in the blender. You can't reach inside while it's running. The only way to get anything out is through the pour spout (the `return` value)."

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "Functions are where you become a real programmer."

---

### 🟢 Beginner Exercises

**Exercise 1: Basic Functions**
```
Write these functions:
a) greet(name) — prints "Hello, {name}!"
b) square(n) — returns n squared
c) is_even(n) — returns True if n is even, False otherwise
d) full_name(first, last) — returns "First Last"
e) maximum(a, b) — returns the larger of two numbers (without using max())

Call each function with at least 2 different arguments.
```

**Exercise 2: Temperature Converter**
```
Write two functions:
a) celsius_to_fahrenheit(c) — returns the Fahrenheit value
b) fahrenheit_to_celsius(f) — returns the Celsius value

Formula: F = C × 9/5 + 32

Test: c_to_f(100) → 212, f_to_c(32) → 0
```

**Exercise 3: Greeting with Defaults**
```
Write a function: greet(name, greeting="Hello", punctuation="!")
Returns: "{greeting}, {name}{punctuation}"

Test with:
  greet("Harvey")                          → "Hello, Harvey!"
  greet("Mike", greeting="Good morning")   → "Good morning, Mike!"
  greet("Louis", punctuation=".")          → "Hello, Louis."
  greet("Donna", "Hi", "!!!")              → "Hi, Donna!!!"
```

---

### 🟡 Intermediate Exercises

**Exercise 4: List Processing Functions**
```
Write these functions (using loops, not built-in functions):
a) my_sum(numbers) — returns sum of all numbers
b) my_min(numbers) — returns the smallest number
c) my_max(numbers) — returns the largest number
d) my_count(items, target) — returns how many times target appears
e) my_filter(numbers, minimum) — returns list of numbers >= minimum

Test each with: [85, 92, 78, 95, 88, 71, 90]
```

**Exercise 5: Password Validator Function**
```
Write: validate_password(password)

Returns a tuple: (is_valid, list_of_issues)

Rules:
- At least 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one digit
- At least one special character

Example:
  validate_password("Hello1!")
  → (False, ["Must be at least 8 characters"])
  
  validate_password("HelloWorld123!")
  → (True, [])
```

**Exercise 6: Statistics Calculator**
```
Write: calculate_stats(numbers)

Returns a dictionary with:
  - count, sum, mean, median, 
  - minimum, maximum, range,
  - is_sorted

Test with several different lists.
Median: sort the list, then:
  - Odd length: middle element
  - Even length: average of two middle elements
```

---

### 🔴 Advanced Exercises

**Exercise 7: Recursive Functions**
```
Write these recursive functions:

a) factorial(n) — n! = n × (n-1) × ... × 1
   factorial(5) → 120

b) fibonacci(n) — returns the nth Fibonacci number
   fibonacci(7) → 13 (sequence: 0,1,1,2,3,5,8,13)

c) power(base, exponent) — base^exponent without using **
   power(2, 10) → 1024

d) reverse_string(s) — reverse a string recursively
   reverse_string("hello") → "olleh"
```

**Exercise 8: Higher-Order Functions**
```
Write functions that take OTHER functions as arguments:

a) apply_to_all(func, items) — applies func to each item, returns list
   apply_to_all(str.upper, ["hello", "world"]) → ["HELLO", "WORLD"]

b) filter_by(func, items) — returns items where func returns True
   filter_by(lambda x: x > 5, [1,8,3,9,2,7]) → [8, 9, 7]

c) reduce_to(func, items, initial) — reduces list to single value
   reduce_to(lambda a,b: a+b, [1,2,3,4], 0) → 10
```

**Exercise 9: Complete Client Management System**
```
Build a modular system using functions:

Functions needed:
1. add_client(db, name, revenue, case_type) → adds to db
2. remove_client(db, name) → removes from db
3. find_client(db, name) → returns client dict or None
4. list_clients(db, sort_by="name") → prints sorted list
5. calculate_firm_revenue(db) → returns total revenue
6. classify_all(db) → returns dict of clients by tier
7. assign_attorneys(db) → assigns attorneys to all clients
8. generate_summary(db) → prints complete firm summary

Use a list of dictionaries as your 'database'.
Create a main() function that demonstrates all functions.
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "Function bugs."

### Buggy Code 🐛

```python
# Bug Hunt: Billing Functions
# Find and fix ALL the bugs!

# Bug 1: No return
def add_tax(amount, rate=0.09):
    total = amount + (amount * rate)

result = add_tax(1000)
print(f"Total: ${result:,.2f}")

# Bug 2: Mutable default
def add_item(item, inventory=[]):
    inventory.append(item)
    return inventory

list1 = add_item("laptop")
list2 = add_item("phone")
print(f"List2: {list2}")    # Expected: ['phone']

# Bug 3: Scope error
discount = 0.10

def apply_discount(price):
    discount = 0.20         # Local variable
    return price * (1 - discount)

result = apply_discount(100)
print(f"After discount: ${result}")
print(f"Discount rate: {discount}")  # Which discount?

# Bug 4: Return in loop
def find_evens(numbers):
    for num in numbers:
        if num % 2 == 0:
            return num      # Only returns FIRST even!

evens = find_evens([1, 4, 7, 8, 3, 6])
print(f"Evens: {evens}")   # Expected: [4, 8, 6]

# Bug 5: Missing parentheses
def get_greeting():
    return "Hello!"

message = get_greeting
print(message)             # Expected: Hello!

# Bug 6: Wrong parameter order
def create_invoice(tax_rate=0.09, client, hours, rate):
    return hours * rate * (1 + tax_rate)
```

---

### Fixed Code ✅

```python
# Fixed: Billing Functions

# Fix 1: Add return statement
def add_tax(amount, rate=0.09):
    total = amount + (amount * rate)
    return total                        # ✅

result = add_tax(1000)
print(f"Total: ${result:,.2f}")        # Total: $1,090.00

# Fix 2: Use None as default for mutable types
def add_item(item, inventory=None):
    if inventory is None:
        inventory = []                  # ✅ New list each call
    inventory.append(item)
    return inventory

list1 = add_item("laptop")
list2 = add_item("phone")
print(f"List2: {list2}")              # ['phone'] ✅

# Fix 3: Be aware of scope — the function uses its LOCAL discount
discount = 0.10

def apply_discount(price, discount_rate=0.10):  # ✅ Pass as parameter
    return price * (1 - discount_rate)

result = apply_discount(100, 0.20)
print(f"After discount: ${result}")    # $80.0
print(f"Global discount: {discount}")  # 0.10 (unchanged)

# Fix 4: Collect all matches, then return
def find_evens(numbers):
    evens = []
    for num in numbers:
        if num % 2 == 0:
            evens.append(num)          # ✅ Collect, don't return yet
    return evens                        # ✅ Return after loop

evens = find_evens([1, 4, 7, 8, 3, 6])
print(f"Evens: {evens}")             # [4, 8, 6] ✅

# Fix 5: Call the function with ()
def get_greeting():
    return "Hello!"

message = get_greeting()               # ✅ Added ()
print(message)                         # Hello!

# Fix 6: Default parameters must come AFTER required ones
def create_invoice(client, hours, rate, tax_rate=0.09):  # ✅ Reordered
    return hours * rate * (1 + tax_rate)
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | Missing `return` — function returns `None` | Return statements |
| 2 | Mutable default `[]` shared across calls | Use `None` as default |
| 3 | Local `discount` shadows global | Scope awareness |
| 4 | `return` inside loop exits on first match | Collect then return |
| 5 | `get_greeting` vs `get_greeting()` | Calling vs referencing |
| 6 | Default param before required params | Parameter ordering |

---

## 8. 🌍 Real-World Application

**Harvey:** "Functions are the building blocks of all professional software."

### Where Functions Are Used:

| Domain | Function Use Case |
|--------|------------------|
| **Web APIs** | Each endpoint is a function (`get_users()`, `create_order()`) |
| **Data Science** | Cleaning functions, feature engineering, model evaluation |
| **Testing** | Each test is a function, setup/teardown are functions |
| **DevOps** | Deployment scripts, health checks, monitoring routines |
| **Libraries** | Every `len()`, `print()`, `sorted()` — all functions! |
| **Microservices** | Each service exposes functions as HTTP endpoints |

### Real Production Example: Validation Library

```python
def validate_email(email):
    """Basic email validation."""
    if not email or "@" not in email:
        return False, "Invalid email format"
    local, domain = email.rsplit("@", 1)
    if not local or not domain or "." not in domain:
        return False, "Invalid email format"
    return True, "Valid"

def validate_phone(phone):
    """Validate US phone number format."""
    digits = "".join(c for c in phone if c.isdigit())
    if len(digits) == 10:
        return True, f"({digits[:3]}) {digits[3:6]}-{digits[6:]}"
    elif len(digits) == 11 and digits[0] == "1":
        return True, f"+1 ({digits[1:4]}) {digits[4:7]}-{digits[7:]}"
    return False, "Must be 10 or 11 digits"

def validate_amount(amount_str):
    """Validate and parse monetary amount."""
    cleaned = amount_str.strip().replace("$", "").replace(",", "")
    try:
        amount = float(cleaned)
        if amount < 0:
            return False, 0, "Amount cannot be negative"
        return True, amount, f"${amount:,.2f}"
    except ValueError:
        return False, 0, "Invalid amount"

# Usage
valid, msg = validate_email("harvey@psl.com")
print(f"Email: {msg}")

valid, formatted = validate_phone("555-0100")
print(f"Phone: {formatted}")

valid, amount, msg = validate_amount("$1,250,000.50")
print(f"Amount: {msg}")
```

---

## 9. ✅ Knowledge Check

**Harvey:** "Functions quiz."

---

**Q1.** What is the difference between a parameter and an argument?

**Q2.** What does a function return if it has no `return` statement?

**Q3.** What is a default parameter? What are the rules for ordering?

**Q4.** Why is `def func(items=[])` dangerous? How do you fix it?

**Q5.** What is the output?
```python
def mystery(a, b=10):
    return a + b

print(mystery(5))
print(mystery(5, 20))
```

**Q6.** What is variable scope? What's the difference between local and global?

**Q7.** What is a docstring? How do you access it?

**Q8.** What is the output? Why?
```python
x = 5
def change():
    x = 10
change()
print(x)
```

**Q9.** Write a function that takes a list of numbers and returns a tuple of `(minimum, maximum, average)`.

**Q10.** What does `*args` do? What does `**kwargs` do?

---

### 📋 Answer Key

> **Q1.** Parameter: variable in the function definition (`def f(x):`). Argument: value passed when calling (`f(5)`).

> **Q2.** `None`.

> **Q3.** A parameter with a preset value that's used if no argument is provided. Rule: default parameters must come AFTER non-default parameters.

> **Q4.** The list is created once and shared across all calls. Fix: `def func(items=None):` then `if items is None: items = []`.

> **Q5.** `15` (uses default b=10), `25` (overrides default with 20).

> **Q6.** Scope determines where a variable is accessible. Local: inside a function only. Global: accessible everywhere. Local variables shadow global ones with the same name.

> **Q7.** A string on the first line of a function describing what it does. Access with `func.__doc__` or `help(func)`.

> **Q8.** `5` — `change()` creates a LOCAL `x = 10`, which doesn't affect the global `x = 5`.

> **Q9.**
```python
def stats(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)
```

> **Q10.** `*args` collects extra positional arguments into a tuple. `**kwargs` collects extra keyword arguments into a dictionary.

---

## 🎬 End of Lesson Scene

*It's 6:00 PM. The billing library is deployed. All three systems — intake, batch, dashboard — now call the same functions. Mike changes the tax rate in ONE place. All three systems update instantly.*

**Mike:** *(leaning back)* "One change. Three systems. Zero bugs."

**Harvey:** "That's how professionals build software. Functions, not copy-paste."

**Louis:** *(reading the docstrings)* "These are... well-documented. I approve."

**Donna:** "Tomorrow is classes and objects. Functions are tools. Classes are **blueprints**."

*Day eleven. You've gone from hardcoded scripts to reusable, well-documented function libraries. You're not writing code anymore — you're designing systems.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | Function definition with `def` | ✅ |
| 2 | Parameters and arguments | ✅ |
| 3 | Return values (including multiple returns as tuples) | ✅ |
| 4 | Default parameters | ✅ |
| 5 | Keyword vs positional arguments | ✅ |
| 6 | Functions with conditionals | ✅ |
| 7 | Functions with loops | ✅ |
| 8 | Functions with conditionals AND loops | ✅ |
| 9 | Docstrings | ✅ |
| 10 | Variable scope — local vs global | ✅ |
| 11 | `*args` and `**kwargs` | ✅ |
| 12 | Mutable default argument trap | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| Tuple unpacking (Lesson 4) | Multiple return values |
| Dictionaries (Lesson 5) | `**kwargs`, returning dicts |
| `if`/`elif`/`else` (Lesson 7) | Conditionals inside functions |
| Boolean operators (Lesson 8) | Complex function logic |
| `for` loops (Lesson 9) | Loops inside functions |
| `while` loops (Lesson 10) | Input validation functions |
| `input()` (Lesson 6) | Interactive functions |
| Built-in functions (Lessons 1–3) | Now you can build YOUR OWN |

---

> **Next Lesson →** Level 1, Lesson 12: *Define a Python Class*  
> *Harvey wants a Client object — not a dictionary, not loose variables. A proper blueprint that enforces structure…*
