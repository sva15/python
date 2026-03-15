# 🏛️ Level 1 – Exploring | Lesson 10: Use While Loops in Python

## *"The Night Watch"*

> **O'Reilly Skill Objective:** *Control program flow with while loops.*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, `input()`, `if`/`elif`/`else`, boolean operators, `for` loops, `break`, `continue`, `range()`, `enumerate()`, `zip()`

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

*It's 11:30 PM. You should have gone home three hours ago, but Harvey is still in his office, staring at his phone.*

**Harvey:** "We have a problem. The e-filing system for the Hessington trial is down. The court deadline is midnight. Our filing needs to go through, and the system needs to keep trying until it accepts the submission."

**You:** "I can write a script that submits the filing—"

**Harvey:** "You already know how to do something once. But this system is unreliable. It might fail three times, five times, twenty times. I need the script to **keep trying** — over and over — until it either succeeds or we run out of time."

*Mike, who's also still here because he has no life outside the firm, walks over.*

**Mike:** "That's a `while` loop. A `for` loop says 'do this for each item in a collection.' A `while` loop says 'keep doing this **as long as** a condition is true.' It doesn't need a collection. It doesn't need to know how many times. It just keeps going."

**Harvey:** "Think of it like a retry mechanism. Keep calling the server. If it fails, wait, then try again. If it succeeds, stop. If we hit the deadline, stop and alert me."

**Mike:** "The key difference from `for` loops: `while` loops don't know when they'll end. They run until a condition changes. That's powerful — and dangerous, if you're not careful."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Let me draw the line between the two loop types."

| Feature | `for` Loop | `while` Loop |
|---------|-----------|-------------|
| **Purpose** | Iterate over a known collection | Repeat until a condition changes |
| **Iterations** | Known in advance | Unknown — could be 0, 1, or infinite |
| **Driven by** | The iterable (list, range, dict) | A boolean condition |
| **Best for** | "Process each item" | "Keep going until done" |
| **Risk** | None (always terminates) | **Infinite loop** if condition never becomes False |

**Harvey:** "Use `for` when you know what you're iterating over. Use `while` when you don't know how many iterations you need — you only know when to **stop**."

**Harvey:** "Examples of `while` loop scenarios:"
- Keep asking for input until the user types "quit"
- Keep retrying an API call until it succeeds
- Keep running a game until the player loses
- Keep monitoring a server until CPU drops below 80%
- Keep reading data until you reach the end of the file

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "While loops are simpler than for loops in syntax — but harder to get right."

---

### 3.1 Basic `while` Loop

```python
# Count from 1 to 5
count = 1
while count <= 5:
    print(count)
    count += 1       # ← CRITICAL: without this, infinite loop!

print("Done!")
```

**Output:**
```
1
2
3
4
5
Done!
```

**Mike:** "The anatomy:"

```python
count = 1                # ← INITIALIZATION: set up the loop variable

while count <= 5:        # ← CONDITION: checked BEFORE each iteration
    print(count)         # ← BODY: runs if condition is True
    count += 1           # ← UPDATE: changes the variable so condition eventually becomes False

print("Done!")           # ← Runs AFTER the loop ends
```

**The three essential parts:**
1. **Initialize** — set up the variable(s) before the loop
2. **Condition** — checked before each iteration (`True` → run body, `False` → exit)
3. **Update** — modify the variable inside the loop so the condition eventually becomes `False`

> ⚠️ **If you forget the update step, the condition stays True forever → infinite loop!**

---

### 3.2 `while` vs `for` — When to Use Which

```python
# TASK: Print numbers 1 to 5

# for loop — when you know the range
for i in range(1, 6):
    print(i)

# while loop — when you're tracking a condition
count = 1
while count <= 5:
    print(count)
    count += 1
```

**Mike:** "Both work, but `for` is cleaner when you know the iteration count. `while` shines when you DON'T:"

