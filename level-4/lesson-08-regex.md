# 🏛️ Level 4 – Advancing | Lesson 8: Use Regex in Python to Match Patterns and Extract Text

## *"Pattern Recognition"*

> **O'Reilly Skill Objective:** *Use regular expressions for simple pattern matching. Use the `re` module to match patterns in strings. Extract substrings from text using regular expressions.*

> **Previously Learned:** String methods, f-strings, `split()`, `replace()`, `in` operator, iteration, `enumerate()`

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

*Discovery dumps 50,000 email files. The firm needs to find every phone number, email address, case reference, and dollar amount — buried in unstructured text.*

```python
# ❌ String methods? One pattern at a time, brittle, verbose...
for line in emails:
    if "@" in line and "." in line:    # Catches "@ . not an email"!
        maybe_email = ???              # How do you extract it?
```

**Mike:** "Regular expressions. One pattern matches thousands of variations."

```python
import re

# One line finds ALL email addresses
emails = re.findall(r'\b[\w.+-]+@[\w-]+\.[\w.]+\b', document)

# One line finds ALL phone numbers
phones = re.findall(r'\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}', document)

# One line finds ALL dollar amounts
amounts = re.findall(r'\$[\d,]+\.?\d*', document)
```

**Harvey:** "Pattern recognition. We do it in court every day. Now teach the computer."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "A regex is a search pattern. Instead of looking for one exact string, you describe the SHAPE of what you want."

| `re` Function | Purpose | Returns |
|---------------|---------|---------|
| `re.search(pattern, string)` | Find first match anywhere | `Match` object or `None` |
| `re.match(pattern, string)` | Match at the START only | `Match` object or `None` |
| `re.fullmatch(pattern, string)` | Match the ENTIRE string | `Match` object or `None` |
| `re.findall(pattern, string)` | Find ALL matches | List of strings |
| `re.finditer(pattern, string)` | Find ALL matches (lazy) | Iterator of `Match` objects |
| `re.sub(pattern, repl, string)` | Find and replace | New string |
| `re.split(pattern, string)` | Split by pattern | List of strings |
| `re.compile(pattern)` | Pre-compile for reuse | `Pattern` object |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Basic Pattern Matching — `re.search()` ⭐

```python
import re

text = "Case WAYNE-2026-001: Harvey Specter representing Wayne Enterprises"

# Search for a literal string
match = re.search(r'Harvey', text)
if match:
    print(f"Found: '{match.group()}'")          # Found: 'Harvey'
    print(f"Position: {match.start()}-{match.end()}")  # Position: 21-27
    print(f"Span: {match.span()}")              # Span: (21, 27)

# re.search returns None if not found
result = re.search(r'Louis', text)
print(result)    # None

# Always check before using:
if result := re.search(r'WAYNE-\d+-\d+', text):
    print(f"Case ref: {result.group()}")        # Case ref: WAYNE-2026-001
```

---

### 3.2 The Regex Character Toolkit ⭐⭐

```python
import re

# === Literal characters ===
re.search(r'Harvey', "Harvey Specter")    # Exact match

# === Special characters (metacharacters) ===
# .   Any character except newline
# ^   Start of string
# $   End of string
# \   Escape special character

re.search(r'H.rvey', "Harvey")      # . matches 'a'
re.search(r'^Case', "Case 001")     # ^ anchors to start
re.search(r'001$', "Case 001")      # $ anchors to end
re.search(r'\$100', "$100")         # \$ matches literal $

# === Character classes ===
# [abc]     Match a, b, or c
# [a-z]     Match any lowercase letter
# [A-Z]     Match any uppercase letter
# [0-9]     Match any digit
# [^abc]    Match anything EXCEPT a, b, c

re.findall(r'[aeiou]', "Harvey Specter")    # ['a', 'e', 'e', 'e']
re.findall(r'[A-Z]', "Harvey Specter")      # ['H', 'S']
re.findall(r'[^a-zA-Z ]', "Price: $450!")   # [':', '$', '4', '5', '0', '!']

# === Shorthand classes ===
# \d   Digit [0-9]
# \D   Non-digit [^0-9]
# \w   Word char [a-zA-Z0-9_]
# \W   Non-word char
# \s   Whitespace [ \t\n\r\f\v]
# \S   Non-whitespace

re.findall(r'\d+', "Case 2026-001, Billed $38,250")
# ['2026', '001', '38', '250']

re.findall(r'\w+', "Harvey Specter (Partner)")
# ['Harvey', 'Specter', 'Partner']
```

