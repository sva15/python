# 🏛️ Level 1 – Exploring | Lesson 8: Use Boolean Operators in Python

## *"The Logic Gate"*

> **O'Reilly Skill Objective:** *Use boolean operators (`and`, `or`, `not`).*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, `input()`, `if`/`elif`/`else`, comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`), truthiness/falsiness, `is` vs `==`

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

*It's 8:15 AM. You walk into Harvey's office and find him, Jessica, and Louis all staring at a whiteboard. It's covered in rules and flowcharts.*

**Jessica:** "This is the new client acceptance policy. We can't keep taking on every case that walks in the door."

*The whiteboard reads:*

```
ACCEPT a client IF:
  - Revenue >= $100K AND case is NOT criminal
  OR
  - Referred by existing Platinum client AND attorney approves
  
REJECT IF:
  - Revenue < $50K AND NOT pro-bono
  OR
  - Client has outstanding debt with us
  
FLAG FOR REVIEW IF:
  - None of the above conditions are cleanly met
```

**Louis:** "This is a nightmare of conditions. How do we even code this?"

**Harvey:** *(looking at you)* "Yesterday you learned `if` statements — single conditions. But real decisions involve **multiple conditions** combined with logic: AND, OR, NOT. That's what you're learning today."

**Mike:** *(at the door)* "Boolean operators. They let you build complex conditions out of simple ones. Think of them as **logic gates** — combining true/false signals into a final answer."

**Jessica:** "By end of day, I want this acceptance policy automated. No more judgment calls on the easy cases."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Let me give you the big picture."

**Harvey:** "In Lesson 7, every `if` statement had one condition: `if revenue > 100000`. Simple. But real-world decisions almost never depend on a single factor."

**Harvey:** "When I decide whether to take a case, I consider:"
- Is the client reputable? **AND**
- Can they pay? **AND**
- Is the case winnable? **AND**
- Does it **NOT** conflict with existing clients?

**Harvey:** "That's four conditions combined with logic. That's boolean operators."

| Operator | Meaning | Real-World Analogy |
|----------|---------|-------------------|
| `and` | Both must be True | "You need a law degree **AND** bar certification" |
| `or` | At least one must be True | "Pay by cash **OR** credit card" |
| `not` | Reverses True ↔ False | "Do **NOT** contact opposing counsel directly" |

**Harvey:** "These three operators — `and`, `or`, `not` — are the complete set. Every logical condition ever conceived can be built from these three. Boolean algebra was invented in 1847, and it still runs every computer on Earth."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "Let's go deep. Boolean logic is the backbone of computation."

---

### 3.1 The `bool` Type — True and False

```python
# Python's boolean type has exactly two values
print(type(True))     # <class 'bool'>
print(type(False))    # <class 'bool'>

# Booleans are actually integers!
print(True == 1)      # True
print(False == 0)     # True
print(True + True)    # 2
print(False + True)   # 1

# bool() converts any value to True or False
print(bool(42))       # True
print(bool(0))        # False
print(bool("hello"))  # True
print(bool(""))       # False
print(bool([1, 2]))   # True
print(bool([]))       # False
print(bool(None))     # False
```

> 💡 **Connection to Lesson 7:** This is the truthiness/falsiness concept — now formalized with `bool()`.

---

### 3.2 The `and` Operator

**Mike:** "Returns `True` only if **BOTH** conditions are `True`."

```python
has_degree = True
passed_bar = True
no_conflicts = False

# Both must be True
print(has_degree and passed_bar)       # True  (T and T = T)
print(has_degree and no_conflicts)     # False (T and F = F)
print(no_conflicts and has_degree)     # False (F and T = F)
print(no_conflicts and no_conflicts)   # False (F and F = F)
```

#### Truth Table for `and`

| A | B | A `and` B |
|---|---|-----------|
| True | True | **True** |
| True | False | False |
| False | True | False |
| False | False | False |

**Mike:** "Only one combination gives `True` — **both** must be `True`. Think of it as a **strict gatekeeper** — everyone must pass."

```python
# Practical example
revenue = 250000
is_active = True
no_debt = True

if revenue >= 100000 and is_active and no_debt:
    print("✅ Client approved")
else:
    print("❌ Client not approved")
```

---

### 3.3 The `or` Operator

**Mike:** "Returns `True` if **at least one** condition is `True`."

