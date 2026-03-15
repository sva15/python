# 🏛️ Level 1 – Exploring | Lesson 6: Get User Input With Python's `input()`

## *"The Interactive Client Intake System"*

> **O'Reilly Skill Objective:** *Get input from the user using `input()`.*

> **Previously Learned:** Variables, strings (methods, f-strings), lists, tuples, dictionaries, `type()`, `int()`, `float()`, `str()`, `print()`

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

*It's Monday morning, 8:20 AM. You walk into the firm and find the reception area packed. Five new potential clients are waiting in the lobby.*

**Donna:** *(whispering)* "Jessica just announced a client acquisition push. Every new walk-in needs to be processed — name, company, case type, estimated value. We need the data *now*, not after a three-day paper trail."

**You:** "I built the client database last week. Can't they just—"

**Donna:** "Those are hardcoded values. The associates need to **type in** client information on the spot. Your program needs to *ask questions* and *capture answers* in real time."

*Harvey walks over carrying two coffees.*

**Harvey:** "Right now, our tools are one-way streets — programs that calculate and display, but never *listen*. Today, that changes. I need you to build an **interactive intake system**. The associate sits down with the client, runs the script, and it asks them questions. Name, case type, budget — all captured and stored."

**Mike:** *(walking in)* "That's the `input()` function. It makes programs **interactive** — they stop, wait for the user, and then continue with whatever the user typed."

**Harvey:** "Simple concept, but critical. Every form you've ever filled out online, every command-line tool you've ever used, every chatbot you've ever talked to — they all start here. `input()`."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Here's why this matters."

**Harvey:** "Until today, every program you've written has been **deterministic** — you write the data in the code, and it always runs the same way. Hardcoded values. Same input, same output, every time."

**Harvey:** "But real software doesn't work that way. Real software **interacts with people**. It asks questions. It processes answers. It adapts."

**Harvey:** "The `input()` function is the simplest form of **user interaction** in Python. It does three things:"

1. **Displays a prompt** — asks the user a question
2. **Pauses execution** — waits for the user to type something and press Enter
3. **Returns the input** — gives you whatever the user typed, as a **string**

**Harvey:** "That third point is crucial. `input()` **always returns a string**. Even if the user types `42`, you get the STRING `"42"`, not the number `42`. You'll need to convert it yourself."

**Harvey:** "Once you understand `input()`, you can make *every* program you've built so far interactive. That contract generator? Interactive. The client database? Interactive. The associate tracker? Interactive."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "This lesson is shorter than the last few, but don't underestimate it. Input handling is where most bugs in beginner programs live."

---

### 3.1 Basic `input()` Usage

```python
# input() pauses the program and waits for user input
name = input("Enter your name: ")
print(f"Hello, {name}!")
```

**How it runs:**
```
Enter your name: Harvey Specter
Hello, Harvey Specter!
```

**Mike:** "Let's break down exactly what happens:"

1. Python encounters `input("Enter your name: ")`
2. It prints the prompt: `Enter your name: ` (with a trailing space for readability)
3. The program **pauses** and waits
4. The user types something and presses **Enter**
5. Everything the user typed (minus the Enter key) is returned as a **string**
6. That string is stored in the variable `name`

```python
# Without a prompt — valid but unfriendly
something = input()    # Blank cursor, user has no idea what to type

# With a prompt — always do this
something = input("What would you like to do? ")
```

> 💡 **Tip:** Always add a trailing space in your prompt (`"Enter name: "` not `"Enter name:"`) — it makes the user's input visually separated from the prompt.

---

### 3.2 `input()` ALWAYS Returns a String

**Mike:** "This is the single most important thing to understand about `input()`. It always, always returns a `str`."

```python
# User types "42" — it's stored as the STRING "42", not the number 42
age = input("Enter your age: ")
print(type(age))    # <class 'str'>
print(age)          # 42
print(age + 10)     # ❌ TypeError: can only concatenate str to str
```

**Mike:** "If you need a number, you must **convert** (cast) the input."

