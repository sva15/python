# 🏛️ Level 1 – Exploring | Lesson 1: Working with Variables and Numerical Data

## *"Your First Day at Pearson Specter Litt"*

> **O'Reilly Skill Objective:** *Write simple scripts that use variables to store, reassign, and perform arithmetic operations on integer and floating-point numbers to solve a problem.*

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

*It's 8:47 AM. The elevator doors open on the 50th floor of Pearson Specter Litt. You walk out clutching your laptop bag, heart pounding. The reception desk gleams under recessed lighting. Donna looks up from her desk with a knowing smile.*

**Donna:** "You must be the new associate. Harvey's been expecting you. Well… *I've* been expecting you. Harvey doesn't expect— he demands."

*She points toward the corner office with the floor-to-ceiling windows overlooking Manhattan.*

*You enter Harvey's office. He's standing at the window, not looking at you.*

**Harvey:** "Sit down."

*You sit. He turns around.*

**Harvey:** "Let me tell you something they didn't teach you in school. This firm doesn't just win cases anymore. We win *with data*. The partners who survive the next decade will be the ones who can automate, analyze, and optimize. That's why you're here."

*He slides a tablet across his desk. On it: a spreadsheet filled with numbers — client billable hours, revenue breakdowns, case costs.*

**Harvey:** "Jessica wants a report by end of day. She wants to know which clients are profitable and which are bleeding us dry. That spreadsheet? It's a mess. Your job is to write a Python script that calculates profit margins for each client."

*You stare at the data. Hundreds of rows.*

**Harvey:** "Don't look at me like that. You're not going to do this manually. That's what *amateurs* do."

*He picks up the phone.*

**Harvey:** "Mike. My office. Now."

*Mike Ross walks in, holding a coffee.*

**Harvey:** "Teach our new associate how to use variables and numbers in Python. They've got until 5 PM."

*Mike grins at you.*

**Mike:** "Alright, rookie. Let's start from the very beginning. And I mean the *very* beginning."

---

## 2. 🎯 Concept Introduction — Harvey Specter

*Harvey leans back in his chair.*

**Harvey:** "Before Mike starts with the technical stuff, let me tell you *why* this matters."

**Harvey:** "Every program you'll ever write does one thing at its core — it **stores information** and **manipulates it**. That's it. A billion-dollar trading algorithm? Variables and math. A self-driving car? Variables and math. Netflix's recommendation engine? Variables and math."

**Harvey:** "Variables are the **foundation of every program ever written**. If you don't understand variables, you don't understand programming. Period."

**Harvey:** "Think of it this way — in a legal case, you need to track facts: who did what, when, how much was involved. You write those facts down. You reference them later. You update them when new evidence comes in. Variables do the **exact same thing** for code."

**Harvey:** "And numbers? Numbers are how you measure *winning*. Revenue, margins, performance metrics, response times — everything that matters in this firm, in this *world*, is quantified."

**Harvey:** "So pay attention. Because if you can't handle variables and numbers, you can't handle anything I'm going to throw at you."

---

## 3. 🔧 Technical Breakdown — Mike Ross

*Mike pulls up a chair and opens a laptop.*

**Mike:** "Alright, let's build this from the ground up. No shortcuts."

---

### 3.1 What Is a Variable?

**Mike:** "A **variable** is a **name** that refers to a **value** stored in your computer's memory. You create a variable by assigning a value to a name using the `=` sign."

```python
# Creating your first variable
client_name = "Pearson Specter Litt"
```

**Mike:** "Here's what just happened, line by line:"

| Component | What it does |
|-----------|-------------|
| `client_name` | The **name** (label) you chose for this variable |
| `=` | The **assignment operator** — it means 'store this value' |
| `"Pearson Specter Litt"` | The **value** being stored (a string of text) |

**Mike:** "But today we're focusing on **numbers**, not text. Let's get into that."

---

### 3.2 Numerical Data Types: Integers and Floats