```python
# This CAN'T be easily done with a for loop:
import random

# Roll a die until you get a 6
rolls = 0
while True:
    roll = random.randint(1, 6)
    rolls += 1
    print(f"Roll {rolls}: {roll}")
    if roll == 6:
        print(f"Got a 6 after {rolls} rolls!")
        break
```

---

### 3.3 Input Validation Loop ⭐

**Mike:** "This is the single most common use of `while` loops — keep asking until the user gives valid input."

```python
# Keep asking until we get a valid number
while True:
    age_str = input("Enter your age: ").strip()
    if age_str.isdigit() and 0 < int(age_str) < 150:
        age = int(age_str)
        break
    print("❌ Please enter a valid age (1-149)")

print(f"✅ Age set to {age}")
```

**Mike:** "This pattern is called a **sentinel loop** or **input validation loop**. You'll use it constantly:"

```python
# Generic validation pattern
while True:
    user_input = input("prompt: ")
    
    if is_valid(user_input):     # Your validation logic
        processed = transform(user_input)
        break
    
    print("Invalid. Try again.")
```

---

### 3.4 Menu-Driven Program

```python
# An interactive menu that keeps running until the user quits
while True:
    print("\n" + "=" * 40)
    print("   PSL CASE MANAGEMENT")
    print("=" * 40)
    print("  1. View clients")
    print("  2. Add client")
    print("  3. Search")
    print("  4. Quit")
    print("=" * 40)
    
    choice = input("  Select (1-4): ").strip()
    
    if choice == "1":
        print("  → Loading client list...")
    elif choice == "2":
        name = input("  Client name: ").strip().title()
        print(f"  → Added: {name}")
    elif choice == "3":
        term = input("  Search term: ").strip()
        print(f"  → Searching for '{term}'...")
    elif choice == "4":
        print("  Goodbye!")
        break
    else:
        print("  ❌ Invalid option. Try 1-4.")
```

**Mike:** "This is how CLI tools work — `while True` with `break` on the exit command."

---

### 3.5 Counter and Accumulator Patterns

```python
# Accumulate values until user types 'done'
print("Enter expenses (type 'done' to finish):")

total = 0
count = 0

while True:
    entry = input(f"  Expense #{count + 1}: $").strip()
    
    if entry.lower() == "done":
        break
    
    try:
        amount = float(entry)
        total += amount
        count += 1
        print(f"  Running total: ${total:,.2f}")
    except ValueError:
        print("  ❌ Not a valid number")

print(f"\n📊 Summary:")
print(f"  Entries: {count}")
print(f"  Total:   ${total:,.2f}")
if count > 0:
    print(f"  Average: ${total / count:,.2f}")
```

---

### 3.6 `while` with `else`

**Mike:** "Like `for-else`, `while-else` runs the `else` block only if the loop exits normally (without `break`)."

```python
# Retry with a maximum number of attempts
import random

max_attempts = 5
attempt = 0

while attempt < max_attempts:
    attempt += 1
    result = random.choice(["fail", "fail", "fail", "success"])
    print(f"  Attempt {attempt}: {result}")
    
    if result == "success":
        print("  ✅ Operation succeeded!")
        break
else:
    # Runs only if we exhausted all attempts WITHOUT breaking
    print(f"  ❌ Failed after {max_attempts} attempts")
```

---

### 3.7 Countdown and Timer Patterns

```python
# Simple countdown
seconds = 5
print("Filing deadline countdown:")

while seconds > 0:
    print(f"  {seconds}...")
    seconds -= 1

print("  ⏰ DEADLINE! File must be submitted!")
```

```python
# Password attempts with lockout
max_attempts = 3
attempts_left = max_attempts
correct_password = "pearsonspecter"

while attempts_left > 0:
    password = input(f"  Password ({attempts_left} attempts left): ")
    
    if password == correct_password:
        print("  🔓 Access granted!")
        break
    else:
        attempts_left -= 1
        if attempts_left > 0:
            print("  ❌ Incorrect. Try again.")
else:
    print("  🔒 Account locked! Contact administrator.")
```

---

### 3.8 Nested While Loops

