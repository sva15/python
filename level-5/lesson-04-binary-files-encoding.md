# 🏛️ Level 5 – Innovating | Lesson 4: Work with Binary Files and Encoding in Python

## *"The Raw Materials"*

> **O'Reilly Skill Objective:** *Work with binary files and handle file encoding and decoding appropriately.*

> **Previously Learned:** File I/O, `open()`, context managers, `bytes`, CSV/JSON, exception handling, `struct` (from sockets)

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

*Discovery produces 500 files — PDFs, images, encrypted archives, and legacy documents in Latin-1. The firm's Python script crashes on half of them.*

```python
# ❌ Text mode can't read binary files!
with open("evidence.pdf") as f:
    data = f.read()    # UnicodeDecodeError!

# ❌ Wrong encoding for legacy files!
with open("old_contract.txt") as f:
    text = f.read()    # Mojibake: "Müller" becomes "MÃ¼ller"!
```

**Mike:** "Two different problems. Binary files aren't text at all — they're raw bytes. And text files can be encoded in dozens of different ways. Python handles both, but you have to tell it what you're dealing with."

```python
# ✅ Binary mode for non-text files
with open("evidence.pdf", "rb") as f:
    data = f.read()    # bytes — no decoding!

# ✅ Explicit encoding for text files
with open("old_contract.txt", encoding="latin-1") as f:
    text = f.read()    # "Müller" ✅
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Everything in a computer is bytes. Text is bytes WITH an encoding. Binary is bytes WITHOUT one."

| Mode | How to Open | Returns | Use For |
|------|:-----------:|:-------:|---------|
| **Text** | `open(f)` or `open(f, 'r')` | `str` | .txt, .csv, .json, .py, .html |
| **Binary** | `open(f, 'rb')` | `bytes` | .pdf, .png, .zip, .exe, .mp3 |

| Encoding | Coverage | Bytes/Char | Usage |
|----------|----------|:----------:|-------|
| **ASCII** | English only (128 chars) | 1 | Legacy systems |
| **Latin-1** (ISO 8859-1) | Western European | 1 | Old documents |
| **UTF-8** | All Unicode (1.1M+ chars) | 1-4 | Modern default ✅ |
| **UTF-16** | All Unicode | 2-4 | Windows internals |
| **UTF-32** | All Unicode | 4 | Fixed-width processing |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Bytes vs Strings — The Fundamental Difference ⭐⭐

```python
# STRINGS: sequences of Unicode characters
text = "Harvey Specter"
print(type(text))       # <class 'str'>
print(len(text))        # 14 characters

# BYTES: sequences of raw byte values (0-255)
raw = b"Harvey Specter"
print(type(raw))        # <class 'bytes'>
print(len(raw))         # 14 bytes (ASCII = 1 byte per char)

# Converting between them:
# str → bytes: ENCODE
encoded = text.encode('utf-8')
print(encoded)           # b'Harvey Specter'
print(type(encoded))     # <class 'bytes'>

# bytes → str: DECODE
decoded = encoded.decode('utf-8')
print(decoded)           # Harvey Specter
print(type(decoded))     # <class 'str'>

# Non-ASCII characters use multiple bytes in UTF-8:
text = "Müller & Associés"
utf8 = text.encode('utf-8')
latin1 = text.encode('latin-1')

print(f"Text length: {len(text)} characters")    # 18 characters
print(f"UTF-8 bytes: {len(utf8)}")                # 20 bytes (ü and é = 2 bytes each)
print(f"Latin-1 bytes: {len(latin1)}")            # 18 bytes (1 byte each)

# Bytes are NOT strings!
# b"hello" + " world"    # TypeError!
# b"hello" + b" world"   # ✅ b"hello world"
```

---

### 3.2 Reading and Writing Binary Files ⭐

```python
# === Reading binary files ===
# 'rb' = read binary
with open("evidence.pdf", "rb") as f:
    data = f.read()           # Returns bytes
    print(type(data))         # <class 'bytes'>
    print(len(data))          # File size in bytes
    print(data[:4])           # b'%PDF' — PDF magic bytes!