```python
is_partner = False
is_senior = True
has_approval = False

print(is_partner or is_senior)       # True  (F or T = T)
print(is_partner or has_approval)    # False (F or F = F)
print(is_senior or has_approval)     # True  (T or F = T)
print(is_partner or is_senior or has_approval)  # True (any one = enough)
```

#### Truth Table for `or`

| A | B | A `or` B |
|---|---|----------|
| True | True | **True** |
| True | False | **True** |
| False | True | **True** |
| False | False | False |

**Mike:** "Only one combination gives `False` — **both** must be `False`. Think of it as a **lenient gatekeeper** — if anyone passes, you're good."

```python
# Practical example
has_referral = True
revenue = 40000    # Below minimum

if revenue >= 100000 or has_referral:
    print("✅ Eligible for consideration")
else:
    print("❌ Not eligible")
# Passes because of the referral, even though revenue is low
```

---

### 3.4 The `not` Operator

**Mike:** "Flips `True` to `False` and `False` to `True`."

```python
is_criminal = True
is_active = False

print(not is_criminal)    # False  (flipped)
print(not is_active)      # True   (flipped)
print(not True)           # False
print(not False)          # True
```

#### Truth Table for `not`

| A | `not` A |
|---|---------|
| True | False |
| False | True |

```python
# Practical examples
case_type = "Merger"

# "NOT criminal" — more readable than checking every other type
if not case_type == "Criminal":
    print("We can take this case")

# Even better with !=
if case_type != "Criminal":
    print("We can take this case")

# Checking for empty/missing data
client_notes = ""
if not client_notes:
    print("No notes on file")   # Prints — empty string is falsy, not falsy = True
```

---

### 3.5 Combining Operators — Complex Conditions

**Mike:** "The real power. Combining `and`, `or`, `not` to build complex decisions."

```python
revenue = 250000
case_type = "Merger"
is_referral = False
has_debt = False

# Complex condition with and + or
if (revenue >= 100000 and case_type != "Criminal") or is_referral:
    print("✅ Accept client")

# Complex condition with not
if revenue >= 50000 and not has_debt:
    print("✅ Financial check passed")

# Multi-part complex condition
if (revenue >= 100000 or is_referral) and not has_debt and case_type != "Criminal":
    print("✅ All checks passed")
```

---

### 3.6 Operator Precedence

**Mike:** "When you combine operators, Python evaluates them in this order:"

```
Highest priority:
  1. not       ← evaluated FIRST
  2. and       ← evaluated SECOND
  3. or        ← evaluated LAST
```

```python
# Without understanding precedence, this is confusing:
result = True or False and False
# Is it: (True or False) and False → False?
# Or:    True or (False and False) → True?

# Answer: 'and' binds tighter than 'or'
# So it's: True or (False and False) → True or False → True
print(result)   # True
```

```python
# Another example
a = False
b = True
c = False

result = a or b and not c
# Step 1: not c → not False → True
# Step 2: b and True → True and True → True
# Step 3: a or True → False or True → True
print(result)   # True
```

**Mike:** "My rule: **ALWAYS use parentheses** when combining `and` and `or`. Don't rely on precedence — make your intent explicit."

```python
# ❌ Relies on precedence — confusing
if revenue >= 100000 or is_referral and not has_debt:
    pass

# ✅ Explicit parentheses — crystal clear
if (revenue >= 100000) or (is_referral and not has_debt):
    pass
```

---

### 3.7 Short-Circuit Evaluation ⭐

**Mike:** "This is a **critical** concept. Python is lazy — it stops evaluating as soon as it knows the answer."

#### `and` Short Circuits on the First `False`
```python
# If the first condition is False, Python SKIPS the second entirely
x = 0

# Without short-circuit, this would crash (division by zero)
if x != 0 and 10 / x > 2:
    print("OK")
# Since x != 0 is False, Python NEVER evaluates 10/x. No crash!
```

#### `or` Short Circuits on the First `True`
```python
# If the first condition is True, Python SKIPS the second
has_cache = True

if has_cache or expensive_database_query():
    print("Got data")
# Since has_cache is True, the database query is NEVER called!
```

**Mike:** "Short-circuit evaluation is not just an optimization — it's a **safety mechanism**. You can use it to prevent errors."

```python
# Common pattern: check before accessing
clients = []

# Safe — short-circuits if list is empty
if clients and clients[0] == "Harvey":
    print("First client is Harvey")
# If 'clients' is empty: bool([]) is False, 'and' short-circuits, 
# clients[0] is NEVER evaluated → no IndexError!

# Safe dictionary access
data = {"name": "Harvey"}
if "age" in data and data["age"] > 30:
    print("Senior")
# 'age' not in data → False → short-circuits → data["age"] never accessed
```