---

### 3.3 Quantifiers — How Many? ⭐⭐

```python
import re

# *     Zero or more
# +     One or more
# ?     Zero or one (optional)
# {n}   Exactly n
# {n,m} Between n and m
# {n,}  n or more

text = "Phone: 212-555-0100, Fax: 2125551234, Mobile: (212) 555-0199"

# \d{3}  = exactly 3 digits
re.findall(r'\d{3}', text)
# ['212', '555', '010', '212', '555', '123', '212', '555', '019']

# \d{3,4}  = 3 or 4 digits
re.findall(r'\d{3,4}', text)
# ['212', '555', '0100', '2125', '5512', '34', '212', '555', '0199']

# Full phone pattern
re.findall(r'\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}', text)
# ['212-555-0100', '2125551234', '(212) 555-0199']

# Greedy vs lazy:
text = "<b>Harvey</b> and <b>Mike</b>"

re.findall(r'<b>.*</b>', text)      # Greedy: ['<b>Harvey</b> and <b>Mike</b>']
re.findall(r'<b>.*?</b>', text)     # Lazy:   ['<b>Harvey</b>', '<b>Mike</b>']
# Add ? after any quantifier to make it lazy (non-greedy)
```

---

### 3.4 Groups — Capturing Parts of a Match ⭐⭐

```python
import re

# Parentheses create GROUPS — capture sub-parts of the match

text = "Case WAYNE-2026-001: Billed 85 hours at $450/hr"

# Named groups: (?P<name>pattern)
pattern = r'(?P<client>\w+)-(?P<year>\d{4})-(?P<number>\d{3})'
match = re.search(pattern, text)

if match:
    print(match.group())           # WAYNE-2026-001  (full match)
    print(match.group(1))          # WAYNE           (group 1)
    print(match.group('client'))   # WAYNE           (named group)
    print(match.group('year'))     # 2026
    print(match.group('number'))   # 001
    print(match.groupdict())       # {'client': 'WAYNE', 'year': '2026', 'number': '001'}

# Multiple groups
billing = r'(\d+)\s+hours?\s+at\s+\$(\d+)/hr'
match = re.search(billing, text)
if match:
    hours, rate = match.groups()
    print(f"Hours: {hours}, Rate: ${rate}")    # Hours: 85, Rate: $450

# findall with groups returns ONLY the groups (not full match!)
text2 = "Harvey: $450/hr, Mike: $300/hr, Louis: $400/hr"
rates = re.findall(r'(\w+):\s+\$(\d+)/hr', text2)
print(rates)    # [('Harvey', '450'), ('Mike', '300'), ('Louis', '400')]

# Non-capturing group: (?:...) — groups but doesn't capture
re.findall(r'(?:\w+):\s+\$(\d+)/hr', text2)
# ['450', '300', '400']  — only the rate, name not captured
```

---

### 3.5 `re.findall()` and `re.finditer()` — Multiple Matches ⭐

```python
import re

document = """
Invoice INV-001: Wayne Enterprises, $38,250.00
Invoice INV-002: Stark Industries, $24,800.00
Invoice INV-003: Oscorp, $25,487.50
Contact: harvey.specter@psl.com, mike.ross@psl.com
Phone: (212) 555-0100
"""

# findall — returns list of strings (or tuples if groups)
invoices = re.findall(r'INV-\d+', document)
print(invoices)    # ['INV-001', 'INV-002', 'INV-003']

amounts = re.findall(r'\$[\d,]+\.\d{2}', document)
print(amounts)     # ['$38,250.00', '$24,800.00', '$25,487.50']

emails = re.findall(r'[\w.]+@[\w.]+', document)
print(emails)      # ['harvey.specter@psl.com', 'mike.ross@psl.com']

# finditer — returns iterator of Match objects (more info per match)
for match in re.finditer(r'INV-(\d+):\s+(.+?),\s+\$([\d,]+\.\d{2})', document):
    inv_id = match.group(1)
    client = match.group(2)
    amount = match.group(3)
    print(f"  {inv_id}: {client} — ${amount}")
# 001: Wayne Enterprises — $38,250.00
# 002: Stark Industries — $24,800.00
# 003: Oscorp — $25,487.50
```