```python
# Outer loop: multiple rounds
# Inner loop: multiple guesses per round
import random

rounds = 3
current_round = 1

while current_round <= rounds:
    target = random.randint(1, 10)
    guesses = 0
    
    print(f"\n--- Round {current_round} of {rounds} ---")
    print("Guess a number between 1 and 10:")
    
    while True:
        guess_str = input("  Your guess: ").strip()
        if not guess_str.isdigit():
            print("  Enter a number!")
            continue
        
        guess = int(guess_str)
        guesses += 1
        
        if guess == target:
            print(f"  ✅ Correct! Got it in {guesses} guesses")
            break
        elif guess < target:
            print("  📈 Higher!")
        else:
            print("  📉 Lower!")
    
    current_round += 1

print("\n🏁 Game over!")
```

---

### 3.9 Converting Between `for` and `while`

**Mike:** "Any `for` loop can be rewritten as a `while` loop (but not always vice versa):"

```python
# FOR loop
for i in range(5):
    print(i)

# Equivalent WHILE loop
i = 0
while i < 5:
    print(i)
    i += 1
```

```python
# FOR loop with a list
clients = ["Wayne", "Stark", "Oscorp"]
for client in clients:
    print(client)

# Equivalent WHILE loop (ugly — don't do this)
i = 0
while i < len(clients):
    print(clients[i])
    i += 1
```

**Mike:** "Rule: If you CAN use a `for` loop, DO. Use `while` only when `for` doesn't fit — unknown iterations, continuous monitoring, input validation."

---

### 3.10 Solving Harvey's Problem — E-Filing Retry System

```python
# ============================================
# Pearson Specter Litt — E-Filing Retry System
# ============================================
import random
import time

print("=" * 60)
print(f"{'E-FILING RETRY SYSTEM':^60}")
print(f"{'Hessington Oil — Case #PSL-2026-001':^60}")
print("=" * 60)
print()

# Configuration
max_retries = 10
retry_delay = 1        # seconds between retries
deadline_seconds = 30  # total time limit

# State
attempt = 0
submitted = False
start_time = time.time()

print("📤 Initiating filing submission...\n")

while attempt < max_retries:
    elapsed = time.time() - start_time
    remaining = deadline_seconds - elapsed
    
    # Check deadline
    if remaining <= 0:
        print(f"\n⏰ DEADLINE EXCEEDED after {elapsed:.1f} seconds!")
        break
    
    attempt += 1
    
    # Simulate server response (30% success rate)
    server_response = random.choice(["timeout", "timeout", "busy", "error", 
                                      "timeout", "busy", "success", 
                                      "timeout", "error", "success"])
    
    # Status display
    status_bar = "█" * attempt + "░" * (max_retries - attempt)
    print(f"  Attempt {attempt:>2}/{max_retries} [{status_bar}] "
          f"({remaining:.0f}s remaining)")
    
    if server_response == "success":
        submitted = True
        print(f"\n  ✅ FILING ACCEPTED by court e-filing system!")
        print(f"  📋 Confirmation #: PSL-{random.randint(100000, 999999)}")
        print(f"  ⏱️  Time elapsed: {elapsed:.1f} seconds")
        print(f"  🔄 Attempts: {attempt}")
        break
    else:
        error_messages = {
            "timeout": "⏳ Server timeout — no response",
            "busy": "🔶 Server busy — too many connections",
            "error": "🔴 Server error — internal failure"
        }
        print(f"       → {error_messages[server_response]}")
        
        if attempt < max_retries:
            # Exponential backoff (wait longer after each failure)
            wait_time = min(retry_delay * (1.5 ** (attempt - 1)), 5)
            print(f"       → Retrying in {wait_time:.1f}s...")
            time.sleep(wait_time)

else:
    # Loop completed without break — all retries exhausted
    print(f"\n  ❌ ALL {max_retries} ATTEMPTS FAILED!")

# Final status
print()
print("=" * 60)
if submitted:
    print(f"{'✅ FILING SUCCESSFULLY SUBMITTED':^60}")
else:
    print(f"{'❌ FILING FAILED — CONTACT COURT CLERK':^60}")
    print(f"{'⚠️  ALERT SENT TO HARVEY SPECTER':^60}")
print("=" * 60)
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

**Louis:** "While loops are the most **dangerous** construct in Python. Here's why."

---

### Pitfall 1: The Infinite Loop 🔥🔥🔥

**Louis:** "The #1 danger. If the condition NEVER becomes False, the loop runs forever."

```python
# ❌ INFINITE LOOP — count never changes!
count = 1
while count <= 5:
    print(count)
    # Forgot: count += 1  ← THIS LINE IS MISSING!

