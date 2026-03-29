# 🏛️ Level 5 – Innovating | Lesson 1: Use Python Sockets for Networking

## *"The Direct Line"*

> **O'Reilly Skill Objective:** *Use sockets for basic network communication.*

> **Previously Learned:** Context managers, exception handling, classes, decorators, threading, iteration, binary data

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

*The firm's offices in New York and Boston need to share case updates in real time. No cloud service — just a direct, private line between the two offices.*

**Harvey:** "I need to talk to Jessica in Boston. Privately. No middleman."

**Mike:** "Sockets. A raw, direct connection between two machines. Python's `socket` module lets us build it from scratch — client, server, protocol, everything."

```python
import socket

# Server (Boston office)
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('0.0.0.0', 9000))
server.listen()
conn, addr = server.accept()
data = conn.recv(1024)
print(f"Received: {data.decode()}")    # "Case update: WAYNE-2026-001 settled"

# Client (New York office)
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('boston-server', 9000))
client.sendall(b"Case update: WAYNE-2026-001 settled")
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "HTTP, email, database connections — they ALL use sockets underneath. A socket is the raw pipe."

| Concept | What It Is |
|---------|-----------|
| **Socket** | An endpoint for network communication |
| **TCP** | Reliable, ordered delivery (like certified mail) |
| **UDP** | Fast, unordered, no guarantee (like shouting across the room) |
| **`AF_INET`** | IPv4 address family |
| **`SOCK_STREAM`** | TCP socket type |
| **`SOCK_DGRAM`** | UDP socket type |
| **`bind()`** | Attach socket to an address/port |
| **`listen()`** | Start accepting connections |
| **`accept()`** | Wait for a client to connect |
| **`connect()`** | Client connects to a server |
| **`send()`/`recv()`** | Send/receive data (bytes!) |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 TCP Server — Step by Step ⭐⭐

```python
import socket

def run_server(host='127.0.0.1', port=9000):
    """Basic TCP server — accepts one connection at a time."""
    
    # 1. Create socket
    #    AF_INET = IPv4, SOCK_STREAM = TCP
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
    # 2. Allow port reuse (avoid "Address already in use" error)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    
    # 3. Bind to address and port
    server_socket.bind((host, port))
    
    # 4. Listen for incoming connections (backlog = 5)
    server_socket.listen(5)
    print(f"🏛️ Server listening on {host}:{port}")
    
    while True:
        # 5. Accept a connection (BLOCKS until client connects)
        conn, addr = server_socket.accept()
        print(f"📞 Connection from {addr}")
        
        try:
            # 6. Receive data (max 4096 bytes at a time)
            while True:
                data = conn.recv(4096)
                if not data:
                    break    # Client disconnected
                
                message = data.decode('utf-8')
                print(f"📨 Received: {message}")
                
                # 7. Send response
                response = f"ACK: {message}"
                conn.sendall(response.encode('utf-8'))
        
        finally:
            # 8. Close connection
            conn.close()
            print(f"🔌 Connection closed from {addr}")

# run_server()
```

---

### 3.2 TCP Client — Connecting to the Server ⭐

```python
import socket

def run_client(host='127.0.0.1', port=9000):
    """Basic TCP client — connects, sends, receives."""
    
    # 1. Create socket
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
    # 2. Connect to server
    client_socket.connect((host, port))
    print(f"✅ Connected to {host}:{port}")
    
    try:
        # 3. Send a message (must be bytes!)
        message = "Case update: WAYNE-2026-001 settled for $2.5M"
        client_socket.sendall(message.encode('utf-8'))
        
        # 4. Receive response
        response = client_socket.recv(4096)
        print(f"📨 Server says: {response.decode('utf-8')}")
    
    finally:
        # 5. Close
        client_socket.close()

# run_client()
```

---

### 3.3 Using Context Managers with Sockets ⭐

```python
import socket

# ✅ Sockets support context managers (auto-close)
def echo_server(host='127.0.0.1', port=9000):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        s.bind((host, port))
        s.listen()
        print(f"Echo server on {host}:{port}")
        
        with s.accept()[0] as conn:
            while data := conn.recv(4096):
                conn.sendall(data)    # Echo back

def echo_client(host='127.0.0.1', port=9000):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((host, port))
        s.sendall(b"Hello from Harvey!")
        response = s.recv(4096)
        print(f"Echo: {response.decode()}")    # "Hello from Harvey!"