# Read in chunks (for large files)
with open("large_file.bin", "rb") as f:
    while chunk := f.read(4096):    # 4KB chunks
        process(chunk)

# === Writing binary files ===
# 'wb' = write binary
with open("output.bin", "wb") as f:
    f.write(b'\x00\x01\x02\x03')    # Write raw bytes
    f.write(bytes([255, 128, 64]))    # From list of ints
    f.write(b"Hello")                 # ASCII text as bytes

# === Append binary ===
with open("output.bin", "ab") as f:    # 'ab' = append binary
    f.write(b'\xFF\xFE')

# === Read specific positions ===
with open("evidence.pdf", "rb") as f:
    f.seek(0)            # Go to beginning
    header = f.read(4)   # Read 4 bytes
    
    f.seek(-4, 2)        # Go 4 bytes from end (2 = from end)
    footer = f.read(4)
    
    pos = f.tell()       # Current position
    print(f"Position: {pos}")
```

---

### 3.3 The `struct` Module — Packing and Unpacking Binary Data ⭐⭐

```python
import struct

# struct converts between Python values and C-style binary data

# === Pack: Python values → bytes ===
# Format: '!' = network byte order, 'I' = unsigned int, 'f' = float, 's' = string
packed = struct.pack('!If10s', 42, 3.14, b'Harvey    ')
print(packed)        # Raw bytes
print(len(packed))   # 18 bytes (4 + 4 + 10)

# === Unpack: bytes → Python values ===
values = struct.unpack('!If10s', packed)
print(values)        # (42, 3.140000104904175, b'Harvey    ')

# === Common format characters ===
# 'b' = signed byte (1)     'B' = unsigned byte (1)
# 'h' = short (2)           'H' = unsigned short (2)
# 'i' = int (4)             'I' = unsigned int (4)
# 'q' = long long (8)       'Q' = unsigned long long (8)
# 'f' = float (4)           'd' = double (8)
# 's' = char[] (string)     '?' = bool (1)

# === Byte order prefixes ===
# '!' = network (big-endian)
# '<' = little-endian
# '>' = big-endian
# '=' = native

# === Practical example: reading a custom binary header ===
# Imagine a file format: [4-byte magic][4-byte version][4-byte record_count]
header_format = '!4sIi'
header_size = struct.calcsize(header_format)    # 12 bytes

# Writing
with open("data.firm", "wb") as f:
    header = struct.pack(header_format, b'FIRM', 2, 100)
    f.write(header)

# Reading
with open("data.firm", "rb") as f:
    raw = f.read(header_size)
    magic, version, count = struct.unpack(header_format, raw)
    print(f"Magic: {magic}, Version: {version}, Records: {count}")
    # Magic: b'FIRM', Version: 2, Records: 100
```

---

### 3.4 Text Encodings — UTF-8, Latin-1, ASCII ⭐⭐

```python
# === UTF-8: The Modern Standard ===
# Variable-length: 1-4 bytes per character
text = "Hello Wörld 你好 🌍"
utf8 = text.encode('utf-8')
print(f"Characters: {len(text)}")    # 15
print(f"UTF-8 bytes: {len(utf8)}")   # 24 (ASCII=1, ö=2, 你=3, 🌍=4)

# UTF-8 is backward-compatible with ASCII!
"Hello".encode('utf-8') == "Hello".encode('ascii')    # True

# === Latin-1 (ISO 8859-1): Western European ===
# Fixed 1 byte per character, covers 256 chars
text = "Müller & Associés"
latin1 = text.encode('latin-1')
print(f"Latin-1 bytes: {len(latin1)}")    # 18 (1 byte each)

# ⚠️ Latin-1 can't encode all Unicode!
# "你好".encode('latin-1')    # UnicodeEncodeError!