> 💡 **Connection to Lesson 1:** Remember `int()`, `float()`, and `str()` from the type conversion section? This is where they become essential.

---

### 3.3 Converting Input to Numbers

```python
# String to integer
age_str = input("Enter your age: ")      # "25"
age = int(age_str)                        # 25
print(f"In 10 years, you'll be {age + 10}")  # 35

# Shorthand — convert inline (most common pattern)
age = int(input("Enter your age: "))
hours = int(input("Billable hours this week: "))
rate = float(input("Hourly rate: "))

# Calculate
weekly_billing = hours * rate
print(f"Weekly billing: ${weekly_billing:,.2f}")
```

**Mike:** "The pattern `int(input("prompt"))` is one you'll write thousands of times. Memorize it."

```python
# PATTERN: Different types
name = input("Name: ")                        # Already a string
age = int(input("Age: "))                     # Convert to integer
salary = float(input("Salary: "))             # Convert to float
is_active = input("Active? (yes/no): ").lower() == "yes"  # Convert to boolean
```

---

### 3.4 Input Validation — Making Programs Robust

**Mike:** "What happens if the user types 'hello' when you expect a number?"

```python
age = int(input("Enter your age: "))   # User types "hello"
# ValueError: invalid literal for int() with base 10: 'hello'
```

**Mike:** "The program **crashes**. In a professional tool, that's unacceptable. Here are validation strategies:"

#### Strategy 1: Check Before Converting

```python
age_str = input("Enter your age: ")

if age_str.isdigit():
    age = int(age_str)
    print(f"You are {age} years old")
else:
    print("Error: Please enter a valid number")
```

**Mike:** "`.isdigit()` checks if the string contains only digits. But it doesn't handle negative numbers or decimals."

#### Strategy 2: Try-Except (Preview — Full lesson in Level 3)

```python
try:
    age = int(input("Enter your age: "))
    print(f"You are {age} years old")
except ValueError:
    print("Error: That's not a valid number!")
```

#### Strategy 3: Loop Until Valid (Using a while loop — Lesson 10)

```python
while True:
    age_str = input("Enter your age: ")
    if age_str.isdigit() and 0 < int(age_str) < 150:
        age = int(age_str)
        break
    print("Please enter a valid age (1-149)")

print(f"Age recorded: {age}")
```

**Mike:** "We'll cover `while` loops fully in Lesson 10, but this pattern is so fundamental to input validation that you need to see it now."

---

### 3.5 Processing String Input

**Mike:** "Since `input()` returns a string, all the string methods from Lesson 2 apply."

```python
# Clean up user input
name = input("Enter client name: ").strip().title()
# User types "   bruce wayne   " → becomes "Bruce Wayne"

# Case-insensitive comparison
response = input("Continue? (yes/no): ").strip().lower()
if response == "yes":
    print("Continuing...")
elif response == "no":
    print("Stopping.")
else:
    print(f"Unknown response: '{response}'")

# Split input into multiple values
date = input("Enter date (MM/DD/YYYY): ")   # "03/13/2026"
month, day, year = date.split("/")
print(f"Month: {month}, Day: {day}, Year: {year}")
```

---

### 3.6 Multiple Inputs in One Line

```python
# Method 1: Split a single input
print("Enter three numbers separated by spaces:")
numbers = input("> ").split()
# User types: "10 20 30"
# numbers = ["10", "20", "30"]

# Convert to integers
a, b, c = int(numbers[0]), int(numbers[1]), int(numbers[2])
print(f"Sum: {a + b + c}")

# Method 2: Using map() — more Pythonic
x, y, z = map(int, input("Enter 3 numbers: ").split())
print(f"Sum: {x + y + z}")

# Method 3: Collect into a list
values = list(map(float, input("Enter values: ").split()))
print(f"Average: {sum(values) / len(values)}")
```

---

### 3.7 Building Menus and Choices