```

---

### 3.4 Multi-Client Server with Threading ⭐⭐

```python
import socket
import threading

def handle_client(conn, addr):
    """Handle a single client in its own thread."""
    print(f"📞 [{addr}] Connected")
    
    with conn:
        while True:
            data = conn.recv(4096)
            if not data:
                break
            
            message = data.decode('utf-8')
            print(f"📨 [{addr}] {message}")
            
            response = f"Server received: {message}"
            conn.sendall(response.encode('utf-8'))
    
    print(f"🔌 [{addr}] Disconnected")

def threaded_server(host='127.0.0.1', port=9000):
    """TCP server that handles multiple clients simultaneously."""
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        s.bind((host, port))
        s.listen(10)
        print(f"🏛️ Multi-client server on {host}:{port}")
        
        while True:
            conn, addr = s.accept()
            # Each client gets its own thread
            thread = threading.Thread(target=handle_client, args=(conn, addr))
            thread.daemon = True    # Dies when main thread exits
            thread.start()

# threaded_server()
```

---

### 3.5 UDP — Datagram Sockets ⭐

```python
import socket

# === UDP Server ===
def udp_server(host='127.0.0.1', port=9001):
    """UDP server — no connection, just receives datagrams."""
    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
        s.bind((host, port))
        print(f"📡 UDP server on {host}:{port}")
        
        while True:
            data, addr = s.recvfrom(4096)    # recvfrom (not recv!)
            print(f"📨 From {addr}: {data.decode()}")
            s.sendto(f"ACK: {data.decode()}".encode(), addr)

# === UDP Client ===
def udp_client(host='127.0.0.1', port=9001):
    """UDP client — no connect, just sends datagrams."""
    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
        message = "Urgent: Case WAYNE-2026-001 hearing at 3pm"
        s.sendto(message.encode(), (host, port))    # sendto (not send!)
        
        data, addr = s.recvfrom(4096)
        print(f"Response: {data.decode()}")

# TCP vs UDP:
# ┌─────────┬──────────────────────┬──────────────────────┐
# │         │ TCP (SOCK_STREAM)    │ UDP (SOCK_DGRAM)     │
# ├─────────┼──────────────────────┼──────────────────────┤
# │ Connect │ Yes (handshake)      │ No (just send)       │
# │ Reliable│ Yes (guaranteed)     │ No (may drop)        │
# │ Order   │ Yes (in-order)       │ No (any order)       │
# │ Speed   │ Slower (overhead)    │ Faster (no overhead) │
# │ Use for │ Files, web, chat     │ Video, DNS, gaming   │
# └─────────┴──────────────────────┴──────────────────────┘
```

---

### 3.6 Sending and Receiving Complete Messages ⭐⭐

```python
import socket
import struct
import json

# PROBLEM: recv() may return partial data!
# A 10,000-byte message might arrive in two recv(4096) calls.

# SOLUTION: Length-prefix protocol
# Send: [4 bytes: message length][message bytes]
# Recv: read 4 bytes → know exactly how much more to read

def send_message(sock, message):
    """Send a message with a 4-byte length prefix."""
    data = message.encode('utf-8')
    length = struct.pack('!I', len(data))    # 4-byte big-endian unsigned int
    sock.sendall(length + data)

def recv_message(sock):
    """Receive a length-prefixed message."""
    # Read the 4-byte length header
    raw_length = recv_exact(sock, 4)
    if not raw_length:
        return None
    length = struct.unpack('!I', raw_length)[0]
    
    # Read exactly 'length' bytes
    data = recv_exact(sock, length)
    return data.decode('utf-8') if data else None

def recv_exact(sock, n):
    """Receive exactly n bytes."""
    data = b''
    while len(data) < n:
        chunk = sock.recv(n - len(data))
        if not chunk:
            return None    # Connection closed
        data += chunk
    return data

# === Usage ===
# Server:
#   msg = recv_message(conn)       # Receives complete message
#   send_message(conn, "ACK")      # Sends complete response

# Client:
#   send_message(sock, "Hello!")   # Sends with length prefix
#   reply = recv_message(sock)     # Receives complete reply

