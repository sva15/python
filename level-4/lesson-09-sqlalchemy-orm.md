# 🏛️ Level 4 – Advancing | Lesson 9: Use SQLAlchemy or Another Python ORM

## *"The Record Keeper"*

> **O'Reilly Skill Objective:** *Use SQLAlchemy or another Python ORM.*

> **Previously Learned:** Classes, inheritance, `__init__`, `@property`, decorators, context managers, exception handling, dunder methods

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

*The firm tracks thousands of clients, cases, and invoices. Currently it's all in raw SQL strings embedded in Python.*

```python
# ❌ Raw SQL — fragile, error-prone, SQL injection risk
import sqlite3

conn = sqlite3.connect("firm.db")
cursor = conn.cursor()

# Hard to maintain, no type safety, SQL injection vulnerable!
name = "Wayne"
cursor.execute(f"SELECT * FROM clients WHERE name = '{name}'")    # ❌ SQL INJECTION!
cursor.execute("SELECT * FROM clients WHERE name = ?", (name,))   # Safer but still raw SQL

# Result is a tuple — no attribute access
row = cursor.fetchone()
print(row[0], row[1])    # What's column 0? column 1? Who knows!
```

**Harvey:** "I don't want to write SQL. I want to write Python."

```python
# ✅ ORM — Python objects, no raw SQL, safe by default
from sqlalchemy.orm import Session

with Session(engine) as session:
    client = session.query(Client).filter_by(name="Wayne").first()
    print(client.name)         # Wayne — attribute access!
    print(client.revenue)      # 2500000
    print(client.cases)        # [Case(...), Case(...)] — relationships!
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "An ORM maps Python classes to database tables. Each instance is a row. Each attribute is a column. No SQL required."

| Concept | Database | Python (ORM) |
|---------|----------|:------------:|
| **Table** | `CREATE TABLE clients` | `class Client(Base)` |
| **Column** | `name VARCHAR(100)` | `name = Column(String(100))` |
| **Row** | `INSERT INTO clients` | `session.add(Client(...))` |
| **Query** | `SELECT * FROM clients WHERE...` | `session.query(Client).filter(...)` |
| **Join** | `JOIN cases ON...` | `client.cases` (relationship) |
| **Update** | `UPDATE clients SET...` | `client.name = "New"` |
| **Delete** | `DELETE FROM clients WHERE...` | `session.delete(client)` |
| **Commit** | `COMMIT` | `session.commit()` |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Setting Up SQLAlchemy ⭐

```python
# pip install sqlalchemy

from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, Session

# Create engine — the connection to the database
# SQLite (file-based, great for learning)
engine = create_engine("sqlite:///firm.db", echo=False)

# Other databases:
# engine = create_engine("postgresql://user:pass@localhost/firm")
# engine = create_engine("mysql+pymysql://user:pass@localhost/firm")

# Base class for all models (SQLAlchemy 2.0 style)
class Base(DeclarativeBase):
    pass

# After defining models, create all tables:
# Base.metadata.create_all(engine)
```

---

### 3.2 Defining Models — Classes as Tables ⭐⭐

```python
from sqlalchemy import Column, Integer, String, Float, Boolean, DateTime, ForeignKey
from sqlalchemy.orm import DeclarativeBase, relationship
from datetime import datetime

class Base(DeclarativeBase):
    pass

