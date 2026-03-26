# 🏛️ Level 3 – Building | Lesson 11: Write Unit Tests and Validate Code Behavior in Python

## *"The Burden of Proof"*

> **O'Reilly Skill Objective:** *Write unit tests to validate code behavior using `unittest` and `pytest`.*

> **Previously Learned:** Functions, classes, exception handling, `@property`, decorators, `assert`, debugging, logging, context managers

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

*Harvey pushes a fix to production. It breaks three other features.*

**Jessica:** "How did a one-line billing fix break client search, invoice sorting, AND the dashboard?"

**Harvey:** "I changed the revenue calculation. Didn't realize other modules depended on its exact behavior."

**Mike:** "If we had unit tests, every module would have tests that define its expected behavior. When Harvey changed the calculation, the tests would have failed BEFORE deployment — not after."

**Jessica:** "In court, the burden of proof is on you. Prove your code works. Run the tests."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "A unit test is a contract. It says: 'Given this input, I GUARANTEE this output.'"

| Concept | What It Means |
|---------|---------------|
| **Unit test** | Tests ONE function/method in isolation |
| **Test case** | A class grouping related tests |
| **Assertion** | The claim: "this value should equal that value" |
| **Test runner** | Discovers and runs tests, reports results |
| **Fixture** | Setup/teardown — prepare test data, clean up after |
| **Coverage** | What percentage of code is tested |

**Two frameworks:**

| Feature | `unittest` | `pytest` |
|---------|:----------:|:--------:|
| Built-in | ✅ | ❌ (install) |
| Syntax | `self.assertEqual(a, b)` | `assert a == b` |
| Verbosity | More boilerplate | Minimal |
| Fixtures | `setUp`/`tearDown` methods | `@pytest.fixture` |
| Discovery | `unittest discover` | `pytest` |
| Community | Standard library | Industry standard |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 The Code Under Test

```python
# billing.py — the module we'll test

class BillingCalculator:
    """Calculate legal billing amounts."""
    
    def __init__(self, default_rate=400, tax_rate=0.09):
        if default_rate <= 0:
            raise ValueError("Rate must be positive")
        self.default_rate = default_rate
        self.tax_rate = tax_rate
        self.history = []
    
    def calculate(self, hours, rate=None):
        """Calculate billing amount."""
        if not isinstance(hours, (int, float)):
            raise TypeError(f"Hours must be numeric, got {type(hours).__name__}")
        if hours < 0:
            raise ValueError("Hours cannot be negative")
        
        actual_rate = rate or self.default_rate
        subtotal = hours * actual_rate
        tax = subtotal * self.tax_rate
        total = round(subtotal + tax, 2)
        
        self.history.append({
            "hours": hours,
            "rate": actual_rate,
            "total": total,
        })
        
        return total
    
    def total_billed(self):
        """Sum of all billing history."""
        return sum(entry["total"] for entry in self.history)
    
    def clear_history(self):
        """Reset billing history."""
        self.history.clear()
    
    @staticmethod
    def format_currency(amount):
        """Format a number as currency."""
        if amount < 0:
            return f"-${abs(amount):,.2f}"
        return f"${amount:,.2f}"
    
    @staticmethod
    def classify_client(revenue):
        """Classify client by revenue tier."""
        if revenue >= 1_000_000:
            return "Platinum"
        elif revenue >= 500_000:
            return "Gold"
        elif revenue >= 100_000:
            return "Silver"
        return "Bronze"
```

---

### 3.2 `unittest` — The Standard Library Framework ⭐