# === JSON over sockets ===
def send_json(sock, obj):
    """Send a JSON-serializable object."""
    send_message(sock, json.dumps(obj))

def recv_json(sock):
    """Receive and parse a JSON message."""
    msg = recv_message(sock)
    return json.loads(msg) if msg else None
```

---

### 3.7 Socket Timeouts and Non-Blocking I/O ⭐

```python
import socket

# === Timeout ===
# By default, socket operations BLOCK forever
# Set a timeout to avoid hanging

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.settimeout(5.0)    # 5 second timeout
    
    try:
        s.connect(('example.com', 80))
        s.sendall(b"GET / HTTP/1.1\r\nHost: example.com\r\n\r\n")
        response = s.recv(4096)
        print(response.decode()[:200])
    except socket.timeout:
        print("⏰ Connection timed out!")
    except ConnectionRefusedError:
        print("❌ Connection refused!")

# === Non-blocking mode ===
# Returns immediately, raises BlockingIOError if no data
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.setblocking(False)
# s.recv(1024)    # BlockingIOError immediately if nothing to read!

# === select() — monitor multiple sockets ===
import select

# Wait for data on multiple sockets (up to 5 second timeout)
# readable, writable, errors = select.select([sock1, sock2], [], [], 5.0)
```

---

### 3.8 A Simple HTTP Request (Understanding the Protocol)

```python
import socket

def http_get(host, path="/", port=80):
    """Minimal HTTP GET using raw sockets."""
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.settimeout(10)
        s.connect((host, port))
        
        # HTTP request is just formatted text!
        request = (
            f"GET {path} HTTP/1.1\r\n"
            f"Host: {host}\r\n"
            f"Connection: close\r\n"
            f"\r\n"
        )
        s.sendall(request.encode())
        
        # Receive response
        response = b""
        while True:
            chunk = s.recv(4096)
            if not chunk:
                break
            response += chunk
        
        return response.decode('utf-8', errors='replace')

# This is what requests.get() does under the hood!
# result = http_get("example.com", "/")
# print(result[:500])

# The layers:
# requests.get()  →  urllib3  →  http.client  →  socket
# Each layer adds convenience; sockets are the foundation.
```

---

### 3.9 `socketserver` — The Standard Library Solution

```python
import socketserver

# Python's socketserver module handles boilerplate for you

class CaseHandler(socketserver.BaseRequestHandler):
    """Handler called for each client connection."""
    
    def handle(self):
        """Process a single client."""
        data = self.request.recv(4096).strip()
        client_addr = self.client_address
        
        message = data.decode('utf-8')
        print(f"📨 [{client_addr}] {message}")
        
        response = f"Received case update: {message}"
        self.request.sendall(response.encode('utf-8'))

# === Single-threaded server ===
# server = socketserver.TCPServer(('127.0.0.1', 9000), CaseHandler)
# server.serve_forever()

# === Multi-threaded server (one line!) ===
class ThreadedServer(socketserver.ThreadingMixIn, socketserver.TCPServer):
    allow_reuse_address = True

# server = ThreadedServer(('127.0.0.1', 9000), CaseHandler)
# server.serve_forever()

# === Forking server (Unix only) ===
# class ForkingServer(socketserver.ForkingMixIn, socketserver.TCPServer):
#     pass
```

---

### 3.10 Solving the Firm's Problem — Case Update System

```python
# ============================================
# Pearson Specter Litt — Real-Time Case Updates
# TCP server/client with JSON messaging
# ============================================

import socket
import threading
import json
import struct
from datetime import datetime

# === Message Protocol ===
def send_message(sock, obj):
    """Send a JSON message with length prefix."""
    data = json.dumps(obj).encode('utf-8')
    header = struct.pack('!I', len(data))
    sock.sendall(header + data)

def recv_message(sock):
    """Receive a length-prefixed JSON message."""
    raw = b''
    while len(raw) < 4:
        chunk = sock.recv(4 - len(raw))
        if not chunk:
            return None
        raw += chunk
    
    length = struct.unpack('!I', raw)[0]
    data = b''
    while len(data) < length:
        chunk = sock.recv(length - len(data))
        if not chunk:
            return None
        data += chunk
    
    return json.loads(data.decode('utf-8'))


