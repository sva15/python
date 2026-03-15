# 🏛️ Level 1 – Exploring | Lesson 2: Working with Strings and Output

## *"The Case of the Missing Contract Clause"*

> **O'Reilly Skill Objective:** *Write Python scripts that construct dynamic text using f-strings, manipulate text with basic string methods, and display the formatted results to the console using the print() function.*

> **Previously Learned:** Variables, integers, floats, arithmetic operators, type conversion, `print()`, `str()`, `type()`

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

*It's 8:12 AM. You walk out of the elevator and Donna intercepts you before you even reach your desk.*

**Donna:** "Don't sit down. Harvey's already in his office, and he's *not* in a good mood."

**You:** "What happened?"

**Donna:** "The merger contract for Dalton Industries. 47 pages. Someone on the associate team sent the wrong version to opposing counsel last night. Names misspelled, dates wrong, exhibit references off by one. Jessica is livid."

*You walk into Harvey's office. He's on the phone. He holds up one finger — wait.*

**Harvey:** *(into the phone)* "Tell them we'll have the corrected version by noon. Yes. *Noon.*" *(hangs up)*

**Harvey:** "Sit. Here's the problem. Our contract templates are filled out manually. Associates copy-paste client names, dates, dollar amounts — and they get it wrong. Every. Single. Time."

*He pulls up a template on his screen:*

```
MERGER AGREEMENT

This agreement is entered into on [DATE] between [PARTY_A] 
(hereinafter referred to as "Buyer") and [PARTY_B] 
(hereinafter referred to as "Seller"), for the total 
consideration of $[AMOUNT].

Case Reference: [CASE_ID]
```

**Harvey:** "I want you to build a Python script that generates these documents automatically. The associate inputs the client data, and the script produces a perfect contract clause every time. No typos. No mistakes."

*He looks at you.*

**Harvey:** "You learned numbers yesterday. Today you learn **strings** — because in law *and* in code, **words matter.**"

*He picks up the phone again.*

**Harvey:** "Mike. Get in here."

---

## 2. 🎯 Concept Introduction — Harvey Specter

*Harvey stands by the window.*

**Harvey:** "Let me tell you why strings are arguably MORE important than numbers."

**Harvey:** "In the real world, most of the data you'll ever work with is **text**. Names, addresses, emails, contracts, log files, API responses, error messages, database records — it's all strings."

**Harvey:** "A program that can crunch numbers but can't handle text is like a lawyer who can do the math on a settlement but can't write the agreement. Useless."

**Harvey:** "Python has the most powerful string handling of any mainstream language. It's like having a legal assistant who can draft, edit, format, search, and rewrite any document in seconds. That's what you're learning today."

**Harvey:** "Three things I need you to master:"
1. **Creating strings** — how to represent text in Python
2. **Manipulating strings** — how to slice, search, replace, and transform text
3. **Formatting output** — how to produce clean, professional, dynamic text with `print()` and f-strings

**Harvey:** "Get these right, and you'll never produce a badly formatted document again."

---

## 3. 🔧 Technical Breakdown — Mike Ross

*Mike walks in with two coffees, hands you one.*

**Mike:** "Alright, lesson two. Strings are where Python really shines. Let's build from the ground up."

---

### 3.1 Creating Strings

**Mike:** "A **string** is a sequence of characters — letters, numbers, symbols, spaces — enclosed in quotes."

```python
# Single quotes
firm_name = 'Pearson Specter Litt'

# Double quotes (identical behavior)
client_name = "Wayne Enterprises"

# Both are exactly equivalent
print(type(firm_name))    # <class 'str'>
print(type(client_name))  # <class 'str'>
```

**Mike:** "Single or double quotes — Python doesn't care. Pick one style and be consistent. The convention at this firm? **Double quotes for strings with text, single quotes for single characters or identifiers.**"

#### When to Use Which?

```python
# Use double quotes when the string contains a single quote (apostrophe)
message = "It's a binding agreement"

# Use single quotes when the string contains double quotes
html_tag = '<div class="contract">'

# Or escape the quote with a backslash
message = 'It\'s a binding agreement'
html_tag = "<div class=\"contract\">"
```

#### Triple Quotes — Multi-line Strings

```python
# Triple quotes for multi-line strings
contract_clause = """
MERGER AGREEMENT

This agreement is entered into on March 13, 2026 
between Pearson Specter Litt (hereinafter referred 
to as "Counsel") and the undersigned parties.
"""
print(contract_clause)
```

**Mike:** "Triple quotes preserve **line breaks** and **indentation**. Perfect for contract templates, SQL queries, and any multi-line text."

#### Empty Strings

```python
# An empty string — valid and useful
notes = ""
print(len(notes))  # 0
print(type(notes))  # <class 'str'>
print(bool(notes))  # False (empty strings are "falsy")
```

---

### 3.2 String Length and Indexing