**Mike:** "Python has two core number types you need to know:"

#### **Integers (`int`)** — Whole numbers, no decimal point

```python
billable_hours = 120
total_cases = 47
year_founded = 2005
negative_balance = -5000
```

**Mike:** "Integers are **exact**. No rounding, no approximation. When you need to count things — cases, clients, years — you use integers."

#### **Floats (`float`)** — Numbers with decimal points

```python
hourly_rate = 450.75
tax_rate = 0.08
pi = 3.14159
```

**Mike:** "Floats represent **real numbers** — anything that needs precision beyond whole numbers. Hourly rates, percentages, measurements."

#### How to check the type of a variable

```python
billable_hours = 120
hourly_rate = 450.75

print(type(billable_hours))  # <class 'int'>
print(type(hourly_rate))     # <class 'float'>
```

**Mike:** "The `type()` function tells you *exactly* what kind of data you're working with. You'll use this constantly when debugging."

---

### 3.3 Variable Naming Rules

**Mike:** "Python has strict rules about what you can name a variable:"

#### ✅ Valid Names
```python
client_revenue = 50000       # snake_case (Python standard)
totalCases = 47              # camelCase (works, but not Pythonic)
_private_data = "secret"     # leading underscore (convention for internal use)
case2024 = "ongoing"         # numbers allowed (not at the start)
FIRM_NAME = "PSL"            # ALL_CAPS (convention for constants)
```

#### ❌ Invalid Names
```python
2024case = "error"           # ❌ Cannot start with a number
client-name = "error"        # ❌ Hyphens not allowed (Python thinks it's subtraction)
class = "error"              # ❌ 'class' is a reserved keyword
my variable = "error"        # ❌ Spaces not allowed
```

**Mike:** "Python convention — and this firm's standard — is **snake_case**. Words separated by underscores, all lowercase. It's clean, it's readable, it's professional."

---

### 3.4 Assignment and Reassignment

**Mike:** "Variables are called 'variables' because their values can **vary** — they can change."

```python
# Initial assignment
client_revenue = 50000
print(client_revenue)        # Output: 50000

# Reassignment — the old value is gone
client_revenue = 75000
print(client_revenue)        # Output: 75000

# You can even change the TYPE (Python is dynamically typed)
client_revenue = "seventy-five thousand"
print(client_revenue)        # Output: seventy-five thousand
print(type(client_revenue))  # <class 'str'>
```

**Mike:** "In Python, variables don't have fixed types. You can reassign a variable to hold a completely different type of data. This is called **dynamic typing**."

> 💡 **Key Insight:** The `=` sign in Python is NOT the same as `=` in math. In math, `x = 5` means "x *equals* 5." In Python, it means "**store** the value 5 into the variable named x."

#### Multiple Assignment

```python
# Assign multiple variables in one line
revenue, expenses, profit = 100000, 65000, 35000
print(revenue)    # 100000
print(expenses)   # 65000
print(profit)     # 35000

# Assign the same value to multiple variables
case_a = case_b = case_c = "pending"
print(case_a)     # pending
print(case_b)     # pending
```

---

### 3.5 Arithmetic Operations

**Mike:** "Now the fun part. Python is a **powerful calculator**. Here are all the arithmetic operators:"

```python
a = 20
b = 6

# Addition
print(a + b)      # 26

# Subtraction
print(a - b)      # 14

# Multiplication
print(a * b)      # 120

# Division (always returns a float!)
print(a / b)      # 3.3333333333333335

# Floor Division (rounds DOWN to nearest integer)
print(a // b)     # 3

# Modulus (the remainder after division)
print(a % b)      # 2

# Exponentiation (power)
print(a ** b)     # 64000000  (20 to the power of 6)
```

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `+` | Addition | `20 + 6` | `26` |
| `-` | Subtraction | `20 - 6` | `14` |
| `*` | Multiplication | `20 * 6` | `120` |
| `/` | Division | `20 / 6` | `3.333...` |
| `//` | Floor Division | `20 // 6` | `3` |
| `%` | Modulus | `20 % 6` | `2` |
| `**` | Exponentiation | `2 ** 10` | `1024` |