---

### 3.6 `re.sub()` — Find and Replace ⭐⭐

```python
import re

# Basic replacement
text = "Contact Harvey at 212-555-0100 or Mike at 212-555-0199"
redacted = re.sub(r'\d{3}-\d{3}-\d{4}', '[REDACTED]', text)
print(redacted)
# Contact Harvey at [REDACTED] or Mike at [REDACTED]

# Replacement with backreferences
text = "2026-03-25"
reformatted = re.sub(r'(\d{4})-(\d{2})-(\d{2})', r'\2/\3/\1', text)
print(reformatted)    # 03/25/2026

# Replacement with named groups
text = "Harvey Specter, Mike Ross, Louis Litt"
result = re.sub(r'(\w+) (\w+)', r'\2, \1', text)
print(result)    # Specter, Harvey Ross, Mike Litt, Louis

# Replacement with a function
def censor_amount(match):
    amount = float(match.group(1).replace(',', ''))
    if amount > 30000:
        return "$[CONFIDENTIAL]"
    return match.group()

text = "Billed $38,250.00, expenses $2,500.00, total $40,750.00"
result = re.sub(r'\$([\d,]+\.\d{2})', censor_amount, text)
print(result)
# Billed $[CONFIDENTIAL], expenses $2,500.00, total $[CONFIDENTIAL]

# Count replacements
result, count = re.subn(r'\d', 'X', "Case 2026-001")
print(f"Result: {result}, Replacements: {count}")
# Result: Case XXXX-XXX, Replacements: 7
```

---

### 3.7 `re.split()` — Split by Pattern ⭐

```python
import re

# Split on multiple delimiters
text = "Harvey;Mike,Louis|Donna Jessica"
names = re.split(r'[;,|\s]+', text)
print(names)    # ['Harvey', 'Mike', 'Louis', 'Donna', 'Jessica']

# Split with capture group keeps the delimiter
text = "Case 001: Open. Case 002: Closed. Case 003: Pending."
parts = re.split(r'(Case \d+)', text)
print(parts)
# ['', 'Case 001', ': Open. ', 'Case 002', ': Closed. ', 'Case 003', ': Pending.']

# maxsplit — limit number of splits
text = "one:two:three:four:five"
print(re.split(r':', text, maxsplit=2))    # ['one', 'two', 'three:four:five']
```

---

### 3.8 `re.compile()` — Pre-compiled Patterns ⭐

```python
import re

# Compile once, use many times — faster for repeated use
CASE_REF = re.compile(r'(?P<client>[A-Z]+)-(?P<year>\d{4})-(?P<num>\d{3})')
EMAIL = re.compile(r'\b[\w.+-]+@[\w-]+\.[\w.]+\b')
PHONE = re.compile(r'\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}')
AMOUNT = re.compile(r'\$[\d,]+(?:\.\d{2})?')

documents = [
    "Case WAYNE-2026-001: Contact harvey@psl.com, (212) 555-0100. Billed $38,250.00",
    "Case STARK-2026-002: Contact mike@psl.com, 212.555.0199. Billed $24,800.00",
    "Case OSCORP-2026-003: Contact louis@psl.com, 212-555-0150. Billed $25,487.50",
]

for doc in documents:
    case = CASE_REF.search(doc)
    email = EMAIL.search(doc)
    phone = PHONE.search(doc)
    amount = AMOUNT.search(doc)
    
    print(f"  Case: {case.group() if case else 'N/A'}")
    print(f"  Email: {email.group() if email else 'N/A'}")
    print(f"  Phone: {phone.group() if phone else 'N/A'}")
    print(f"  Amount: {amount.group() if amount else 'N/A'}")
    print()
```

---

### 3.9 Flags — Modifying Regex Behavior ⭐