# === Case Update Server ===
class CaseUpdateServer:
    """Multi-client TCP server for case updates."""
    
    def __init__(self, host='127.0.0.1', port=9000):
        self.host = host
        self.port = port
        self.clients = []
        self.updates = []
        self.lock = threading.Lock()
    
    def start(self):
        self.server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server_socket.bind((self.host, self.port))
        self.server_socket.listen(10)
        print(f"🏛️ Case Update Server on {self.host}:{self.port}")
        
        try:
            while True:
                conn, addr = self.server_socket.accept()
                with self.lock:
                    self.clients.append(conn)
                thread = threading.Thread(target=self._handle, args=(conn, addr))
                thread.daemon = True
                thread.start()
        except KeyboardInterrupt:
            print("\n🔌 Server shutting down")
        finally:
            self.server_socket.close()
    
    def _handle(self, conn, addr):
        print(f"📞 {addr} connected")
        try:
            while True:
                msg = recv_message(conn)
                if msg is None:
                    break
                
                msg['timestamp'] = datetime.now().isoformat()
                msg['from'] = str(addr)
                
                with self.lock:
                    self.updates.append(msg)
                
                print(f"📨 [{msg.get('attorney', '?')}] {msg.get('type', '?')}: "
                      f"{msg.get('case', '?')}")
                
                response = {
                    "status": "ok",
                    "message": f"Update #{len(self.updates)} received",
                    "timestamp": msg['timestamp'],
                }
                send_message(conn, response)
        
        except (ConnectionResetError, BrokenPipeError):
            pass
        finally:
            with self.lock:
                if conn in self.clients:
                    self.clients.remove(conn)
            conn.close()
            print(f"🔌 {addr} disconnected")


# === Case Update Client ===
class CaseUpdateClient:
    """TCP client for sending case updates."""
    
    def __init__(self, host='127.0.0.1', port=9000):
        self.host = host
        self.port = port
        self.sock = None
    
    def connect(self):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((self.host, self.port))
        print(f"✅ Connected to {self.host}:{self.port}")
    
    def send_update(self, attorney, case_ref, update_type, details):
        msg = {
            "attorney": attorney,
            "case": case_ref,
            "type": update_type,
            "details": details,
        }
        send_message(self.sock, msg)
        response = recv_message(self.sock)
        return response
    
    def close(self):
        if self.sock:
            self.sock.close()
    
    def __enter__(self):
        self.connect()
        return self
    
    def __exit__(self, *args):
        self.close()


# === Demo (run server and client in same process for demonstration) ===
def demo():
    """Demonstrate the system with server thread + client."""
    import time
    
    # Start server in background thread
    server = CaseUpdateServer()
    server_thread = threading.Thread(target=server.start, daemon=True)
    server_thread.start()
    time.sleep(0.5)    # Let server start
    
    # Client sends updates
    with CaseUpdateClient() as client:
        updates = [
            ("Harvey", "WAYNE-2026-001", "status_change", "Moved to discovery phase"),
            ("Mike", "STARK-2026-002", "billing", "Billed 40 hours for patent research"),
            ("Louis", "QUEEN-2026-003", "filing", "Motion to dismiss filed"),
            ("Harvey", "WAYNE-2026-001", "settlement", "Offer received: $2.5M"),
        ]
        
        print(f"\n{'='*60}")
        print(f"{'🏛️  CASE UPDATE DEMO':^60}")
        print(f"{'='*60}\n")
        
        for attorney, case, update_type, details in updates:
            response = client.send_update(attorney, case, update_type, details)
            status = response.get('status', '?') if response else 'error'
            print(f"  ✅ {attorney}: {case} — {update_type} [{status}]")
        
        print(f"\n📊 Server received {len(server.updates)} updates")
        print(f"{'='*60}")

# demo()
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Partial `recv()` — Data Comes in Chunks 🔥

```python
# ❌ recv() may NOT return the full message!
data = conn.recv(4096)    # Might get 100 bytes of a 10,000-byte message!

# ✅ Use a length-prefix protocol (Section 3.6)
# Or loop until you have all the data:
def recv_all(sock, length):
    data = b''
    while len(data) < length:
        chunk = sock.recv(min(4096, length - len(data)))
        if not chunk:
            raise ConnectionError("Connection closed")
        data += chunk
    return data
```

---

### Pitfall 2: "Address Already in Use" 🔥