# ❌ INFINITE LOOP — wrong variable updated
x = 10
while x > 0:
    print(x)
    y -= 1     # Updating 'y' instead of 'x'!

# ❌ INFINITE LOOP — condition can never be False
while True:
    print("Help!")
    # No break statement!
```

**Louis:** "If your program freezes, it's probably an infinite loop. Press **Ctrl+C** to kill it."

**Louis:** "Defense strategies:"
```python
# Strategy 1: Always ensure the update moves toward termination
count = 0
while count < 10:
    print(count)
    count += 1    # ✅ Guaranteed to reach 10

# Strategy 2: Add a safety limit
max_iterations = 10000
iteration = 0
while some_condition and iteration < max_iterations:
    # ... work ...
    iteration += 1
if iteration >= max_iterations:
    print("⚠️ Safety limit reached!")

# Strategy 3: Use 'while True' with an explicit break
while True:
    result = do_something()
    if result == "done":
        break    # ✅ Clear exit condition
```

---

### Pitfall 2: Off-by-One in While Loops

```python
# How many times does this print?
i = 0
while i < 5:
    print(i)
    i += 1
# Prints: 0, 1, 2, 3, 4 → 5 times

# vs
i = 0
while i <= 5:
    print(i)
    i += 1
# Prints: 0, 1, 2, 3, 4, 5 → 6 times!

# vs
i = 1
while i < 5:
    print(i)
    i += 1
# Prints: 1, 2, 3, 4 → 4 times
```

**Louis:** "The start value AND the condition (`<` vs `<=`) both affect how many iterations you get."

---

### Pitfall 3: `while True` Without `break`

```python
# ❌ This NEVER stops
while True:
    name = input("Name: ")
    print(f"Hello, {name}")
    # Where's the exit?!

# ✅ Always have an exit path
while True:
    name = input("Name (or 'quit'): ")
    if name.lower() == "quit":
        break              # ✅ Exit condition
    print(f"Hello, {name}")
```

---

### Pitfall 4: Condition Checked at the TOP, Not the Bottom

**Louis:** "The condition is checked BEFORE each iteration — including the first:"

```python
# This loop body might NEVER execute!
x = 10
while x < 5:          # 10 < 5 is False → body NEVER runs
    print("Never printed")
    x += 1

print(f"x is still {x}")   # x is still 10
```

**Louis:** "If you need the body to run AT LEAST once, use `while True` + `break`:"
```python
# Guaranteed to run at least once
while True:
    response = input("Enter something: ")
    if response:      # If they typed something
        break
```

---

### Pitfall 5: Floating-Point Loop Variables

```python
# ❌ DANGEROUS — float imprecision may prevent termination
x = 0.0
while x != 1.0:
    x += 0.1
    print(f"{x:.20f}")  # Shows imprecision!
# 0.1 + 0.1 + ... might NEVER exactly equal 1.0!

# ✅ Use comparison instead of equality
x = 0.0
while x < 1.0:        # < instead of !=
    x += 0.1
    print(f"{x:.1f}")
```

---

### Pitfall 6: Using `while` When `for` Is Better

```python
# ❌ Unnecessarily complex — using while for a known collection
clients = ["Wayne", "Stark", "Oscorp"]
i = 0
while i < len(clients):
    print(clients[i])
    i += 1

# ✅ Just use a for loop!
for client in clients:
    print(client)