```python
import re

# re.IGNORECASE (re.I) — case-insensitive matching
re.findall(r'harvey', "Harvey Specter, HARVEY specter", re.IGNORECASE)
# ['Harvey', 'HARVEY', 'harvey']  (if the string had 'harvey')

print(re.findall(r'harvey', "Harvey HARVEY harvey", re.I))
# ['Harvey', 'HARVEY', 'harvey']

# re.MULTILINE (re.M) — ^ and $ match line starts/ends
text = """Case 001: Open
Case 002: Closed
Case 003: Pending"""

re.findall(r'^Case \d+', text)            # ['Case 001'] — only first line
re.findall(r'^Case \d+', text, re.M)      # ['Case 001', 'Case 002', 'Case 003']

# re.DOTALL (re.S) — . matches newline too
text = "<note>\nConfidential\ninfo\n</note>"
re.findall(r'<note>.*?</note>', text)          # [] — . doesn't match \n
re.findall(r'<note>.*?</note>', text, re.S)    # ['<note>\nConfidential\ninfo\n</note>']

# re.VERBOSE (re.X) — allow comments and whitespace
PHONE_PATTERN = re.compile(r'''
    \(?         # Optional opening parenthesis
    \d{3}       # Area code (3 digits)
    \)?         # Optional closing parenthesis
    [-.\s]?     # Optional separator
    \d{3}       # Exchange (3 digits)
    [-.\s]?     # Optional separator
    \d{4}       # Number (4 digits)
''', re.VERBOSE)

# Combine flags
re.findall(r'^case \d+', text, re.IGNORECASE | re.MULTILINE)
```

---

### 3.10 Solving the Discovery Problem — Document Scanner

```python
# ============================================
# Pearson Specter Litt — Document Scanner
# Regex-powered extraction from legal documents
# ============================================

import re
from collections import defaultdict

class DocumentScanner:
    """Extract structured data from unstructured legal documents."""
    
    # Pre-compiled patterns
    PATTERNS = {
        'case_ref': re.compile(
            r'(?P<client>[A-Z][A-Z]+)-(?P<year>\d{4})-(?P<num>\d{3})'
        ),
        'email': re.compile(
            r'\b[\w.+-]+@[\w-]+\.[\w.]+\b'
        ),
        'phone': re.compile(
            r'\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}'
        ),
        'amount': re.compile(
            r'\$(?P<value>[\d,]+(?:\.\d{2})?)' 
        ),
        'date': re.compile(
            r'\b(?P<month>\d{1,2})/(?P<day>\d{1,2})/(?P<year>\d{4})\b'
            r'|'
            r'\b(?P<year2>\d{4})-(?P<month2>\d{2})-(?P<day2>\d{2})\b'
        ),
        'ssn': re.compile(
            r'\b\d{3}-\d{2}-\d{4}\b'
        ),
    }
    
    def __init__(self):
        self.results = defaultdict(list)
    
    def scan(self, text):
        """Scan text for all patterns."""
        self.results.clear()
        
        for name, pattern in self.PATTERNS.items():
            for match in pattern.finditer(text):
                self.results[name].append({
                    'value': match.group(),
                    'position': match.span(),
                    'groups': match.groupdict(),
                })
        
        return dict(self.results)
    
    def redact(self, text, patterns=None):
        """Redact sensitive information."""
        patterns_to_redact = patterns or ['ssn', 'phone']
        redacted = text
        
        for name in patterns_to_redact:
            if name in self.PATTERNS:
                redacted = self.PATTERNS[name].sub(f'[{name.upper()} REDACTED]', redacted)
        
        return redacted
    
    def extract_amounts(self, text):
        """Extract and sum all dollar amounts."""
        amounts = []
        for match in self.PATTERNS['amount'].finditer(text):
            value_str = match.group('value').replace(',', '')
            amounts.append(float(value_str))
        return amounts
    
    def validate_email(self, email):
        """Validate an email address format."""
        pattern = re.compile(
            r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        )
        return bool(pattern.match(email))


# === Demonstrate ===
print("=" * 65)
print(f"{'🏛️  DOCUMENT SCANNER':^65}")
print("=" * 65)

document = """
CONFIDENTIAL — Pearson Specter Litt

Re: WAYNE-2026-001 — Merger Proceedings

Dear Mr. Wayne,

This letter confirms that Harvey Specter (harvey.specter@psl.com)
will represent Wayne Enterprises in the merger with STARK-2026-002.

Key financial details:
  - Legal fees: $38,250.00
  - Filing costs: $2,500.00
  - Expert witness: $15,000.00
  - Total estimated: $55,750.00

Please contact our office at (212) 555-0100 or reach
Mike Ross at mike.ross@psl.com / 212-555-0199.

SSN on file: 123-45-6789

Signed on 03/25/2026.

Best regards,
Harvey Specter
Partner, Pearson Specter Litt
"""

scanner = DocumentScanner()

# Full scan
results = scanner.scan(document)
print("\n📋 SCAN RESULTS:")
for category, matches in results.items():
    print(f"\n  {category.upper()} ({len(matches)} found):")
    for m in matches:
        print(f"    → {m['value']} at position {m['position']}")

# Extract amounts
amounts = scanner.extract_amounts(document)
print(f"\n💰 AMOUNTS FOUND:")
for amt in amounts:
    print(f"  ${amt:,.2f}")
print(f"  Total: ${sum(amounts):,.2f}")

# Redaction
print(f"\n🔒 REDACTED VERSION:")
redacted = scanner.redact(document, ['ssn', 'phone'])
# Show just the relevant lines
for line in redacted.split('\n'):
    if 'REDACTED' in line:
        print(f"  {line.strip()}")

# Email validation
test_emails = ["harvey@psl.com", "bad@", "mike.ross@psl.com", "not-email"]
print(f"\n📧 EMAIL VALIDATION:")
for email in test_emails:
    valid = scanner.validate_email(email)
    print(f"  {email:<25} {'✅' if valid else '❌'}")

print(f"\n{'='*65}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Greedy vs Lazy Matching 🔥

```python
import re

