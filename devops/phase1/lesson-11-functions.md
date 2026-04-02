# Phase 1 · Lesson 11 — Functions
### *"The Toolbox"*

> **O'Reilly Skill:** Write functions with conditionals or loops.
> **DevOps connection:** Every CLI tool, automation script, and deployment pipeline is built from functions. Write once, run everywhere. Change in one place, fixed everywhere.
> **10-min sprint:** Take the billing calculation you wrote in Lesson 1 and turn it into a function. Call it 3 times with different inputs.

---

## Why This Matters (Harvey's take)

Louis changed the tax rate. It was hardcoded in 3 different systems. Now all three show different numbers. The fix? **Functions**. Write the billing calculation ONCE. Give it a name. Every system calls that function. Change the tax rate in one place — done.

Without functions you're writing **scripts**. With functions you're building **systems**.

---

## The Core Concept (Mike's breakdown)

### Defining and Calling

```python
# DEFINE — creates the function, does NOT run it
def check_server(server_name):
    print(f"Checking {server_name}...")
    print(f"  Status: running")

# CALL — actually runs it
check_server("web-01")
check_server("api-01")
check_server("db-01")
```

**Anatomy:**
```python
def function_name(parameter):    # def, name, params, colon
    body_code_here               # indented body
    return something             # optional return value
```

### Parameters and Arguments

```python
# Parameters = variables in the definition
def calculate_billing(hours, rate):
    return hours * rate

# Arguments = values passed when calling
bill = calculate_billing(85, 450.00)   # 85 and 450 are arguments
print(f"${bill:,.2f}")                 # $38,250.00
```

### Return Values

```python
# WITHOUT return — does something but gives nothing back
def greet(name):
    print(f"Hello {name}")

result = greet("Harvey")
print(result)           # None — nothing returned!

# WITH return — gives back a value you can use
def calculate_billing(hours, rate):
    return hours * rate

bill = calculate_billing(85, 450)
tax = calculate_billing(85, 450) * 0.09    # returned value used in expression
```

**Returning multiple values (uses tuples from Lesson 4):**
```python
def analyze_server(server_name, cpu, memory):
    is_healthy = cpu < 90 and memory < 85
    status = "healthy" if is_healthy else "critical"
    return status, cpu, memory     # returns a tuple

# Unpack the return value
status, cpu, mem = analyze_server("web-01", 67.0, 72.0)
print(f"{status}: CPU {cpu}%, MEM {mem}%")
```

### Default Parameters

```python
def deploy(image, environment="staging", replicas=3):
    print(f"Deploying {image} to {environment} ({replicas} replicas)")

deploy("myapp:latest")                           # uses both defaults
deploy("myapp:latest", "production")             # overrides environment
deploy("myapp:latest", "production", replicas=5) # overrides replicas
```

> ⚠️ Default parameters must come AFTER non-default ones: `def f(a, b, c=10)` ✅

### Keyword Arguments

```python
def create_server(name, region, instance_type, count):
    print(f"Creating {count}x {instance_type} in {region} named {name}")

# Positional — order matters
create_server("web", "us-east-1", "t3.medium", 3)

# Keyword — order doesn't matter
create_server(count=3, name="web", instance_type="t3.medium", region="us-east-1")
```

### Functions with Conditionals

```python
def classify_alert(cpu, memory, disk):
    """Return alert level based on resource usage."""
    if cpu > 95 or memory > 95 or disk > 95:
        return "CRITICAL"
    elif cpu > 80 or memory > 80 or disk > 80:
        return "HIGH"
    elif cpu > 60 or memory > 60 or disk > 60:
        return "MEDIUM"
    else:
        return "NORMAL"

print(classify_alert(91.0, 45.0, 55.0))   # HIGH
print(classify_alert(45.0, 78.0, 62.0))   # MEDIUM
print(classify_alert(98.0, 92.0, 45.0))   # CRITICAL
```

### Functions with Loops

```python
def find_down_servers(servers, statuses):
    """Return list of servers that are not running."""
    down = []
    for server, status in zip(servers, statuses):
        if status != "running":
            down.append(server)
    return down

servers = ["web-01", "web-02", "api-01", "db-01"]
statuses = ["running", "stopped", "running", "stopped"]

down = find_down_servers(servers, statuses)
print(f"Down servers: {down}")   # ['web-02', 'db-01']
```

### Docstrings — professional function documentation

```python
def calculate_billing(hours, rate, tax_rate=0.09):
    """
    Calculate total billing with tax.

    Args:
        hours (float): Billable hours
        rate (float): Hourly rate in dollars
        tax_rate (float): Tax rate as decimal (default 9%)

    Returns:
        dict: Subtotal, tax, and total amounts
    """
    subtotal = hours * rate
    tax = subtotal * tax_rate
    return {"subtotal": subtotal, "tax": tax, "total": subtotal + tax}
```

### Variable Scope

```python
TAX_RATE = 0.09    # global — accessible everywhere

def calculate(hours, rate):
    subtotal = hours * rate         # local — only exists inside this function
    tax = subtotal * TAX_RATE      # ✅ CAN read globals
    return subtotal + tax

# print(subtotal)    # ❌ NameError — local variable doesn't exist outside
```

**Best practice:** Pass values in as parameters. Return values out. Avoid modifying globals.

---

## DevOps Example — Deployment Library