```python
# Simple menu system
print("=" * 40)
print("   PEARSON SPECTER LITT - MAIN MENU")
print("=" * 40)
print("  1. Look up client")
print("  2. Add new client")
print("  3. Generate report")
print("  4. Exit")
print("=" * 40)

choice = input("Select an option (1-4): ").strip()

if choice == "1":
    print("Loading client lookup...")
elif choice == "2":
    print("Opening new client form...")
elif choice == "3":
    print("Generating report...")
elif choice == "4":
    print("Goodbye!")
else:
    print(f"Invalid option: '{choice}'")
```

**Mike:** "Notice we compare `choice` as a **string** (`"1"`, not `1`). Since `input()` returns a string, it's simpler to compare directly than to convert."

---

### 3.8 Solving the Client Intake Problem

**Mike:** "Let's build that interactive intake system Harvey asked for."

```python
# ============================================
# Pearson Specter Litt — Client Intake System
# ============================================

print()
print("=" * 55)
print(f"{'PEARSON SPECTER LITT':^55}")
print(f"{'New Client Intake Form':^55}")
print("=" * 55)
print()

# --- Gather client information ---
client_name = input("  Client Name:      ").strip().title()
company = input("  Company:          ").strip().title()
phone = input("  Phone:            ").strip()
email = input("  Email:            ").strip().lower()

print()
print("  Case Types: Merger | Litigation | Patent | Corporate | Criminal")
case_type = input("  Case Type:        ").strip().title()

# --- Numeric inputs with basic validation ---
amount_str = input("  Estimated Value ($): ").strip().replace(",", "").replace("$", "")
if amount_str.replace(".", "", 1).isdigit():
    estimated_value = float(amount_str)
else:
    estimated_value = 0.0
    print("  ⚠️  Could not parse value. Set to $0.00")

# --- Attorney assignment ---
print()
print("  Available Attorneys:")
print("    1. Harvey Specter (Senior Partner)")
print("    2. Mike Ross (Senior Associate)")
print("    3. Louis Litt (Named Partner)")
attorney_choice = input("  Assign Attorney (1-3): ").strip()

attorneys = {"1": "Harvey Specter", "2": "Mike Ross", "3": "Louis Litt"}
attorney = attorneys.get(attorney_choice, "Unassigned")

# --- Confirmation ---
urgency = input("  Urgent case? (yes/no): ").strip().lower()
is_urgent = urgency == "yes"

notes = input("  Additional Notes:  ").strip()

# --- Build the client record ---
import random
case_id = f"PSL-2026-{random.randint(1000, 9999)}"

client_record = {
    "case_id": case_id,
    "client_name": client_name,
    "company": company,
    "phone": phone,
    "email": email,
    "case_type": case_type,
    "estimated_value": estimated_value,
    "attorney": attorney,
    "urgent": is_urgent,
    "notes": notes
}

# --- Display the completed record ---
print()
print("=" * 55)
print(f"{'CLIENT RECORD CREATED':^55}")
print("=" * 55)
print()
print(f"  Case ID:      {client_record['case_id']}")
print(f"  Client:       {client_record['client_name']}")
print(f"  Company:      {client_record['company']}")
print(f"  Phone:        {client_record['phone']}")
print(f"  Email:        {client_record['email']}")
print(f"  Case Type:    {client_record['case_type']}")
print(f"  Value:        ${client_record['estimated_value']:,.2f}")
print(f"  Attorney:     {client_record['attorney']}")
print(f"  Urgent:       {'🔴 YES' if client_record['urgent'] else '🟢 No'}")
print(f"  Notes:        {client_record['notes'] if client_record['notes'] else 'None'}")
print()
print("=" * 55)
print("  ✅ Record saved successfully!")
print("=" * 55)
```