# ❌ Greedy (default) — matches as MUCH as possible
text = '<b>Harvey</b> and <b>Mike</b>'
re.findall(r'<b>.*</b>', text)
# ['<b>Harvey</b> and <b>Mike</b>']  ← One giant match!

# ✅ Lazy (add ?) — matches as LITTLE as possible
re.findall(r'<b>.*?</b>', text)
# ['<b>Harvey</b>', '<b>Mike</b>']  ← Two separate matches!
```

---

### Pitfall 2: `re.match()` vs `re.search()` 🔥

```python
import re

text = "Invoice: INV-001"

# match only checks the START of the string!
print(re.match(r'INV-\d+', text))     # None! (doesn't start with INV)
print(re.search(r'INV-\d+', text))    # <Match: 'INV-001'>

# ✅ Use search() unless you specifically need start-of-string matching
# Use fullmatch() for exact string matching
```

---

### Pitfall 3: Forgetting Raw Strings 🔥

```python
import re

# ❌ Without raw string — backslashes are Python escape sequences!
re.findall('\d+', "Case 2026")     # Works... but only by luck
re.findall('\bword\b', "a word")   # [] ← \b is Python backspace!

# ✅ Always use raw strings (r'...')
re.findall(r'\d+', "Case 2026")    # ['2026']
re.findall(r'\bword\b', "a word")  # ['word'] ✅
```

---

### Pitfall 4: Catastrophic Backtracking

```python
import re

# ❌ Nested quantifiers can cause exponential runtime!
# pattern = r'(a+)+b'
# re.match(pattern, 'a' * 30)    # Takes FOREVER!

# ✅ Simplify nested quantifiers
pattern = r'a+b'    # Same result, no backtracking issue

# ✅ Use atomic groups or possessive quantifiers (Python 3.11+)
# Or set a timeout / limit input size
```

---

### Pitfall 5: Backslash Confusion in Replacement

```python
import re

# ❌ Backreference confusion
text = "Harvey Specter"
# re.sub(r'(\w+) (\w+)', '\2, \1', text)  # SyntaxWarning: unknown escape