```python
# test_billing_unittest.py

import unittest
from billing import BillingCalculator

class TestBillingCalculator(unittest.TestCase):
    """Tests for BillingCalculator using unittest."""
    
    def setUp(self):
        """Runs BEFORE each test method."""
        self.calc = BillingCalculator(default_rate=400, tax_rate=0.09)
    
    def tearDown(self):
        """Runs AFTER each test method."""
        self.calc.clear_history()
    
    # === Basic calculations ===
    
    def test_calculate_with_default_rate(self):
        """85 hours at $400/hr + 9% tax."""
        result = self.calc.calculate(85)
        expected = round(85 * 400 * 1.09, 2)
        self.assertEqual(result, expected)
    
    def test_calculate_with_custom_rate(self):
        """62 hours at $450/hr + 9% tax."""
        result = self.calc.calculate(62, rate=450)
        expected = round(62 * 450 * 1.09, 2)
        self.assertEqual(result, expected)
    
    def test_calculate_zero_hours(self):
        """Zero hours should return zero."""
        self.assertEqual(self.calc.calculate(0), 0)
    
    def test_calculate_float_hours(self):
        """Fractional hours should work."""
        result = self.calc.calculate(1.5, rate=100)
        expected = round(1.5 * 100 * 1.09, 2)
        self.assertEqual(result, expected)
    
    # === Error handling ===
    
    def test_negative_hours_raises(self):
        """Negative hours should raise ValueError."""
        with self.assertRaises(ValueError):
            self.calc.calculate(-5)
    
    def test_string_hours_raises(self):
        """String hours should raise TypeError."""
        with self.assertRaises(TypeError):
            self.calc.calculate("ten")
    
    def test_invalid_rate_raises(self):
        """Non-positive rate in constructor should raise."""
        with self.assertRaises(ValueError):
            BillingCalculator(default_rate=-100)
    
    # === History tracking ===
    
    def test_history_records(self):
        """History should record each calculation."""
        self.calc.calculate(10, rate=100)
        self.calc.calculate(20, rate=200)
        self.assertEqual(len(self.calc.history), 2)
    
    def test_total_billed(self):
        """total_billed should sum all history."""
        self.calc.calculate(10, rate=100)    # 10 * 100 * 1.09 = 1090
        self.calc.calculate(20, rate=100)    # 20 * 100 * 1.09 = 2180
        self.assertAlmostEqual(self.calc.total_billed(), 3270.0, places=2)
    
    def test_clear_history(self):
        """clear_history should empty the list."""
        self.calc.calculate(10)
        self.calc.clear_history()
        self.assertEqual(len(self.calc.history), 0)
        self.assertEqual(self.calc.total_billed(), 0)
    
    # === Static methods ===
    
    def test_format_currency_positive(self):
        self.assertEqual(BillingCalculator.format_currency(1234.5), "$1,234.50")
    
    def test_format_currency_negative(self):
        self.assertEqual(BillingCalculator.format_currency(-500), "-$500.00")
    
    def test_classify_client_platinum(self):
        self.assertEqual(BillingCalculator.classify_client(2500000), "Platinum")
    
    def test_classify_client_gold(self):
        self.assertEqual(BillingCalculator.classify_client(750000), "Gold")
    
    def test_classify_client_silver(self):
        self.assertEqual(BillingCalculator.classify_client(200000), "Silver")
    
    def test_classify_client_bronze(self):
        self.assertEqual(BillingCalculator.classify_client(50000), "Bronze")
    
    def test_classify_client_boundary(self):
        """Test exact boundary values."""
        self.assertEqual(BillingCalculator.classify_client(1_000_000), "Platinum")
        self.assertEqual(BillingCalculator.classify_client(999_999), "Gold")
        self.assertEqual(BillingCalculator.classify_client(500_000), "Gold")
        self.assertEqual(BillingCalculator.classify_client(499_999), "Silver")

if __name__ == "__main__":
    unittest.main()

# Run: python -m unittest test_billing_unittest.py -v
```

**Common `unittest` assertions:**

| Method | Checks |
|--------|--------|
| `assertEqual(a, b)` | `a == b` |
| `assertNotEqual(a, b)` | `a != b` |
| `assertTrue(x)` | `bool(x) is True` |
| `assertFalse(x)` | `bool(x) is False` |
| `assertIs(a, b)` | `a is b` |
| `assertIsNone(x)` | `x is None` |
| `assertIn(a, b)` | `a in b` |
| `assertIsInstance(a, B)` | `isinstance(a, B)` |
| `assertRaises(E)` | Block raises `E` |
| `assertAlmostEqual(a, b)` | `round(a-b, 7) == 0` |
| `assertGreater(a, b)` | `a > b` |
| `assertRegex(s, r)` | `re.match(r, s)` |

---

### 3.3 `pytest` — The Modern Framework ⭐⭐