class Client(Base):
    """Each Client instance = one row in the 'clients' table."""
    __tablename__ = 'clients'
    
    # Columns
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(100), nullable=False, unique=True)
    revenue = Column(Float, default=0.0)
    tier = Column(String(20), default="Bronze")
    active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # Relationship: one client has many cases
    cases = relationship("Case", back_populates="client", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"Client(id={self.id}, name='{self.name}', tier='{self.tier}')"

class Attorney(Base):
    __tablename__ = 'attorneys'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    role = Column(String(50), default="Associate")
    hourly_rate = Column(Float, nullable=False)
    equity_share = Column(Float, default=0.0)
    
    # One attorney has many cases
    cases = relationship("Case", back_populates="attorney")
    
    def __repr__(self):
        return f"Attorney('{self.name}', ${self.hourly_rate}/hr)"

class Case(Base):
    __tablename__ = 'cases'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    status = Column(String(20), default="open")
    hours_billed = Column(Float, default=0.0)
    
    # Foreign keys — link to other tables
    client_id = Column(Integer, ForeignKey('clients.id'), nullable=False)
    attorney_id = Column(Integer, ForeignKey('attorneys.id'), nullable=False)
    
    # Relationships
    client = relationship("Client", back_populates="cases")
    attorney = relationship("Attorney", back_populates="cases")
    
    # One case has many invoices
    invoices = relationship("Invoice", back_populates="case", cascade="all, delete-orphan")
    
    @property
    def total_billed(self):
        return sum(inv.amount for inv in self.invoices)
    
    def __repr__(self):
        return f"Case('{self.title}', status='{self.status}')"

class Invoice(Base):
    __tablename__ = 'invoices'
    
    id = Column(Integer, primary_key=True)
    amount = Column(Float, nullable=False)
    description = Column(String(500))
    paid = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    case_id = Column(Integer, ForeignKey('cases.id'), nullable=False)
    case = relationship("Case", back_populates="invoices")
    
    def __repr__(self):
        return f"Invoice(${self.amount:,.2f}, paid={self.paid})"
```

---

### 3.3 SQLAlchemy 2.0 Mapped Columns (Modern Style) ⭐

```python
from sqlalchemy import String, ForeignKey
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship
from datetime import datetime

class Base(DeclarativeBase):
    pass

class Client(Base):
    """SQLAlchemy 2.0 style — type-annotated columns."""
    __tablename__ = 'clients'
    
    # Mapped columns with Python type hints
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100), unique=True)
    revenue: Mapped[float] = mapped_column(default=0.0)
    tier: Mapped[str] = mapped_column(String(20), default="Bronze")
    active: Mapped[bool] = mapped_column(default=True)
    
    # Optional columns
    notes: Mapped[str | None] = mapped_column(String(500), default=None)
    
    # Relationships
    cases: Mapped[list["Case"]] = relationship(back_populates="client")
    
    def __repr__(self):
        return f"Client('{self.name}')"

class Case(Base):
    __tablename__ = 'cases'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(200))
    status: Mapped[str] = mapped_column(String(20), default="open")
    
    client_id: Mapped[int] = mapped_column(ForeignKey("clients.id"))
    client: Mapped["Client"] = relationship(back_populates="cases")

# Benefits of 2.0 style:
# ✅ Type hints for IDE autocomplete
# ✅ Cleaner syntax
# ✅ Better mypy/pyright support
```

---

### 3.4 Creating, Reading, Updating, Deleting (CRUD) ⭐⭐

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import Session

engine = create_engine("sqlite:///firm.db")
Base.metadata.create_all(engine)    # Create tables

# === CREATE: Adding records ===
with Session(engine) as session:
    # Create objects
    wayne = Client(name="Wayne Enterprises", revenue=2500000, tier="Platinum")
    stark = Client(name="Stark Industries", revenue=1800000, tier="Gold")
    oscorp = Client(name="Oscorp", revenue=600000, tier="Silver")
    
    harvey = Attorney(name="Harvey Specter", role="Partner", hourly_rate=450, equity_share=0.15)
    mike = Attorney(name="Mike Ross", role="Associate", hourly_rate=300)
    
    # Add to session
    session.add_all([wayne, stark, oscorp, harvey, mike])
    
    # Create with relationships
    case1 = Case(title="Wayne Merger", client=wayne, attorney=harvey, status="open")
    case2 = Case(title="Stark Patent", client=stark, attorney=harvey, status="open")
    case3 = Case(title="Oscorp Restructuring", client=oscorp, attorney=mike, status="pending")
    session.add_all([case1, case2, case3])
    
    # Add invoices
    session.add_all([
        Invoice(amount=38250.00, description="Legal fees Q1", case=case1),
        Invoice(amount=12500.00, description="Filing costs", case=case1),
        Invoice(amount=24800.00, description="Patent research", case=case2),
    ])
    
    session.commit()    # Save to database!
    print(f"✅ Created {session.query(Client).count()} clients")


# === READ: Querying records ===
with Session(engine) as session:
    # Get all clients
    clients = session.query(Client).all()
    for c in clients:
        print(f"  {c.name}: ${c.revenue:,} [{c.tier}]")
    
    # Get one by primary key
    client = session.get(Client, 1)
    print(f"\nClient #1: {client.name}")
    
    # Filter
    platinum = session.query(Client).filter_by(tier="Platinum").all()
    print(f"\nPlatinum clients: {[c.name for c in platinum]}")
    
    # Complex filter
    big_clients = (
        session.query(Client)
        .filter(Client.revenue > 1_000_000)
        .filter(Client.active == True)
        .order_by(Client.revenue.desc())
        .all()
    )
    print(f"\nBig clients: {[c.name for c in big_clients]}")


# === UPDATE: Modifying records ===
with Session(engine) as session:
    client = session.query(Client).filter_by(name="Wayne Enterprises").first()
    client.revenue = 3000000          # Just set the attribute!
    client.tier = "Platinum VIP"
    session.commit()                   # Save changes
    print(f"✅ Updated: {client.name} → ${client.revenue:,}")


# === DELETE: Removing records ===
with Session(engine) as session:
    oscorp = session.query(Client).filter_by(name="Oscorp").first()
    if oscorp:
        session.delete(oscorp)         # Mark for deletion
        session.commit()               # Execute deletion
        print(f"✅ Deleted: Oscorp")
```