**Mike:** "Every character in a string has a **position** — called an **index**. Python uses **zero-based indexing**, meaning the FIRST character is at position 0."

```python
firm = "SPECTER"
#       S P E C T E R
#       0 1 2 3 4 5 6    ← positive index
#      -7-6-5-4-3-2-1    ← negative index
```

```python
firm = "SPECTER"

# Length
print(len(firm))      # 7

# Accessing individual characters
print(firm[0])        # S  (first character)
print(firm[1])        # P  (second character)
print(firm[6])        # R  (last character)
print(firm[-1])       # R  (last character — using negative indexing)
print(firm[-2])       # E  (second to last)
```

> 💡 **Connection to Lesson 1:** Remember `type()`? You can use it on individual characters too — `type(firm[0])` returns `<class 'str'>`. In Python, there's no separate "character" type. A single character is just a string of length 1.

---

### 3.3 String Slicing

**Mike:** "Slicing lets you extract a **portion** of a string. The syntax is `string[start:stop:step]`."

```python
case_id = "PSL-2026-MERGER-0042"

# Basic slicing: [start:stop] — stop is EXCLUSIVE
print(case_id[0:3])       # PSL
print(case_id[4:8])       # 2026
print(case_id[9:15])      # MERGER

# Omitting start or stop
print(case_id[:3])        # PSL (from beginning to index 3)
print(case_id[9:])        # MERGER-0042 (from index 9 to end)

# Negative slicing
print(case_id[-4:])       # 0042 (last 4 characters)

# Step — every Nth character
alphabet = "ABCDEFGHIJ"
print(alphabet[::2])      # ACEGI (every 2nd character)

# Reverse a string!
print(case_id[::-1])      # 2400-REGREM-6202-LSP
```

#### Slicing Cheat Sheet

| Syntax | Meaning | Example on `"SPECTER"` | Result |
|--------|---------|----------------------|--------|
| `s[2:5]` | Index 2 to 4 (stop exclusive) | `"SPECTER"[2:5]` | `"ECT"` |
| `s[:3]` | Start to index 2 | `"SPECTER"[:3]` | `"SPE"` |
| `s[3:]` | Index 3 to end | `"SPECTER"[3:]` | `"CTER"` |
| `s[-3:]` | Last 3 characters | `"SPECTER"[-3:]` | `"TER"` |
| `s[::2]` | Every 2nd character | `"SPECTER"[::2]` | `"SETR"` |
| `s[::-1]` | Reversed string | `"SPECTER"[::-1]` | `"RETCEPS"` |

---

### 3.4 Strings Are Immutable

**Mike:** "This is **critical**: strings in Python are **immutable**. You **cannot** change a character inside a string."

```python
name = "Harvey"

# This will CRASH
name[0] = "h"  # TypeError: 'str' object does not support item assignment

# Instead, create a NEW string
name = "h" + name[1:]
print(name)  # "harvey"
```

**Mike:** "Every string operation you learn today creates a **new** string. The original is never modified. This connects to **mutability** — a concept Louis will drill into you in Level 2."

---

### 3.5 String Concatenation and Repetition

**Mike:** "You already saw concatenation in Lesson 1 — joining strings with `+`."

```python
# Concatenation with +
first = "Harvey"
last = "Specter"
full_name = first + " " + last
print(full_name)  # Harvey Specter

# Repetition with *
separator = "=" * 40
print(separator)  # ========================================

# Building a header
title = "CLIENT REPORT"
print(separator)
print(title.center(40))  # Centers text within 40 characters
print(separator)
```

**Output:**
```
========================================
             CLIENT REPORT              
========================================
```

> ⚠️ **You CANNOT concatenate strings and numbers directly:**
```python
age = 35
# print("Age: " + age)     # TypeError!
print("Age: " + str(age))   # ✅ Convert number to string first
```

**Mike:** "But there's a MUCH better way to combine text and data. Meet **f-strings**."

---

### 3.6 f-Strings (Formatted String Literals) ⭐

**Mike:** "f-strings are Python's **most powerful** string formatting tool. Introduced in Python 3.6, they're the standard everyone uses."

```python
# Prefix the string with 'f' and put expressions inside {curly braces}
client = "Wayne Enterprises"
amount = 2500000.50
date = "March 13, 2026"

# Basic f-string
message = f"Contract for {client} — Total: ${amount}"
print(message)
# Output: Contract for Wayne Enterprises — Total: $2500000.5
```

#### Expressions Inside f-strings

```python
hours = 85
rate = 375.00

# You can do MATH inside f-strings
print(f"Total billing: ${hours * rate}")
# Output: Total billing: $31875.0

# You can call FUNCTIONS inside f-strings
name = "harvey specter"
print(f"Attorney: {name.title()}")
# Output: Attorney: Harvey Specter

# You can use CONDITIONALS (ternary) inside f-strings
profit = -5000
print(f"Status: {'Profit' if profit > 0 else 'Loss'}")
# Output: Status: Loss
```