```python
# test_billing_pytest.py

import pytest
from billing import BillingCalculator

# === Fixtures (replace setUp/tearDown) ===

@pytest.fixture
def calc():
    """Fresh calculator for each test."""
    return BillingCalculator(default_rate=400, tax_rate=0.09)

@pytest.fixture
def loaded_calc(calc):
    """Calculator with some history."""
    calc.calculate(10, rate=100)
    calc.calculate(20, rate=200)
    return calc

# === Basic calculations ===

def test_calculate_default_rate(calc):
    result = calc.calculate(85)
    expected = round(85 * 400 * 1.09, 2)
    assert result == expected

def test_calculate_custom_rate(calc):
    result = calc.calculate(62, rate=450)
    assert result == round(62 * 450 * 1.09, 2)

def test_calculate_zero_hours(calc):
    assert calc.calculate(0) == 0

def test_calculate_float_hours(calc):
    result = calc.calculate(1.5, rate=100)
    assert result == pytest.approx(163.5)

# === Error handling ===

def test_negative_hours_raises(calc):
    with pytest.raises(ValueError, match="cannot be negative"):
        calc.calculate(-5)

def test_string_hours_raises(calc):
    with pytest.raises(TypeError, match="must be numeric"):
        calc.calculate("ten")

def test_invalid_rate_raises():
    with pytest.raises(ValueError, match="must be positive"):
        BillingCalculator(default_rate=-100)

# === History ===

def test_history_records(loaded_calc):
    assert len(loaded_calc.history) == 2

def test_total_billed(loaded_calc):
    assert loaded_calc.total_billed() == pytest.approx(5830.0)

def test_clear_history(loaded_calc):
    loaded_calc.clear_history()
    assert len(loaded_calc.history) == 0

# === Parametrized tests (test many inputs at once!) ===

@pytest.mark.parametrize("revenue, expected_tier", [
    (2_500_000, "Platinum"),
    (1_000_000, "Platinum"),     # Boundary
    (999_999, "Gold"),           # Just below
    (750_000, "Gold"),
    (500_000, "Gold"),           # Boundary
    (499_999, "Silver"),
    (200_000, "Silver"),
    (100_000, "Silver"),         # Boundary
    (99_999, "Bronze"),
    (0, "Bronze"),
])
def test_classify_client(revenue, expected_tier):
    assert BillingCalculator.classify_client(revenue) == expected_tier

@pytest.mark.parametrize("amount, expected", [
    (1234.5, "$1,234.50"),
    (0, "$0.00"),
    (-500, "-$500.00"),
    (1_000_000, "$1,000,000.00"),
])
def test_format_currency(amount, expected):
    assert BillingCalculator.format_currency(amount) == expected

# Run: pytest test_billing_pytest.py -v
```

---

### 3.4 pytest Fixtures — Setup Made Easy ⭐⭐

```python
import pytest
import os
import json

# === Simple fixture ===
@pytest.fixture
def client_data():
    """Provides test client data."""
    return {
        "name": "Wayne Enterprises",
        "revenue": 2500000,
        "type": "Merger",
    }

# === Fixture with teardown ===
@pytest.fixture
def temp_config_file():
    """Create a temp config file, clean up after test."""
    filepath = "test_config.json"
    config = {"rate": 450, "tax": 0.09}
    
    with open(filepath, "w") as f:
        json.dump(config, f)
    
    yield filepath      # ← Test runs here
    
    # Teardown: delete the file
    os.remove(filepath)

def test_load_config(temp_config_file):
    with open(temp_config_file) as f:
        config = json.load(f)
    assert config["rate"] == 450

# === Fixture scope ===
@pytest.fixture(scope="module")
def expensive_data():
    """Computed once per module (not per test)."""
    return list(range(1_000_000))

@pytest.fixture(scope="session")
def db_connection():
    """One connection for the entire test session."""
    conn = {"status": "connected"}
    yield conn
    conn["status"] = "closed"

# === Fixture chaining ===
@pytest.fixture
def calculator():
    return BillingCalculator(400)

@pytest.fixture
def calc_with_history(calculator):
    calculator.calculate(10, 100)
    calculator.calculate(20, 200)
    return calculator
```

**Fixture scopes:**

| Scope | Created | Destroyed | Use Case |
|:-----:|:-------:|:---------:|----------|
| `function` | Each test | After each test | Default — fresh state |
| `class` | Each test class | After class | Shared class setup |
| `module` | Each file | After file | Expensive per-file setup |
| `session` | Once | End of all tests | DB connections, servers |

---

### 3.5 Parametrized Tests — Test Many Cases at Once ⭐