---

### 3.5 Querying — Filters, Ordering, Aggregation ⭐⭐

```python
from sqlalchemy import func, and_, or_, desc, asc

with Session(engine) as session:
    # === Comparison filters ===
    session.query(Client).filter(Client.revenue > 1000000).all()
    session.query(Client).filter(Client.revenue >= 1000000).all()
    session.query(Client).filter(Client.name != "Oscorp").all()
    session.query(Client).filter(Client.tier.in_(["Platinum", "Gold"])).all()
    session.query(Client).filter(Client.name.like("Wayne%")).all()
    session.query(Client).filter(Client.name.ilike("%stark%")).all()    # Case-insensitive
    session.query(Client).filter(Client.notes.is_(None)).all()
    session.query(Client).filter(Client.notes.isnot(None)).all()
    
    # === Logical operators ===
    session.query(Client).filter(
        and_(Client.revenue > 500000, Client.active == True)
    ).all()
    
    session.query(Client).filter(
        or_(Client.tier == "Platinum", Client.revenue > 2000000)
    ).all()
    
    # === Ordering ===
    session.query(Client).order_by(Client.revenue.desc()).all()
    session.query(Client).order_by(Client.name.asc()).all()
    
    # === Limiting ===
    session.query(Client).limit(5).all()
    session.query(Client).offset(10).limit(5).all()    # Pagination
    
    # === Aggregation ===
    total_revenue = session.query(func.sum(Client.revenue)).scalar()
    avg_revenue = session.query(func.avg(Client.revenue)).scalar()
    client_count = session.query(func.count(Client.id)).scalar()
    max_revenue = session.query(func.max(Client.revenue)).scalar()
    
    print(f"  Clients: {client_count}")
    print(f"  Total revenue: ${total_revenue:,.2f}")
    print(f"  Avg revenue: ${avg_revenue:,.2f}")
    print(f"  Max revenue: ${max_revenue:,.2f}")
    
    # === Group by ===
    tier_stats = (
        session.query(Client.tier, func.count(Client.id), func.sum(Client.revenue))
        .group_by(Client.tier)
        .all()
    )
    for tier, count, revenue in tier_stats:
        print(f"  {tier}: {count} clients, ${revenue:,.2f}")
```

---

### 3.6 Relationships — Joins Without SQL ⭐⭐