#### Number Formatting in f-strings

```python
revenue = 2500000.5
tax_rate = 0.085

# Comma separator for thousands
print(f"Revenue: ${revenue:,.2f}")
# Output: Revenue: $2,500,000.50

# Fixed decimal places
print(f"Tax Rate: {tax_rate:.2%}")
# Output: Tax Rate: 8.50%

# Padding and alignment
for item in ["Revenue", "Costs", "Profit"]:
    print(f"{item:<15} {'$1,000':>10}")
```

**Output:**
```
Revenue          $1,000
Costs            $1,000
Profit           $1,000
```

#### f-string Format Specifiers Cheat Sheet

| Specifier | Meaning | Example | Result |
|-----------|---------|---------|--------|
| `:.2f` | 2 decimal places | `f"{3.14159:.2f}"` | `"3.14"` |
| `:,.2f` | Commas + 2 decimals | `f"{1234567.89:,.2f}"` | `"1,234,567.89"` |
| `:.2%` | Percentage (×100) | `f"{0.085:.2%}"` | `"8.50%"` |
| `:>10` | Right-align, width 10 | `f"{'hi':>10}"` | `"        hi"` |
| `:<10` | Left-align, width 10 | `f"{'hi':<10}"` | `"hi        "` |
| `:^10` | Center, width 10 | `f"{'hi':^10}"` | `"    hi    "` |
| `:010` | Zero-pad to width 10 | `f"{42:010}"` | `"0000000042"` |
| `:+` | Show sign always | `f"{42:+}"` | `"+42"` |
| `:#x` | Hex with prefix | `f"{255:#x}"` | `"0xff"` |

---

### 3.7 The `print()` Function — Deep Dive

**Mike:** "You've been using `print()` since day one. Let me show you what it can *really* do."

```python
# Basic print
print("Hello, Pearson Specter Litt")

# Multiple arguments — separated by space by default
print("Harvey", "Specter", "Attorney at Law")
# Output: Harvey Specter Attorney at Law

# Change the separator
print("Harvey", "Specter", sep=" | ")
# Output: Harvey | Specter

print("2026", "03", "13", sep="-")
# Output: 2026-03-13

# Change end character (default is newline \n)
print("Loading", end="")
print("...", end="")
print("Done!")
# Output: Loading...Done!

# Print to stay on same line
print("Processing: ", end="")
print("Complete")
# Output: Processing: Complete
```

#### Escape Characters

```python
# \n — newline
print("Line 1\nLine 2\nLine 3")

# \t — tab
print("Name:\tHarvey Specter")
print("Title:\tSenior Partner")

# \\ — literal backslash
print("File path: C:\\Users\\Documents")

# \" or \' — literal quotes
print("He said, \"I don't lose.\"")

# Raw string — ignores escape characters
print(r"C:\Users\new_folder")  # \n is NOT a newline here
```

**Output:**
```
Line 1
Line 2
Line 3
Name:	Harvey Specter
Title:	Senior Partner
File path: C:\Users\Documents
He said, "I don't lose."
C:\Users\new_folder
```

---

### 3.8 Essential String Methods

**Mike:** "Strings come loaded with **methods** — built-in functions you call on the string itself. Here are the ones you'll use every day."

#### Case Transformation

```python
name = "harvey specter"

print(name.upper())       # HARVEY SPECTER
print(name.lower())       # harvey specter
print(name.title())       # Harvey Specter
print(name.capitalize())  # Harvey specter (only first letter)
print(name.swapcase())    # HARVEY SPECTER

# Check case
print("HELLO".isupper())  # True
print("hello".islower())  # True
print("Hello World".istitle())  # True
```

#### Searching and Finding

```python
contract = "This agreement between Buyer and Seller is binding"

# Check if a substring exists
print("Buyer" in contract)       # True
print("buyer" in contract)       # False (case-sensitive!)
print("buyer" in contract.lower())  # True

# Find the position of a substring
print(contract.find("Buyer"))    # 24 (index where "Buyer" starts)
print(contract.find("Judge"))    # -1 (not found — no error)
print(contract.index("Buyer"))   # 24 (same, but raises ValueError if not found)

# Count occurrences
text = "the case of the century in the court"
print(text.count("the"))         # 3

# Starts/ends with
print(contract.startswith("This"))    # True
print(contract.endswith("binding"))   # True
```

#### Stripping Whitespace

```python
messy_input = "   Harvey Specter   \n"

print(messy_input.strip())    # "Harvey Specter" (both sides)
print(messy_input.lstrip())   # "Harvey Specter   \n" (left only)
print(messy_input.rstrip())   # "   Harvey Specter" (right only)

# Strip specific characters
ref = "...Case #2024..."
print(ref.strip("."))         # "Case #2024"
```

#### Replacing and Splitting