```

**Louis:** "If you catch yourself writing `i = 0` and `i += 1` around a while loop that iterates over a collection, you want a `for` loop."

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "Two analogies today."

---

### 🔁 Analogy 1: `while` Loop = A Security Guard

**Donna:** "Think of a while loop as a **security guard** at the door:"

```
Guard's instructions:
  "WHILE the building is open, keep checking IDs."
  "IF a threat is detected, lock down immediately (break)."
  "IF someone has pre-clearance, let them through without checking (continue)."
  "When the building closes (condition becomes False), go home."
```

- **The condition** = Is the building still open?
- **The body** = Check the next person's ID
- **break** = Emergency lockdown — stop everything
- **continue** = Skip this person, check the next
- **Infinite loop** = The building NEVER closes — guard works forever 😱

---

### 🔁 Analogy 2: `for` vs `while` — The Key Difference

```
FOR Loop (Assembly Line):
  "Here are 10 boxes. Process each one."
  ✅ You KNOW there are 10 boxes.
  ✅ The line ENDS when all boxes are processed.
  ✅ CANNOT run forever.

WHILE Loop (Security Guard):
  "Keep checking until the building closes."
  ❓ You DON'T KNOW when that will be.
  ⚠️ If the building NEVER closes, you work forever.
  ⚠️ CAN run forever if condition never changes.
```

**Donna:** "Use with care."

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "While loops — where you prove you can handle infinite possibilities."

---

### 🟢 Beginner Exercises

**Exercise 1: Countdown**
```
Ask the user for a starting number.
Count down to 0, then print "🚀 LAUNCH!"
Use a while loop.

Example:
  Start from: 5
  5... 4... 3... 2... 1... 🚀 LAUNCH!
```

**Exercise 2: Even Numbers**
```
Using a while loop, print all even numbers from 2 to 50.
Also calculate their sum.
```

**Exercise 3: Password Gate**
```
Set a secret password (e.g., "pearsonspecter").
Keep asking the user for the password.
- If correct: print "Access Granted" and exit
- If wrong: print "Access Denied. Try again."
- After 3 wrong attempts: print "Account Locked" and exit

Track the number of attempts.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Number Guessing Game**
```
Generate a random number between 1 and 100.
The user keeps guessing until they get it.

After each guess, tell them:
- "Too high!" or "Too low!" or "Correct!"
- How many guesses they've used

When they win, classify their performance:
  1-5 guesses: "Excellent! 🏆"
  6-10 guesses: "Good! 👍"
  11+ guesses: "Keep practicing! 📚"
```

**Exercise 5: Data Collector**
```
Build a data collection loop that:
1. Asks for a student name (empty string to finish)
2. Asks for their score (validate it's 0-100)
3. Stores name and score in a dictionary
4. After finishing, displays:
   - All students and scores
   - Average score
   - Highest and lowest scorer
   - Number of students above average
```

**Exercise 6: Collatz Conjecture**
```
Ask for a positive integer n.
Apply these rules repeatedly:
- If n is even: n = n // 2
- If n is odd: n = 3 * n + 1
Keep going until n reaches 1.

Print each step and count the total steps.
Example: 6 → 3 → 10 → 5 → 16 → 8 → 4 → 2 → 1 (8 steps)

Test with: 6, 27, 100
```

---

### 🔴 Advanced Exercises

**Exercise 7: ATM Simulator**
```
Build an ATM with a balance of $10,000.
Menu: Check Balance, Deposit, Withdraw, Transaction History, Quit

Rules:
- Cannot withdraw more than balance
- Cannot withdraw negative amounts
- Keep a list of all transactions: ("Deposit", $500) or ("Withdraw", $200)
- Show transaction history with running balance
- Print balance after every operation
```

**Exercise 8: Text Adventure Game**
```
Create a simple text adventure:
- Start in a "Lobby"
- Rooms: Lobby, Harvey's Office, Louis's Office, Library, Server Room
- Each room has a description and 2-3 exits
- Items to collect: "security badge" (lobby), "case file" (Harvey's), "encryption key" (server room)
- Win condition: collect all 3 items and return to Lobby
- Quit command available from any room

Use a while loop for the game loop.
Store rooms as a dictionary of dictionaries.
Track inventory as a list.
```