# === Encoding detection ===
# When you don't know the encoding:
raw_bytes = b'\xc3\xbc'    # "ü" in UTF-8
print(raw_bytes.decode('utf-8'))      # ü ✅
print(raw_bytes.decode('latin-1'))    # Ã¼ ❌ (mojibake!)

raw_bytes2 = b'\xfc'    # "ü" in Latin-1
# raw_bytes2.decode('utf-8')          # UnicodeDecodeError!
print(raw_bytes2.decode('latin-1'))   # ü ✅
```

---

### 3.5 Handling Encoding Errors ⭐

```python
# === Error handling modes ===
text = "Müller — 你好"

# 'strict' (default) — raise error
try:
    text.encode('ascii')    # UnicodeEncodeError!
except UnicodeEncodeError as e:
    print(f"Error: {e}")

# 'ignore' — skip unencodable characters
print(text.encode('ascii', errors='ignore'))
# b'Mller  '  (dropped ü, —, 你好)

# 'replace' — substitute with '?'
print(text.encode('ascii', errors='replace'))
# b'M?ller ? ??'

# 'xmlcharrefreplace' — XML escape
print(text.encode('ascii', errors='xmlcharrefreplace'))
# b'M&#252;ller &#8212; &#20320;&#22909;'

# 'backslashreplace' — Python escape
print(text.encode('ascii', errors='backslashreplace'))
# b'M\\xfcller \\u2014 \\u4f60\\u597d'

# === Decoding with error handling ===
bad_bytes = b'\xff\xfe\x00\x01'
# bad_bytes.decode('utf-8')    # UnicodeDecodeError!

print(bad_bytes.decode('utf-8', errors='replace'))     # ��\x00\x01
print(bad_bytes.decode('utf-8', errors='ignore'))      # \x00\x01

# === Opening files with error handling ===
with open("legacy.txt", encoding="utf-8", errors="replace") as f:
    text = f.read()    # Replaces bad bytes with '�'
```

---

### 3.6 `bytearray` — Mutable Bytes ⭐

```python
# bytes is IMMUTABLE (like str)
b = b"Hello"
# b[0] = 72    # TypeError: bytes doesn't support item assignment!

# bytearray is MUTABLE (like list)
ba = bytearray(b"Hello")
ba[0] = 0x68    # 'h'
ba.append(0x21)  # '!'
ba.extend(b" World")
print(ba)        # bytearray(b'hello! World')

# Useful for building binary data incrementally
buffer = bytearray()
buffer.extend(b'\x89PNG')         # PNG magic bytes
buffer.extend(struct.pack('!I', 100))  # Width
buffer.extend(struct.pack('!I', 50))   # Height
print(f"Buffer: {len(buffer)} bytes")

# Convert back to immutable bytes
final = bytes(buffer)
```

---

### 3.7 File Magic Bytes — Identifying File Types ⭐

```python
# Files often start with "magic bytes" that identify their type

MAGIC_BYTES = {
    b'%PDF':      'PDF',
    b'\x89PNG':   'PNG',
    b'\xff\xd8':  'JPEG',
    b'GIF87a':    'GIF87a',
    b'GIF89a':    'GIF89a',
    b'PK':        'ZIP/DOCX/XLSX',
    b'\x1f\x8b':  'GZIP',
    b'BM':        'BMP',
    b'\x7fELF':   'ELF (Linux executable)',
    b'MZ':        'EXE (Windows)',
    b'Rar!':      'RAR',
    b'\x00\x00\x00':  'Possible MP4/MOV',
}

def identify_file(filepath):
    """Identify file type by magic bytes."""
    with open(filepath, 'rb') as f:
        header = f.read(8)
    
    for magic, filetype in MAGIC_BYTES.items():
        if header.startswith(magic):
            return filetype
    
    return "Unknown"

# Usage:
# print(identify_file("evidence.pdf"))     # PDF
# print(identify_file("photo.png"))        # PNG
# print(identify_file("archive.zip"))      # ZIP/DOCX/XLSX
```

---

### 3.8 Base64 Encoding — Binary as Text ⭐

```python
import base64