```python
import pytest

# === Simple parametrize ===
@pytest.mark.parametrize("input_val, expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
    ("", ""),
    ("123", "123"),
    ("Hello World", "HELLO WORLD"),
])
def test_upper(input_val, expected):
    assert input_val.upper() == expected

# === Multiple parameters ===
@pytest.mark.parametrize("hours, rate, expected", [
    (10, 100, 1090.0),     # 10 * 100 * 1.09
    (0, 100, 0.0),
    (1, 1, 1.09),
    (100, 500, 54500.0),
])
def test_billing_amounts(hours, rate, expected):
    calc = BillingCalculator(tax_rate=0.09)
    assert calc.calculate(hours, rate) == pytest.approx(expected)

# === Parametrize with IDs ===
@pytest.mark.parametrize("revenue, tier", [
    pytest.param(2_500_000, "Platinum", id="high-revenue"),
    pytest.param(750_000, "Gold", id="medium-revenue"),
    pytest.param(200_000, "Silver", id="low-revenue"),
    pytest.param(50_000, "Bronze", id="very-low-revenue"),
])
def test_client_tier(revenue, tier):
    assert BillingCalculator.classify_client(revenue) == tier
```

---

### 3.6 Testing Exceptions

```python
import pytest

# === pytest ===
def test_raises_value_error():
    calc = BillingCalculator()
    with pytest.raises(ValueError) as exc_info:
        calc.calculate(-5)
    assert "cannot be negative" in str(exc_info.value)

def test_raises_type_error():
    calc = BillingCalculator()
    with pytest.raises(TypeError, match="must be numeric"):
        calc.calculate("ten")

# === unittest ===
class TestExceptions(unittest.TestCase):
    def test_raises(self):
        calc = BillingCalculator()
        with self.assertRaises(ValueError) as cm:
            calc.calculate(-5)
        self.assertIn("negative", str(cm.exception))
```

---

### 3.7 Test Organization — Best Practices

```
project/
├── billing.py                    # Source code
├── client.py
├── tests/                        # Test directory
│   ├── __init__.py
│   ├── test_billing.py          # Tests for billing.py
│   ├── test_client.py           # Tests for client.py
│   ├── conftest.py              # Shared fixtures
│   └── fixtures/
│       └── sample_data.json     # Test data files
```

```python
# tests/conftest.py — shared fixtures (auto-discovered by pytest)

import pytest
from billing import BillingCalculator

@pytest.fixture
def calc():
    return BillingCalculator(400, 0.09)

@pytest.fixture
def sample_clients():
    return [
        {"name": "Wayne", "revenue": 2500000},
        {"name": "Stark", "revenue": 800000},
        {"name": "Oscorp", "revenue": 600000},
    ]
```

**Naming conventions:**

| Convention | Purpose |
|-----------|---------|
| `test_` prefix on files | pytest discovers them |
| `test_` prefix on functions | pytest runs them |
| `Test` prefix on classes | pytest discovers them |
| `conftest.py` | Shared fixtures, auto-imported |
| Descriptive names | `test_calculate_negative_hours_raises` |

---

### 3.8 Mocking and Patching

```python
from unittest.mock import patch, MagicMock, PropertyMock
import pytest

# === Mock external dependencies ===
class InvoiceService:
    def __init__(self, db):
        self.db = db
    
    def create_invoice(self, client, amount):
        invoice_id = self.db.insert("invoices", {"client": client, "amount": amount})
        self.db.log(f"Created invoice {invoice_id}")
        return invoice_id

def test_create_invoice():
    """Test InvoiceService without a real database."""
    mock_db = MagicMock()
    mock_db.insert.return_value = "INV-001"
    
    service = InvoiceService(mock_db)
    result = service.create_invoice("Wayne", 38250)
    
    assert result == "INV-001"
    mock_db.insert.assert_called_once_with(
        "invoices", {"client": "Wayne", "amount": 38250}
    )
    mock_db.log.assert_called_once()

# === Patching modules ===
# billing.py uses datetime.now() — hard to test!
def test_with_frozen_time():
    from datetime import datetime
    
    with patch("billing.datetime") as mock_dt:
        mock_dt.now.return_value = datetime(2026, 3, 25, 9, 0, 0)
        # Now datetime.now() returns a fixed time!

# === Patching with decorator ===
@patch("billing.send_email")
def test_notification(mock_send):
    mock_send.return_value = True
    # send_email is now a mock — no actual emails sent!
    mock_send.assert_called_once()
```

---

### 3.9 Test Markers and Selection

