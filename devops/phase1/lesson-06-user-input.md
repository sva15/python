# Phase 1 · Lesson 6 — User Input with `input()`
### *"The Interactive Client Intake System"*

> **O'Reilly Skill:** Get input from the user using `input()`.
> **DevOps connection:** CLI tools (`kubectl`, `terraform`, deployment scripts) all accept user input. This is how you build them.
> **10-min sprint:** Build a script that asks for a server name, an IP, and a port. Print back a formatted connection string: `"ssh user@10.0.1.45 -p 22"`.

---

## Why This Matters (Harvey's take)

Until today every program has been deterministic — hardcoded values, same output every time. Real software **interacts with people**. It asks questions. It adapts. `input()` is the simplest form of user interaction. Once you understand it, every program you've built so far can become interactive.

---

## The Core Concept (Mike's breakdown)

### Basic Usage

```python
name = input("Enter your name: ")
print(f"Hello, {name}!")
```

**What happens:**
1. Python prints the prompt: `Enter your name: `
2. Program **pauses** and waits
3. User types something and presses Enter
4. Everything typed (minus the Enter) is returned as a **string**
5. That string is stored in `name`

> **Always add a trailing space in your prompt** → `"Enter name: "` not `"Enter name:"` — makes the cursor visually separated.

### `input()` ALWAYS Returns a String

This is the #1 thing to remember:

```python
age = input("Enter your age: ")   # User types "25"
print(type(age))    # <class 'str'>
print(age + 10)     # ❌ TypeError — "25" is text, not a number
```

### Converting Input to Numbers

```python
# Convert immediately (most common pattern)
age = int(input("Enter your age: "))
rate = float(input("Hourly rate: "))

# Calculate
hours = float(input("Hours worked: "))
print(f"Total: ${hours * rate:,.2f}")
```

**Memorize these patterns:**
- `int(input("prompt"))` — for whole numbers
- `float(input("prompt"))` — for decimals

### Cleaning String Input

```python
# Always strip whitespace and normalize case
name = input("Client name: ").strip().title()
# User types "  bruce wayne  " → stored as "Bruce Wayne"

email = input("Email: ").strip().lower()
# User types "Bruce.Wayne@PSL.COM" → stored as "bruce.wayne@psl.com"

response = input("Continue? (yes/no): ").strip().lower()
if response == "yes":
    print("Continuing...")
```

### Handling Multiple Values on One Line

```python
# User types: "10.0.1.45 8080"
ip, port_str = input("Enter IP and port: ").split()
port = int(port_str)
print(f"Connecting to {ip}:{port}")

# Map pattern — convert all values at once
x, y, z = map(int, input("Enter 3 numbers: ").split())
print(f"Sum: {x + y + z}")
```

### Building a Menu

```python
print("=" * 40)
print("   PSL DEPLOYMENT TOOL")
print("=" * 40)
print("  1. Deploy to staging")
print("  2. Deploy to production")
print("  3. Rollback")
print("  4. Exit")

choice = input("Select (1-4): ").strip()

if choice == "1":
    print("Deploying to staging...")
elif choice == "2":
    print("Deploying to production...")
elif choice == "3":
    print("Rolling back...")
elif choice == "4":
    print("Goodbye!")
else:
    print(f"Invalid: '{choice}'")
```

> Notice: compare `choice` as a **string** (`"1"` not `1`) — since `input()` always returns a string.

### Default Values Pattern

```python
# Press Enter to accept the default
host = input("Host [localhost]: ").strip() or "localhost"
port_str = input("Port [5432]: ").strip() or "5432"
port = int(port_str)
print(f"Connecting to {host}:{port}")
```

The `or` works because empty strings are falsy — if input is empty, `or "localhost"` kicks in.

---

## DevOps Example — CLI Config Wizard

