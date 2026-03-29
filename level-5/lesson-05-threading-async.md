# 🏛️ Level 5 – Innovating | Lesson 5: Write Multi-Threaded and Asynchronous Code in Python

## *"All at Once"*

> **O'Reilly Skill Objective:** *Reason about the Global Interpreter Lock (GIL) and its effect on threading. Write multi-threaded code using `concurrent.futures` or `threading`. Write asynchronous code using `asyncio`.*

> **Previously Learned:** Functions, decorators, context managers, generators, `yield`, exception handling, GIL (Level 4 Lesson 10)

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

*The firm needs to pull case updates from 50 court APIs, process 500 documents, and send 200 notification emails — all before the 9 AM meeting.*

```python
# ❌ Sequential: one at a time
for api in court_apis:
    result = fetch(api)        # Waits 2 seconds each
    process(result)            # 50 APIs × 2s = 100 seconds!

# ✅ Concurrent: all at once
import asyncio

async def main():
    results = await asyncio.gather(
        *[fetch(api) for api in court_apis]
    )    # 50 APIs × 2s = ~2 seconds total!
```

**Harvey:** "I don't do things one at a time. Neither should your code."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Three ways to do things simultaneously in Python. Each for a different problem."

| Approach | Module | Best For | GIL? |
|----------|--------|----------|:----:|
| **Threading** | `threading` / `concurrent.futures` | I/O-bound (network, file, DB) | Held per thread |
| **Multiprocessing** | `multiprocessing` / `concurrent.futures` | CPU-bound (computation) | Separate per process |
| **Asyncio** | `asyncio` | Many I/O operations (1000s of connections) | Single thread |

```
SEQUENTIAL:     ████████████████████████████████████████  40s
THREADING:      ████████████                              12s (I/O overlaps)
MULTIPROCESSING:████████                                   8s (true parallel)
ASYNCIO:        ██████████                                10s (cooperative)
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Threading Basics ⭐

```python
import threading
import time

def fetch_court_data(court_name, delay=2):
    """Simulate a slow API call."""
    print(f"  📡 Fetching {court_name}...")
    time.sleep(delay)    # Simulates network wait
    print(f"  ✅ {court_name} complete")
    return f"{court_name}: 5 cases"

# === Sequential (slow) ===
start = time.perf_counter()
for court in ["Supreme", "Federal", "District"]:
    fetch_court_data(court)
sequential = time.perf_counter() - start
print(f"Sequential: {sequential:.1f}s")    # ~6s