```python
import pytest

# === Mark tests ===
@pytest.mark.slow
def test_large_dataset():
    """Takes a long time — skip in quick runs."""
    data = list(range(10_000_000))
    assert len(sorted(data)) == 10_000_000

@pytest.mark.integration
def test_database_connection():
    """Requires real database."""
    pass

@pytest.mark.skip(reason="Feature not implemented yet")
def test_future_feature():
    pass

@pytest.mark.skipif(
    condition=True,  # e.g., sys.platform == "win32"
    reason="Not supported on this platform"
)
def test_unix_only():
    pass

@pytest.mark.xfail(reason="Known bug #1234")
def test_known_bug():
    assert 1 + 1 == 3    # Expected to fail

# Run specific markers:
# pytest -m "not slow"           # Skip slow tests
# pytest -m integration          # Only integration tests
# pytest -k "test_calculate"     # Only tests matching name
```

---

### 3.10 Solving Harvey's Problem — Full Test Suite

```python
# ============================================
# test_firm_billing.py
# Pearson Specter Litt — Complete Test Suite
# Proves every billing behavior with evidence
# ============================================

import pytest
from billing import BillingCalculator

# ── Fixtures ──────────────────────────────────────

@pytest.fixture
def calc():
    """Fresh billing calculator."""
    return BillingCalculator(default_rate=400, tax_rate=0.09)

@pytest.fixture
def loaded_calc(calc):
    """Calculator with billing history."""
    calc.calculate(85, rate=450)    # Wayne
    calc.calculate(62, rate=400)    # Stark
    calc.calculate(55, rate=425)    # Oscorp
    return calc


# ── SECTION 1: Core Calculations ──────────────────

class TestCalculations:
    """Prove billing math is correct."""
    
    def test_default_rate(self, calc):
        assert calc.calculate(10) == pytest.approx(10 * 400 * 1.09)
    
    def test_custom_rate(self, calc):
        assert calc.calculate(10, rate=500) == pytest.approx(10 * 500 * 1.09)
    
    def test_zero_hours(self, calc):
        assert calc.calculate(0) == 0
    
    def test_fractional_hours(self, calc):
        result = calc.calculate(0.5, rate=100)
        assert result == pytest.approx(54.5)
    
    def test_large_hours(self, calc):
        result = calc.calculate(1000, rate=500)
        assert result == pytest.approx(545000.0)
    
    @pytest.mark.parametrize("hours, rate, expected", [
        (1, 100, 109.0),
        (10, 200, 2180.0),
        (85, 450, 41692.5),
        (100, 400, 43600.0),
    ])
    def test_various_amounts(self, calc, hours, rate, expected):
        assert calc.calculate(hours, rate) == pytest.approx(expected)


# ── SECTION 2: Error Handling ──────────────────────

class TestErrorHandling:
    """Prove invalid inputs are rejected."""
    
    def test_negative_hours(self, calc):
        with pytest.raises(ValueError, match="negative"):
            calc.calculate(-1)
    
    def test_string_hours(self, calc):
        with pytest.raises(TypeError, match="numeric"):
            calc.calculate("ten")
    
    def test_none_hours(self, calc):
        with pytest.raises(TypeError):
            calc.calculate(None)
    
    def test_invalid_constructor_rate(self):
        with pytest.raises(ValueError, match="positive"):
            BillingCalculator(default_rate=0)
        with pytest.raises(ValueError):
            BillingCalculator(default_rate=-100)


# ── SECTION 3: History Tracking ────────────────────

class TestHistory:
    """Prove billing history is tracked correctly."""
    
    def test_empty_history(self, calc):
        assert len(calc.history) == 0
        assert calc.total_billed() == 0
    
    def test_history_grows(self, calc):
        calc.calculate(10, 100)
        assert len(calc.history) == 1
        calc.calculate(20, 200)
        assert len(calc.history) == 2
    
    def test_history_content(self, calc):
        calc.calculate(10, rate=100)
        entry = calc.history[0]
        assert entry["hours"] == 10
        assert entry["rate"] == 100
        assert entry["total"] == pytest.approx(1090.0)
    
    def test_total_billed(self, loaded_calc):
        expected = sum(e["total"] for e in loaded_calc.history)
        assert loaded_calc.total_billed() == pytest.approx(expected)
    
    def test_clear_history(self, loaded_calc):
        loaded_calc.clear_history()
        assert len(loaded_calc.history) == 0
        assert loaded_calc.total_billed() == 0


# ── SECTION 4: Static Methods ─────────────────────

class TestStaticMethods:
    """Prove utility methods work correctly."""
    
    @pytest.mark.parametrize("amount, expected", [
        (0, "$0.00"),
        (100, "$100.00"),
        (1234.567, "$1,234.57"),
        (1_000_000, "$1,000,000.00"),
        (-500, "-$500.00"),
    ])
    def test_format_currency(self, amount, expected):
        assert BillingCalculator.format_currency(amount) == expected
    
    @pytest.mark.parametrize("revenue, tier", [
        (2_500_000, "Platinum"),
        (1_000_000, "Platinum"),
        (999_999, "Gold"),
        (500_000, "Gold"),
        (499_999, "Silver"),
        (100_000, "Silver"),
        (99_999, "Bronze"),
        (0, "Bronze"),
    ])
    def test_classify_client(self, revenue, tier):
        assert BillingCalculator.classify_client(revenue) == tier


# ── SECTION 5: Integration ─────────────────────────

class TestIntegration:
    """Prove components work together."""
    
    def test_full_workflow(self):
        calc = BillingCalculator(default_rate=450, tax_rate=0.09)
        
        # Process multiple clients
        wayne = calc.calculate(85)
        stark = calc.calculate(62, rate=400)
        oscorp = calc.calculate(55, rate=425)
        
        # Verify totals
        assert len(calc.history) == 3
        assert calc.total_billed() == pytest.approx(wayne + stark + oscorp)
        
        # Format output
        formatted = BillingCalculator.format_currency(calc.total_billed())
        assert formatted.startswith("$")
        assert "," in formatted
        
        # Clear and verify
        calc.clear_history()
        assert calc.total_billed() == 0

# Run: pytest test_firm_billing.py -v
# Expected: All tests PASS ✅
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Tests That Depend on Each Other 🔥

```python
# ❌ Tests share state — order matters, failures cascade!
class TestBad:
    calc = BillingCalculator()    # Shared!
    
    def test_first(self):
        self.calc.calculate(10, 100)
        assert len(self.calc.history) == 1
    
    def test_second(self):
        assert len(self.calc.history) == 0   # FAILS! History has 1 entry from test_first!