**Exercise 9: Binary Search Implementation**
```
Implement binary search using a while loop:

Given a SORTED list of 100 numbers (1 to 100),
find a target number.

Steps:
1. Set low = 0, high = len(list) - 1
2. While low <= high:
   a. mid = (low + high) // 2
   b. If list[mid] == target: FOUND!
   c. If list[mid] < target: low = mid + 1
   d. If list[mid] > target: high = mid - 1
3. If loop ends without finding: NOT FOUND

Print each step (what low, high, mid are).
Show how many comparisons were needed.
Compare this to linear search (for loop through every element).
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "While loop bugs — the most terrifying kind."

### Buggy Code 🐛

```python
# Bug Hunt: Server Monitor
# Find and fix ALL the bugs!

# Bug 1: Infinite loop
count = 10
while count > 0:
    print(f"Countdown: {count}")
    count + 1              # Will this ever end?

# Bug 2: Logic error
total = 0
num = 1
while num <= 10:
    total += num
    num += 2              # What does this actually sum?
print(f"Sum 1-10: {total}")

# Bug 3: Off-by-one
i = 1
while i < 5:
    print(f"Step {i}")
    i += 1
# Expected: Steps 1 through 5

# Bug 4: Wrong exit condition
password = ""
while password != "secret":
    password = input("Password: ")
    if password != "secret":
        print("Wrong!")
print("Welcome!")
# What if user types "Secret" (capitalized)?

# Bug 5: Unreachable code
while True:
    print("Working...")
    break
    print("This never runs")  # Dead code!

# Bug 6: Variable not updated correctly
items = ["a", "b", "c", "d", "e"]
i = 0
while i < len(items):
    if items[i] == "c":
        items.remove("c")
    i += 1
print(items)   # Did we remove "c"? Did we skip anything?
```

---

### Fixed Code ✅

```python
# Fixed: Server Monitor

# Fix 1: count + 1 doesn't change count! Need count -= 1
count = 10
while count > 0:
    print(f"Countdown: {count}")
    count -= 1            # ✅ Decrement (also: was incrementing wrong direction!)

# Fix 2: num += 2 only sums odd numbers (1,3,5,7,9)
# To sum ALL numbers 1-10, use num += 1
total = 0
num = 1
while num <= 10:
    total += num
    num += 1              # ✅ Increment by 1
print(f"Sum 1-10: {total}")  # 55

# Fix 3: Use <= instead of < to include 5
i = 1
while i <= 5:             # ✅ Include 5
    print(f"Step {i}")
    i += 1

# Fix 4: Handle case sensitivity
password = ""
while True:
    password = input("Password: ")
    if password.lower() == "secret":  # ✅ Case-insensitive
        break
    print("Wrong!")
print("Welcome!")

# Fix 5: Code after break is dead — move or remove it
while True:
    print("Working...")
    print("Almost done...")  # ✅ Before break
    break

# Fix 6: Removing while iterating causes index skip
# Use a different approach
items = ["a", "b", "c", "d", "e"]
items = [item for item in items if item != "c"]  # ✅ Build new list
print(items)   # ['a', 'b', 'd', 'e']
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | `count + 1` doesn't modify count; also going wrong direction | `+=` vs `+`; decrement |
| 2 | `+= 2` skips even numbers | Step size matters |
| 3 | `< 5` excludes 5 | `<` vs `<=` |
| 4 | Case-sensitive password comparison | `.lower()` normalization |
| 5 | Code after `break` never executes | Dead code |
| 6 | Removing items shifts indices during iteration | Build new list |

---

## 8. 🌍 Real-World Application

**Harvey:** "While loops run the systems that never sleep."

### Where `while` Loops Are Used in Production:

| Domain | While Loop Use Case |
|--------|-------------------|
| **Servers** | `while True:` — accept and handle connections forever |
| **Game Loops** | `while game_running:` — update, render, repeat |
| **Monitoring** | `while True:` — check CPU, memory, disk every 30 seconds |
| **Retry Logic** | `while attempts < max:` — retry failed API calls |
| **Stream Processing** | `while data_available:` — process incoming data packets |
| **CLI Tools** | `while True:` — REPL loops (Read-Eval-Print Loop) |
| **State Machines** | `while state != 'done':` — process events until completion |

### Real Production Example: Server Health Monitor

```python
import random
import time

def check_server_health():
    """Simulate checking server metrics."""
    return {
        "cpu": random.randint(10, 100),
        "memory": random.randint(30, 95),
        "disk": random.randint(40, 98),
        "status": random.choice(["healthy", "healthy", "healthy", "degraded", "error"])
    }

print("🖥️ PSL Server Monitor — Starting...")
print("=" * 55)

check_count = 0
alerts = []

while check_count < 5:    # In production: while True
    check_count += 1
    metrics = check_server_health()
    
    # Evaluate health
    issues = []
    if metrics["cpu"] > 80:
        issues.append(f"⚠️ CPU HIGH: {metrics['cpu']}%")
    if metrics["memory"] > 90:
        issues.append(f"⚠️ MEMORY CRITICAL: {metrics['memory']}%")
    if metrics["disk"] > 95:
        issues.append(f"🔴 DISK FULL: {metrics['disk']}%")
    if metrics["status"] == "error":
        issues.append("🔴 SERVER ERROR detected")
    
    # Display
    status_icon = "✅" if not issues else "⚠️"
    print(f"\n  [{check_count}] {status_icon} CPU={metrics['cpu']:>3}% | "
          f"MEM={metrics['memory']:>3}% | DISK={metrics['disk']:>3}% | "
          f"Status={metrics['status']}")
    
    if issues:
        for issue in issues:
            print(f"       {issue}")
            alerts.append((check_count, issue))
    
    time.sleep(0.5)    # In production: time.sleep(30)

# Summary
print("\n" + "=" * 55)
print(f"  Checks completed: {check_count}")
print(f"  Alerts triggered: {len(alerts)}")
if alerts:
    print("  Alert log:")
    for check, alert in alerts:
        print(f"    Check #{check}: {alert}")
print("=" * 55)
```

### Real Production Example: Retry with Exponential Backoff

```python
import random
import time

def api_call():
    """Simulate an unreliable API — 30% success rate."""
    return random.random() < 0.3

def retry_with_backoff(func, max_retries=5, base_delay=1.0):
    """Retry a function with exponential backoff."""
    attempt = 0
    
    while attempt < max_retries:
        attempt += 1
        print(f"  Attempt {attempt}/{max_retries}...", end=" ")
        
        if func():
            print("✅ SUCCESS")
            return True
        
        print("❌ FAILED")
        
        if attempt < max_retries:
            delay = base_delay * (2 ** (attempt - 1))  # 1, 2, 4, 8, 16...
            delay = min(delay, 30)  # Cap at 30 seconds
            print(f"    Waiting {delay:.1f}s before retry...")
            time.sleep(min(delay, 1))  # Shortened for demo
    
    print(f"  ❌ All {max_retries} attempts failed!")
    return False

# Usage
print("📡 Calling external API...")
success = retry_with_backoff(api_call)
print(f"\nFinal result: {'Success' if success else 'Failure'}")
```

---

## 9. ✅ Knowledge Check

**Harvey:** "Final quiz."

---

**Q1.** What are the three essential parts of a `while` loop? What happens if you forget one?

**Q2.** When should you use a `while` loop instead of a `for` loop?

**Q3.** What is an infinite loop? How do you prevent one? How do you stop one?

**Q4.** What is the output?
```python
x = 10
while x > 0:
    x -= 3
    print(x)
```

**Q5.** Write a `while` loop that keeps asking for a number > 0 and valid, then prints it.