```python
with Session(engine) as session:
    # Navigate relationships like Python attributes!
    
    # Client → Cases (one-to-many)
    wayne = session.query(Client).filter_by(name="Wayne Enterprises").first()
    print(f"\n{wayne.name}'s cases:")
    for case in wayne.cases:
        print(f"  📋 {case.title} ({case.status})")
        print(f"     Attorney: {case.attorney.name}")
        print(f"     Invoices: {len(case.invoices)}")
    
    # Attorney → Cases (one-to-many)
    harvey = session.query(Attorney).filter_by(name="Harvey Specter").first()
    print(f"\n{harvey.name}'s caseload:")
    for case in harvey.cases:
        print(f"  📋 {case.title} for {case.client.name}")
    
    # Case → Invoices (one-to-many)
    merger = session.query(Case).filter_by(title="Wayne Merger").first()
    print(f"\nInvoices for '{merger.title}':")
    for inv in merger.invoices:
        print(f"  💰 {inv.description}: ${inv.amount:,.2f} (paid: {inv.paid})")
    
    # Explicit join query
    results = (
        session.query(Case.title, Client.name, Attorney.name)
        .join(Client, Case.client_id == Client.id)
        .join(Attorney, Case.attorney_id == Attorney.id)
        .all()
    )
    print(f"\nAll cases:")
    for title, client, attorney in results:
        print(f"  {title}: {client} → {attorney}")
```

---

### 3.7 SQLAlchemy 2.0 `select()` Statement (Modern Query API) ⭐

```python
from sqlalchemy import select

with Session(engine) as session:
    # Modern select() — replaces session.query()
    
    # Simple select
    stmt = select(Client).where(Client.revenue > 1000000)
    clients = session.scalars(stmt).all()
    
    # With ordering
    stmt = (
        select(Client)
        .where(Client.active == True)
        .order_by(Client.revenue.desc())
        .limit(5)
    )
    top_clients = session.scalars(stmt).all()
    
    # Select specific columns
    stmt = select(Client.name, Client.revenue).where(Client.tier == "Platinum")
    rows = session.execute(stmt).all()
    for name, revenue in rows:
        print(f"  {name}: ${revenue:,}")
    
    # Join with select
    stmt = (
        select(Case.title, Client.name, Attorney.name)
        .join(Client)
        .join(Attorney)
        .where(Case.status == "open")
    )
    for title, client, attorney in session.execute(stmt):
        print(f"  {title} — {client} → {attorney}")
```

---

### 3.8 Sessions and Transactions ⭐

```python
from sqlalchemy.orm import Session

# === Session as context manager (recommended) ===
with Session(engine) as session:
    client = Client(name="New Client", revenue=500000)
    session.add(client)
    session.commit()    # Saves changes
# Session automatically closed when block exits

# === Transaction with rollback ===
with Session(engine) as session:
    try:
        client = session.query(Client).filter_by(name="Wayne Enterprises").first()
        client.revenue = -999    # Bad value!
        
        if client.revenue < 0:
            session.rollback()    # Undo ALL changes in this transaction!
            print("❌ Rolled back — invalid data!")
        else:
            session.commit()
    except Exception as e:
        session.rollback()
        print(f"❌ Error: {e}")

# === Nested transaction (savepoint) ===
with Session(engine) as session:
    session.begin()
    try:
        session.add(Client(name="Client A", revenue=100000))
        
        # Nested savepoint
        nested = session.begin_nested()
        try:
            session.add(Client(name="Client A", revenue=200000))  # Duplicate!
            nested.commit()
        except Exception:
            nested.rollback()    # Only rolls back the nested part
        
        session.commit()    # Client A (first one) is saved
    except Exception:
        session.rollback()
```

---

### 3.9 Migrations with Alembic (Overview)

```python
# Alembic manages database schema changes over time
# pip install alembic

# === Setup (one time) ===
# $ alembic init migrations

# === After changing models, generate a migration ===
# $ alembic revision --autogenerate -m "Add notes column to clients"

# Generated migration file:
"""
def upgrade():
    op.add_column('clients', sa.Column('notes', sa.String(500), nullable=True))

def downgrade():
    op.drop_column('clients', 'notes')
"""

# === Apply migrations ===
# $ alembic upgrade head       # Apply all pending migrations
# $ alembic downgrade -1       # Rollback last migration
# $ alembic history            # View migration history
# $ alembic current            # Show current version

# WHY ALEMBIC?
# ✅ Track schema changes in version control
# ✅ Deploy database updates safely
# ✅ Rollback if something goes wrong
# ✅ Team collaboration on schema changes
```

---

### 3.10 Solving the Firm's Problem — Complete ORM System