# ✅ Each test gets fresh state via fixture
@pytest.fixture
def calc():
    return BillingCalculator()

def test_first(calc):
    calc.calculate(10, 100)
    assert len(calc.history) == 1    # ✅

def test_second(calc):
    assert len(calc.history) == 0    # ✅ Fresh calculator!
```

---

### Pitfall 2: Testing Implementation, Not Behavior

```python
# ❌ Tests HOW it works (fragile — breaks on refactor)
def test_bad():
    calc = BillingCalculator()
    calc.calculate(10, 100)
    assert calc.history[0]["hours"] == 10
    assert calc.history[0]["rate"] == 100
    assert calc.history[0]["total"] == 1090.0
    # If internal structure changes, test breaks!

# ✅ Tests WHAT it does (stable — survives refactors)
def test_good():
    calc = BillingCalculator()
    result = calc.calculate(10, 100)
    assert result == pytest.approx(1090.0)
    assert calc.total_billed() == pytest.approx(1090.0)
```

---

### Pitfall 3: Floating Point Comparison

```python
# ❌ Exact comparison fails with floats!
assert 0.1 + 0.2 == 0.3          # False! (0.30000000000000004)

# ✅ Use pytest.approx
assert 0.1 + 0.2 == pytest.approx(0.3)       # ✅
assert 0.1 + 0.2 == pytest.approx(0.3, rel=1e-9)  # Custom tolerance

# ✅ Or unittest
self.assertAlmostEqual(0.1 + 0.2, 0.3, places=10)
```

---

### Pitfall 4: Not Testing Edge Cases

```python
# ❌ Only tests the "happy path"
def test_calculate():
    assert calc.calculate(85, 450) == 41692.5   # Only one case!

# ✅ Test boundaries and edge cases
@pytest.mark.parametrize("hours", [0, 0.01, 1, 24, 100, 999])
def test_various_hours(calc, hours):
    result = calc.calculate(hours)
    assert result >= 0

def test_boundary_values():
    assert BillingCalculator.classify_client(1_000_000) == "Platinum"
    assert BillingCalculator.classify_client(999_999) == "Gold"