**Mike:** "Pay special attention to these three:"

#### 🔹 Division Always Returns a Float
```python
print(10 / 2)     # 5.0 (NOT 5!)
print(type(10/2)) # <class 'float'>
```

**Mike:** "Even when the result is a whole number, `/` gives you a float. If you need an integer, use `//`."

#### 🔹 Floor Division Rounds DOWN (Not Toward Zero)
```python
print(7 // 2)     # 3  (rounds down)
print(-7 // 2)    # -4 (rounds DOWN, not toward zero!)
```

**Mike:** "This catches people all the time. With negative numbers, floor division rounds toward *negative infinity*, not toward zero."

#### 🔹 Modulus Gives You the Remainder
```python
print(17 % 5)     # 2 (because 17 = 5*3 + 2)
print(20 % 4)     # 0 (perfectly divisible — no remainder)
```

**Mike:** "Modulus is incredibly useful. You can use it to check if a number is even or odd, cycle through values, and much more."

---

### 3.6 Augmented Assignment Operators

**Mike:** "When you want to update a variable based on its current value, Python has shorthand:"

```python
revenue = 100000

# These two lines do the exact same thing:
revenue = revenue + 5000
revenue += 5000

print(revenue)  # 110000
```

All augmented assignment operators:

```python
x = 50
x += 10    # x = x + 10  → 60
x -= 5     # x = x - 5   → 55
x *= 2     # x = x * 2   → 110
x /= 4     # x = x / 4   → 27.5
x //= 3    # x = x // 3  → 9.0
x **= 2    # x = x ** 2  → 81.0
x %= 7     # x = x % 7   → 4.0
```

---

### 3.7 Operator Precedence (Order of Operations)

**Mike:** "Python follows mathematical order of operations — **PEMDAS**:"

```
Priority (high to low):
1. () — Parentheses
2. ** — Exponentiation
3. *, /, //, % — Multiplication, Division, Floor Div, Modulus
4. +, - — Addition, Subtraction
```

```python
# Without parentheses
result = 2 + 3 * 4
print(result)     # 14 (multiplication first: 3*4=12, then 2+12=14)

# With parentheses — YOU control the order
result = (2 + 3) * 4
print(result)     # 20 (parentheses first: 2+3=5, then 5*4=20)

# Exponentiation has higher precedence than multiplication
result = 2 * 3 ** 2
print(result)     # 18 (exponent first: 3**2=9, then 2*9=18)
```

**Mike:** "My rule: **when in doubt, use parentheses**. They make your code easier to read AND guarantee the order you want."

---

### 3.8 Type Conversion (Casting)

**Mike:** "Sometimes you need to convert between number types:"

```python
# Float to Integer (truncates — doesn't round!)
price = 49.99
print(int(price))      # 49 (decimal part is CHOPPED OFF)

# Integer to Float
count = 10
print(float(count))    # 10.0

# String to Integer
user_input = "42"
number = int(user_input)
print(number + 8)      # 50

# String to Float
rate = "3.14"
pi = float(rate)
print(pi * 2)          # 6.28

# Number to String
age = 25
message = "I am " + str(age) + " years old"
print(message)         # I am 25 years old
```

> ⚠️ **Critical:** `int()` does NOT round — it **truncates** (cuts off the decimal). `int(9.9)` is `9`, not `10`. If you want rounding, use `round()`.

```python
print(int(9.9))      # 9  — truncated
print(round(9.9))    # 10 — rounded
print(round(9.5))    # 10 — rounds to nearest even (banker's rounding)
print(round(8.5))    # 8  — rounds to nearest even!
```

---

### 3.9 Useful Built-in Functions for Numbers