---

### 3.8 What `and` and `or` Actually Return

**Mike:** "Here's something that surprises most people. `and` and `or` don't necessarily return `True` or `False` — they return **the value that determined the result**."

```python
# 'and' returns the first falsy value, or the last value if all truthy
print(5 and 3)          # 3 (both truthy → returns last)
print(0 and 3)          # 0 (first is falsy → returns it)
print("" and "hello")   # "" (first is falsy → returns it)
print("hi" and "bye")   # "bye" (both truthy → returns last)

# 'or' returns the first truthy value, or the last value if all falsy
print(5 or 3)           # 5 (first is truthy → returns it)
print(0 or 3)           # 3 (first is falsy → tries second, truthy → returns it)
print("" or "hello")    # "hello" (first falsy, second truthy)
print(0 or "" or None)  # None (all falsy → returns last)
print(0 or "" or 42)    # 42 (first truthy value)
```

**Mike:** "This is why the default value pattern from Lesson 6 works:"

```python
# If user input is empty (falsy), 'or' returns the default
name = input("Name: ").strip() or "Anonymous"
# "" or "Anonymous" → "Anonymous"
# "Harvey" or "Anonymous" → "Harvey"
```

---

### 3.9 Membership Testing with `in` and Boolean Operators

```python
valid_types = ("Merger", "Litigation", "Patent", "Corporate")
excluded_clients = ["Luthor Corp", "Oscorp"]

case_type = "Merger"
client = "Wayne Enterprises"

# Combine 'in' with boolean operators
if case_type in valid_types and client not in excluded_clients:
    print("✅ Case type valid and client eligible")

# Multiple membership checks
day = "Saturday"
if day in ("Saturday", "Sunday"):
    print("Weekend rate applies")

# Check that ALL inputs are non-empty
name = "Harvey"
email = "h@psl.com"
phone = ""

if name and email and phone:
    print("All fields filled")
else:
    print("Missing required fields")  # phone is empty → falsy
```

---

### 3.10 Solving Jessica's Problem — Client Acceptance Engine

```python
# ============================================
# Pearson Specter Litt — Client Acceptance Policy
# ============================================

print("=" * 60)
print(f"{'CLIENT ACCEPTANCE EVALUATION':^60}")
print("=" * 60)
print()

# --- Gather data ---
client_name = input("  Client Name: ").strip().title()

revenue_str = input("  Expected Revenue ($): ").strip().replace(",", "")
try:
    revenue = float(revenue_str)
except ValueError:
    revenue = 0.0

case_type = input("  Case Type (Merger/Litigation/Patent/Corporate/Criminal): ").strip().title()
is_referral = input("  Referred by existing client? (yes/no): ").strip().lower() == "yes"
has_debt = input("  Outstanding debt with firm? (yes/no): ").strip().lower() == "yes"
is_pro_bono = input("  Pro-bono case? (yes/no): ").strip().lower() == "yes"

referral_tier = ""
if is_referral:
    referral_tier = input("  Referring client tier (Platinum/Gold/Silver/Bronze): ").strip().title()

attorney_approved = input("  Partner-approved? (yes/no): ").strip().lower() == "yes"

# --- Define the acceptance rules ---
valid_case_types = ("Merger", "Litigation", "Patent", "Corporate")
is_valid_type = case_type in valid_case_types

# Rule 1: Standard acceptance
standard_accept = revenue >= 100000 and is_valid_type and not has_debt

# Rule 2: Referral acceptance
referral_accept = is_referral and referral_tier == "Platinum" and attorney_approved

# Rule 3: Pro-bono acceptance
probono_accept = is_pro_bono and is_valid_type and attorney_approved

# Rule 4: Rejection conditions
reject_low_value = revenue < 50000 and not is_pro_bono
reject_debt = has_debt
reject_criminal = case_type == "Criminal"

# --- Make the decision ---
reasons = []

if reject_criminal:
    decision = "🔴 REJECTED"
    reasons.append("Criminal cases not accepted")
elif reject_debt:
    decision = "🔴 REJECTED"
    reasons.append("Client has outstanding debt")
elif reject_low_value and not is_referral:
    decision = "🔴 REJECTED"
    reasons.append("Revenue below $50K minimum (non-referred)")
elif standard_accept or referral_accept or probono_accept:
    decision = "🟢 ACCEPTED"
    if standard_accept:
        reasons.append("Meets standard revenue and case type requirements")
    if referral_accept:
        reasons.append("Platinum client referral with partner approval")
    if probono_accept:
        reasons.append("Approved pro-bono case")
else:
    decision = "🟡 FLAGGED FOR REVIEW"
    if not is_valid_type:
        reasons.append(f"Unusual case type: {case_type}")
    if revenue < 100000 and not is_pro_bono:
        reasons.append(f"Revenue ${revenue:,.0f} below standard threshold")
    if is_referral and referral_tier != "Platinum":
        reasons.append(f"Referral from {referral_tier} client (not Platinum)")
    if not attorney_approved:
        reasons.append("Missing partner approval")
    if not reasons:
        reasons.append("Does not meet standard acceptance criteria")

# --- Display result ---
print()
print("=" * 60)
print(f"{'EVALUATION RESULT':^60}")
print("=" * 60)
print()
print(f"  Client:    {client_name}")
print(f"  Revenue:   ${revenue:,.2f}")
print(f"  Type:      {case_type}")
print(f"  Referral:  {'Yes (' + referral_tier + ')' if is_referral else 'No'}")
print(f"  Pro-bono:  {'Yes' if is_pro_bono else 'No'}")
print(f"  Debt:      {'Yes ⚠️' if has_debt else 'No'}")
print(f"  Approved:  {'Yes' if attorney_approved else 'No'}")
print()
print(f"  Decision:  {decision}")
print()
print("  Reasoning:")
for i, reason in enumerate(reasons, 1):
    print(f"    {i}. {reason}")
print()
print("=" * 60)
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

**Louis:** "Boolean logic is where even experienced programmers make mistakes."

---

### Pitfall 1: The `or` Trap — Revisited One Last Time 🔥

**Louis:** "We've mentioned this before. Now let me show you *exactly* why it's wrong."

```python
color = "green"