**Sample Session:**
```
=======================================================
                 PEARSON SPECTER LITT                  
                New Client Intake Form                 
=======================================================

  Client Name:      bruce wayne
  Company:          wayne enterprises
  Phone:            555-0101
  Email:            Bruce.Wayne@Wayne.COM

  Case Types: Merger | Litigation | Patent | Corporate | Criminal
  Case Type:        merger
  Estimated Value ($): 2,500,000

  Available Attorneys:
    1. Harvey Specter (Senior Partner)
    2. Mike Ross (Senior Associate)
    3. Louis Litt (Named Partner)
  Assign Attorney (1-3): 1
  Urgent case? (yes/no): yes
  Additional Notes:  High-profile acquisition, NDA required

=======================================================
                 CLIENT RECORD CREATED                 
=======================================================

  Case ID:      PSL-2026-4287
  Client:       Bruce Wayne
  Company:      Wayne Enterprises
  Phone:        555-0101
  Email:        bruce.wayne@wayne.com
  Case Type:    Merger
  Value:        $2,500,000.00
  Attorney:     Harvey Specter
  Urgent:       🔴 YES
  Notes:        High-profile acquisition, NDA required

=======================================================
  ✅ Record saved successfully!
=======================================================
```

**Mike:** "Notice how we cleaned every input: `.strip()` removes whitespace, `.title()` fixes casing, `.lower()` normalizes emails. We also validated the numeric input and used a dictionary for the attorney lookup. This is **everything from Lessons 1–5** coming together."

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

**Louis:** "User input is the most **dangerous** thing in programming. Users will type anything."

---

### Pitfall 1: Forgetting That `input()` Returns a String

**Louis:** "I've seen this bug more times than I can count:"

```python
# User types "5"
num = input("Enter a number: ")

# WRONG — Comparing string to int
if num == 5:                   # ALWAYS False!
    print("You entered five")

# RIGHT — Convert first
if int(num) == 5:
    print("You entered five")

# ALSO WRONG — Accidentally multiplying a string
quantity = input("How many? ")   # "3"
total = quantity * 10            # "3333333333" — string repeated 10 times!

# RIGHT
total = int(quantity) * 10       # 30
```

---

### Pitfall 2: Crashing on Invalid Numeric Input

**Louis:** "If you blindly convert without checking, you WILL crash:"

```python
# User types "abc"
age = int(input("Age: "))   # 💥 ValueError

# User types ""  (just presses Enter)
age = int(input("Age: "))   # 💥 ValueError

# User types "25.5"
age = int(input("Age: "))   # 💥 ValueError (can't int() a decimal string!)
```

**Louis:** "Defense patterns:"

```python
# Pattern 1: isdigit() for integers
value = input("Enter a whole number: ")
if value.isdigit():
    number = int(value)
else:
    print("Invalid! Not a number.")

# Pattern 2: Try-except for any conversion
try:
    amount = float(input("Enter amount: "))
except ValueError:
    print("Invalid amount!")
    amount = 0.0

# Pattern 3: is-numeric check for floats (handles decimals)
def is_number(s):
    """Check if a string can be converted to a float."""
    try:
        float(s)
        return True
    except ValueError:
        return False

value = input("Enter any number: ")
if is_number(value):
    number = float(value)
    print(f"Valid: {number}")
else:
    print("Not a valid number!")
```

---

### Pitfall 3: Leading/Trailing Whitespace

**Louis:** "Users are sloppy. They add spaces everywhere:"

```python
name = input("Name: ")      # User types "  Harvey  "
print(f"'{name}'")           # '  Harvey  '
print(len(name))             # 10 (not 6!)
print(name == "Harvey")      # False!

# ALWAYS strip
name = input("Name: ").strip()
print(f"'{name}'")           # 'Harvey'
print(name == "Harvey")      # True ✅
```

---

### Pitfall 4: Case Sensitivity in Comparisons

**Louis:** "The user's 'Yes' is not your 'yes':"

```python
answer = input("Continue? (yes/no): ")

# FRAGILE — only catches exact lowercase
if answer == "yes":
    print("OK")

# ROBUST — normalize case
if answer.strip().lower() == "yes":
    print("OK")

# MOST ROBUST — accept multiple variations
if answer.strip().lower() in ("yes", "y", "yeah", "yep", "sure"):
    print("OK")
```

---

### Pitfall 5: Empty Input

**Louis:** "What if the user just presses Enter?"