```python
# ============================================
# Pearson Specter Litt — ORM-Powered Database
# Full CRUD with relationships and reporting
# ============================================

from sqlalchemy import create_engine, Column, Integer, String, Float, Boolean, DateTime, ForeignKey, func
from sqlalchemy.orm import DeclarativeBase, relationship, Session
from datetime import datetime

# === Setup ===
class Base(DeclarativeBase):
    pass

class Client(Base):
    __tablename__ = 'clients'
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False, unique=True)
    revenue = Column(Float, default=0.0)
    tier = Column(String(20), default="Bronze")
    active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    cases = relationship("Case", back_populates="client", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"Client('{self.name}', ${self.revenue:,.0f})"

class Attorney(Base):
    __tablename__ = 'attorneys'
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    role = Column(String(50))
    hourly_rate = Column(Float)
    
    cases = relationship("Case", back_populates="attorney")
    
    def __repr__(self):
        return f"Attorney('{self.name}')"

class Case(Base):
    __tablename__ = 'cases'
    id = Column(Integer, primary_key=True)
    title = Column(String(200))
    status = Column(String(20), default="open")
    hours = Column(Float, default=0.0)
    client_id = Column(Integer, ForeignKey('clients.id'))
    attorney_id = Column(Integer, ForeignKey('attorneys.id'))
    
    client = relationship("Client", back_populates="cases")
    attorney = relationship("Attorney", back_populates="cases")
    invoices = relationship("Invoice", back_populates="case", cascade="all, delete-orphan")
    
    @property
    def billing(self):
        return self.hours * (self.attorney.hourly_rate if self.attorney else 0)
    
    def __repr__(self):
        return f"Case('{self.title}')"

class Invoice(Base):
    __tablename__ = 'invoices'
    id = Column(Integer, primary_key=True)
    amount = Column(Float)
    description = Column(String(500))
    paid = Column(Boolean, default=False)
    case_id = Column(Integer, ForeignKey('cases.id'))
    
    case = relationship("Case", back_populates="invoices")

# === Create database ===
engine = create_engine("sqlite:///:memory:")    # In-memory for demo
Base.metadata.create_all(engine)

# === Seed data ===
with Session(engine) as session:
    # Attorneys
    harvey = Attorney(name="Harvey Specter", role="Partner", hourly_rate=450)
    mike = Attorney(name="Mike Ross", role="Associate", hourly_rate=300)
    louis = Attorney(name="Louis Litt", role="Partner", hourly_rate=400)
    session.add_all([harvey, mike, louis])
    
    # Clients
    wayne = Client(name="Wayne Enterprises", revenue=2500000, tier="Platinum")
    stark = Client(name="Stark Industries", revenue=1800000, tier="Gold")
    oscorp = Client(name="Oscorp", revenue=600000, tier="Silver")
    queen = Client(name="Queen Consolidated", revenue=400000, tier="Bronze")
    session.add_all([wayne, stark, oscorp, queen])
    
    # Cases with relationships
    cases = [
        Case(title="Wayne Merger", status="open", hours=85, client=wayne, attorney=harvey),
        Case(title="Wayne IP Defense", status="open", hours=40, client=wayne, attorney=mike),
        Case(title="Stark Patent", status="closed", hours=120, client=stark, attorney=harvey),
        Case(title="Oscorp Restructuring", status="pending", hours=55, client=oscorp, attorney=mike),
        Case(title="Queen Compliance", status="open", hours=30, client=queen, attorney=louis),
    ]
    session.add_all(cases)
    
    # Invoices
    session.add_all([
        Invoice(amount=38250, description="Merger Q1 fees", case=cases[0]),
        Invoice(amount=12500, description="Filing costs", case=cases[0]),
        Invoice(amount=12000, description="IP research", case=cases[1], paid=True),
        Invoice(amount=54000, description="Patent work", case=cases[2], paid=True),
        Invoice(amount=16500, description="Due diligence", case=cases[3]),
        Invoice(amount=12000, description="Compliance audit", case=cases[4]),
    ])
    
    session.commit()

# === Reports ===
print("=" * 70)
print(f"{'🏛️  PEARSON SPECTER LITT — DATABASE REPORT':^70}")
print("=" * 70)

with Session(engine) as session:
    # Client roster
    print(f"\n📋 CLIENT ROSTER:")
    print(f"  {'Name':<25} {'Revenue':>12} {'Tier':<12} {'Cases':>5}")
    print(f"  {'─'*25} {'─'*12} {'─'*12} {'─'*5}")
    
    clients = session.query(Client).order_by(Client.revenue.desc()).all()
    for c in clients:
        print(f"  {c.name:<25} ${c.revenue:>10,.0f} {c.tier:<12} {len(c.cases):>5}")
    
    # Attorney billing
    print(f"\n👤 ATTORNEY BILLING:")
    attorneys = session.query(Attorney).all()
    for a in attorneys:
        total_hours = sum(c.hours for c in a.cases)
        total_billing = sum(c.billing for c in a.cases)
        print(f"  {a.name:<20} {a.role:<12} {total_hours:>5.0f}h  ${total_billing:>10,.0f}")
    
    # Case status breakdown
    print(f"\n📊 CASE STATUS:")
    status_stats = (
        session.query(Case.status, func.count(Case.id), func.sum(Case.hours))
        .group_by(Case.status)
        .all()
    )
    for status, count, hours in status_stats:
        print(f"  {status:<10}: {count} cases, {hours:.0f} hours")
    
    # Invoice summary
    total_invoiced = session.query(func.sum(Invoice.amount)).scalar()
    total_paid = session.query(func.sum(Invoice.amount)).filter(Invoice.paid == True).scalar() or 0
    total_outstanding = total_invoiced - total_paid
    
    print(f"\n💰 INVOICE SUMMARY:")
    print(f"  Total invoiced:    ${total_invoiced:>12,.2f}")
    print(f"  Paid:              ${total_paid:>12,.2f}")
    print(f"  Outstanding:       ${total_outstanding:>12,.2f}")
    
    # Navigate relationships
    print(f"\n🔗 RELATIONSHIP NAVIGATION:")
    wayne = session.query(Client).filter_by(name="Wayne Enterprises").first()
    print(f"  {wayne.name} has {len(wayne.cases)} cases:")
    for case in wayne.cases:
        print(f"    📋 {case.title} ({case.status})")
        print(f"       Attorney: {case.attorney.name} at ${case.attorney.hourly_rate}/hr")
        print(f"       Hours: {case.hours}h → Billing: ${case.billing:,.2f}")
        print(f"       Invoices: {len(case.invoices)}")

print(f"\n{'='*70}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Forgetting `session.commit()` 🔥

```python
with Session(engine) as session:
    client = Client(name="New Client")
    session.add(client)
    # ❌ Forgot commit!
    # Nothing saved to database!