# ❌ WRONG — "blue" is ALWAYS truthy
if color == "red" or "blue":
    print("Match!")    # ALWAYS prints!

# What Python sees:
# if (color == "red") or ("blue"):
# if False or "blue"
# if "blue"            ← non-empty string = truthy!
# if True              ← ALWAYS True!

# ✅ RIGHT — check both sides explicitly
if color == "red" or color == "blue":
    print("Match!")

# ✅ BEST — use 'in'
if color in ("red", "blue"):
    print("Match!")
```

---

### Pitfall 2: De Morgan's Laws

**Louis:** "When you negate complex conditions, the logic inverts in ways that catch people off guard."

```python
# De Morgan's Laws:
# not (A and B) == (not A) or (not B)
# not (A or B)  == (not A) and (not B)

# Example: "Not (rich and famous)" = "Not rich OR not famous"
rich = True
famous = False

# These are equivalent:
print(not (rich and famous))        # True
print((not rich) or (not famous))   # True

# These are equivalent:
print(not (rich or famous))         # False
print((not rich) and (not famous))  # False
```

**Louis:** "When you negate an `and`, it becomes an `or`, and vice versa. Forgetting this leads to logic bugs that are nearly impossible to find."

```python
# Real-world example:
# "Reject if client does NOT have (revenue >= 100K AND good credit)"
revenue = 80000
good_credit = True

# ❌ Common mistake — forgetting De Morgan's
if not revenue >= 100000 and good_credit:
    print("Rejected")   # not applies only to revenue check!
# Python reads: (not (revenue >= 100000)) and good_credit
# → (True) and True → True  ... but is that what you meant?

# ✅ Correct — negate the entire expression
if not (revenue >= 100000 and good_credit):
    print("Rejected — missing revenue OR credit")
```

---

### Pitfall 3: Short-Circuit Side Effects

**Louis:** "Short-circuiting means some code might NEVER run:"

```python
# Dangerous: relying on the second condition to do something
count = 0

def increment():
    global count
    count += 1
    return True

# If first condition is True, increment() is NEVER called!
result = True or increment()
print(count)    # 0 — increment was skipped!

# If first condition is False, increment() IS called
result = False or increment()
print(count)    # 1 — increment ran this time
```

**Louis:** "Never put important side-effects (like logging, counting, or modifying data) in short-circuit conditions."

---

### Pitfall 4: Comparing Booleans Directly

**Louis:** "Don't compare booleans to `True` or `False` — it's redundant:"

```python
is_active = True