```python
# Absolute value
print(abs(-42))          # 42

# Rounding
print(round(3.14159, 2)) # 3.14 (round to 2 decimal places)

# Min and Max
print(min(10, 20, 5))    # 5
print(max(10, 20, 5))    # 20

# Power (same as **)
print(pow(2, 10))        # 1024

# Divmod — returns quotient AND remainder together
print(divmod(17, 5))     # (3, 2) → 17 = 5*3 + 2
```

---

### 3.10 Solving Harvey's Problem

**Mike:** "Now let's put it all together and solve Harvey's actual problem — calculate profit margins for clients."

```python
# ============================================
# Pearson Specter Litt — Client Profitability
# ============================================

# Client data: revenue earned vs costs incurred
client_a_revenue = 250000
client_a_costs = 180000

client_b_revenue = 120000
client_b_costs = 95000

client_c_revenue = 500000
client_c_costs = 620000   # This client is LOSING money

# Calculate profit for each client
profit_a = client_a_revenue - client_a_costs
profit_b = client_b_revenue - client_b_costs
profit_c = client_c_revenue - client_c_costs

# Calculate profit margin (as a percentage)
margin_a = (profit_a / client_a_revenue) * 100
margin_b = (profit_b / client_b_revenue) * 100
margin_c = (profit_c / client_c_revenue) * 100

# Display results
print("=== Client Profitability Report ===")
print()
print("Client A:")
print("  Revenue:       $" + str(client_a_revenue))
print("  Costs:         $" + str(client_a_costs))
print("  Profit:        $" + str(profit_a))
print("  Profit Margin: " + str(round(margin_a, 2)) + "%")
print()
print("Client B:")
print("  Revenue:       $" + str(client_b_revenue))
print("  Costs:         $" + str(client_b_costs))
print("  Profit:        $" + str(profit_b))
print("  Profit Margin: " + str(round(margin_b, 2)) + "%")
print()
print("Client C:")
print("  Revenue:       $" + str(client_c_revenue))
print("  Costs:         $" + str(client_c_costs))
print("  Profit:        $" + str(profit_c))
print("  Profit Margin: " + str(round(margin_c, 2)) + "%")
print()

# Determine which client is most/least profitable
best_margin = max(margin_a, margin_b, margin_c)
worst_margin = min(margin_a, margin_b, margin_c)

print("Most profitable margin:  " + str(round(best_margin, 2)) + "%")
print("Least profitable margin: " + str(round(worst_margin, 2)) + "%")
```

**Expected Output:**
```
=== Client Profitability Report ===

Client A:
  Revenue:       $250000
  Costs:         $180000
  Profit:        $70000
  Profit Margin: 28.0%

Client B:
  Revenue:       $120000
  Costs:         $95000
  Profit:        $25000
  Profit Margin: 20.83%

Client C:
  Revenue:       $500000
  Costs:         $620000
  Profit:        $-120000
  Profit Margin: -24.0%

Most profitable margin:  28.0%
Least profitable margin: -24.0%
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

*The door bursts open. Louis Litt walks in, holding a coffee mug that reads "WORLD'S BEST MANAGING PARTNER."*

**Louis:** "Oh, so we're teaching the new associate Python? Let me tell you something — I've seen plenty of so-called 'coders' crash and burn because they don't understand the *gotchas*. Let me stress-test what Mike just taught you."

---

### Pitfall 1: Floating-Point Precision 🔥

**Louis:** "Try this. Tell me the answer."

```python
print(0.1 + 0.2)
```

*You type it. The screen shows:*

```
0.30000000000000004
```

**Louis:** "SURPRISE! `0.1 + 0.2` is NOT `0.3` in Python. Welcome to the real world."

**Louis:** "Computers store numbers in **binary** (base 2). Some decimal fractions — like 0.1 — *cannot be represented exactly* in binary. It's like trying to write 1/3 as a decimal: 0.33333… It never ends."

**Louis:** "So how do you handle money and precision-critical calculations?"

```python
# BAD — never compare floats directly
print(0.1 + 0.2 == 0.3)         # False!