# ✅ Always commit after changes
with Session(engine) as session:
    client = Client(name="New Client")
    session.add(client)
    session.commit()    # ✅ Now it's saved
```

---

### Pitfall 2: Using Objects Outside Their Session 🔥

```python
# ❌ DetachedInstanceError
with Session(engine) as session:
    client = session.query(Client).first()

# Session is closed here!
# print(client.cases)    # DetachedInstanceError! Lazy load can't reach DB!

# ✅ Fix 1: Access everything inside the session
with Session(engine) as session:
    client = session.query(Client).first()
    cases = client.cases    # Access while session is open ✅

# ✅ Fix 2: Eager loading
from sqlalchemy.orm import joinedload

with Session(engine) as session:
    client = (
        session.query(Client)
        .options(joinedload(Client.cases))    # Load cases immediately
        .first()
    )

# Now cases are loaded even after session closes
print(client.cases)    # ✅ Works!
```

---

### Pitfall 3: N+1 Query Problem

```python
# ❌ N+1: one query for clients + one query per client's cases
with Session(engine) as session:
    clients = session.query(Client).all()    # 1 query
    for c in clients:
        print(c.cases)    # N additional queries! One per client!

# ✅ Fix: eager loading with joinedload
from sqlalchemy.orm import joinedload

with Session(engine) as session:
    clients = (
        session.query(Client)
        .options(joinedload(Client.cases))    # 1 query with JOIN
        .all()
    )
    for c in clients:
        print(c.cases)    # No extra queries!
```

---

### Pitfall 4: Not Handling Unique Constraint Violations

```python
from sqlalchemy.exc import IntegrityError