# ❌ Redundant — comparing True to True
if is_active == True:
    print("Active")

# ❌ Also redundant
if is_active is True:
    print("Active")

# ✅ Pythonic — just use the boolean directly
if is_active:
    print("Active")

# For checking False:
# ❌ if is_active == False:
# ✅ if not is_active:
```

---

### Pitfall 5: `0` Is Falsy — Watch Out in Numeric Logic

**Louis:** "Zero is a valid number, but it's falsy:"

```python
score = 0

# ❌ This treats 0 as "no value" — but 0 IS a valid score!
if score:
    print(f"Score: {score}")
else:
    print("No score recorded")  # Prints! But the score IS 0!

# ✅ Check for None instead
score = 0

if score is not None:
    print(f"Score: {score}")    # Prints: Score: 0 ✅
else:
    print("No score recorded")
```

---

### Pitfall 6: Chaining `and`/`or` with Assignment

**Louis:** "Using `and`/`or` for conditional assignment can be tricky:"

```python
# 'or' for defaults — common and usually safe
name = "" or "Default"      # "Default" (empty string is falsy)
name = "Harvey" or "Default" # "Harvey" (non-empty is truthy)
name = 0 or "Default"       # "Default" (0 is falsy — GOTCHA!)

# 'and' for conditional values
result = True and "Success"   # "Success"
result = False and "Success"  # False
result = "data" and "valid"   # "valid" (both truthy → last value)
result = "" and "valid"       # "" (first is falsy → returns it)
```

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "Perfect analogy for this one."

---

### 🚪 Analogy: Boolean Operators Are Like Security Doors

**Donna:** "Imagine trying to get into the firm's secure server room."

#### `and` — Both Badges Required
```
Door with TWO badge readers:
  [Badge 1: ✅] AND [Badge 2: ✅] → 🚪 Door OPENS
  [Badge 1: ✅] AND [Badge 2: ❌] → 🚪 Door STAYS LOCKED
  [Badge 1: ❌] AND [Badge 2: ✅] → 🚪 Door STAYS LOCKED
  [Badge 1: ❌] AND [Badge 2: ❌] → 🚪 Door STAYS LOCKED