# Base64 converts binary data to ASCII text
# Used for: email attachments, embedding images, API payloads

# === Encode ===
binary_data = b'\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR'
encoded = base64.b64encode(binary_data)
print(encoded)    # b'iVBORw0KGgoAAAANSUhEUg=='

# === Decode ===
decoded = base64.b64decode(encoded)
print(decoded == binary_data)    # True

# === File to Base64 string ===
def file_to_base64(filepath):
    with open(filepath, 'rb') as f:
        return base64.b64encode(f.read()).decode('ascii')

def base64_to_file(b64_string, filepath):
    with open(filepath, 'wb') as f:
        f.write(base64.b64decode(b64_string))

# === Data URLs (embedding in HTML/JSON) ===
# data:image/png;base64,iVBORw0KGgo...
def make_data_url(filepath, mime_type):
    b64 = file_to_base64(filepath)
    return f"data:{mime_type};base64,{b64}"

# === URL-safe Base64 ===
url_safe = base64.urlsafe_b64encode(binary_data)    # Uses - and _ instead of + and /
```

---

### 3.9 `io.BytesIO` and `io.StringIO` — In-Memory Files ⭐

```python
import io

# === BytesIO: in-memory binary file ===
# Behaves like an open file but lives in memory
buffer = io.BytesIO()
buffer.write(b"Hello ")
buffer.write(b"World!")

# Read it back
buffer.seek(0)
content = buffer.read()
print(content)    # b'Hello World!'

# Use with libraries that expect file objects
import csv
import json

# JSON to in-memory buffer
data = {"name": "Harvey", "revenue": 2500000}
buffer = io.BytesIO()
buffer.write(json.dumps(data).encode('utf-8'))

# === StringIO: in-memory text file ===
text_buffer = io.StringIO()
text_buffer.write("name,revenue\n")
text_buffer.write("Wayne,2500000\n")
text_buffer.write("Stark,1800000\n")

text_buffer.seek(0)
reader = csv.DictReader(text_buffer)
for row in reader:
    print(f"  {row['name']}: ${int(row['revenue']):,}")

# Why use them?
# ✅ Testing without creating temp files
# ✅ Processing data in pipelines without disk I/O
# ✅ Working with APIs that return/accept file-like objects
```

---

### 3.10 Solving the Firm's Problem — Multi-Format Document Processor

```python
# ============================================
# Pearson Specter Litt — Document Processor
# Handle binary files, encodings, and formats
# ============================================

import struct
import base64
import io
import json
import os
from datetime import datetime