```python
ENVIRONMENTS = {"staging": 1, "production": 3, "dev": 1}

def validate_deployment(image, environment, replicas):
    """Check if deployment config is valid."""
    if environment not in ENVIRONMENTS:
        return False, f"Unknown environment: {environment}"
    if not image or ":" not in image:
        return False, f"Invalid image format: {image}"
    if replicas < 1 or replicas > 10:
        return False, f"Replicas must be 1-10, got {replicas}"
    return True, "Valid"

def get_deploy_config(image, environment, replicas=None):
    """Build deployment config dict."""
    if replicas is None:
        replicas = ENVIRONMENTS.get(environment, 1)
    return {
        "image": image,
        "environment": environment,
        "replicas": replicas,
        "namespace": f"psl-{environment}"
    }

def deploy(image, environment="staging", replicas=None):
    """Full deployment with validation."""
    is_valid, message = validate_deployment(image, environment, replicas or 1)
    if not is_valid:
        print(f"❌ Deployment rejected: {message}")
        return False

    config = get_deploy_config(image, environment, replicas)
    print(f"🚀 Deploying {config['image']} to {config['environment']}")
    print(f"   Replicas: {config['replicas']}")
    print(f"   Namespace: {config['namespace']}")
    return True

# Use the library
deploy("myapp:1.2.3")
deploy("myapp:1.2.3", "production", 5)
deploy("badimage", "unknown-env")
```

---

## Gotchas (Louis's warnings)

**1. Mutable default arguments — THE #1 function bug in Python:**
```python
# ❌ DANGEROUS — the list is shared across ALL calls!
def add_server(name, servers=[]):
    servers.append(name)
    return servers

result1 = add_server("web-01")   # ['web-01']
result2 = add_server("api-01")   # ['web-01', 'api-01'] — WHAT?!

# ✅ ALWAYS use None as default for mutable types
def add_server(name, servers=None):
    if servers is None:
        servers = []       # new list each call
    servers.append(name)
    return servers
```

**2. Forgetting to return:**
```python
def calculate(hours, rate):
    total = hours * rate    # calculated but not returned!

result = calculate(85, 450)
print(result)               # None — forgot return!
```

**3. `return` in a loop exits on first match:**
```python
def find_all_errors(logs):
    for log in logs:
        if "ERROR" in log:
            return log       # ❌ returns FIRST error only!
    
    # ✅ collect all, then return
    errors = []
    for log in logs:
        if "ERROR" in log:
            errors.append(log)
    return errors
```

**4. Calling vs referencing (missing parentheses):**
```python
def get_status():
    return "running"

result = get_status    # ❌ this is the function OBJECT
result = get_status()  # ✅ this CALLS it and gets "running"
```

**5. Modifying arguments in-place:**
```python
def sort_servers(servers):
    servers.sort()         # ❌ modifies the ORIGINAL list!
    return servers

# ✅ work on a copy
def sort_servers(servers):
    return sorted(servers)  # new list, original unchanged
```

---

## Mental Model (Donna's analogy)

Functions are **kitchen appliances** (a blender):

```
Parameters (what goes in): Fruits, ice, milk
Body (what happens):       Blend everything
Return value (what comes out): A smoothie

You don't need to know HOW the blender works.
Put stuff in → get result out.
```

- **Parameters** = the ingredient slots
- **Body** = what happens inside (invisible to caller)
- **Return** = the pour spout — only way to get something out
- **Scope** = what happens in the blender stays in the blender

---

## Practice (pick one, 10 minutes)

**Beginner:** Write `is_healthy(cpu, memory, disk)` — returns `True` if all are below 80, `False` otherwise. Write `get_grade(score)` — returns "A"/"B"/"C"/"D"/"F". Call each function with 3 different sets of arguments.

**Intermediate:** Write `calculate_stats(numbers)` — returns a dict with count, sum, mean, min, max, and range. No built-in `sum()`/`min()`/`max()` — use loops.

**DevOps-specific:** Write a `parse_log_line(line)` function that takes `"[09:15] ERROR: timeout | server=db-01"` and returns a dict with `{timestamp, level, message, server}`. Handle the case where there's no server field.

---

## Quick Debug — find all 6 bugs

```python
def add_tax(amount, rate=0.09):       # Bug 1
    total = amount + (amount * rate)

result = add_tax(1000)
print(f"Total: ${result:,.2f}")

def add_item(item, inventory=[]):     # Bug 2
    inventory.append(item)
    return inventory

list1 = add_item("laptop")
list2 = add_item("phone")
print(list2)   # Expected: ['phone']

def find_evens(numbers):
    for num in numbers:
        if num % 2 == 0:
            return num              # Bug 3

evens = find_evens([1,4,7,8,3,6])
print(evens)    # Expected: [4, 8, 6]

def get_greeting():
    return "Hello!"

message = get_greeting             # Bug 4
print(message)

def create_invoice(tax_rate=0.09, client, hours, rate):  # Bug 5
    return hours * rate * (1 + tax_rate)
```

---

## Knowledge Check

1. What's the difference between a parameter and an argument?
2. What does a function return if there's no `return` statement?
3. Why is `def func(items=[])` dangerous? How do you fix it?
4. What's the output? `def f(a, b=10): return a+b; print(f(5)); print(f(5,20))`
5. What is variable scope? What's the difference between local and global?

**Answers:** (1) Parameter = variable in definition; argument = value passed when calling. (2) `None`. (3) The list is created once and shared across all calls; fix with `items=None` and `if items is None: items = []`. (4) `15` then `25`. (5) Scope = where a variable is accessible. Local = inside the function only. Global = everywhere. Local shadows global with the same name.

---

> **Next:** Phase 2 begins → Exception Handling, Logging, subprocess, os/pathlib, requests, and YAML — the tools DevOps engineers use every day.