# GOOD — use a tolerance (epsilon)
result = 0.1 + 0.2
print(abs(result - 0.3) < 1e-9) # True

# BEST — use the decimal module for financial calculations
from decimal import Decimal
a = Decimal('0.1')
b = Decimal('0.2')
print(a + b)                     # 0.3 (exact!)
print(a + b == Decimal('0.3'))   # True
```

> 🚨 **Louis's Rule #1:** "NEVER use floats for money. Use the `decimal` module or work in cents (integers)."

---

### Pitfall 2: Integer Division Surprise

**Louis:** "Quick — what's the type of this?"

```python
result = 10 / 5
print(result)       # 2.0 — NOT 2!
print(type(result)) # <class 'float'>
```

**Louis:** "The `/` operator ALWAYS returns a float, even when dividing evenly. If you need an integer, you must explicitly use `//` or `int()`."

---

### Pitfall 3: Division by Zero

**Louis:** "What happens when you divide by zero?"

```python
print(10 / 0)   # ZeroDivisionError: division by zero
print(10 // 0)  # ZeroDivisionError: integer division or modulo by zero
print(10 % 0)   # ZeroDivisionError: integer division or modulo by zero
```

**Louis:** "Your program **crashes**. No warning, no mercy. In Level 3, you'll learn `try-except` to handle this. For now, just know: **always validate your divisor.**"

```python
# Safe division pattern
divisor = 0
if divisor != 0:
    result = 100 / divisor
else:
    result = 0
    print("Warning: Cannot divide by zero!")
```

---

### Pitfall 4: Chained Assignment Confusion

**Louis:** "What does `x = y = 5` actually do? Does it chain left-to-right or right-to-left?"

```python
x = y = 5
print(x)  # 5
print(y)  # 5

# They BOTH point to the same value in memory
print(id(x) == id(y))  # True
```

**Louis:** "For integers and floats this is harmless. But when you get to *mutable* objects in Level 2, this distinction will *haunt* you."

---

### Pitfall 5: Large Numbers and Underscores

**Louis:** "Can Python handle obscenely large numbers?"

```python
# Python handles arbitrarily large integers!
big_number = 999999999999999999999999999999999999
print(big_number + 1)  # 1000000000000000000000000000000000000

# Use underscores for readability (Python 3.6+)
firm_revenue = 1_000_000_000  # Same as 1000000000
print(firm_revenue)           # 1000000000
```

**Louis:** "Python integers have **unlimited precision**. There's no overflow like in C or Java. However, floats are limited to about 15-17 significant digits."

```python
# Float precision limit
print(123456789.123456789)  # 123456789.12345679 — precision lost!
```

---

### Pitfall 6: The Walrus Operator `:=` (Python 3.8+)

**Louis:** "Since we're being thorough, know that Python 3.8 introduced the **walrus operator** — assignment inside expressions:"

```python
# Without walrus — calculates twice
import math
value = 225
if math.sqrt(value) > 10:
    print(math.sqrt(value))  # Calculated TWICE

# With walrus — calculates once and assigns
if (root := math.sqrt(value)) > 10:
    print(root)               # Calculated ONCE, reused
```

**Louis:** "You won't need this today, but knowing it exists puts you ahead."

---

## 5. 🧠 Mental Model — Donna Paulsen

*Donna appears at the doorway, arms crossed, looking amused.*

**Donna:** "Boys, you've overwhelmed the rookie. Let me simplify this."

---

### 🏷️ Analogy: Variables Are Labeled Boxes

**Donna:** "Imagine a storage room in the firm. You have boxes on shelves."

```
📦 [client_name]      → contains: "Pearson Specter Litt"
📦 [billable_hours]    → contains: 120
📦 [hourly_rate]       → contains: 450.75
```

**Donna:** "A **variable** is a **labeled box**. The label is the **name**, and what's inside is the **value**."