```python
name = input("Name: ")  # User presses Enter immediately
print(f"Name is: '{name}'")  # ''
print(len(name))             # 0
print(bool(name))            # False — empty strings are falsy!

# Guard against empty input
name = input("Name: ").strip()
if not name:
    print("Error: Name cannot be empty!")
else:
    print(f"Welcome, {name}")
```

---

### Pitfall 6: The `isdigit()` Limitation

**Louis:** "`.isdigit()` doesn't handle everything you'd expect:"

```python
print("42".isdigit())      # True ✅
print("3.14".isdigit())    # False ❌ (decimals!)
print("-5".isdigit())      # False ❌ (negative sign!)
print("".isdigit())        # False ❌ (empty string!)
print(" 42 ".isdigit())    # False ❌ (spaces!)
```

**Louis:** "For floats and negatives, you need `try/except float()`. For integers, strip first: `value.strip().isdigit()`."

---

### Pitfall 7: `input()` in Python 2 vs Python 3

**Louis:** "If you're ever working with legacy code:"

```python
# Python 3: input() always returns a string ✅
# Python 2: input() EVALUATES the input as Python code! ☠️
# Python 2: raw_input() returns a string (equivalent to Python 3's input())

# NEVER use Python 2's input() for user data — it's a security vulnerability
# Always use Python 3
```

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "Let me simplify this."

---

### 🎤 Analogy: `input()` Is Like a Receptionist

**Donna:** "Think of `input()` as the firm's receptionist."

```
You → Receptionist: "What is the client's name?"   ← The prompt
     ← Receptionist WAITS for client to answer      ← Program pauses
Client → Receptionist: "Bruce Wayne"                ← User types
Receptionist → You: hands a NOTE that says "Bruce Wayne"  ← Returns string
```

- **The prompt** = The question the receptionist asks
- **Waiting** = The program pauses until the user types something
- **The return value** = A **handwritten note** — it's ALWAYS a piece of paper (string), even if the client says a number. "42" on paper is still text, not a number
- **Type conversion** = YOU read the note and interpret it. If the note says "42", you decide whether it's a number (`int("42")`) or just text

**Donna:** "The key insight? The receptionist doesn't interpret. She doesn't decide if '42' is a number or text. She just writes it down and hands it to you. **You** do the interpretation."

**Donna:** "And validation? That's like the receptionist asking the same question again if the answer doesn't make sense:"

```python
# Receptionist asks → Client gives a bad answer → Ask again
answer = input("Phone number: ")        # Client says "banana"
# "That doesn't look like a phone number. Could you repeat that?"
```

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "Time to make your programs interactive."

---

### 🟢 Beginner Exercises

**Exercise 1: Personal Greeter**
```
Ask the user for their first name and last name.
Print a greeting like:
  "Welcome to Pearson Specter Litt, Harvey Specter!"

Make sure it's properly capitalized even if the user types in ALL CAPS or all lowercase.
```

**Exercise 2: Simple Calculator**
```
Ask the user for two numbers and an operation (+, -, *, /).
Perform the calculation and print the result.

Example:
  Enter first number: 42
  Enter second number: 8
  Enter operation (+, -, *, /): *
  Result: 42 * 8 = 336
```

**Exercise 3: Age Calculator**
```
Ask the user for their birth year.
Calculate and print:
  - Their current age (use 2026 as the current year)
  - The year they will turn 100

Handle invalid input: if they enter something that's not a number,
print an error message instead of crashing.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Temperature Converter**
```
Build an interactive temperature converter:

1. Ask the user: "Convert from (C)elsius or (F)ahrenheit?"
2. Ask for the temperature value
3. Convert and display the result with 1 decimal place
4. Handle invalid choice (not C or F)
5. Handle invalid temperature (not a number)

Example:
  Convert from (C)elsius or (F)ahrenheit? C
  Enter temperature: 100
  100.0°C = 212.0°F
```

**Exercise 5: Interactive Shopping Cart**
```
Build a shopping cart where the user can:
- Type an item name and price to add it
- Type "done" to finish shopping
- See a running total after each item
- See a final receipt with all items, subtotal, tax (8.5%), and total