```python
# ❌ Restarting a server quickly fails
# OSError: [Errno 98] Address already in use

# ✅ Set SO_REUSEADDR before bind
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server_socket.bind((host, port))
```

---

### Pitfall 3: Sending Strings Instead of Bytes

```python
# ❌ Sockets work with BYTES, not strings!
# sock.sendall("Hello")    # TypeError: a bytes-like object is required!

# ✅ Encode strings to bytes
sock.sendall("Hello".encode('utf-8'))
# ✅ Decode bytes to strings
data = sock.recv(4096)
message = data.decode('utf-8')
```

---

### Pitfall 4: Not Closing Sockets

```python
# ❌ Socket leak — eventually run out of file descriptors!
sock = socket.socket()
sock.connect(('server', 9000))
# ... use socket ...
# Forgot to close!

# ✅ Use context manager
with socket.socket() as sock:
    sock.connect(('server', 9000))
    # Auto-closed when block exits
```

---

### Pitfall 5: Blocking Forever on `accept()` or `recv()`

```python
# ❌ Blocks forever if no client connects or no data arrives
conn, addr = server_socket.accept()    # Hangs!

# ✅ Set a timeout
server_socket.settimeout(30)    # 30 second timeout
try:
    conn, addr = server_socket.accept()
except socket.timeout:
    print("No connection in 30 seconds")
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📞 Analogy: Sockets = The Firm's Phone System

```
TCP SOCKET = PHONE CALL:
  📞 Dial the number (connect)
  🤝 Other party picks up (accept)
  🗣️ Talk back and forth (send/recv)
  📞 Hang up (close)
  ✅ Guaranteed delivery, in order

UDP SOCKET = INTERCOM ANNOUNCEMENT:
  📢 Press the button, speak (sendto)
  🔊 Everyone on the channel hears it (recvfrom)
  ⚠️ No confirmation anyone heard it
  ✅ Fast, no connection needed

DONNA'S RULE:
  "TCP is a phone call — private, reliable, two-way.
   UDP is the intercom — fast, public, no guarantee.
   Use the phone for contracts.
   Use the intercom for lunch orders."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Echo Server**
```
Build a TCP echo server that sends back whatever it receives.
Test with a client that sends 5 messages.
```

**Exercise 2: Calculator Server**
```
Client sends: "5 + 3", "10 * 4", "100 / 7"
Server parses, calculates, sends back the result.
Handle division by zero gracefully.
```

**Exercise 3: Time Server**
```
Client connects → server sends current time as string → close.
No input needed from client.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Multi-Client Chat**
```
Server accepts multiple clients (threading).
Each message is broadcast to ALL connected clients.
Handle disconnections gracefully.
```

**Exercise 5: File Transfer**
```
Client sends a filename → server reads file → sends contents.
Use length-prefix protocol for reliable transfer.
Handle: file not found, large files, binary files.
```

**Exercise 6: UDP Ping/Pong**
```
Client sends "PING" to server.
Server responds with "PONG" + timestamp.
Measure round-trip latency.
Handle packet loss (timeout + retry).
```

---

### 🔴 Advanced Exercises

**Exercise 7: HTTP Server**
```
Build a minimal HTTP server:
  - Serves static files from a directory
  - Returns proper HTTP headers (Content-Type, Content-Length)
  - Handles 200, 404, 500 responses
  - Multi-threaded
```

**Exercise 8: RPC Framework**
```
Build a simple RPC system:
  Client: result = rpc.call("add", 5, 3)  → 8
  Server: routes "add" to a registered function
  JSON-based protocol with length prefix.
```

**Exercise 9: Connection Pool**
```
Build a socket connection pool:
  pool = ConnectionPool(host, port, max_connections=5)
  with pool.get() as conn:
      conn.sendall(b"...")
  Reuses connections instead of creating new ones.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Sending string instead of bytes
sock.sendall("Hello")    # TypeError!

# Bug 2: No SO_REUSEADDR → "Address already in use" on restart
server.bind(('0.0.0.0', 9000))

# Bug 3: Assuming recv gets everything
data = sock.recv(4096)    # May be partial!

# Bug 4: Resource leak
sock = socket.socket()
sock.connect(('server', 9000))
# ... exception thrown here ...
# sock.close() never called!