```python
print("PSL Deploy Tool Setup")
print("Press Enter to accept [defaults]\n")

environment = input("Environment [staging]: ").strip() or "staging"
image = input("Docker image [myapp:latest]: ").strip() or "myapp:latest"
replicas_str = input("Replicas [3]: ").strip() or "3"
replicas = int(replicas_str)

config = {
    "environment": environment,
    "image": image,
    "replicas": replicas
}

print(f"\nDeploying {config['image']} to {config['environment']} ({config['replicas']} replicas)")
confirm = input("Confirm? (yes/no): ").strip().lower()

if confirm in ("yes", "y"):
    print("✅ Deployment started!")
else:
    print("❌ Cancelled.")
```

---

## Gotchas (Louis's warnings)

**1. Forgetting that input returns a string:**
```python
qty = input("Quantity: ")
total = qty * 10              # "3" * 10 = "3333333333" — string repeated!
total = int(qty) * 10         # ✅ 30
```

**2. Crashing on invalid numeric input:**
```python
age = int(input("Age: "))     # User types "abc" → ValueError, crash!
# Fix — check first:
age_str = input("Age: ")
if age_str.isdigit():
    age = int(age_str)
else:
    print("Invalid!")
```

**3. The `isdigit()` limitations:**
```python
"42".isdigit()      # True ✅
"3.14".isdigit()    # False ❌ (decimals)
"-5".isdigit()      # False ❌ (negative sign)
" 42 ".isdigit()    # False ❌ (spaces — strip first!)
```

**4. Hidden whitespace in comparisons:**
```python
name = input("Name: ")        # User types "Harvey " (trailing space)
name == "Harvey"               # False!
name.strip() == "Harvey"       # True ✅
```

**5. The `or "yes"` trap:**
```python
if answer == "yes" or "yes":   # ALWAYS True — "yes" is always truthy!
if answer == "yes" or answer == "y":   # ✅
if answer in ("yes", "y"):             # ✅ best
```

---

## Mental Model (Donna's analogy)

`input()` is like a **receptionist**:
- The **prompt** = the question the receptionist asks
- **Waiting** = the program pauses until the user responds
- The **return value** = a handwritten note — always paper (string), even if the client says a number
- **Type conversion** = YOU decide if that note means a number or text
- **`.strip()`** = the receptionist ignores leading/trailing spaces before handing you the note

---

## Practice (pick one, 10 minutes)

**Beginner:** Ask for first name and last name separately. Print a properly formatted greeting: `"Welcome to PSL, Harvey Specter!"` — even if user types in all-caps.

**Intermediate:** Build a temperature converter. Ask: Celsius or Fahrenheit? Then ask for the value. Convert and display with 1 decimal. Handle invalid choice.

**DevOps-specific:** Build a server provisioning prompt. Ask: environment (dev/staging/prod), instance type, number of instances. Print a formatted summary and ask for confirmation.

---

## Quick Debug — find all 6 bugs

```python
name = input("Client name: ")
print("Welcome, " + Name)             # Bug 1

rate = input("Hourly rate: ")
print(f"Rate: ${rate:.2f}")           # Bug 2

hours = input("Hours: ")
total = rate * hours                  # Bug 3

discount = input("Discount %: ")
discounted = total - (total * discount / 100)  # Bug 4

confirm = input("Confirm? (yes/no): ")
if confirm == "Yes" or "yes":        # Bug 5
    print("Confirmed!")
```

---

## Knowledge Check

1. What type does `input()` always return?
2. What does `int(input("Age: "))` do if the user types "twenty"?
3. Why should you always call `.strip()` on user input?
4. Write one line that asks for a float and stores it.
5. What does `input("Host [localhost]: ").strip() or "localhost"` do when the user presses Enter?

**Answers:** (1) Always `str`. (2) `ValueError` — crashes. (3) Users often add spaces; strips prevent comparison failures. (4) `num = float(input("Enter a number: "))`. (5) Empty string is falsy, so `or "localhost"` returns the default.

---

> **Next:** Lesson 7 — If Statements → Harvey needs the system to make decisions — which cases are urgent, which clients are profitable.