Example:
  Item: Coffee
  Price: 4.50
  Added Coffee ($4.50) — Running total: $4.50

  Item: Sandwich
  Price: 8.75
  Added Sandwich ($8.75) — Running total: $13.25

  Item: done

  === RECEIPT ===
  Coffee          $  4.50
  Sandwich        $  8.75
  ----------------
  Subtotal        $ 13.25
  Tax (8.5%)      $  1.13
  TOTAL           $ 14.38
```

**Exercise 6: Number Guessing Game (Simplified)**
```
The program "thinks" of a number (hardcode it, e.g., 42).
The user has 5 guesses.

Each round:
1. Ask for a guess
2. Tell them if it's too high, too low, or correct
3. If correct, print "You got it in X guesses!"
4. If they run out of guesses, reveal the answer

Handle non-numeric input gracefully.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Multi-Client Intake**
```
Extend the intake system to handle MULTIPLE clients:

1. Ask "How many clients to process?"
2. For each client, collect: name, company, case type, estimated value
3. Store all clients in a list of dictionaries
4. After all clients are entered, display:
   - A formatted table of all clients
   - Total estimated value across all clients
   - Average estimated value
   - The highest-value client
5. Ask "Export to screen? (yes/no)" and display accordingly
```

**Exercise 8: Mad Libs Generator**
```
Create an interactive Mad Libs story:

Ask the user for:
  - A person's name
  - An adjective
  - A noun
  - A verb (past tense)
  - A place
  - A number

Build a funny story using f-strings:
  "{name} walked into {place} carrying a {adjective} {noun}. 
   They {verb} exactly {number} times before anyone noticed.
   'This is perfectly normal,' said {name}."
```

**Exercise 9: Quiz Engine**
```
Build an interactive quiz with at least 5 questions.

Store questions as a list of dictionaries:
  questions = [
      {"question": "What does len() return?", "answer": "length"},
      {"question": "Are lists mutable?", "answer": "yes"},
      ...
  ]

For each question:
1. Display the question
2. Get the user's answer
3. Compare (case-insensitive) and say correct/incorrect
4. Track the score
5. At the end, display: score, percentage, and a grade (A/B/C/D/F)
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "Input bugs. These are where most beginner programs fail."

### Buggy Code 🐛

```python
# Bug Hunt: Client Fee Calculator
# Find and fix ALL the bugs!

# Bug 1: Get client name
name = input("Client name: ")
print("Welcome, " + Name)            # Hmm...

# Bug 2: Get hourly rate
rate = input("Hourly rate: ")
print(f"Rate: ${rate:.2f}")           # Will the format work?

# Bug 3: Get hours
hours = input("Hours worked: ")
total = rate * hours                  # What types are these?
print(f"Total: ${total}")

# Bug 4: Get discount
discount = input("Discount %: ")     # User enters "10"
discounted = total - (total * discount / 100)   # Math on strings?

# Bug 5: Yes/No confirmation
confirm = input("Confirm billing? (yes/no): ")
if confirm == "Yes" or "yes":        # Logic error?
    print("Billing confirmed!")

# Bug 6: Percentage display
print(f"Discount was: {discount}%")   # What if discount is "10"?
# Expected: "Discount was: 10%"
# Actually prints: "Discount was: 10%" — but what TYPE is it?
```

---

### Fixed Code ✅

```python
# Fixed: Client Fee Calculator

# Fix 1: Python is case-sensitive — 'Name' vs 'name'
name = input("Client name: ").strip().title()
print("Welcome, " + name)             # ✅ Correct variable name

# Fix 2: Convert to float BEFORE formatting
rate = float(input("Hourly rate: "))   # ✅ Convert immediately
print(f"Rate: ${rate:.2f}")

# Fix 3: Both must be numbers for multiplication
hours = float(input("Hours worked: "))  # ✅ Convert to number
total = rate * hours                     # ✅ float * float works
print(f"Total: ${total:,.2f}")