- **Creating a variable** = Getting a new box, putting a label on it, and placing something inside
- **Reassigning** = Opening the box, throwing away what's inside, and putting something new in
- **Reading a variable** = Looking inside the box to see what's there

**Donna:** "Numbers come in two flavors — think of it like this:"

| Type | Real-World Analogy | Example |
|------|-------------------|---------|
| `int` (Integer) | Counting on your fingers — whole, exact | Number of clients: `47` |
| `float` (Floating-point) | Reading a thermometer — precise, with decimals | Temperature: `98.6` |

**Donna:** "And arithmetic? It's just math you already know. Python is the world's most expensive calculator— but it works *exactly* like the one you had in school."

**Donna:** "One last thing. When Louis tells you that `0.1 + 0.2 ≠ 0.3`, think about it like this: imagine trying to cut a pizza into *exactly* three equal slices. You can't. There's always a tiny, invisible sliver left over. That's floating-point math. The sliver is real, but it's so small it almost never matters — unless you're counting *money*."

*She looks at Louis.*

**Donna:** "See? Not that complicated."

*Louis mutters something under his breath.*

---

## 6. 💪 Practice Exercises — Rachel Zane

*Rachel walks in with a notepad.*

**Rachel:** "Great lesson. But knowing isn't doing. Here are your exercises. Complete all three levels before you leave today."

---

### 🟢 Beginner Exercises

**Exercise 1: Variable Basics**
```
Create variables to store the following information about a legal case:
- Case number (integer): 2024001
- Client name (string): "Wayne Enterprises"
- Settlement amount (float): 2500000.50
- Case is active (boolean): True

Print each variable and its type.
```

**Exercise 2: Simple Arithmetic**
```
A client was billed for 85 hours at a rate of $375.00 per hour.
- Calculate the total billing amount
- Calculate the tax (8.5%)
- Calculate the total with tax
- Print all three values
```

**Exercise 3: Temperature Converter**
```
Convert a temperature from Fahrenheit to Celsius.
Formula: celsius = (fahrenheit - 32) * 5 / 9

Test with: 98.6°F → should output 37.0°C
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Profit Calculator**
```
The firm handled 3 cases this quarter:
- Case 1: Revenue $150,000 | Cost $95,000
- Case 2: Revenue $280,000 | Cost $310,000
- Case 3: Revenue $420,000 | Cost $275,000