# === Threaded (fast for I/O) ===
start = time.perf_counter()
threads = []
for court in ["Supreme", "Federal", "District"]:
    t = threading.Thread(target=fetch_court_data, args=(court,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()    # Wait for all threads to complete
threaded = time.perf_counter() - start
print(f"Threaded: {threaded:.1f}s")    # ~2s (all run simultaneously!)
```

---

### 3.2 Sharing Data Between Threads — Locks ⭐⭐

```python
import threading

# ❌ Race condition without locks!
counter = 0

def increment_unsafe(n):
    global counter
    for _ in range(n):
        counter += 1    # NOT atomic! Read-modify-write can interleave!

threads = [threading.Thread(target=increment_unsafe, args=(100_000,)) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()
print(f"Unsafe: {counter:,} (expected 500,000)")    # Less than 500,000!

# ✅ Thread-safe with Lock
counter = 0
lock = threading.Lock()

def increment_safe(n):
    global counter
    for _ in range(n):
        with lock:             # Only ONE thread can enter at a time
            counter += 1

threads = [threading.Thread(target=increment_safe, args=(100_000,)) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()
print(f"Safe: {counter:,} (expected 500,000)")    # Exactly 500,000 ✅

# Other synchronization primitives:
# threading.RLock()      — re-entrant lock (same thread can acquire multiple times)
# threading.Semaphore(n) — allows n threads simultaneously
# threading.Event()      — one thread signals, others wait
# threading.Barrier(n)   — n threads wait for each other
# queue.Queue()          — thread-safe data exchange (best practice!)
```

---

### 3.3 `concurrent.futures` — The High-Level API ⭐⭐

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed
import time

def fetch_api(url):
    """Simulate API fetch."""
    time.sleep(1)    # Network delay
    return f"Data from {url}"

def crunch_numbers(n):
    """CPU-intensive work."""
    return sum(i * i for i in range(n))

# === ThreadPoolExecutor — for I/O-bound work ===
urls = [f"https://court-{i}.gov/api" for i in range(10)]

start = time.perf_counter()
with ThreadPoolExecutor(max_workers=5) as executor:
    # submit() — one at a time, get Future objects
    futures = [executor.submit(fetch_api, url) for url in urls]
    
    # Collect results as they complete
    for future in as_completed(futures):
        result = future.result()
        print(f"  Got: {result}")

elapsed = time.perf_counter() - start
print(f"ThreadPool ({len(urls)} URLs): {elapsed:.1f}s")    # ~2s (not 10s!)

# === map() — simpler when order matters ===
with ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(fetch_api, urls))
    # Results are in ORDER (unlike as_completed)

# === ProcessPoolExecutor — for CPU-bound work ===
numbers = [10_000_000] * 4

start = time.perf_counter()
with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(crunch_numbers, numbers))
elapsed = time.perf_counter() - start
print(f"ProcessPool: {elapsed:.1f}s")    # ~Nx faster on N cores!
```

---

### 3.4 `queue.Queue` — Thread-Safe Communication ⭐

```python
import threading
import queue
import time

# Queue is the BEST way for threads to exchange data
# No locks needed — it's thread-safe by design!

def producer(q, items):
    """Produces work items."""
    for item in items:
        time.sleep(0.5)    # Simulate finding work
        q.put(item)
        print(f"  📥 Produced: {item}")
    q.put(None)    # Sentinel: signals "done"

def consumer(q, name):
    """Consumes work items."""
    while True:
        item = q.get()    # Blocks until item available
        if item is None:
            q.put(None)   # Pass sentinel to next consumer
            break
        time.sleep(0.3)   # Simulate processing
        print(f"  📤 {name} processed: {item}")
        q.task_done()

# Create queue
work_queue = queue.Queue(maxsize=10)    # Max 10 items buffered

# Start workers
cases = ["WAYNE-001", "STARK-002", "OSCORP-003", "QUEEN-004", "LUTHOR-005"]

prod = threading.Thread(target=producer, args=(work_queue, cases))
cons1 = threading.Thread(target=consumer, args=(work_queue, "Worker-1"))
cons2 = threading.Thread(target=consumer, args=(work_queue, "Worker-2"))

prod.start()
cons1.start()
cons2.start()

prod.join()
cons1.join()
cons2.join()
```

---

### 3.5 Asyncio Basics — `async`/`await` ⭐⭐

```python
import asyncio
import time

# asyncio = COOPERATIVE multitasking in a SINGLE thread
# Functions voluntarily yield control at 'await' points

async def fetch_court(court_name):
    """Async function (coroutine)."""
    print(f"  📡 Fetching {court_name}...")
    await asyncio.sleep(2)    # Non-blocking sleep! Yields control.
    print(f"  ✅ {court_name} done")
    return f"{court_name}: 5 cases"

# === Running a single coroutine ===
async def main():
    result = await fetch_court("Supreme")
    print(result)

asyncio.run(main())

# === Running multiple coroutines concurrently ===
async def main_concurrent():
    start = time.perf_counter()
    
    # asyncio.gather — run all concurrently
    results = await asyncio.gather(
        fetch_court("Supreme"),
        fetch_court("Federal"),
        fetch_court("District"),
    )
    
    elapsed = time.perf_counter() - start
    print(f"\nAll done in {elapsed:.1f}s")    # ~2s (not 6s!)
    for r in results:
        print(f"  {r}")

asyncio.run(main_concurrent())
```

---

### 3.6 Asyncio Patterns — Tasks, Timeouts, Semaphores ⭐⭐

```python
import asyncio

# === Creating Tasks ===
async def main():
    # Tasks run in the background immediately
    task1 = asyncio.create_task(fetch_court("Supreme"))
    task2 = asyncio.create_task(fetch_court("Federal"))
    
    # Do other work while tasks run...
    print("  ⏳ Doing other work...")
    await asyncio.sleep(1)
    
    # Collect results
    result1 = await task1
    result2 = await task2
    print(f"  Results: {result1}, {result2}")

# === Timeouts ===
async def fetch_with_timeout():
    try:
        result = await asyncio.wait_for(
            fetch_court("Slow Court"),
            timeout=1.0    # Max 1 second!
        )
    except asyncio.TimeoutError:
        print("  ⏰ Timed out!")

# === Semaphore — limit concurrency ===
semaphore = asyncio.Semaphore(5)    # Max 5 concurrent

async def limited_fetch(court_name):
    async with semaphore:    # Only 5 at a time
        return await fetch_court(court_name)

async def fetch_all_limited():
    courts = [f"Court-{i}" for i in range(20)]
    tasks = [limited_fetch(c) for c in courts]
    results = await asyncio.gather(*tasks)
    return results

# === asyncio.wait — more control ===
async def wait_example():
    tasks = [asyncio.create_task(fetch_court(f"Court-{i}")) for i in range(5)]
    
    # Wait for first to complete
    done, pending = await asyncio.wait(tasks, return_when=asyncio.FIRST_COMPLETED)
    print(f"First done: {done.pop().result()}")
    
    # Wait for the rest
    done2, _ = await asyncio.wait(pending)
```

---

### 3.7 Async Context Managers and Iterators ⭐

```python
import asyncio

# === Async Context Manager ===
class AsyncDatabase:
    """Async database connection."""
    
    async def connect(self):
        print("  🔌 Connecting...")
        await asyncio.sleep(0.5)
        print("  ✅ Connected")
    
    async def disconnect(self):
        print("  🔌 Disconnecting...")
        await asyncio.sleep(0.2)
    
    async def query(self, sql):
        await asyncio.sleep(0.3)
        return [{"id": 1, "name": "Wayne"}, {"id": 2, "name": "Stark"}]
    
    async def __aenter__(self):
        await self.connect()
        return self
    
    async def __aexit__(self, *args):
        await self.disconnect()

async def main():
    async with AsyncDatabase() as db:
        results = await db.query("SELECT * FROM clients")
        print(f"  Results: {results}")

# === Async Iterator ===
class AsyncCaseStream:
    """Async iterator — yields cases one at a time."""
    
    def __init__(self, case_ids):
        self.case_ids = case_ids
        self.index = 0
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        if self.index >= len(self.case_ids):
            raise StopAsyncIteration
        case_id = self.case_ids[self.index]
        self.index += 1
        await asyncio.sleep(0.1)    # Simulate async fetch
        return {"id": case_id, "status": "open"}

async def main():
    async for case in AsyncCaseStream(["W-001", "S-002", "O-003"]):
        print(f"  Case: {case}")

# === Async Generator ===
async def case_stream(case_ids):
    for case_id in case_ids:
        await asyncio.sleep(0.1)
        yield {"id": case_id, "status": "open"}

async def main():
    async for case in case_stream(["W-001", "S-002"]):
        print(f"  Case: {case}")
```

---

### 3.8 Threading vs Asyncio — When to Use Which ⭐

```python
# ┌────────────────────┬──────────────────┬──────────────────┐
# │                    │ THREADING         │ ASYNCIO           │
# ├────────────────────┼──────────────────┼──────────────────┤
# │ Concurrency model  │ Preemptive        │ Cooperative       │
# │ Threads            │ Multiple OS       │ Single thread     │
# │ Switching          │ OS decides        │ You decide (await)│
# │ Shared state       │ Needs locks       │ No locks needed   │
# │ Blocking calls     │ ✅ Supported      │ ❌ Blocks loop!   │
# │ Libraries          │ Any (requests)    │ Async only (aiohttp)│
# │ Scalability        │ 100s of threads   │ 10,000s of tasks  │
# │ Debugging          │ Race conditions   │ Stack traces OK   │
# │ Best for           │ Legacy code, I/O  │ New async I/O     │
# └────────────────────┴──────────────────┴──────────────────┘

# === Threading: use with synchronous libraries ===
import requests    # Blocking library

def fetch_sync(url):
    return requests.get(url).text

# Works fine in threads:
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor(5) as pool:
    results = list(pool.map(fetch_sync, urls))

# === Asyncio: use with async libraries ===
# pip install aiohttp
# import aiohttp
#
# async def fetch_async(url):
#     async with aiohttp.ClientSession() as session:
#         async with session.get(url) as response:
#             return await response.text()
#
# async def main():
#     tasks = [fetch_async(url) for url in urls]
#     results = await asyncio.gather(*tasks)
```

---

### 3.9 Combining Async with Threading ⭐

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor
import time

def blocking_io(name):
    """A blocking function (can't use await)."""
    time.sleep(1)
    return f"Result from {name}"

async def main():
    loop = asyncio.get_running_loop()
    
    # Run blocking code in a thread pool from async context
    result = await loop.run_in_executor(
        None,    # Default executor (ThreadPoolExecutor)
        blocking_io,
        "court-api"
    )
    print(result)    # Result from court-api
    
    # Multiple blocking calls concurrently
    tasks = [
        loop.run_in_executor(None, blocking_io, f"api-{i}")
        for i in range(5)
    ]
    results = await asyncio.gather(*tasks)
    print(results)    # All 5 results in ~1 second!

asyncio.run(main())

# asyncio.to_thread (Python 3.9+) — simpler!
async def main_simple():
    result = await asyncio.to_thread(blocking_io, "court-api")
    print(result)
```

---

### 3.10 Solving the Firm's Problem — Concurrent Case Pipeline

```python
# ============================================
# Pearson Specter Litt — Concurrent Pipeline
# Fetch, process, and notify — all at once
# ============================================

import asyncio
import time
import random
from dataclasses import dataclass

@dataclass
class CaseUpdate:
    case_id: str
    client: str
    status: str
    details: str
    fetched_in: float = 0.0

# === Simulated API calls ===
async def fetch_court_api(case_id):
    """Simulate fetching from court API (I/O bound)."""
    start = time.perf_counter()
    delay = random.uniform(0.5, 2.0)
    await asyncio.sleep(delay)
    
    statuses = ["Motion Filed", "Hearing Scheduled", "Discovery Complete", 
                "Settlement Offered", "Trial Date Set"]
    
    return CaseUpdate(
        case_id=case_id,
        client=case_id.split("-")[0],
        status=random.choice(statuses),
        details=f"Updated at court system",
        fetched_in=time.perf_counter() - start,
    )

async def process_document(case_id):
    """Simulate document processing."""
    await asyncio.sleep(random.uniform(0.2, 0.8))
    return {"case_id": case_id, "pages": random.randint(10, 200), "status": "processed"}

async def send_notification(recipient, message):
    """Simulate sending notification."""
    await asyncio.sleep(random.uniform(0.1, 0.3))
    return {"to": recipient, "status": "sent"}


# === Pipeline orchestrator ===
async def run_pipeline():
    """Fetch, process, and notify — concurrently."""
    
    cases = [
        "WAYNE-2026-001", "WAYNE-2026-002",
        "STARK-2026-001", "STARK-2026-002",
        "OSCORP-2026-001", "QUEEN-2026-001",
        "LUTHOR-2026-001", "PALMER-2026-001",
    ]
    
    attorneys = {
        "WAYNE": "Harvey", "STARK": "Harvey",
        "OSCORP": "Mike", "QUEEN": "Louis",
        "LUTHOR": "Mike", "PALMER": "Katrina",
    }
    
    print("=" * 65)
    print(f"{'🏛️  CONCURRENT CASE PIPELINE':^65}")
    print("=" * 65)
    
    overall_start = time.perf_counter()
    
    # Phase 1: Fetch all court updates concurrently
    print(f"\n📡 Phase 1: Fetching {len(cases)} court APIs...")
    semaphore = asyncio.Semaphore(4)    # Max 4 concurrent API calls
    
    async def limited_fetch(case_id):
        async with semaphore:
            return await fetch_court_api(case_id)
    
    fetch_start = time.perf_counter()
    updates = await asyncio.gather(*[limited_fetch(c) for c in cases])
    fetch_time = time.perf_counter() - fetch_start
    
    for u in updates:
        print(f"  ✅ {u.case_id}: {u.status} ({u.fetched_in:.2f}s)")
    print(f"  ⏱️ All fetched in {fetch_time:.2f}s (sequential would be ~{sum(u.fetched_in for u in updates):.1f}s)")
    
    # Phase 2: Process documents concurrently
    print(f"\n📄 Phase 2: Processing {len(cases)} documents...")
    process_start = time.perf_counter()
    docs = await asyncio.gather(*[process_document(c) for c in cases])
    process_time = time.perf_counter() - process_start
    
    total_pages = sum(d["pages"] for d in docs)
    print(f"  ✅ {len(docs)} documents, {total_pages} total pages in {process_time:.2f}s")
    
    # Phase 3: Send notifications concurrently
    print(f"\n📧 Phase 3: Sending notifications...")
    notifications = []
    for update in updates:
        client_prefix = update.client
        attorney = attorneys.get(client_prefix, "Unassigned")
        msg = f"{update.case_id}: {update.status}"
        notifications.append(send_notification(attorney, msg))
    
    notify_start = time.perf_counter()
    results = await asyncio.gather(*notifications)
    notify_time = time.perf_counter() - notify_start
    
    sent = sum(1 for r in results if r["status"] == "sent")
    print(f"  ✅ {sent}/{len(results)} notifications sent in {notify_time:.2f}s")
    
    # Summary
    total_time = time.perf_counter() - overall_start
    sequential_est = sum(u.fetched_in for u in updates) + process_time * 3 + notify_time * 3
    
    print(f"\n📊 PIPELINE SUMMARY:")
    print(f"  Cases processed: {len(cases)}")
    print(f"  Documents:       {total_pages} pages")
    print(f"  Notifications:   {sent} sent")
    print(f"  Total time:      {total_time:.2f}s")
    print(f"  Est. sequential: {sequential_est:.1f}s")
    print(f"  Speedup:         {sequential_est/total_time:.1f}x")
    
    print(f"\n{'='*65}")

asyncio.run(run_pipeline())
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Using Threads for CPU-Bound Work 🔥

```python
import threading, time

# ❌ Threads DON'T speed up CPU work (GIL blocks parallelism!)
def cpu_heavy():
    return sum(i * i for i in range(5_000_000))

start = time.perf_counter()
threads = [threading.Thread(target=cpu_heavy) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()
threaded = time.perf_counter() - start
# Threaded: ~same as single-threaded!

# ✅ Use ProcessPoolExecutor for CPU work
from concurrent.futures import ProcessPoolExecutor
with ProcessPoolExecutor(4) as pool:
    results = list(pool.map(cpu_heavy, range(4)))
# Actually ~4x faster!
```

---

### Pitfall 2: Blocking Calls in Asyncio 🔥

```python
import asyncio, time

# ❌ time.sleep blocks the ENTIRE event loop!
async def bad():
    time.sleep(2)    # Blocks everything — no other tasks run!

# ✅ Use asyncio.sleep for async code
async def good():
    await asyncio.sleep(2)    # Yields control — other tasks can run!

# ✅ For blocking libraries, use run_in_executor
async def with_blocking():
    loop = asyncio.get_running_loop()
    await loop.run_in_executor(None, time.sleep, 2)
```

---

### Pitfall 3: Race Conditions

```python
import threading

# ❌ Shared mutable state without locks
results = []

def worker(item):
    result = process(item)
    results.append(result)    # NOT guaranteed thread-safe for complex ops!

# ✅ Use queue.Queue for thread communication
import queue

result_queue = queue.Queue()

def safe_worker(item):
    result = process(item)
    result_queue.put(result)    # Thread-safe!
```

---

### Pitfall 4: Forgetting to `await`

```python
# ❌ Forgot await — gets a coroutine object, not the result!
async def main():
    result = fetch_court("Supreme")    # Missing await!
    print(result)    # <coroutine object fetch_court at 0x...>

# ✅ Always await coroutines
async def main():
    result = await fetch_court("Supreme")
    print(result)    # "Supreme: 5 cases"
```

---

### Pitfall 5: Daemon Threads and Lost Work

```python
import threading, time

# ❌ Daemon threads die when main thread exits!
def background_task():
    time.sleep(5)
    print("Done!")    # May never print!

t = threading.Thread(target=background_task, daemon=True)
t.start()
# Main thread exits → daemon killed → "Done!" never prints!

# ✅ Join non-daemon threads
t = threading.Thread(target=background_task)
t.start()
t.join()    # Wait for completion
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🏢 Analogy: The Law Firm Office

```
SEQUENTIAL:
  Harvey handles each case alone, one at a time.
  Case 1 → Case 2 → Case 3.
  Simple but slow.

THREADING:
  Harvey, Mike, and Louis each take a case.
  They share the conference room (GIL) — only one
  can present at a time, but they prepare simultaneously.
  Great when most time is spent WAITING (for courts, clients).

MULTIPROCESSING:
  Each attorney has their OWN office, OWN resources.
  True parallel work, no sharing bottleneck.
  Great for heavy research (CPU work).

ASYNCIO:
  Donna manages 50 phone calls from ONE desk.
  She puts each caller on hold (await), handles the next,
  comes back when each is ready.
  One person, many tasks, no waiting.

DONNA'S RULE:
  "Waiting for the court? → Threading or asyncio.
   Heavy number crunching? → Multiprocessing.
   1000 simultaneous connections? → Asyncio.
   Simple script? → Don't bother — keep it sequential."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Threaded Downloader**
```
Download 5 web pages using ThreadPoolExecutor.
Compare time: sequential vs threaded.
```

**Exercise 2: Async Timer**
```
Create 3 async timers that count down simultaneously:
  asyncio.gather(countdown("A", 5), countdown("B", 3), countdown("C", 7))
```

**Exercise 3: Producer-Consumer**
```
Create a producer thread that generates numbers 1-20,
and 2 consumer threads that process them.
Use queue.Queue for safe communication.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Parallel File Processor**
```
Process 50 text files using ThreadPoolExecutor:
  - Count words in each file
  - Collect results
  - Report total words across all files
Compare with sequential processing.
```

**Exercise 5: Async Web Scraper**
```
Build an async scraper with rate limiting (Semaphore):
  - Fetch 20 URLs concurrently
  - Max 5 concurrent connections
  - Timeout after 10 seconds per URL
  - Retry failed requests once
```

**Exercise 6: Thread-Safe Cache**
```
Build a thread-safe LRU cache:
  cache = ThreadSafeCache(max_size=100)
  cache.get(key)  → value or None
  cache.set(key, value)
  Multiple threads reading/writing simultaneously.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Task Scheduler**
```
Build an async task scheduler:
  scheduler = Scheduler()
  scheduler.every(5).seconds.do(check_api)
  scheduler.every(1).hour.do(send_report)
  await scheduler.run()
```

**Exercise 8: Worker Pool**
```
Build a worker pool that auto-selects threading vs
multiprocessing based on task type:
  pool = SmartPool()
  pool.submit(cpu_task, type="cpu")    → ProcessPool
  pool.submit(io_task, type="io")      → ThreadPool
```

**Exercise 9: Async Pipeline**
```
Build an async data pipeline:
  Pipeline()
    .source(async_fetch_cases)
    .filter(async_check_status)
    .transform(async_enrich_data)
    .sink(async_save_to_db)
    .run(concurrency=10)
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Blocking call in async
async def bad_async():
    time.sleep(5)    # Blocks the entire event loop!

# Bug 2: Threads for CPU work
for _ in range(4):
    threading.Thread(target=cpu_heavy).start()    # No speedup!

# Bug 3: Race condition
total = 0
def add(n):
    global total
    for _ in range(n):
        total += 1    # Not atomic!

# Bug 4: Forgot to join threads
t = threading.Thread(target=long_task)
t.start()
print("Done!")    # Prints before task is done!

# Bug 5: Forgot await
async def main():
    result = fetch_data()    # Returns coroutine, not result!
```

### Fixed Code ✅

```python
# Fix 1: Use async sleep
async def good_async():
    await asyncio.sleep(5)    # Other tasks can run!

# Fix 2: Use multiprocessing
with ProcessPoolExecutor(4) as pool:
    pool.map(cpu_heavy, range(4))    # True parallelism!

# Fix 3: Use lock
lock = threading.Lock()
def add_safe(n):
    global total
    for _ in range(n):
        with lock: total += 1

# Fix 4: Join!
t.start()
t.join()    # Wait for completion
print("Done!")

# Fix 5: Add await
async def main():
    result = await fetch_data()
```

---

## 8. 🌍 Real-World Application

### Concurrency in Production

| Service | Approach | Why |
|---------|----------|-----|
| **Django** | Threading (WSGI) | Handles concurrent HTTP requests |
| **FastAPI** | Asyncio (ASGI) | High-concurrency APIs |
| **Celery** | Multiprocessing | Background task queue |
| **Scrapy** | Asyncio (Twisted) | Web scraping at scale |
| **aiohttp** | Asyncio | Async HTTP client/server |
| **Gunicorn** | Multiprocessing + threading | Production WSGI server |
| **Uvicorn** | Asyncio | Production ASGI server |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is the difference between concurrency and parallelism?

**Q2.** Why don't threads speed up CPU-bound Python code?

**Q3.** What is `concurrent.futures.ThreadPoolExecutor`?

**Q4.** What is `asyncio.gather()`?

**Q5.** What happens if you call `time.sleep()` inside an async function?

**Q6.** What is a `queue.Queue` and why use it with threads?

**Q7.** When should you use `ProcessPoolExecutor`?

**Q8.** What is an async context manager?

**Q9.** How do you limit concurrency in asyncio?

**Q10.** When should you NOT use concurrency?

---

### 📋 Answer Key

> **Q1.** Concurrency = managing multiple tasks at once (may interleave on one core). Parallelism = executing multiple tasks at the same time (requires multiple cores).

> **Q2.** The GIL allows only one thread to execute Python bytecode at a time. Threads take turns instead of running truly in parallel.

> **Q3.** A high-level pool of threads. Submit tasks, get `Future` objects, collect results. Manages thread creation/cleanup automatically.

> **Q4.** Runs multiple coroutines concurrently and returns all results. `await asyncio.gather(coro1(), coro2(), coro3())`.

> **Q5.** It BLOCKS the entire event loop — no other async tasks can run. Use `await asyncio.sleep()` or `run_in_executor()` instead.

> **Q6.** A thread-safe FIFO queue. Threads can safely put and get items without locks. Best practice for inter-thread communication.

> **Q7.** For CPU-bound work. Each process has its own interpreter and GIL, enabling true parallelism on multiple cores.

> **Q8.** A class with `__aenter__` and `__aexit__` async methods. Used with `async with` for async setup/teardown (e.g., database connections).

> **Q9.** Use `asyncio.Semaphore(n)`: `async with semaphore:` limits to n concurrent tasks.

> **Q10.** For simple scripts, CPU-bound work with I/O libraries, or when the overhead exceeds the benefit. Profile first.

---

## 🎬 End of Lesson Scene

**Harvey:** "The morning report?"

**Mike:** "50 court APIs fetched, 500 documents processed, 200 notifications sent. All before 9 AM."

**Harvey:** "How long?"

**Mike:** "4.2 seconds. Sequentially it would have been 8 minutes. Asyncio for the APIs, thread pool for the documents, asyncio again for the notifications."

**Harvey:** "Next: design patterns. The architecture behind the architecture."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Threading basics and `threading.Thread` | ✅ |
| 2 | Locks and thread synchronization | ✅ |
| 3 | `concurrent.futures` (ThreadPool/ProcessPool) | ✅ |
| 4 | `queue.Queue` for thread communication | ✅ |
| 5 | Asyncio basics (`async`/`await`) | ✅ |
| 6 | `asyncio.gather`, tasks, timeouts, semaphores | ✅ |
| 7 | Async context managers and iterators | ✅ |
| 8 | Threading vs asyncio decision criteria | ✅ |
| 9 | Combining async with threading | ✅ |
| 10 | Concurrent pipeline architecture | ✅ |

---

> **Next Lesson →** Level 5, Lesson 6: *Apply Common Design Patterns in Python*