class DocumentProcessor:
    """Process discovery documents: identify, convert, validate."""
    
    MAGIC_BYTES = {
        b'%PDF': ('PDF', 'application/pdf'),
        b'\x89PNG': ('PNG', 'image/png'),
        b'\xff\xd8': ('JPEG', 'image/jpeg'),
        b'PK': ('ZIP', 'application/zip'),
        b'\x1f\x8b': ('GZIP', 'application/gzip'),
        b'GIF8': ('GIF', 'image/gif'),
        b'BM': ('BMP', 'image/bmp'),
    }
    
    ENCODING_FALLBACK = ['utf-8', 'latin-1', 'cp1252', 'ascii']
    
    def __init__(self):
        self.processed = []
        self.errors = []
    
    def identify(self, filepath):
        """Identify file type by magic bytes."""
        with open(filepath, 'rb') as f:
            header = f.read(8)
        
        for magic, (name, mime) in self.MAGIC_BYTES.items():
            if header.startswith(magic):
                return name, mime
        
        return 'UNKNOWN', 'application/octet-stream'
    
    def detect_encoding(self, filepath, sample_size=8192):
        """Try to detect the text encoding of a file."""
        with open(filepath, 'rb') as f:
            raw = f.read(sample_size)
        
        # Check for BOM (Byte Order Mark)
        if raw.startswith(b'\xef\xbb\xbf'):
            return 'utf-8-sig'
        if raw.startswith(b'\xff\xfe'):
            return 'utf-16-le'
        if raw.startswith(b'\xfe\xff'):
            return 'utf-16-be'
        
        # Try encodings in order
        for encoding in self.ENCODING_FALLBACK:
            try:
                raw.decode(encoding)
                return encoding
            except (UnicodeDecodeError, ValueError):
                continue
        
        return 'latin-1'    # Latin-1 accepts ALL byte values
    
    def read_text(self, filepath, encoding=None):
        """Read a text file with auto-detected encoding."""
        if encoding is None:
            encoding = self.detect_encoding(filepath)
        
        with open(filepath, encoding=encoding, errors='replace') as f:
            return f.read(), encoding
    
    def file_info(self, filepath):
        """Get comprehensive info about a file."""
        stat = os.stat(filepath)
        filetype, mime = self.identify(filepath)
        
        info = {
            "path": filepath,
            "name": os.path.basename(filepath),
            "size_bytes": stat.st_size,
            "size_human": self._human_size(stat.st_size),
            "type": filetype,
            "mime": mime,
            "modified": datetime.fromtimestamp(stat.st_mtime).isoformat(),
        }
        
        # For text files, detect encoding
        if filetype == 'UNKNOWN':
            info["encoding"] = self.detect_encoding(filepath)
        
        return info
    
    def to_base64(self, filepath):
        """Convert file to base64 string."""
        with open(filepath, 'rb') as f:
            return base64.b64encode(f.read()).decode('ascii')
    
    def process_batch(self, filepaths):
        """Process multiple files and generate a report."""
        results = []
        
        for path in filepaths:
            try:
                info = self.file_info(path)
                info['status'] = 'ok'
                results.append(info)
                self.processed.append(path)
            except Exception as e:
                results.append({
                    'path': path,
                    'status': 'error',
                    'error': str(e),
                })
                self.errors.append((path, str(e)))
        
        return results
    
    @staticmethod
    def _human_size(size_bytes):
        for unit in ['B', 'KB', 'MB', 'GB']:
            if size_bytes < 1024:
                return f"{size_bytes:.1f} {unit}"
            size_bytes /= 1024
        return f"{size_bytes:.1f} TB"


# === Custom Binary Format: Firm Evidence Archive ===
class EvidenceArchive:
    """Custom binary format for evidence packaging.
    
    Format:
        Header: [4B magic][4B version][4B file_count]
        For each file:
            [4B name_length][name_bytes][4B data_length][data_bytes]
    """
    MAGIC = b'EVID'
    VERSION = 1
    
    @classmethod
    def pack(cls, files_dict, output_path):
        """Pack multiple files into an archive.
        files_dict = {"filename": b"data", ...}
        """
        with open(output_path, 'wb') as f:
            # Header
            f.write(struct.pack('!4sII', cls.MAGIC, cls.VERSION, len(files_dict)))
            
            # Files
            for name, data in files_dict.items():
                name_bytes = name.encode('utf-8')
                f.write(struct.pack('!I', len(name_bytes)))
                f.write(name_bytes)
                f.write(struct.pack('!I', len(data)))
                f.write(data)
        
        return output_path
    
    @classmethod
    def unpack(cls, archive_path):
        """Unpack an archive to dict."""
        files = {}
        
        with open(archive_path, 'rb') as f:
            # Header
            magic, version, count = struct.unpack('!4sII', f.read(12))
            assert magic == cls.MAGIC, f"Invalid archive: {magic}"
            
            # Files
            for _ in range(count):
                name_len = struct.unpack('!I', f.read(4))[0]
                name = f.read(name_len).decode('utf-8')
                data_len = struct.unpack('!I', f.read(4))[0]
                data = f.read(data_len)
                files[name] = data
        
        return files


# === Demo ===
print("=" * 65)
print(f"{'🏛️  DOCUMENT PROCESSOR DEMO':^65}")
print("=" * 65)

# Demonstrate archive
files_to_pack = {
    "memo.txt": "Confidential merger memo.\nWayne + Stark.".encode('utf-8'),
    "contract.txt": "Merger agreement v2.1\n$2.5M settlement.".encode('utf-8'),
    "notes.txt": "Meeting notes: Müller & Associés".encode('utf-8'),
}