# Bug 5: Server only handles one client
conn, addr = server.accept()
# Process client...
# Never loops back to accept another!
```

### Fixed Code ✅

```python
# Fix 1: Encode!
sock.sendall("Hello".encode('utf-8'))

# Fix 2: Set SO_REUSEADDR
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind(('0.0.0.0', 9000))

# Fix 3: Length-prefix protocol or loop
data = recv_all(sock, expected_length)

# Fix 4: Context manager
with socket.socket() as sock:
    sock.connect(('server', 9000))
    # Auto-closed even on exception

# Fix 5: Loop + threading
while True:
    conn, addr = server.accept()
    threading.Thread(target=handle, args=(conn,)).start()
```

---

## 8. 🌍 Real-World Application

### Sockets Power Everything

| Application | Socket Usage |
|-------------|-------------|
| **Web browsers** | HTTP/HTTPS over TCP sockets |
| **Databases** | PostgreSQL, MySQL connections |
| **Email** | SMTP, IMAP, POP3 |
| **SSH** | Remote shell connections |
| **DNS** | Domain resolution (UDP) |
| **Gaming** | Real-time state sync (UDP) |
| **Chat apps** | WebSockets (TCP upgrade) |
| **File transfer** | FTP, SFTP |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is a socket?

**Q2.** What is the difference between TCP and UDP?

**Q3.** What do `AF_INET` and `SOCK_STREAM` mean?

**Q4.** Why must you encode strings before sending?

**Q5.** What does `setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)` do?

**Q6.** Why is `recv()` unreliable for complete messages?

**Q7.** How do you solve the partial-recv problem?

**Q8.** How do you handle multiple clients?

**Q9.** What is the difference between `send()` and `sendall()`?

**Q10.** When should you use UDP instead of TCP?

---

### 📋 Answer Key

> **Q1.** An endpoint for sending and receiving data across a network. Like a phone jack — one end of a communication channel.

> **Q2.** TCP is reliable, ordered, and connection-based (like a phone call). UDP is fast, unordered, connectionless, with no delivery guarantee (like sending postcards).

> **Q3.** `AF_INET` = IPv4 address family. `SOCK_STREAM` = TCP (reliable stream). Together they create an IPv4 TCP socket.

> **Q4.** Sockets transmit raw bytes, not strings. `"Hello".encode('utf-8')` converts the string to bytes. `data.decode('utf-8')` converts back.

> **Q5.** Allows reusing a port immediately after the server stops. Without it, restarting the server quickly fails with "Address already in use."

> **Q6.** `recv(4096)` returns UP TO 4096 bytes, but may return fewer. A 10KB message might arrive in multiple chunks.

> **Q7.** Use a length-prefix protocol: send 4 bytes containing the message length, then the message. The receiver reads the length first, then reads exactly that many bytes.

> **Q8.** Use threading — each client gets its own thread via `threading.Thread(target=handler, args=(conn,))`. Or use `socketserver.ThreadingMixIn`.

> **Q9.** `send()` may send only PART of the data and returns the number of bytes sent. `sendall()` guarantees ALL data is sent (loops internally).

> **Q10.** When speed matters more than reliability: video streaming, real-time gaming, DNS lookups, sensor data where occasional loss is acceptable.

---

## 🎬 End of Lesson Scene

**Harvey:** "So HTTP, databases, email — all sockets underneath?"

**Mike:** "Everything. `requests.get('https://api.com')` is syntactic sugar for: open a TCP socket, send an HTTP request as text, receive the response as bytes, parse it. We just built that from scratch."

**Harvey:** "The direct line. No middleman. Next: metaclasses — customizing how Python creates classes themselves."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | TCP server: bind, listen, accept, recv, send | ✅ |
| 2 | TCP client: connect, send, recv | ✅ |
| 3 | Context managers with sockets | ✅ |
| 4 | Multi-client server with threading | ✅ |
| 5 | UDP datagrams (sendto, recvfrom) | ✅ |
| 6 | Length-prefix protocol for complete messages | ✅ |
| 7 | Timeouts and non-blocking I/O | ✅ |
| 8 | Raw HTTP over sockets | ✅ |
| 9 | `socketserver` module | ✅ |
| 10 | JSON messaging over TCP | ✅ |

---

> **Next Lesson →** Level 5, Lesson 2: *Use Metaclasses to Customize Class Creation in Python*