**Q6.** What is the difference between `while x:` and `while x != 0:`? Are there cases where they differ?

**Q7.** What does `while-else` do? When does the `else` block execute?

**Q8.** Why is `while True` with `break` useful? Give an example.

**Q9.** Can a `while` loop's body execute zero times? When?

**Q10.** What is exponential backoff? Why is it used in retry logic?

---

### 📋 Answer Key

> **Q1.** (1) Initialize variable, (2) Condition, (3) Update variable. Missing initialization → NameError. Missing condition → SyntaxError. Missing update → infinite loop!

> **Q2.** When you don't know how many iterations you need — input validation, retry logic, monitoring, games, reading until end-of-data.

> **Q3.** A loop whose condition never becomes False. Prevent: ensure the update moves toward termination. Stop: `Ctrl+C`.

> **Q4.** `7`, `4`, `1`, `-2` — the loop subtracts 3 each time. When x becomes -2, the condition `x > 0` is False, so the loop stops.

> **Q5.**
```python
while True:
    val = input("Enter a positive number: ")
    if val.isdigit() and int(val) > 0:
        print(f"You entered: {int(val)}")
        break
    print("Invalid!")
```

> **Q6.** `while x:` checks truthiness — `0`, `None`, `""`, `[]` are all falsy. `while x != 0:` only checks numeric zero. They differ if x could be `None`, `""`, or `[]`.

> **Q7.** The `else` block runs only if the loop exits normally (condition becomes False). It does NOT run if `break` is used.

> **Q8.** Useful when the exit condition is complex or needs to be checked in the middle of the loop body. Example: input validation — process input, check validity, break if valid.

> **Q9.** Yes! If the condition is already False before the first iteration: `x = 0; while x > 5: print(x)` — body never runs.

> **Q10.** Waiting progressively longer between retries (1s, 2s, 4s, 8s…). Prevents overwhelming a struggling server with rapid retries.

---

## 🎬 End of Lesson Scene

*It's 11:58 PM. The e-filing script is running. Attempt 1: timeout. Attempt 2: server busy. Attempt 3: timeout. Attempt 4—*

```
  ✅ FILING ACCEPTED by court e-filing system!
  📋 Confirmation #: PSL-847291
  ⏱️  Time elapsed: 12.3 seconds
  🔄 Attempts: 4
```

*Harvey looks at his watch. 11:58:47 PM. With 73 seconds to spare.*

**Harvey:** *(picking up his jacket)* "Good work. Go home."

**Mike:** "Two more lessons and we finish Level 1. Next up: **functions**. We'll take all the code you've been writing — the loops, the conditions, the calculations — and package them into reusable, named blocks."

*You head for the elevator at midnight. Ten days in. Your programs can store data, make decisions, repeat actions, retry operations, and run indefinitely. You're not writing scripts anymore. You're building systems.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | `while` loop basics — condition, body, update | ✅ |
| 2 | `while True` + `break` pattern | ✅ |
| 3 | Input validation loops | ✅ |
| 4 | Menu-driven programs | ✅ |
| 5 | Counter and accumulator with while | ✅ |
| 6 | `while-else` | ✅ |
| 7 | Countdown and timer patterns | ✅ |
| 8 | Nested while loops | ✅ |
| 9 | `for` vs `while` — when to use each | ✅ |
| 10 | Retry logic with exponential backoff | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| `for` loops (Lesson 9) | Comparison: for vs while |
| `break`, `continue` (Lesson 9) | Same control in while loops |
| `input()` validation (Lesson 6) | Input validation loops |
| Boolean operators (Lesson 8) | Complex while conditions |
| `if` statements (Lesson 7) | Decision-making inside loops |
| Dictionaries (Lesson 5) | State tracking, menu systems |
| Truthiness (Lesson 7) | `while x:` relies on truthiness |

---

> **Next Lesson →** Level 1, Lesson 11: *Write Functions with Conditionals or Loops*  
> *Harvey wants reusable tools. "Stop copying code. Build functions I can call with a single line."*