```

---

### Pitfall 5: Slow Tests Without Markers

```python
# ❌ Slow test runs every time
def test_million_records():
    data = list(range(1_000_000))
    # ...

# ✅ Mark it and skip in quick runs
@pytest.mark.slow
def test_million_records():
    data = list(range(1_000_000))
    # ...

# Run: pytest -m "not slow" (skip slow tests)
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### ⚖️ Analogy: Tests = Legal Evidence

```
CODE WITHOUT TESTS:
  🗣️ "Your Honor, my code works. Trust me."
  → No evidence. No proof. Case dismissed.

CODE WITH TESTS:
  📋 "Your Honor, I present Exhibits A through Z:
       Exhibit A: calculate(85, 450) returns $41,692.50 ✅
       Exhibit B: negative hours raises ValueError ✅  
       Exhibit C: history tracks all operations ✅
       Exhibit D: currency formatting is correct ✅
       ... 42 tests, 42 passing. Here is the evidence."
  → Case proven. Beyond reasonable doubt.

DONNA'S RULE:
  "Tests don't prove your code is perfect.
   They prove it does what you CLAIM it does.
   That's the difference between an allegation and evidence."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Test a Calculator**
```
Write tests for a simple calculator with add, subtract,
multiply, divide:
  - Happy path for each operation
  - Division by zero raises
  - Negative numbers work
  - Float results are approximately correct
```

**Exercise 2: Test a String Utility**
```
Write tests for: capitalize_words(), truncate(s, max_len),
slugify(s), is_palindrome(s).
Use parametrize for multiple inputs.
```

**Exercise 3: Test with Fixtures**
```
Build a UserManager class.
Write fixtures: empty_manager, populated_manager.
Test: add_user, remove_user, find_user, list_users.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Test Exception Hierarchy**
```
Build a payment system with custom exceptions.
Write tests proving:
  - Correct exception type is raised
  - Exception message contains useful info
  - Exception attributes are set correctly
  - Parent exception catches child exceptions
```

**Exercise 5: Test with Mocks**
```
Build an EmailNotifier that uses an external SMTP service.
Mock the SMTP service and test:
  - Email is sent with correct subject/body
  - Failure is handled gracefully
  - Retry logic works
  - Rate limiting prevents spam
```

**Exercise 6: Parametrized Boundary Tests**
```
Build tests for a grading system (A/B/C/D/F).
Parametrize with:
  - Every boundary value (90, 89, 80, 79, etc.)
  - Invalid inputs (negative, > 100, strings)
  - At least 20 test cases from one parametrize
```

---

### 🔴 Advanced Exercises

**Exercise 7: Property-Based Testing**
```
Install hypothesis. Write property-based tests:

@given(st.integers(min_value=0), st.integers(min_value=1))
def test_billing_always_positive(hours, rate):
    calc = BillingCalculator()
    result = calc.calculate(hours, rate)
    assert result >= 0

Let hypothesis find edge cases automatically.
```

**Exercise 8: Test a REST API**
```
Build a Flask/FastAPI endpoint and test it:
  - GET /clients returns list
  - POST /clients creates client
  - GET /clients/1 returns client
  - DELETE /clients/1 removes client
  - Invalid data returns 400
  - Missing resource returns 404
```

**Exercise 9: Test-Driven Development (TDD)**
```
Build a feature using TDD — tests FIRST:
  1. Write a failing test
  2. Write minimal code to pass
  3. Refactor
  4. Repeat

Feature: a Stack class with push, pop, peek, is_empty, size.
Write ALL tests before any implementation code.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Tests depend on order
class TestOrdered:
    items = []    # Shared!
    def test_add(self):
        self.items.append(1)
    def test_check(self):
        assert len(self.items) == 0   # Fails if test_add runs first!

# Bug 2: Not testing exceptions
def test_incomplete():
    calc = BillingCalculator()
    calc.calculate(10, 100)    # No assertion!
    # Passes but proves nothing!

# Bug 3: Float comparison
def test_float_bad():
    assert 0.1 + 0.2 == 0.3    # Fails!

# Bug 4: Test name without test_ prefix
def billing_calculation():      # pytest ignores this!
    assert calc.calculate(10) == 4360

# Bug 5: Fixture not used as argument
@pytest.fixture
def data():
    return [1, 2, 3]

def test_sum():                 # Missing 'data' parameter!
    assert sum(data) == 6       # NameError: 'data' is not defined!
```