# ✅ Use raw string for replacement too
result = re.sub(r'(\w+) (\w+)', r'\2, \1', text)
print(result)    # Specter, Harvey
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🔍 Analogy: Regex = Discovery Search Warrant

```
STRING METHODS (manual search):
  📝 "Find every mention of '$' in these 50,000 emails"
  → You read each email, character by character
  → Takes forever, miss variations like "$1,000" vs "$1000.00"

REGEX (search warrant):
  📜 "Search for: dollar sign followed by digits, 
      optionally with commas and decimal point"
  → \$[\d,]+\.?\d*
  → One command, catches every variation

DONNA'S RULE:
  "A regex is a search warrant for text.
   You describe WHAT you're looking for,
   and the engine finds EVERY instance.
   Be specific in your warrant — or you get false matches."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Phone Validator**
```
Write a regex that matches these phone formats:
  (212) 555-0100
  212-555-0100
  212.555.0100
  2125550100
Reject: 12345, abc-def-ghij, 1-2-3
```

**Exercise 2: Date Extractor**
```
Extract dates in these formats:
  03/25/2026, 2026-03-25, March 25, 2026
Return as structured data: {month, day, year}
```

**Exercise 3: Word Counter**
```
Use regex to count:
  - Total words
  - Words starting with capital letters
  - Words longer than 5 characters
  - Words containing digits
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Log Parser**
```
Parse log lines:
  "2026-03-25 14:30:22 [ERROR] Database connection failed"
Extract: timestamp, level, message from each line.
Build a summary: count per level, latest error, etc.
```

**Exercise 5: URL Parser**
```
Extract parts from URLs:
  "https://www.example.com:8080/path/page?key=val&k2=v2#section"
Groups: protocol, domain, port, path, query, fragment
```

**Exercise 6: Markdown Link Extractor**
```
From markdown text, extract all links:
  [link text](http://url.com)
  ![image alt](image.png)
Return: [(text, url), ...]
```

---

### 🔴 Advanced Exercises

**Exercise 7: Email Validator (RFC 5321)**
```
Build a comprehensive email validator:
  - Local part: letters, digits, ._%+-
  - Domain: letters, digits, .-
  - TLD: 2-63 letters
  - No consecutive dots
  - Max length: 254 characters
```

**Exercise 8: Code Tokenizer**
```
Build a simple Python tokenizer using regex:
  - Keywords: if, else, for, while, def, class
  - Identifiers: variable names
  - Numbers: int, float
  - Strings: single, double, triple quoted
  - Operators: +, -, *, /, ==, !=
  - Comments: # to end of line
```

**Exercise 9: Data Redactor**
```
Build a configurable redactor:
  redactor = Redactor()
  redactor.add_pattern("SSN", r'\d{3}-\d{2}-\d{4}')
  redactor.add_pattern("credit_card", r'\d{4}[- ]?\d{4}[- ]?\d{4}[- ]?\d{4}')
  redactor.redact(document)    # All SSNs and credit cards hidden
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
import re

# Bug 1: match vs search
text = "Price: $450"
match = re.match(r'\$\d+', text)    # None! match checks START only!

# Bug 2: No raw string
pattern = '\bcase\b'                 # \b is ASCII backspace, not word boundary!
re.findall(pattern, "this case is important")    # []

# Bug 3: Greedy matching
html = "<p>Hello</p><p>World</p>"
re.findall(r'<p>.*</p>', html)       # ['<p>Hello</p><p>World</p>'] — one match!

# Bug 4: findall with groups returns only groups
text = "Harvey: 450, Mike: 300"
re.findall(r'(\w+): (\d+)', text)    # [('Harvey', '450'), ('Mike', '300')]
# Expected full matches? Use finditer instead.

# Bug 5: Not escaping special characters
re.search(r'$450', "$450/hr")        # None! $ means end-of-string in regex!
```

### Fixed Code ✅