```

**Donna:** "You need BOTH badges to get in. If the first badge fails, the guard doesn't even check the second one — that's **short-circuit** evaluation."

#### `or` — Either Badge Works
```
Door with TWO badge readers (either works):
  [Badge 1: ✅] OR  [Badge 2: ---] → 🚪 Door OPENS (didn't check #2)
  [Badge 1: ❌] OR  [Badge 2: ✅] → 🚪 Door OPENS
  [Badge 1: ❌] OR  [Badge 2: ❌] → 🚪 Door STAYS LOCKED
```

**Donna:** "One valid badge is enough. If the first works, the guard doesn't bother checking the second."

#### `not` — Reverses Your Badge
```
🟢 Valid badge + NOT → 🔴 Invalid
🔴 Invalid badge + NOT → 🟢 Valid
```

**Donna:** "And **precedence**? Think of it this way:"
- `not` = deciding if a SINGLE badge is valid (individual check)
- `and` = checking if a PAIR of conditions are met (both doors)
- `or` = checking if ANY path works (alternative routes)

**Donna:** "Individual checks happen first, then pairs, then alternatives. `not` → `and` → `or`."

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "Logic exercises. These will train your brain to think in boolean."

---

### 🟢 Beginner Exercises

**Exercise 1: Access Control**
```
Ask for a username and password.
Set the correct values: username = "admin", password = "python123"

Grant access only if BOTH match (case-sensitive).
Print "Access Granted" or "Access Denied"
```

**Exercise 2: Weekend Detector**
```
Ask for a day of the week.
Using 'in' and boolean operators, determine:
- Is it a weekday?
- Is it a weekend?
- Print the result

Handle any capitalization (e.g., "MONDAY", "monday", "Monday")
```

**Exercise 3: Number Classifier**
```
Ask for a number. Using boolean operators, print ALL that apply:
- Positive or negative or zero
- Even or odd
- Divisible by 5
- Divisible by both 3 AND 5
- Between 1 and 100 (inclusive)
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Eligibility Checker**
```
Build a loan eligibility checker:

Ask for:
- Annual income
- Credit score (300-850)
- Years employed
- Outstanding loans (yes/no)

Rules:
- APPROVED if: income >= $50K AND credit >= 700 AND employed >= 2 years AND no loans
- CONDITIONAL if: income >= $35K AND credit >= 650 AND (employed >= 1 year OR no loans)
- DENIED if: none of the above

Print the decision with all the reasons.
```

**Exercise 5: Logical Evaluator**
```
Write a program that asks the user for two boolean values
(each as "true" or "false") and an operator ("and", "or").

Evaluate the expression and print:
  "True AND False = False"
  "True OR False = True"

Also show the result of NOT for each value.
```

**Exercise 6: Date Validator**
```
Ask for a month (1-12), day, and year.
Validate whether the date is valid:

Rules:
- Month must be 1-12
- Day must be valid for the month (handle 30/31-day months)
- February has 28 days, or 29 in a leap year
- Leap year: divisible by 4 AND (NOT divisible by 100 OR divisible by 400)
- Year must be positive

Print "Valid date" or "Invalid date" with the specific error.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Truth Table Generator**
```
Generate the complete truth table for any expression.

Ask for the operator: "and", "or", "nand", "nor", "xor", "xnor"

Print the full truth table:

Example for "and":
  A     | B     | A AND B
  ------|-------|--------
  True  | True  | True
  True  | False | False
  False | True  | False
  False | False | False

Implement all six operators:
- AND, OR (built-in)
- NAND = not (A and B)
- NOR = not (A or B)
- XOR = A or B, but not both → (A or B) and not (A and B)
- XNOR = not XOR → (A and B) or (not A and not B)
```

**Exercise 8: Password Strength Checker**
```
Ask for a password. Evaluate its strength:

Criteria (each is True/False):
1. At least 8 characters long
2. Contains at least one uppercase letter
3. Contains at least one lowercase letter
4. Contains at least one digit
5. Contains at least one special character (!@#$%^&*)
6. Does NOT contain spaces
7. Is NOT a common password (check against: "password", "123456", "qwerty")

Scoring:
- Meet all 7: "Strong 💪"
- Meet 5-6: "Moderate ⚠️"
- Meet 3-4: "Weak 🔴"
- Meet 0-2: "Unacceptable ❌"

Print which criteria pass and fail.
Hint: use any() to check for character types:
  any(c.isupper() for c in password)
```

**Exercise 9: Boolean Algebra Simplifier**
```
Given these expressions, simplify them using boolean algebra
and verify with Python:

1. not (not A)                    → should simplify to A
2. A and A                        → should simplify to A
3. A or A                         → should simplify to A
4. A and (A or B)                 → should simplify to A (absorption)
5. not (A and B)                  → not A or not B (De Morgan's)
6. not (A or B)                   → not A and not B (De Morgan's)
7. (A and B) or (A and not B)     → should simplify to A (consensus)

Write Python code that tests all combinations of A and B (True/False)
for both the original and simplified expressions, confirming they're
equivalent.
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "Boolean logic bugs."

### Buggy Code 🐛

```python
# Bug Hunt: Access Control System
# Find and fix ALL the bugs!

role = "admin"
active = True
login_attempts = 3

# Bug 1: The or trap
if role == "admin" or "manager":
    print("Admin access granted")

# Bug 2: Comparing to True directly
if active == True and login_attempts < 5:
    print("Account in good standing")

# Bug 3: Logic error with not
banned_users = ["spam_bot", "hacker_42"]
user = "harvey"
if not user in banned_users:   # Does this work?
    print("User allowed")

# Bug 4: Precedence confusion
is_vip = True
has_ticket = False
has_invitation = True
if is_vip or has_ticket and has_invitation:
    print("Entry allowed")
    # Does this check (is_vip or has_ticket) and has_invitation?
    # Or is_vip or (has_ticket and has_invitation)?

# Bug 5: Multiple conditions wrong
age = 25
income = 45000
if age >= 18 and <= 65:          # Syntax error!
    print("Working age")

# Bug 6: Falsy zero
score = 0
if not score:
    print("No score recorded")  # But 0 IS a valid score!
```

---

### Fixed Code ✅

```python
# Fixed: Access Control System

role = "admin"
active = True
login_attempts = 3

# Fix 1: Explicit comparison on both sides
if role == "admin" or role == "manager":    # ✅
    print("Admin access granted")
# Better: if role in ("admin", "manager"):

# Fix 2: Don't compare booleans to True — just use them
if active and login_attempts < 5:           # ✅ Pythonic
    print("Account in good standing")

# Fix 3: Works, but 'not in' is preferred syntax
banned_users = ["spam_bot", "hacker_42"]
user = "harvey"
if user not in banned_users:                # ✅ More Pythonic
    print("User allowed")

# Fix 4: Add explicit parentheses for your intended logic
is_vip = True
has_ticket = False
has_invitation = True
# If you meant: VIP gets in regardless, others need both ticket AND invitation
if is_vip or (has_ticket and has_invitation):  # ✅ Clear intent
    print("Entry allowed")

# Fix 5: Must repeat the variable on each side of 'and'
age = 25
income = 45000
if age >= 18 and age <= 65:                # ✅
    print("Working age")
# Better: if 18 <= age <= 65:  (chained comparison)

# Fix 6: Check for None instead of using truthiness for numeric values
score = 0
if score is not None:                      # ✅ 0 is a valid score
    print(f"Score: {score}")
else:
    print("No score recorded")
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | `or "manager"` is always truthy | The `or` trap |
| 2 | `== True` is redundant | Direct boolean usage |
| 3 | `not user in` works but `user not in` is idiomatic | `not in` syntax |
| 4 | `and` binds tighter than `or` — no explicit parentheses | Operator precedence |
| 5 | `and <= 65` — can't omit the variable | Must repeat variable |
| 6 | `not score` treats 0 as falsy | `is not None` for numerics |

---

## 8. 🌍 Real-World Application

**Harvey:** "Boolean logic runs everything."

### Where Boolean Operators Are Used in Production:

| Domain | Boolean Logic Example |
|--------|---------------------|
| **Authentication** | `valid_password and not account_locked and not expired` |
| **Search Filters** | `(category == "shoes") and (price < 100 or on_sale)` |
| **Form Validation** | `name and email and (phone or address)` — required fields |
| **Feature Flags** | `feature_enabled and (user_is_beta or user_is_admin)` |
| **Circuit Breakers** | `failures > threshold and not manual_override` |
| **Rate Limiting** | `requests > limit and not whitelisted_ip` |
| **Business Rules** | `(order > minimum or prime_member) and in_stock` |

### Real Production Example: Search Filter Engine

```python
# Simplified e-commerce search filter
products = [
    {"name": "Suit - Navy", "price": 1200, "category": "suits", "in_stock": True, "on_sale": False},
    {"name": "Suit - Grey", "price": 800, "category": "suits", "in_stock": True, "on_sale": True},
    {"name": "Tie - Silk", "price": 150, "category": "accessories", "in_stock": False, "on_sale": False},
    {"name": "Shirt - White", "price": 200, "category": "shirts", "in_stock": True, "on_sale": True},
    {"name": "Cufflinks", "price": 300, "category": "accessories", "in_stock": True, "on_sale": False},
]

# User's filter criteria
max_price = 1000
categories = ("suits", "shirts")
must_be_in_stock = True
only_on_sale = False

print("Search Results:")
print("-" * 50)

for product in products:
    # Complex boolean filter
    price_ok = product["price"] <= max_price
    category_ok = product["category"] in categories
    stock_ok = not must_be_in_stock or product["in_stock"]
    sale_ok = not only_on_sale or product["on_sale"]

    if price_ok and category_ok and stock_ok and sale_ok:
        sale_tag = " [SALE]" if product["on_sale"] else ""
        stock_tag = " [OUT OF STOCK]" if not product["in_stock"] else ""
        print(f"  ${product['price']:>6} | {product['name']}{sale_tag}{stock_tag}")
```

### Real Production Example: Access Control Matrix

```python
# Role-based access control (RBAC)
def check_access(user_role, resource, action):
    """Determine if a user has access to perform an action on a resource."""
    
    # Admin can do everything
    if user_role == "admin":
        return True
    
    # Partners can read and write all resources
    if user_role == "partner" and action in ("read", "write"):
        return True
    
    # Associates can read all, but only write their own cases
    if user_role == "associate":
        if action == "read":
            return True
        if action == "write" and resource != "finances":
            return True
    
    # Clients can only read their own cases
    if user_role == "client" and action == "read" and resource == "own_cases":
        return True
    
    return False

# Test
tests = [
    ("admin", "finances", "delete"),
    ("partner", "cases", "write"),
    ("associate", "finances", "write"),
    ("associate", "cases", "write"),
    ("client", "own_cases", "read"),
    ("client", "all_cases", "read"),
]

for role, resource, action in tests:
    allowed = check_access(role, resource, action)
    icon = "✅" if allowed else "❌"
    print(f"  {icon} {role:>10} | {action:>6} | {resource}")
```

---

## 9. ✅ Knowledge Check

**Harvey:** "Final boolean quiz."

---

**Q1.** What is the truth table for `and`? For `or`?

**Q2.** What is **short-circuit evaluation**? Give an example where it prevents an error.

**Q3.** What does `5 and 3` return? What about `0 or "hello"`? Why?

**Q4.** What are De Morgan's Laws? Write both rules.

**Q5.** What is the output?
```python
print(not True or False and True)
```

**Q6.** Why is `if x == 1 or 2 or 3` wrong? How do you fix it?

**Q7.** What is the output?
```python
print(bool(0), bool(""), bool([]), bool(None), bool(0.0))
```

**Q8.** Why is `if active == True:` considered bad style? What's the Pythonic way?

**Q9.** Write a one-line expression that checks if a number `n` is between 1 and 100 (inclusive) AND is even.

**Q10.** What is the value of `result`?
```python
result = "" or 0 or None or [] or "finally" or 42
```

---

### 📋 Answer Key

> **Q1.** `and`: T∧T=T, T∧F=F, F∧T=F, F∧F=F. `or`: T∨T=T, T∨F=T, F∨T=T, F∨F=F.

> **Q2.** Python stops evaluating once the result is known. Example: `if x != 0 and 10/x > 2:` — if x is 0, the division is never evaluated, preventing ZeroDivisionError.

> **Q3.** `5 and 3` → `3` (both truthy, returns last). `0 or "hello"` → `"hello"` (first falsy, returns first truthy).

> **Q4.** `not (A and B)` = `(not A) or (not B)`. `not (A or B)` = `(not A) and (not B)`.

> **Q5.** Precedence: `not` first → `(not True) or (False and True)` → `False or False` → `False`.

> **Q6.** `2` and `3` are always truthy — the condition is always True. Fix: `if x in (1, 2, 3):`.

> **Q7.** `False False False False False` — all are falsy values.

> **Q8.** Redundant comparison. Pythonic: `if active:`. For False check: `if not active:`.

> **Q9.** `1 <= n <= 100 and n % 2 == 0`

> **Q10.** `"finally"` — Python evaluates `or` left to right, returning the first truthy value. `""`, `0`, `None`, `[]` are all falsy. `"finally"` is the first truthy value.

---

## 🎬 End of Lesson Scene

*It's 5:30 PM. Jessica tests the acceptance engine with five different client profiles. Every decision is correct, every reason is documented.*

**Jessica:** "This saves us hours of committee discussions on obvious cases. Good work."

**Harvey:** "Tomorrow, you learn loops. Once you can repeat actions, you can process *every client* in the system automatically."

**Mike:** "Today your programs can *decide*. Tomorrow they'll *repeat*. And when you combine decisions with repetition — that's when programs become truly powerful."

**Louis:** "I want to see De Morgan's Laws applied correctly. I'll be checking."

**Donna:** "Good thing I prepared flashcards for the elevator ride down."

*Day eight. Your programs can now reason through complex multi-condition logic. The brain is fully wired.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | `bool` type — `True`, `False`, `bool()` | ✅ |
| 2 | `and` — both must be True | ✅ |
| 3 | `or` — at least one must be True | ✅ |
| 4 | `not` — reverses True/False | ✅ |
| 5 | Truth tables | ✅ |
| 6 | Operator precedence — `not` > `and` > `or` | ✅ |
| 7 | Short-circuit evaluation | ✅ |
| 8 | What `and`/`or` actually return (not always bool!) | ✅ |
| 9 | De Morgan's Laws | ✅ |
| 10 | Common traps — `or` trap, falsy zero, redundant comparisons | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| `if`/`elif`/`else` (Lesson 7) | All boolean operators used inside conditions |
| Truthiness/falsiness (Lesson 7) | Formalized with `bool()` and truth tables |
| `in` operator (Lessons 2–5) | Combined with `and`/`or`/`not` |
| Default values with `or` (Lesson 6) | Explained WHY it works via return values |
| `input()` (Lesson 6) | Interactive boolean-driven programs |
| Float precision (Lesson 1) | Boolean comparisons with floats |

---

> **Next Lesson →** Level 1, Lesson 9: *Use `for` Loops in Python*  
> *The firm has 200 clients. Harvey needs EVERY one processed automatically. Time to learn repetition…*