# ❌ Crashes on duplicate
with Session(engine) as session:
    session.add(Client(name="Wayne Enterprises"))    # Already exists!
    session.commit()    # IntegrityError!

# ✅ Handle gracefully
with Session(engine) as session:
    try:
        session.add(Client(name="Wayne Enterprises"))
        session.commit()
    except IntegrityError:
        session.rollback()
        print("Client already exists!")
```

---

### Pitfall 5: Raw SQL When ORM Suffices

```python
# ❌ Raw SQL in an ORM project — defeats the purpose
with Session(engine) as session:
    result = session.execute("SELECT * FROM clients WHERE revenue > 1000000")

# ✅ Use ORM queries
with Session(engine) as session:
    clients = session.query(Client).filter(Client.revenue > 1000000).all()

# Sometimes raw SQL IS appropriate (complex reports, DB-specific features)
# But use text() for safety:
from sqlalchemy import text
with Session(engine) as session:
    result = session.execute(text("SELECT * FROM clients WHERE revenue > :min"), {"min": 1000000})
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📁 Analogy: ORM = Donna's Filing System

```
RAW SQL:
  📝 You write a note: "Please find file #2026-001 in cabinet B, 
     drawer 3, folder wayne, and bring me the revenue field."
  → Error-prone, verbose, you need to know the physical layout.

ORM:
  📞 You call Donna: "Get me Wayne's revenue."
  → Donna knows where everything is.
  → She translates your request into the filing system.
  → You get a Python object back: client.revenue

DONNA'S RULE:
  "You don't need to know which drawer the file is in.
   That's my job. You just tell me what you need —
   in Python, not SQL."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Simple Model**
```
Create a Book model: id, title, author, year, genre, rating.
Insert 5 books. Query: all books, by genre, top-rated.
```

**Exercise 2: CRUD Operations**
```
Create a Task model: id, title, done, priority.
Implement: add, list, complete, delete, list_by_priority.
```

**Exercise 3: Filtering Practice**
```
With 20+ records, practice:
  - filter_by, filter with operators (>, <, ==)
  - like/ilike, in_, order_by, limit
  - Aggregations: count, sum, avg, max
```

---

### 🟡 Intermediate Exercises

**Exercise 4: One-to-Many**
```
Models: Author → Books (one author, many books).
Query: author with most books, books by a specific author,
authors with no books, total books per author.
```

**Exercise 5: Many-to-Many**
```
Models: Student ↔ Course (many students, many courses).
Association table: enrollments.
Query: students in a course, courses for a student, 
popular courses, students taking multiple courses.
```

**Exercise 6: Blog System**
```
Models: User → Post → Comment.
With timestamps, status, tags.
Query: posts by user, comments on a post,
most commented post, recent posts.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Repository Pattern**
```
Build a generic Repository class:
  repo = Repository(Client, session)
  repo.create(name="Wayne", revenue=2500000)
  repo.find(id=1)
  repo.find_all(tier="Platinum")
  repo.update(1, revenue=3000000)
  repo.delete(1)
```

**Exercise 8: Query Builder**
```
Build a fluent query builder:
  results = (
    Query(Client)
    .where(revenue__gt=1000000)
    .where(active=True)
    .order_by("-revenue")
    .limit(10)
    .execute(session)
  )
```

**Exercise 9: Seed + Report Script**
```
Build a complete script:
  1. Create schema from models
  2. Seed with realistic data (50+ records)
  3. Generate formatted report
  4. Export to JSON
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Forgot commit
session.add(Client(name="Test"))
# No commit — not saved!

# Bug 2: Accessing lazy-loaded relationship outside session
with Session(engine) as s:
    c = s.query(Client).first()
print(c.cases)    # DetachedInstanceError!

# Bug 3: Duplicate key
session.add(Client(name="Wayne Enterprises"))    # Already exists!
session.commit()    # IntegrityError!

# Bug 4: Wrong filter syntax
session.query(Client).filter(name="Wayne")    # TypeError!

# Bug 5: Missing ForeignKey
class Case(Base):
    client_id = Column(Integer)    # Missing ForeignKey!
```

### Fixed Code ✅