archive_path = "evidence.evid"
EvidenceArchive.pack(files_to_pack, archive_path)

# Unpack and verify
unpacked = EvidenceArchive.unpack(archive_path)

print(f"\n📦 Archive: {archive_path}")
print(f"   Files packed: {len(files_to_pack)}")
print(f"   Archive size: {os.path.getsize(archive_path)} bytes")

print(f"\n📂 Unpacked contents:")
for name, data in unpacked.items():
    text = data.decode('utf-8')
    preview = text[:50].replace('\n', '\\n')
    print(f"  📄 {name} ({len(data)} bytes): {preview}...")

# Verify integrity
for name in files_to_pack:
    assert unpacked[name] == files_to_pack[name], f"Mismatch: {name}"
print(f"\n✅ Integrity check passed — all files match!")

# Base64 demo
b64 = base64.b64encode(files_to_pack["memo.txt"]).decode('ascii')
print(f"\n📎 memo.txt as Base64: {b64[:50]}...")

# Cleanup
os.remove(archive_path)

print(f"\n{'='*65}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Text Mode for Binary Files 🔥

```python
# ❌ Text mode corrupts binary data!
with open("image.png") as f:    # Text mode!
    data = f.read()    # UnicodeDecodeError or corrupted data!

# ✅ Always use 'rb' for non-text files
with open("image.png", "rb") as f:
    data = f.read()    # Clean bytes
```

---

### Pitfall 2: Wrong Encoding = Mojibake 🔥

```python
# "Mojibake" = garbled text from wrong encoding
# "Müller" in UTF-8 is b'\xc3\xbc', in Latin-1 it's b'\xfc'

# ❌ Reading Latin-1 file as UTF-8
raw = b'M\xfcller'    # Latin-1 encoded "Müller"
# raw.decode('utf-8')    # UnicodeDecodeError!

# ✅ Decode with correct encoding
print(raw.decode('latin-1'))    # Müller ✅

# ✅ When unsure, try multiple encodings
for enc in ['utf-8', 'latin-1', 'cp1252']:
    try:
        text = raw.decode(enc)
        print(f"  {enc}: {text}")
        break
    except UnicodeDecodeError:
        continue
```

---

### Pitfall 3: Platform-Dependent Line Endings

```python
# Windows: \r\n,  Unix: \n,  Old Mac: \r

# Text mode auto-converts (usually fine)
with open("file.txt", "r") as f:
    text = f.read()    # \r\n → \n on Windows

# Binary mode preserves original line endings
with open("file.txt", "rb") as f:
    raw = f.read()    # \r\n stays as \r\n

# ✅ Use universal newlines (default in text mode)
with open("file.txt", "r", newline=None) as f:    # Default
    text = f.read()    # All \r\n, \r, \n → \n
```

---

### Pitfall 4: Mixing Binary and Text Operations

```python
# ❌ Can't write str to binary file
with open("file.bin", "wb") as f:
    # f.write("Hello")    # TypeError: a bytes-like object is required!
    f.write("Hello".encode('utf-8'))    # ✅

# ❌ Can't write bytes to text file
with open("file.txt", "w") as f:
    # f.write(b"Hello")    # TypeError: write() argument must be str!
    f.write(b"Hello".decode('utf-8'))    # ✅
```

---

### Pitfall 5: BOM (Byte Order Mark) Confusion

```python
# Some editors add a BOM to UTF-8 files: b'\xef\xbb\xbf'
# This can cause issues with parsers

# ❌ BOM appears as invisible character
with open("bom_file.txt", encoding="utf-8") as f:
    text = f.read()
    print(repr(text[:5]))    # '\ufeffHell'  ← BOM!

# ✅ Use utf-8-sig to strip BOM automatically
with open("bom_file.txt", encoding="utf-8-sig") as f:
    text = f.read()
    print(repr(text[:5]))    # 'Hello'  ✅
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📜 Analogy: Encoding = Language, Binary = Sealed Package

```
TEXT FILES = LETTERS:
  Written in a LANGUAGE (encoding):
    English (ASCII), French (Latin-1), Universal (UTF-8)
  
  You MUST know the language to read it.
  Wrong language = gibberish (mojibake).

BINARY FILES = SEALED PACKAGES:
  Not text at all — raw materials inside.
  PDFs, images, ZIP files.
  
  Opening as text is like reading a package label
  and trying to "read" the contents.

DONNA'S RULE:
  "If it's a letter, know the language.
   If it's a package, don't try to read it — unpack it.
   UTF-8 is the universal translator.
   When in doubt, start there."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Bytes Explorer**
```
Encode "Hello, World! 🌍" in UTF-8, Latin-1, and ASCII.
Compare the byte lengths.
Which encodings support all characters?
```

**Exercise 2: Binary File Copy**
```
Write a script that copies any file (binary-safe):
  copy_file("source.pdf", "copy.pdf")
Verify the copy is identical using file comparison.
```

**Exercise 3: Hex Viewer**
```
Build a simple hex viewer:
  hex_dump("file.bin", num_bytes=64)
Output:
  00000000  89 50 4e 47 0d 0a 1a 0a  |.PNG....|
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Encoding Detector**
```
Build a function that guesses the encoding of a file:
  encoding = detect_encoding("mystery.txt")
Try UTF-8, UTF-16, Latin-1, cp1252.
Handle BOM detection.
```

**Exercise 5: Binary Record Format**
```
Define a custom binary format for employee records:
  [ID: 4B][Name: 50B][Salary: 8B float][Active: 1B bool]
Write 10 records, read them back, verify.
```

**Exercise 6: File Splitter/Joiner**
```
split_file("large.bin", chunk_size=1024*1024)  → part_001.bin, part_002.bin, ...
join_files("part_*.bin", "reassembled.bin")
Verify reassembled file matches original.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Simple File Encryption**
```
Implement XOR encryption on binary files:
  encrypt_file("secret.txt", key=b"MyKey123", output="secret.enc")
  decrypt_file("secret.enc", key=b"MyKey123", output="secret.dec")
```

**Exercise 8: WAV File Reader**
```
Parse a WAV audio file header:
  - Sample rate, channels, bits per sample
  - Duration, file size
  - Extract raw audio data
(WAV uses a well-documented binary format)
```

**Exercise 9: Binary Diff Tool**
```
Compare two binary files and report differences:
  diff = binary_diff("file1.bin", "file2.bin")
  → Offset 0x1A4: 0xFF → 0x00
  → Offset 0x1A5: 0xAB → 0xCD
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Opening PDF as text
with open("doc.pdf") as f:
    data = f.read()    # UnicodeDecodeError!

# Bug 2: Wrong encoding → mojibake
with open("german.txt", encoding="ascii") as f:
    text = f.read()    # UnicodeDecodeError on "ü"!

# Bug 3: Writing string to binary file
with open("out.bin", "wb") as f:
    f.write("Hello")    # TypeError!

# Bug 4: Forgetting BOM
with open("utf8_bom.csv", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row)    # First header has invisible BOM character!

# Bug 5: Lost data from wrong encoding
data = "Müller".encode('utf-8')
text = data.decode('latin-1')    # "MÃ¼ller" — mojibake!
```

### Fixed Code ✅

```python
# Fix 1: Binary mode
with open("doc.pdf", "rb") as f:
    data = f.read()

# Fix 2: Correct encoding
with open("german.txt", encoding="utf-8") as f:
    text = f.read()

# Fix 3: Encode first
with open("out.bin", "wb") as f:
    f.write("Hello".encode('utf-8'))

# Fix 4: Use utf-8-sig
with open("utf8_bom.csv", encoding="utf-8-sig") as f:
    reader = csv.DictReader(f)

# Fix 5: Match encoding
data = "Müller".encode('utf-8')
text = data.decode('utf-8')    # "Müller" ✅
```

---

## 8. 🌍 Real-World Application

### Binary and Encoding in Practice

| Scenario | Approach |
|----------|----------|
| Reading PDFs | `rb` mode + libraries like PyPDF2 |
| Image processing | `rb` mode + Pillow/PIL |
| Network protocols | `struct` for packet headers |
| File uploads | Base64 for JSON APIs |
| Legacy data | Detect encoding, convert to UTF-8 |
| Databases | UTF-8 everywhere |
| Email attachments | Base64 + MIME types |
| Config files | UTF-8, handle BOM |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is the difference between text mode and binary mode?

**Q2.** How do you convert a string to bytes?

**Q3.** What is UTF-8 and why is it the default?

**Q4.** What is mojibake?

**Q5.** What does the `struct` module do?

**Q6.** What is a BOM?

**Q7.** What is `bytearray` and how does it differ from `bytes`?

**Q8.** What is Base64 encoding used for?

**Q9.** What is `io.BytesIO`?

**Q10.** How do you handle files with unknown encoding?

---

### 📋 Answer Key

> **Q1.** Text mode (`'r'`/`'w'`) reads/writes `str` with encoding. Binary mode (`'rb'`/`'wb'`) reads/writes raw `bytes` with no encoding.

> **Q2.** `"text".encode('utf-8')` → `b'text'`. The encoding parameter specifies which byte representation to use.

> **Q3.** A variable-length Unicode encoding (1-4 bytes per char). It's backward-compatible with ASCII and can represent all 1.1M+ Unicode characters.

> **Q4.** Garbled text resulting from decoding bytes with the wrong encoding. Example: UTF-8 "Müller" decoded as Latin-1 becomes "MÃ¼ller."

> **Q5.** Converts between Python values and C-style binary data. `struct.pack()` creates bytes, `struct.unpack()` extracts values.

> **Q6.** Byte Order Mark — special bytes at the start of a file indicating encoding and byte order. UTF-8 BOM: `b'\xef\xbb\xbf'`. Use `encoding='utf-8-sig'` to handle it.

> **Q7.** `bytearray` is a MUTABLE sequence of bytes (like a list). `bytes` is IMMUTABLE (like a tuple/str). Use `bytearray` when building binary data incrementally.

> **Q8.** Encoding binary data as ASCII text. Used in email attachments, embedding images in HTML/JSON, and APIs that only accept text.

> **Q9.** An in-memory binary file object. Behaves like an open file but lives in RAM. Useful for testing and data pipelines without disk I/O.

> **Q10.** Try common encodings in order (UTF-8, Latin-1, cp1252), check for BOMs, use `errors='replace'` as fallback, or use the `chardet` library.

---

## 🎬 End of Lesson Scene

**Mike:** "500 discovery files — 312 PDFs, 89 images, 45 legacy Latin-1 documents, 54 UTF-8 texts. All processed. I built a custom binary archive to package the evidence."

**Harvey:** "The legacy documents?"

**Mike:** "Auto-detected as Latin-1, converted to UTF-8. No more mojibake. Every 'Müller' reads correctly now."

**Harvey:** "Next: threading and async. Make it all run simultaneously."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Bytes vs strings (`bytes` vs `str`) | ✅ |
| 2 | Reading/writing binary files (`rb`/`wb`) | ✅ |
| 3 | `struct` module (pack/unpack) | ✅ |
| 4 | Text encodings (UTF-8, Latin-1, ASCII) | ✅ |
| 5 | Handling encoding errors | ✅ |
| 6 | `bytearray` (mutable bytes) | ✅ |
| 7 | File magic bytes (type identification) | ✅ |
| 8 | Base64 encoding | ✅ |
| 9 | `io.BytesIO` / `io.StringIO` | ✅ |
| 10 | Custom binary format with `struct` | ✅ |

---

> **Next Lesson →** Level 5, Lesson 5: *Write Multi-Threaded and Asynchronous Code in Python*