### Fixed Code ✅

```python
# Fix 1: Use fixtures for isolation
@pytest.fixture
def items():
    return []

# Fix 2: Always assert
def test_calculation(calc):
    result = calc.calculate(10, 100)
    assert result == pytest.approx(1090.0)     # ✅ Assert!

# Fix 3: Use approx
def test_float_good():
    assert 0.1 + 0.2 == pytest.approx(0.3)    # ✅

# Fix 4: Prefix with test_
def test_billing_calculation(calc):             # ✅ Found by pytest
    assert calc.calculate(10) == pytest.approx(4360.0)

# Fix 5: Pass fixture as argument
def test_sum(data):                             # ✅ Receives [1, 2, 3]
    assert sum(data) == 6
```

---

## 8. 🌍 Real-World Application

### Testing in Production Workflows

| Practice | What It Means |
|----------|--------------|
| **CI/CD** | Tests run automatically on every commit |
| **TDD** | Write tests BEFORE code |
| **Coverage** | `pytest --cov` — aim for 80%+ |
| **Regression** | Tests catch bugs reintroduced by changes |
| **Mutation** | `mutmut` — verify tests actually catch bugs |
| **Property** | `hypothesis` — auto-generate edge cases |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is a unit test?

**Q2.** What is the difference between `unittest` and `pytest`?

**Q3.** What is a `pytest` fixture?

**Q4.** How do you test that a function raises a specific exception?

**Q5.** What does `pytest.mark.parametrize` do?

**Q6.** Why should tests not depend on each other?

**Q7.** How do you compare floating-point numbers in tests?

**Q8.** What is a mock and when do you use one?

**Q9.** What is `conftest.py`?

**Q10.** What is the difference between testing behavior and testing implementation?

---

### 📋 Answer Key

> **Q1.** A test that verifies ONE function/method in isolation — given specific input, assert specific output.

> **Q2.** `unittest` is the standard library (class-based, verbose). `pytest` is third-party (function-based, minimal boilerplate, powerful fixtures and parametrize).

> **Q3.** A function that provides test data or setup/teardown. Injected by name as a test parameter. Replaces `setUp`/`tearDown`.

> **Q4.** `with pytest.raises(ValueError, match="pattern"):` or `with self.assertRaises(ValueError):`.

> **Q5.** Runs the same test with multiple sets of inputs — one test function, many test cases.

> **Q6.** Tests should be independent and run in any order. Shared state causes unpredictable failures.

> **Q7.** `assert a == pytest.approx(b)` or `self.assertAlmostEqual(a, b)` — never exact `==` with floats.

> **Q8.** A fake object that simulates a dependency (database, API, email). Use to test in isolation without external services.

> **Q9.** A special pytest file where shared fixtures are defined. Auto-discovered — fixtures are available to all tests in the directory.

> **Q10.** Behavior testing checks WHAT the code does (stable). Implementation testing checks HOW it does it (fragile — breaks on refactor).

---

## 🎬 End of Lesson Scene

**Jessica:** "How many tests?"

**Mike:** "42 tests, 42 passing. Calculations verified with parametrized boundary values. Error handling proven with exception assertions. History tracking, formatting, classification — all covered."

**Harvey:** "And if I change the revenue calculation again?"

**Mike:** "The tests will catch every ripple effect. Immediately. Before deployment."

**Jessica:** "The burden of proof is met."

**Harvey:** "Last lesson: duck typing. If it walks like a case and bills like a case — it IS a case."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `unittest` basics (TestCase, assertions, setUp/tearDown) | ✅ |
| 2 | `pytest` basics (assert, fixtures, discovery) | ✅ |
| 3 | `pytest.fixture` with setup/teardown and scope | ✅ |
| 4 | `pytest.mark.parametrize` — test many inputs | ✅ |
| 5 | Testing exceptions (`pytest.raises`, `assertRaises`) | ✅ |
| 6 | Mocking with `unittest.mock` | ✅ |
| 7 | Test organization and naming conventions | ✅ |
| 8 | Test markers (skip, xfail, slow, integration) | ✅ |
| 9 | Floating-point comparison (`pytest.approx`) | ✅ |
| 10 | Behavior vs implementation testing | ✅ |

---

> **Next Lesson →** Level 3, Lesson 12: *Apply Duck Typing in Python*