# Fix 4: Convert discount to a number
discount = float(input("Discount %: "))  # ✅ Convert
discounted = total - (total * discount / 100)
print(f"After discount: ${discounted:,.2f}")

# Fix 5: 'or' doesn't work this way — need explicit comparisons
confirm = input("Confirm billing? (yes/no): ").strip().lower()
if confirm == "yes" or confirm == "y":   # ✅ Compare both sides
    print("Billing confirmed!")
# EVEN BETTER:
# if confirm in ("yes", "y"):

# Fix 6: This actually "works" but discount is now a float, not string
# Just demonstrating awareness of types
print(f"Discount was: {discount}%")      # ✅ Works fine since we converted
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | `Name` vs `name` — case-sensitive variable names | Python is case-sensitive |
| 2 | Can't format a string with `:.2f` | Must convert to float first |
| 3 | String × String doesn't give numeric result | Convert inputs to numbers |
| 4 | Math on strings raises TypeError | Type conversion |
| 5 | `if x == "Yes" or "yes"` always True! | Boolean logic (Lesson 8 preview) |
| 6 | Not a bug, but awareness of types matters | Type tracking |

> 🔍 **Bug 5 Deep Dive:** `if confirm == "Yes" or "yes"` is parsed as `if (confirm == "Yes") or ("yes")`. Since `"yes"` is a non-empty string, it's **always truthy**, so the condition is **always True**, regardless of what the user types!

---

## 8. 🌍 Real-World Application

**Harvey:** "Every piece of software that interacts with humans starts with input."

### Where `input()` and User Input Concepts Apply:

| Domain | Input Mechanism |
|--------|----------------|
| **Web Applications** | HTML forms, search bars, login pages |
| **APIs** | Request parameters, POST body data |
| **CLI Tools** | Command-line arguments, interactive prompts |
| **Mobile Apps** | Text fields, voice input, gestures |
| **Games** | Keyboard input, mouse clicks, controller |
| **Chatbots** | Natural language messages |
| **Databases** | SQL queries from user interfaces |

### Real Production Example: CLI Tool

```python
# A simplified command-line tool (like git, docker, npm, etc.)
print("PSL Case Tool v1.0")
print("Type 'help' for available commands.\n")

while True:
    command = input("psl> ").strip().lower()

    if command == "help":
        print("  list    — List all clients")
        print("  add     — Add a new client")
        print("  search  — Search for a client")
        print("  quit    — Exit the tool")
    elif command == "list":
        print("  1. Wayne Enterprises")
        print("  2. Stark Industries")
        print("  3. Oscorp")
    elif command == "search":
        term = input("  Search term: ").strip().lower()
        clients = ["wayne enterprises", "stark industries", "oscorp"]
        results = [c for c in clients if term in c]
        if results:
            for r in results:
                print(f"  Found: {r.title()}")
        else:
            print("  No results found.")
    elif command in ("quit", "exit", "q"):
        print("Goodbye!")
        break
    elif command == "":
        continue
    else:
        print(f"  Unknown command: '{command}'. Type 'help'.")
```

### Real Production Example: Configuration Wizard

```python
# Setup wizard — common in tools like npm init, python setup.py, etc.
print("=" * 50)
print("PSL Project Setup Wizard")
print("=" * 50)
print("Press Enter to accept [default values]\n")

project_name = input("Project name [untitled]: ").strip() or "untitled"
version = input("Version [1.0.0]: ").strip() or "1.0.0"
author = input("Author []: ").strip() or "Anonymous"
license_type = input("License [MIT]: ").strip() or "MIT"

config = {
    "name": project_name,
    "version": version,
    "author": author,
    "license": license_type
}

print("\nGenerated configuration:")
for key, value in config.items():
    print(f"  {key}: {value}")

confirm = input("\nLooks good? (yes/no): ").strip().lower()
if confirm in ("yes", "y", ""):
    print("✅ Configuration saved!")
else:
    print("❌ Setup cancelled.")
```

