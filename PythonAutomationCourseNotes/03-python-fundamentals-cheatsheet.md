# Section 3: Python Fundamentals - DevOps Cheat Sheet

## ✅ Core Syntax
- Shebang: `#!/usr/bin/env python3`
- Indentation: 4 spaces; no tabs mixing
- F-strings: `f"{var=}" -> var=value`
- Walrus operator: `if (n := len(items)) > 0:`

## 📦 Data Types & Structures
```python
# Numbers, strings, booleans
x: int = 42
pi: float = 3.14
name: str = "devops"
flag: bool = True

# Lists (ordered, mutable)
servers = ["web-1", "web-2"]
servers.append("web-3")

# Tuples (ordered, immutable)
coords = (10, 20)

# Sets (unique, unordered)
racks = {"r1", "r2", "r3"}
racks.add("r4")

# Dicts (key-value)
config = {"env": "prod", "replicas": 3}
config["region"] = "eu-west-1"
```

### Comprehensions (Fast and Pythonic)
```python
squares = [n*n for n in range(10)]
unique_envs = {svc["env"] for svc in services}
lookup = {svc["name"]: svc for svc in services}
```

## 🔁 Control Flow
```python
for host in servers:
    if host.startswith("web"):
        continue
    break
else:
    print("Loop finished without break")

# Ternary
status = "ok" if healthy else "fail"
```

## 🧩 Functions
```python
def deploy(app: str, *, region: str = "us-east-1") -> bool:
    """Deploy app to region."""
    return True

# Default, args, kwargs
def run(cmd, /, *args, verbose=False, **kwargs):
    ...

# Lambda
key_fn = lambda item: item["priority"]
```

### Useful Patterns
```python
from typing import Iterable

def batched(iterable: Iterable, size: int):
    buf = []
    for item in iterable:
        buf.append(item)
        if len(buf) == size:
            yield buf
            buf = []
    if buf:
        yield buf
```

## 🏷️ Type Hints (Basics)
```python
from typing import List, Dict, Optional, Tuple, Callable

Hosts = List[str]
Config = Dict[str, str]

def get_host(ip: str) -> Optional[str]:
    ...
```

## 📁 Files and Paths (quick intro)
```python
from pathlib import Path
p = Path("/var/log")
for f in p.glob("*.log"):
    print(f.name, f.stat().st_size)
```

## 🧰 Standard Library Must-Knows
```python
import os, sys, json, time, subprocess

# Environment
token = os.environ.get("TOKEN", "")

# JSON
payload = json.dumps({"name": "svc"})
obj = json.loads(payload)

# Subprocess (basic)
out = subprocess.check_output(["echo", "ok"], text=True).strip()
```

## 🧪 Assertions and Simple Tests
```python
def add(a, b):
    return a + b

if __name__ == "__main__":
    assert add(1, 2) == 3
    print("✅ basic tests passed")
```

## 🧱 Error Handling (Basics)
```python
try:
    risky()
except (TimeoutError, ConnectionError) as e:
    print(f"Network issue: {e}")
except Exception as e:
    print(f"Unexpected: {e}")
else:
    print("No exceptions")
finally:
    cleanup()
```

## ⚙️ CLI Basics with argparse
```python
import argparse

parser = argparse.ArgumentParser(description="DevOps tool")
parser.add_argument("action", choices=["deploy", "status"]) 
parser.add_argument("--env", default="dev")
args = parser.parse_args()
```

## 🔄 Iterables & Generators (preview)
```python
def tail(lines, n=10):
    from collections import deque
    return list(deque(lines, maxlen=n))

# Generator
def grep(lines, needle):
    for line in lines:
        if needle in line:
            yield line
```

## 🧪 Quick DevOps Examples
```python
# Parse simple INI-like file
config = {}
for line in Path("settings.cfg").read_text().splitlines():
    if not line or line.startswith("#"): continue
    k, v = line.split("=", 1)
    config[k.strip()] = v.strip()

# Roll up log sizes per service
from collections import defaultdict
sizes = defaultdict(int)
for log in Path("/var/log").glob("*.log"):
    svc = log.stem.split("-")[0]
    sizes[svc] += log.stat().st_size
```

## 🧭 DevOps Tips
- Prefer pathlib over os.path
- Use list/dict comprehensions for clarity and speed
- Keep functions small and pure; make side effects explicit
- Add type hints early; run mypy in CI
- Fail fast with clear error messages
- Structure scripts to allow import and reuse (`if __name__ == "__main__":`)

---
This cheat sheet covers essential Python syntax and patterns you’ll use constantly in DevOps automation.