```python
# Replace
clause = "The Buyer shall pay the Seller"
updated = clause.replace("Buyer", "Acquiring Party")
print(updated)  # The Acquiring Party shall pay the Seller

# Replace with a limit
text = "a-b-c-d-e"
print(text.replace("-", " ", 2))  # "a b c-d-e" (only first 2)

# Split — turns a string into a LIST of substrings
csv_data = "Harvey,Mike,Louis,Donna,Rachel"
names = csv_data.split(",")
print(names)     # ['Harvey', 'Mike', 'Louis', 'Donna', 'Rachel']
print(names[0])  # Harvey

# Split by whitespace (default)
sentence = "Python is amazing"
words = sentence.split()
print(words)     # ['Python', 'is', 'amazing']
print(len(words))  # 3

# Join — opposite of split (turns a list into a string)
names = ["Harvey", "Mike", "Louis"]
result = " & ".join(names)
print(result)    # Harvey & Mike & Louis
```

#### Checking Content Type

```python
print("12345".isdigit())      # True — all digits
print("Hello".isalpha())      # True — all letters
print("Hello123".isalnum())   # True — letters AND/OR digits
print("   ".isspace())        # True — all whitespace
```

#### Padding and Alignment

```python
# Center, left-justify, right-justify
title = "CONTRACT"
print(title.center(30, "="))    # ===========CONTRACT===========
print(title.ljust(30, "-"))     # CONTRACT----------------------
print(title.rjust(30, "-"))     # ----------------------CONTRACT

# Zero-fill
case_num = "42"
print(case_num.zfill(6))       # 000042
```

---

### 3.9 String Method Chaining

**Mike:** "Since every string method returns a **new** string, you can **chain** methods together:"

```python
raw_input = "   HARVEY SPECTER, esq.   "

# Clean and format in one line
cleaned = raw_input.strip().title().replace(", Esq.", ", Esq.")
print(cleaned)  # Harvey Specter, Esq.

# Multi-step chain
messy_email = "  Harvey.Specter@PSL.COM  "
clean_email = messy_email.strip().lower()
print(clean_email)  # harvey.specter@psl.com
```

---

### 3.10 Solving Harvey's Problem — Contract Generator

**Mike:** "Alright, let's build that contract generator Harvey asked for."

```python
# ============================================
# Pearson Specter Litt — Contract Generator
# ============================================

# Client Data (in a real app, this comes from user input)
party_a = "Pearson Specter Litt Holdings"
party_b = "Dalton Industries"
agreement_date = "March 13, 2026"
total_amount = 15750000.00
case_reference = "PSL-2026-MRG-0042"
governing_state = "New York"

# Format the dollar amount
formatted_amount = f"${total_amount:,.2f}"

# Build the contract using f-strings
contract = f"""
{'='*60}
               MERGER AND ACQUISITION AGREEMENT
{'='*60}

Case Reference: {case_reference}
Date:           {agreement_date}

This Merger and Acquisition Agreement ("Agreement") is 
entered into on {agreement_date} between:

  BUYER:  {party_a.upper()}
          (hereinafter referred to as "Buyer")

  SELLER: {party_b.upper()}
          (hereinafter referred to as "Seller")

ARTICLE 1 — CONSIDERATION

The Buyer agrees to pay the Seller a total consideration 
of {formatted_amount} (USD) for the complete acquisition 
of all assets, intellectual property, and operational 
rights of {party_b}.

ARTICLE 2 — GOVERNING LAW

This Agreement shall be governed by and construed in 
accordance with the laws of the State of {governing_state}.

ARTICLE 3 — SIGNATURES

Buyer Representative:  ________________________
                       {party_a}

Seller Representative: ________________________
                       {party_b}

{'='*60}
Generated by PSL Automation System
Case: {case_reference} | Date: {agreement_date}
{'='*60}
"""

print(contract)
```

**Expected Output:**
```
============================================================
               MERGER AND ACQUISITION AGREEMENT
============================================================

Case Reference: PSL-2026-MRG-0042
Date:           March 13, 2026

This Merger and Acquisition Agreement ("Agreement") is 
entered into on March 13, 2026 between:

  BUYER:  PEARSON SPECTER LITT HOLDINGS
          (hereinafter referred to as "Buyer")

  SELLER: DALTON INDUSTRIES
          (hereinafter referred to as "Seller")

ARTICLE 1 — CONSIDERATION

The Buyer agrees to pay the Seller a total consideration 
of $15,750,000.00 (USD) for the complete acquisition 
of all assets, intellectual property, and operational 
rights of Dalton Industries.

ARTICLE 2 — GOVERNING LAW

This Agreement shall be governed by and construed in 
accordance with the laws of the State of New York.

ARTICLE 3 — SIGNATURES

Buyer Representative:  ________________________
                       Pearson Specter Litt Holdings

Seller Representative: ________________________
                       Dalton Industries

============================================================
Generated by PSL Automation System
Case: PSL-2026-MRG-0042 | Date: March 13, 2026
============================================================
```

**Mike:** "Every placeholder is filled programmatically. No manual copy-paste. No typos. Harvey's problem — *solved*."

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