```python
# Fix 1: Use search
match = re.search(r'\$\d+', text)    # <Match: '$450'> ✅

# Fix 2: Use raw string
pattern = r'\bcase\b'               # ✅
re.findall(pattern, "this case is important")    # ['case']

# Fix 3: Lazy matching
re.findall(r'<p>.*?</p>', html)      # ['<p>Hello</p>', '<p>World</p>'] ✅

# Fix 4: Use finditer for full Match objects
for m in re.finditer(r'(\w+): (\d+)', text):
    print(f"{m.group()} → {m.groups()}")    # Full match + groups

# Fix 5: Escape special characters
re.search(r'\$450', "$450/hr")       # ✅
# Or use re.escape() for dynamic strings:
re.search(re.escape("$450"), "$450/hr")
```

---

## 8. 🌍 Real-World Application

### Regex in Production

| Domain | Common Patterns |
|--------|----------------|
| **Web scraping** | Extract data from HTML |
| **Log analysis** | Parse timestamps, levels, errors |
| **Data cleaning** | Normalize phone numbers, addresses |
| **Validation** | Email, URL, phone, credit card formats |
| **Security** | Detect SQL injection, XSS patterns |
| **NLP** | Tokenization, entity extraction |
| **DevOps** | Parse config files, grep logs |
| **Legal tech** | Extract case refs, dates, amounts |

---

## 9. ✅ Knowledge Check

---

**Q1.** What module provides regex in Python?

**Q2.** What is the difference between `re.match()` and `re.search()`?

**Q3.** What does `\d+` match?

**Q4.** Why should you always use raw strings (`r'...'`) for regex?

**Q5.** What is the difference between greedy and lazy matching?

**Q6.** How do you create a capturing group?

**Q7.** What does `re.findall()` return when the pattern has groups?

**Q8.** How do you do case-insensitive matching?

**Q9.** What does `re.compile()` do and why use it?

**Q10.** How do you replace matches using a function?

---

### 📋 Answer Key

> **Q1.** The `re` module — `import re`.

> **Q2.** `match()` checks only at the START of the string. `search()` checks ANYWHERE in the string. Use `search()` for general matching.

> **Q3.** One or more digits. `\d` = digit [0-9], `+` = one or more.

> **Q4.** To prevent Python from interpreting backslashes as escape sequences. Without `r`, `\b` is a backspace character, not a word boundary.

> **Q5.** Greedy (`*`, `+`) matches as MUCH as possible. Lazy (`*?`, `+?`) matches as LITTLE as possible.

> **Q6.** With parentheses: `(pattern)`. Named: `(?P<name>pattern)`. Non-capturing: `(?:pattern)`.

> **Q7.** It returns a list of tuples containing only the group contents, not the full matches.

> **Q8.** `re.search(pattern, text, re.IGNORECASE)` or `re.I` flag.

> **Q9.** Pre-compiles a regex pattern into a reusable `Pattern` object. Faster when the same pattern is used repeatedly.

> **Q10.** `re.sub(pattern, replacement_function, text)` — the function receives a `Match` object and returns the replacement string.

---

## 🎬 End of Lesson Scene

**Harvey:** "How many documents did you scan?"

**Mike:** "50,000 emails. Found 2,341 phone numbers, 8,912 email addresses, 156 SSNs that needed redaction, and $14.2 million in referenced amounts. All with six regex patterns."

**Harvey:** "And the SSNs?"

**Mike:** "Redacted. `re.sub` replaced them with `[REDACTED]` before the documents went to opposing counsel."

**Harvey:** "Next: ORMs — querying databases without writing SQL."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `re.search()`, `re.match()`, `re.fullmatch()` | ✅ |
| 2 | Character classes and shorthand (`\d`, `\w`, `\s`) | ✅ |
| 3 | Quantifiers (`*`, `+`, `?`, `{n,m}`) | ✅ |
| 4 | Capturing and named groups | ✅ |
| 5 | `re.findall()` and `re.finditer()` | ✅ |
| 6 | `re.sub()` with strings and functions | ✅ |
| 7 | `re.split()` by pattern | ✅ |
| 8 | `re.compile()` for reuse | ✅ |
| 9 | Flags: `IGNORECASE`, `MULTILINE`, `DOTALL`, `VERBOSE` | ✅ |
| 10 | Greedy vs lazy matching | ✅ |

---

> **Next Lesson →** Level 4, Lesson 9: *Use SQLAlchemy or Another Python ORM*