```python
# Fix 1: Add commit
session.add(Client(name="Test"))
session.commit()    # ✅

# Fix 2: Access inside session or use eager loading
with Session(engine) as s:
    c = s.query(Client).options(joinedload(Client.cases)).first()
    cases = c.cases    # ✅ Loaded eagerly

# Fix 3: Handle IntegrityError
try:
    session.add(Client(name="Wayne Enterprises"))
    session.commit()
except IntegrityError:
    session.rollback()    # ✅

# Fix 4: Use filter_by for keyword args, filter for expressions
session.query(Client).filter_by(name="Wayne")         # ✅ filter_by
session.query(Client).filter(Client.name == "Wayne")  # ✅ filter

# Fix 5: Add ForeignKey
client_id = Column(Integer, ForeignKey('clients.id'))    # ✅
```

---

## 8. 🌍 Real-World Application

### ORMs in the Real World

| Framework | ORM |
|-----------|-----|
| **Django** | Django ORM (built-in) |
| **Flask** | SQLAlchemy (via Flask-SQLAlchemy) |
| **FastAPI** | SQLAlchemy / Tortoise ORM |
| **Pyramid** | SQLAlchemy |
| **SQLModel** | SQLAlchemy + Pydantic hybrid |
| **Peewee** | Lightweight alternative |
| **Tortoise** | Async ORM for asyncio |

---

## 9. ✅ Knowledge Check

---

**Q1.** What does ORM stand for?

**Q2.** What does a model class map to?

**Q3.** How do you create a new record?

**Q4.** What is the difference between `filter()` and `filter_by()`?

**Q5.** What does `session.commit()` do?

**Q6.** How do you define a one-to-many relationship?

**Q7.** What is the N+1 query problem?

**Q8.** What is eager loading and why use it?

**Q9.** What does `session.rollback()` do?

**Q10.** What is Alembic used for?

---

### 📋 Answer Key

> **Q1.** Object-Relational Mapper — maps Python objects to database tables.

> **Q2.** A database table. Each class attribute maps to a column, each instance is a row.

> **Q3.** Create an instance (`client = Client(name="Wayne")`), add it (`session.add(client)`), commit (`session.commit()`).

> **Q4.** `filter_by()` uses keyword arguments (`filter_by(name="Wayne")`). `filter()` uses expressions (`filter(Client.name == "Wayne")`). `filter()` supports more complex conditions.

> **Q5.** Flushes pending changes to the database and commits the current transaction, making changes permanent.

> **Q6.** Add a `ForeignKey` column on the "many" side and a `relationship()` on both sides with `back_populates`.

> **Q7.** When iterating over N parent objects triggers N separate queries for their children. Fix with `joinedload()`.

> **Q8.** Loading related objects in the same query (JOIN) instead of separate lazy queries. Prevents N+1 problems.

> **Q9.** Undoes all uncommitted changes in the current transaction, restoring the session to its last committed state.

> **Q10.** Database migration management — tracks, generates, and applies schema changes over time.

---

## 🎬 End of Lesson Scene

**Harvey:** "So instead of writing SQL, I just…"

**Mike:** "Write Python. `client = session.query(Client).filter_by(name='Wayne').first()`. That generates the SQL, executes it, and returns a Python object with attribute access, relationships, and type safety. No SQL injection, no tuple indexing, no raw strings."

**Harvey:** "And when we change the schema?"

**Mike:** "Alembic generates a migration. One command updates every database — dev, staging, production."

**Harvey:** "Final lesson next. How Python itself runs."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | SQLAlchemy setup and engine | ✅ |
| 2 | Model definition (Column, types, constraints) | ✅ |
| 3 | SQLAlchemy 2.0 Mapped columns | ✅ |
| 4 | CRUD operations | ✅ |
| 5 | Filters, ordering, aggregation | ✅ |
| 6 | Relationships and navigation | ✅ |
| 7 | Modern select() API | ✅ |
| 8 | Sessions and transactions | ✅ |
| 9 | Alembic migrations overview | ✅ |
| 10 | N+1 problem and eager loading | ✅ |

---

> **Next Lesson →** Level 4, Lesson 10: *Compare Python Implementations (CPython, PyPy, etc.)*