*Louis walks in, peering at your screen over his reading glasses.*

**Louis:** "Cute contract generator. Now let me show you where it'll break."

---

### Pitfall 1: String Immutability Traps

**Louis:** "You think you changed a string? Think again."

```python
name = "Harvey"
name.upper()        # Returns "HARVEY" — but name is STILL "Harvey"!
print(name)         # Harvey  ← UNCHANGED

# You must CAPTURE the return value
name = name.upper()
print(name)         # HARVEY ← Now it's changed
```

**Louis:** "Every string method returns a **new** string. If you don't assign it to a variable, the result is **lost forever**. I see this mistake every single day."

---

### Pitfall 2: `find()` vs `index()` — Silent Failure

**Louis:** "Watch this carefully:"

```python
text = "Pearson Specter Litt"

# find() returns -1 when not found — NO error
pos = text.find("Harvey")
print(pos)  # -1

# index() raises ValueError when not found — CRASHES
pos = text.index("Harvey")  # ValueError: substring not found
```

**Louis:** "Use `find()` when you're not sure the substring exists. Use `index()` when it *must* exist and you want an error if it doesn't."

---

### Pitfall 3: String Comparison Is Case-Sensitive

**Louis:** "This one will destroy your logic if you forget it:"

```python
input_name = "harvey specter"
stored_name = "Harvey Specter"

print(input_name == stored_name)          # False!
print(input_name.lower() == stored_name.lower())  # True ✅
```

**Louis:** "**Always normalize case** before comparing strings. In the legal world, 'ACME Corp' and 'acme corp' are the same entity. Your code should know that too."

---

### Pitfall 4: Newline Characters Are Invisible Enemies

**Louis:** "Data from files and user input often has hidden characters:"

```python
# This looks clean...
user_input = "Harvey Specter\n"
print(len(user_input))  # 15 (not 14!)

# Comparison fails silently
print(user_input == "Harvey Specter")  # False! Hidden newline!

# Always strip!
print(user_input.strip() == "Harvey Specter")  # True ✅
```

---

### Pitfall 5: `+` for Concatenation Is SLOW

**Louis:** "Building strings with `+` in a loop is a **performance disaster**:"

```python
# BAD — creates a new string object every iteration
result = ""
for i in range(10000):
    result = result + str(i) + ","  # O(n²) time!

# GOOD — use join()
result = ",".join(str(i) for i in range(10000))  # O(n) time!
```

**Louis:** "With `+`, Python creates a brand new string every loop iteration, copying everything. With `join()`, it calculates the total size first, then builds the string once. For 10,000 items, `join()` can be **100x faster**."

---

### Pitfall 6: f-string vs `.format()` vs `%` — Old Styles

**Louis:** "You'll see three different formatting styles in the wild:"

```python
name = "Harvey"
age = 45

# ❌ OLD — % formatting (C-style, avoid)
print("Name: %s, Age: %d" % (name, age))

# ⚠️ OLDER BUT OKAY — .format() method
print("Name: {}, Age: {}".format(name, age))

# ✅ BEST — f-strings (Python 3.6+)
print(f"Name: {name}, Age: {age}")
```

**Louis:** "Use f-strings. Always. The other styles exist for backward compatibility, and you'll encounter them in legacy code."

---

### Pitfall 7: Multiline f-string Gotcha

**Louis:** "Watch the indentation with multiline f-strings inside functions:"

```python
def generate_report(client):
    # BAD — the indentation is INCLUDED in the string
    report = f"""
        Client: {client}
        Status: Active
    """
    print(report)
    # Output:
    #
    #         Client: Acme
    #         Status: Active
    #

    # GOOD — use textwrap.dedent or avoid leading indentation
    report = f"""Client: {client}
Status: Active"""
    print(report)
    # Output:
    # Client: Acme
    # Status: Active

generate_report("Acme")
```

---

## 5. 🧠 Mental Model — Donna Paulsen

*Donna leans against the doorframe.*

**Donna:** "Alright, I can see your eyes glazing over. Let me break this down."

---

### 📿 Analogy: Strings Are Like Charm Bracelets

**Donna:** "Think of a string as a **charm bracelet**."

```
String:  "HARVEY"

Bracelet: [H] [A] [R] [V] [E] [Y]
           0   1   2   3   4   5    ← each charm has a position
```

- **Each character** is a **charm** on the bracelet
- **Indexing** (`bracelet[2]`) = picking out the charm at position 2 → `R`
- **Slicing** (`bracelet[1:4]`) = cutting a section of charms → `ARV`
- **Immutability** = the charms are **welded on**. You can't replace one. You have to make a **whole new bracelet** with the change
- **Methods** (`.upper()`, `.replace()`) = they don't modify your bracelet. They **create a copy** with the modifications

**Donna:** "And **f-strings**? Think of them as a **form letter** with blanks to fill in."