Calculate:
a) Profit for each case
b) Total profit for the quarter
c) Average profit per case
d) The profit margin percentage for each case
e) Which case had the highest profit margin?
```

**Exercise 5: Time Calculator**
```
An associate worked 14,350 minutes this month.
Convert this into:
- Total hours and remaining minutes (use // and %)
- Total days, remaining hours, and remaining minutes
Display the result in the format: "X days, Y hours, Z minutes"
```

**Exercise 6: Compound Interest**
```
The firm invests $500,000 in a trust fund at 6.5% annual interest,
compounded annually.

Formula: A = P * (1 + r) ** t

Calculate the value after 1, 5, 10, and 20 years.
Calculate how much interest was earned in each case.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Digit Extractor**
```
Given the number 83,472:
- Extract each digit using only arithmetic operations (%, //, no strings!)
- Print each digit on a separate line, from left to right.

Hint: Use combinations of // and % with powers of 10.
```

**Exercise 8: Number System Converter**  
```
Take a decimal number (e.g., 156) and convert it to:
- Binary representation (use bin())
- Octal representation (use oct())  
- Hexadecimal representation (use hex())

Also, convert the binary string '10110101' back to decimal (use int() with base).
```

**Exercise 9: Payroll Calculator**
```
Build a complete payroll calculator for the firm:

Associate data:
- Base salary: $185,000/year
- Bonus percentage: 15%
- Federal tax rate: 24%
- State tax rate: 6.85%
- 401k contribution: 6%
- Health insurance (monthly): $450

Calculate:
a) Annual gross pay (salary + bonus)
b) Annual deductions (federal tax, state tax, 401k, health insurance)
c) Annual net pay
d) Monthly net pay
e) Effective tax rate (total deductions / gross pay * 100)
f) Display a formatted payroll summary
```

---

## 7. 🐛 Debugging Scenario

*Mike slides his laptop over to you.*

**Mike:** "Before you go, one more thing. You need to learn to **debug**. Here's a script I wrote at 3 AM. It has bugs. Find them."

### Buggy Code 🐛

```python
# Bug Hunt: Client Billing Calculator
# Find and fix ALL the bugs!

hourly_rate = 375
hours_worked = "40"         # Bug 1: Why might this be a problem?
tax rate = 8.5              # Bug 2: What's wrong with this variable name?

# Calculate total billing
total = hourly_rate * hours_worked    # Bug 3: What happens here?

# Apply discount for loyal clients
discount_percent = 10
discounted_total = total - total * discount_percent   # Bug 4: Is this the right formula?

# Calculate tax
tax_amount = discounted_total * tax rate / 100        # Bug 5: Syntax issue

# Final amount
final = discounted_total + tax_amount

print("Billing Summary")
print("Rate: $" hourly_rate)          # Bug 6: Missing something?
print("Total: $" + total)             # Bug 7: Can you concatenate like this?
```

---

### Fixed Code ✅

```python
# Fixed: Client Billing Calculator

hourly_rate = 375
hours_worked = 40                     # Fix 1: Changed from string "40" to integer 40

tax_rate = 8.5                        # Fix 2: Replaced space with underscore

# Calculate total billing
total = hourly_rate * hours_worked    # Fix 3: Now both are numbers, multiplication works

# Apply discount for loyal clients  
discount_percent = 10
discounted_total = total - total * (discount_percent / 100)  # Fix 4: Convert percent properly

# Calculate tax
tax_amount = discounted_total * tax_rate / 100               # Fix 5: Variable name is now valid

# Final amount
final = discounted_total + tax_amount

print("Billing Summary")
print("Rate: $" + str(hourly_rate))    # Fix 6: Added + for concatenation
                                        #         AND str() to convert int to string
print("Total: $" + str(final))         # Fix 7: Used str() to convert float to string
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | `"40"` is a string, not an integer | Data types matter |
| 2 | Space in variable name `tax rate` | Variable naming rules |
| 3 | Can't multiply string × int meaningfully | Type errors |
| 4 | `discount_percent` used as whole number, not percentage | Math logic |
| 5 | Invalid variable name breaks syntax | Naming rules |
| 6 | Missing `+` operator and `str()` conversion | String concatenation |
| 7 | Can't concatenate `str` + `float/int` directly | Type conversion |

---

## 8. 🌍 Real-World Application

*Harvey returns to his office.*

**Harvey:** "Good. You learned the basics. Now let me tell you where this matters in the real world — because everything we do here applies outside this firm."

### Where Variables & Numbers Are Used in Production Software:

| Domain | Example Use Case |
|--------|-----------------|
| **Finance** | Calculating interest rates, portfolio values, risk metrics |
| **E-Commerce** | Shopping cart totals, discount calculations, tax computation |
| **Data Science** | Statistical analysis, feature engineering, data normalization |
| **Game Development** | Player scores, health points, physics calculations |
| **DevOps** | Server metrics, uptime percentages, resource thresholds |
| **Machine Learning** | Model weights, learning rates, loss calculations |
| **Web Development** | Pagination math, API rate limiting, session timeouts |

### Real Production Example: API Rate Limiter

```python
# A simplified rate limiter — used in every major web application
max_requests_per_minute = 100
current_requests = 73
time_remaining_seconds = 22

# Calculate available request budget
remaining_requests = max_requests_per_minute - current_requests
requests_per_second = remaining_requests / time_remaining_seconds

print("Remaining requests: " + str(remaining_requests))      # 27
print("Safe rate: " + str(round(requests_per_second, 2)) + " req/sec")  # ~1.23 req/sec
```

**Harvey:** "Every startup, every enterprise, every system in the world — they all started with someone storing a number in a variable. Don't underestimate the fundamentals."

---

## 9. ✅ Knowledge Check

*Harvey stands up.*

**Harvey:** "Before you leave this office, answer these questions. Get them wrong, and we're starting over tomorrow."

---

**Q1.** What is the difference between `/` and `//` in Python? Give an example where they produce different results.

**Q2.** What will `print(type(10 / 2))` output? Why?

**Q3.** Why does `0.1 + 0.2 != 0.3` in Python? How would you fix this for a financial application?

**Q4.** What is wrong with this variable name: `2nd_client = "Acme Corp"`?

**Q5.** What does the `%` (modulus) operator do? How would you use it to check if a number is even?

**Q6.** What is the output of this code?
```python
x = 10
x += 5
x *= 2
x //= 3
print(x)
```

**Q7.** Explain the difference between `int(9.8)` and `round(9.8)`. What does each return?

**Q8.** What is the output of `-7 // 2`? Why isn't it `-3`?

**Q9.** How do you write the number `one million` in Python so it's readable?

**Q10.** Write a single line of Python that calculates the area of a circle with radius 7. (Area = π × r²)

---

### 📋 Answer Key

> **Q1.** `/` is true division (always returns float). `//` is floor division (rounds toward negative infinity). Example: `7/2 = 3.5` but `7//2 = 3`.

> **Q2.** `<class 'float'>` — because `/` always returns a float, even for `10/2 = 5.0`.

> **Q3.** Binary can't represent 0.1 exactly. Fix: use `from decimal import Decimal` and `Decimal('0.1') + Decimal('0.2')`.

> **Q4.** Variable names cannot start with a digit. Use `second_client` instead.

> **Q5.** `%` returns the remainder of division. Even check: `number % 2 == 0`.

> **Q6.** `10 → 15 → 30 → 10` (30 // 3 = 10). Output: `10`.

> **Q7.** `int(9.8)` = `9` (truncates). `round(9.8)` = `10` (rounds).

> **Q8.** `-4`. Floor division rounds toward *negative infinity*, not zero. -3.5 rounds down to -4.

> **Q9.** `1_000_000` — underscores for readability.

> **Q10.** `area = 3.14159 * 7 ** 2` (or `import math; area = math.pi * 7 ** 2`).

---

## 🎬 End of Lesson Scene

*It's 5:02 PM. You close your laptop. Harvey looks at his watch.*

**Harvey:** "Not bad for day one."

**Mike:** "You picked it up fast. Tomorrow we tackle strings — and trust me, text manipulation is where things get *really* interesting."

**Donna:** *(from outside)* "I already scheduled your desk for 8 AM. Don't be late."

**Louis:** *(walking past)* "And don't forget about floating-point precision. I'll be quizzing you."

*You walk toward the elevator, laptop bag over your shoulder. The city lights of Manhattan shimmer through the windows. Day one: complete.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | Variables — creation, naming, reassignment | ✅ |
| 2 | Data types — `int`, `float`, `type()` | ✅ |
| 3 | Arithmetic operators — `+`, `-`, `*`, `/`, `//`, `%`, `**` | ✅ |
| 4 | Augmented assignment — `+=`, `-=`, `*=`, etc. | ✅ |
| 5 | Operator precedence — PEMDAS | ✅ |
| 6 | Type conversion — `int()`, `float()`, `str()`, `round()` | ✅ |
| 7 | Built-in functions — `abs()`, `min()`, `max()`, `pow()`, `divmod()` | ✅ |
| 8 | Floating-point precision & `Decimal` | ✅ |
| 9 | Variable naming rules and conventions | ✅ |
| 10 | Debugging type-related errors | ✅ |

---

> **Next Lesson →** Level 1, Lesson 2: *Working with Strings and Output*  
> *Harvey needs you to generate formatted legal documents using Python string methods…*