**Mike:** "Notice the `or` pattern: `input("Name [default]: ").strip() or "default"`. If the user presses Enter (empty string, which is falsy), the default value is used. This is a production pattern you'll see everywhere."

---

## 9. ✅ Knowledge Check

**Harvey:** "Quick quiz on input."

---

**Q1.** What type does `input()` always return?

**Q2.** What is the output of this code if the user types `5`?
```python
x = input("Number: ")
print(x * 3)
```

**Q3.** How do you convert user input to an integer? To a float?

**Q4.** What happens if you run `int(input("Age: "))` and the user types `"twenty"`?

**Q5.** Why should you always call `.strip()` on user input?

**Q6.** What's wrong with this code?
```python
answer = input("Yes or No? ")
if answer == "yes" or "Yes":
    print("Confirmed")
```

**Q7.** How do you handle empty input (user just presses Enter)?

**Q8.** Write one line that asks for a number and stores it as a float.

**Q9.** What does `input("Name: ").strip().lower()` do? When would you use it?

**Q10.** How would you accept a default value — i.e., user can press Enter to accept "42" as the default?

---

### 📋 Answer Key

> **Q1.** Always `str` (string).

> **Q2.** `"555"` — string `"5"` repeated 3 times. To get `15`, convert: `int(x) * 3`.

> **Q3.** `int(input("prompt"))` for integer, `float(input("prompt"))` for float.

> **Q4.** `ValueError: invalid literal for int() with base 10: 'twenty'` — the program crashes.

> **Q5.** Users often accidentally add spaces. `.strip()` removes leading/trailing whitespace that would cause comparison failures.

> **Q6.** `or "Yes"` is always truthy. Should be: `if answer.lower() in ("yes", "y"):`.

> **Q7.** Check with `if not name:` or `if name == "":` — empty strings are falsy.

> **Q8.** `num = float(input("Enter a number: "))`

> **Q9.** Strips whitespace and converts to lowercase — used for **case-insensitive comparison** of user input.

> **Q10.** `value = input("Enter value [42]: ").strip() or "42"` — uses `or` to fall back to default when input is empty.

---

## 🎬 End of Lesson Scene

*It's 4:30 PM. The client intake system is running on a terminal at reception. An associate processes a new walk-in client in under two minutes. Donna watches approvingly.*

**Donna:** "No more paper forms. No more illegible handwriting. I might actually like this."

**Harvey:** *(walking past)* "Tomorrow, you learn to make your programs **think**. `if` statements — the brain of every program."

**Mike:** "Today your programs can *hear*. Tomorrow they'll *decide*."

**Louis:** *(from his office)* "I want input validation on EVERYTHING! If someone types 'banana' where a number should go, your program should NOT crash!"

*You head for the elevator. Six lessons in. Your programs can calculate, format, store, protect, organize, and now listen. Next: they learn to think.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | `input()` basics — prompt, pause, return | ✅ |
| 2 | `input()` always returns `str` | ✅ |
| 3 | Type conversion — `int(input())`, `float(input())` | ✅ |
| 4 | Input validation — `isdigit()`, try-except, loops | ✅ |
| 5 | String cleanup — `.strip()`, `.lower()`, `.title()` on input | ✅ |
| 6 | Multiple inputs — `split()`, `map()` | ✅ |
| 7 | Menu systems and choices | ✅ |
| 8 | Default values with `or` | ✅ |
| 9 | Empty input handling | ✅ |
| 10 | Building interactive programs | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| `int()`, `float()`, `str()` (Lesson 1) | Essential for converting input |
| `.strip()`, `.lower()`, `.title()` (Lesson 2) | Cleaning user input |
| `.split()` (Lesson 2) | Parsing multi-value input |
| Lists (Lesson 3) | Collecting multiple inputs |
| Dictionaries (Lesson 5) | Menu lookups, storing intake records |
| f-strings (Lesson 2) | Formatting output with user data |

---

> **Next Lesson →** Level 1, Lesson 7: *Use `if` Statements in Python*  
> *Harvey needs the system to make decisions — which cases are urgent, which clients are profitable, which associates need attention…*