```
Dear {client_name},

Your invoice for {amount} is due on {date}.

Regards,
{firm_name}
```

**Donna:** "The computer reads the template, fills in the `{blanks}` with actual values, and produces a perfect letter every time. That's exactly what f-strings do."

**Donna:** "The `print()` function? It's the **printer**. It takes whatever you give it and puts it on paper — or in our case, the screen. The `sep` parameter is the *space between pages*, and `end` is *what happens after the last page rolls out*."

*She points at you.*

**Donna:** "Charm bracelets, form letters, and a printer. That's strings, f-strings, and print. You're welcome."

---

## 6. 💪 Practice Exercises — Rachel Zane

*Rachel walks over with her tablet.*

**Rachel:** "Yesterday you did math. Today, you do words. Here are your exercises."

---

### 🟢 Beginner Exercises

**Exercise 1: String Basics**
```
Create variables for:
- Your name (as entered: "harvey specter")
- Print it in UPPERCASE, lowercase, and Title Case
- Print the length of the name
- Print the first and last characters using indexing
```

**Exercise 2: f-String Introduction**
```
Create variables for a client's case:
- client_name = "Stark Industries"
- case_type = "Patent Infringement"
- damages = 4500000

Using an f-string, print:
"Case: Patent Infringement | Client: Stark Industries | Damages: $4,500,000.00"

The dollar amount must have commas and exactly 2 decimal places.
```

**Exercise 3: String Slicing**
```
Given case_id = "PSL-2026-CORP-0089"

Use slicing to extract and print:
- The firm code: "PSL"
- The year: "2026"
- The case type: "CORP"
- The case number: "0089"
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Email Generator**
```
Given:
  first_name = "  JESSICA  "
  last_name = "  PEARSON  "
  domain = "PSL.COM"

Generate a properly formatted email address:
  "jessica.pearson@psl.com"

Steps: strip whitespace, convert to lowercase, join with a dot,
add @ and domain. Do this in as few lines as possible.
```

**Exercise 5: Text Analyzer**
```
Given this paragraph:
text = "The law is reason free from passion. Justice delayed is justice denied. 
The first duty of society is justice."

Calculate and print:
a) Total character count (with and without spaces)
b) Total word count
c) Total sentence count (hint: count '.')
d) Average word length
e) How many times the word "justice" appears (case-insensitive)
```

**Exercise 6: Receipt Formatter**
```
Create a formatted receipt using f-strings with alignment:

==================================
       PEARSON SPECTER LITT       
       LEGAL SERVICES RECEIPT     
==================================
Client:          Wayne Enterprises
Case:            PSL-2026-0042
----------------------------------
Consultation     $   2,500.00
Document Review  $   4,750.00
Court Filing     $     850.00
Trial Prep       $  12,000.00
----------------------------------
Subtotal         $  20,100.00
Tax (8.5%)       $   1,708.50
----------------------------------
TOTAL            $  21,808.50
==================================

Use variables for all amounts. Calculate subtotal, tax,
and total programmatically. Align all dollar amounts.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Simple Cipher**
```
Implement a Caesar cipher that shifts each letter by N positions.

Given: 
  message = "MEET AT NOON"
  shift = 3

Encrypted: "PHHW DW QRRQ"

Rules:
- Only shift letters (A-Z). Keep spaces as-is.
- Wrap around: X shifted by 3 = A
- Hint: use ord() and chr() to convert between characters and numbers

Also write the decrypt function (shift in the opposite direction).
```

**Exercise 8: Log Parser**
```
Parse this server log entry and extract each field:

log = "[2026-03-13 14:22:05] ERROR: ConnectionTimeout | Server: api-node-3 | Client: 192.168.1.45 | Duration: 30.5s"

Extract and print each field:
- Date and time
- Log level
- Error message
- Server name
- Client IP
- Duration (as a float, without the 's')

Use only string methods (split, find, strip, etc.) — no regex.
```

**Exercise 9: Contract Template Engine**
```
Build a mini template engine. Given a template and a dictionary
of values, replace all placeholders:

template = """
Dear {CLIENT_NAME},

This letter confirms that {FIRM_NAME} will represent you 
in the matter of {CASE_TITLE}. Our estimated fees are 
{ESTIMATED_FEES}. The case is assigned to {ATTORNEY_NAME}, 
who will be your primary contact.

Your case reference number is {CASE_ID}.

Regards,
{FIRM_NAME}
"""

data = {
    "CLIENT_NAME": "Bruce Wayne",
    "FIRM_NAME": "Pearson Specter Litt",
    "CASE_TITLE": "Wayne Enterprises v. Luthor Corp",
    "ESTIMATED_FEES": "$850,000",
    "ATTORNEY_NAME": "Harvey Specter",
    "CASE_ID": "PSL-2026-LIT-0101"
}

Write code that replaces ALL placeholders in the template
with corresponding values from the dictionary.
Print the finished letter.
```

---

## 7. 🐛 Debugging Scenario

*Mike pushes his laptop toward you again.*

**Mike:** "Round two. Another 3 AM script. Find the bugs."

### Buggy Code 🐛

```python
# Bug Hunt: Client Name Formatter
# Find and fix ALL the bugs!

# Input data
raw_name = "   JESSICA   PEARSON   "

# Bug 1: Strip whitespace
clean_name = raw_name.strip           # Something's missing...

# Bug 2: Convert to title case
proper_name = clean_name.Title()      # Method name wrong?

# Bug 3: Build email
email = proper_name.lower.replace(" ", ".") + "@psl.com"   # Missing something?

# Bug 4: Build greeting
greeting = f"Welcome, {proper_name}!"
print(greeting)

# Bug 5: Extract initials
first_initial = proper_name[0]
last_initial = proper_name[proper_name.find(" ")]  # Wrong index?
initials = first_initial + last_initial
print(f"Initials: {initials}")

# Bug 6: Check domain
company_email = "jessica.pearson@psl.com"
if company_email.endswith("@psl.com") or company_email.endswith("@PSL.COM"):
    print("Valid PSL email")
# Is there a simpler way that catches ALL cases?

# Bug 7: String repetition
separator = "-" * "40"   # Hmm...
print(separator)
```

---

### Fixed Code ✅

```python
# Fixed: Client Name Formatter

# Input data
raw_name = "   JESSICA   PEARSON   "

# Fix 1: strip() is a METHOD — needs parentheses ()
clean_name = raw_name.strip()

# Fix 2: Python is case-sensitive — it's .title(), not .Title()
proper_name = clean_name.title()

# Note: clean_name after strip is "JESSICA   PEARSON" (internal spaces remain)
# We should also normalize internal spaces:
proper_name = " ".join(clean_name.split()).title()
# split() without args splits on ANY whitespace and removes extras
# Result: "Jessica Pearson"

# Fix 3: .lower() is a method — needs parentheses
email = proper_name.lower().replace(" ", ".") + "@psl.com"
print(email)  # jessica.pearson@psl.com

# Fix 4: This one was actually correct! ✅
greeting = f"Welcome, {proper_name}!"
print(greeting)  # Welcome, Jessica Pearson!

# Fix 5: find(" ") returns the index of the space, not the character after it
first_initial = proper_name[0]
last_initial = proper_name[proper_name.find(" ") + 1]  # +1 to get past the space
initials = first_initial + last_initial
print(f"Initials: {initials}")  # Initials: JP

# Fix 6: Better approach — normalize case before checking
company_email = "jessica.pearson@psl.com"
if company_email.lower().endswith("@psl.com"):
    print("Valid PSL email")

# Fix 7: Can't multiply string by string — needs an integer
separator = "-" * 40
print(separator)
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | `strip` without `()` — references the method, doesn't call it | Methods need parentheses |
| 2 | `.Title()` — Python is case-sensitive, method is `.title()` | Case sensitivity |
| 3 | `.lower` without `()` — same as Bug 1 | Method call syntax |
| 4 | No bug! Distraction to test your confidence | — |
| 5 | `find(" ")` returns space index, need `+1` for next char | Off-by-one error |
| 6 | Checking two cases manually — use `.lower()` instead | Case normalization |
| 7 | `"-" * "40"` — can't multiply str × str, needs `int` | Type error |

---

## 8. 🌍 Real-World Application

*Harvey returns.*

**Harvey:** "Strings and formatting aren't just for contracts. Here's where they're used in every piece of software you'll ever touch."

### Where Strings Are Used in Production Software:

| Domain | String Operations |
|--------|------------------|
| **Web Development** | URL parsing, HTML template rendering, request/response headers |
| **Data Engineering** | CSV parsing, log file analysis, data cleaning pipelines |
| **APIs** | JSON key handling, endpoint construction, error message formatting |
| **Security** | Input validation, sanitization, password hashing, token generation |
| **DevOps** | Log parsing, configuration management, deployment scripts |
| **Machine Learning** | Text preprocessing, tokenization, NLP feature extraction |
| **Automation** | Email generation, report formatting, file path manipulation |

### Real Production Example: Log Analyzer

```python
# Simplified production log analyzer
log_line = "[2026-03-13 14:22:05] ERROR: Database connection failed | Server: db-primary | Retry: 3/5"

# Extract timestamp
timestamp = log_line[1:20]
print(f"Time: {timestamp}")           # 2026-03-13 14:22:05

# Extract log level
level_start = log_line.find("] ") + 2
level_end = log_line.find(":")
level = log_line[level_start:level_end]
print(f"Level: {level}")              # ERROR

# Extract error message
msg_start = level_end + 2
msg_end = log_line.find(" |")
message = log_line[msg_start:msg_end]
print(f"Message: {message}")          # Database connection failed

# Quick status check
is_critical = level in ("ERROR", "FATAL")
print(f"Needs attention: {is_critical}")  # True
```

### Real Production Example: URL Builder

```python
# Building API URLs dynamically
base_url = "https://api.pearsonspecter.com"
version = "v2"
resource = "clients"
client_id = 2024001
params = "fields=name,revenue&sort=desc"

# Construct the full URL
url = f"{base_url}/{version}/{resource}/{client_id}?{params}"
print(url)
# https://api.pearsonspecter.com/v2/clients/2024001?fields=name,revenue&sort=desc
```

**Harvey:** "Every web request, every log entry, every email, every chat message — it's all strings. Master them, and you control the flow of information."

---

## 9. ✅ Knowledge Check

*Harvey crosses his arms.*

**Harvey:** "Same drill as yesterday. Answer these before you leave."

---

**Q1.** What is the difference between `str.find()` and `str.index()`? When would you use each?

**Q2.** What does `"Hello"[1:4]` return? Why doesn't it include index 4?

**Q3.** Why can't you do `name[0] = "h"` in Python? How would you change the first character of a string?

**Q4.** What is the output of: `"Python" * 3`?

**Q5.** Write an f-string that formats the number `1234567.891` as `"$1,234,567.89"`.

**Q6.** What's the difference between `print("A", "B", "C")` and `print("A" + "B" + "C")`?

**Q7.** What does `"  hello world  ".strip().split()` return?

**Q8.** What is a **raw string** (`r"..."`), and when would you use one?

**Q9.** Why is `",".join(list_of_strings)` better than concatenating with `+` in a loop?

**Q10.** Given `text = "Harvey harvey HARVEY"`, how would you count all occurrences of "harvey" regardless of case?

---

### 📋 Answer Key

> **Q1.** `find()` returns `-1` if not found (safe). `index()` raises `ValueError` (strict). Use `find()` when the substring might not exist; `index()` when it must.

> **Q2.** `"ell"` — slicing is `[start:stop)`, stop is *exclusive*. It returns characters at indices 1, 2, 3.

> **Q3.** Strings are **immutable**. Solution: `name = "h" + name[1:]` — creates a new string.

> **Q4.** `"PythonPythonPython"` — the `*` operator repeats strings.

> **Q5.** `f"${1234567.891:,.2f}"` — `:,.2f` = comma separator + 2 decimal places.

> **Q6.** First: prints `A B C` (space-separated, three arguments). Second: prints `ABC` (concatenated, one argument).

> **Q7.** `['hello', 'world']` — `strip()` removes outer spaces, `split()` splits on whitespace into a list.

> **Q8.** A raw string treats backslashes as literal characters. Used for regex patterns and Windows file paths: `r"C:\Users\new"` (without `r`, `\n` = newline).

> **Q9.** `join()` is O(n) — calculates total size, builds once. `+` concatenation in a loop is O(n²) — creates a new string each iteration.

> **Q10.** `text.lower().count("harvey")` — normalize to lowercase first. Result: `3`.

---

## 🎬 End of Lesson Scene

*It's 5:15 PM. The contract generator is working. Harvey reviews the output on his screen.*

**Harvey:** "Jessica just saw the automated contract. She wants every department using it by next week."

*He looks at you.*

**Harvey:** "Not bad. You know numbers. You know strings. But tomorrow, you're going to learn something that changes everything — **lists**. Because in the real world, you don't deal with one client at a time. You deal with *hundreds.*"

**Mike:** *(closing his laptop)* "Lists are where you go from writing scripts to building *systems*. Get some sleep."

**Donna:** *(walking past)* "I left a snack at your desk. You'll need the energy."

**Louis:** *(from down the hall)* "And don't forget — strings are IMMUTABLE! I *will* quiz you!"

*You head to the elevator, contract generator saved and committed. Day two: complete.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | String creation — single, double, triple quotes | ✅ |
| 2 | String indexing — zero-based, positive & negative | ✅ |
| 3 | String slicing — `[start:stop:step]` | ✅ |
| 4 | String immutability | ✅ |
| 5 | Concatenation (`+`) and repetition (`*`) | ✅ |
| 6 | f-strings — expressions, formatting, alignment | ✅ |
| 7 | `print()` — `sep`, `end`, escape characters | ✅ |
| 8 | String methods — case, search, strip, replace, split, join | ✅ |
| 9 | Method chaining | ✅ |
| 10 | Debugging string-related errors | ✅ |

### 🔗 Connections to Lesson 1
| From Lesson 1 | Used in Lesson 2 |
|---------------|-------------------|
| `type()` function | Checking `type("hello")` → `<class 'str'>` |
| `str()` conversion | Converting numbers to strings for concatenation |
| `print()` basics | Deep dive into `sep`, `end`, escape chars |
| Arithmetic operators | f-string format specifiers for numbers |

---

> **Next Lesson →** Level 1, Lesson 3: *Work With Lists in Python*  
> *Harvey's managing 200 clients. He needs you to build a system that stores, sorts, and filters them all…*
