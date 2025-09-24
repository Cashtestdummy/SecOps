# Section 4: Generators and Decorators - DevOps Cheat Sheet

## ⚡ Generators: Memory-Efficient Pipelines
```python
# Basic generator
def stream_lines(path):
    with open(path) as f:
        for line in f:
            yield line.rstrip("\n")

# Pipeline: grep -> parse -> transform
lines = stream_lines("/var/log/app.log")
errors = (l for l in lines if "ERROR" in l)
parsed = (l.split(" | ") for l in errors)
service_names = (p[1] for p in parsed)
```

### Generator Utilities
```python
# Chunk/batch items
def chunks(iterable, size):
    batch = []
    for item in iterable:
        batch.append(item)
        if len(batch) == size:
            yield batch
            batch = []
    if batch:
        yield batch

# Sliding window
def window(seq, n=2):
    it = iter(seq)
    from collections import deque
    d = deque(maxlen=n)
    for x in it:
        d.append(x)
        if len(d) == n:
            yield tuple(d)
```

### Practical DevOps Use
- Stream large logs without loading into memory
- Read huge CSV/JSONL and process incrementally
- Batch API calls to respect rate limits

## 🧩 Decorators: Reusable Cross-Cutting Concerns
```python
import time
import functools

# Timing
def timing(fn):
    @functools.wraps(fn)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        try:
            return fn(*args, **kwargs)
        finally:
            dur = (time.perf_counter() - start) * 1000
            print(f"{fn.__name__} took {dur:.1f} ms")
    return wrapper

# Retry with backoff
from time import sleep

def retry(times=3, backoff=0.5, exceptions=(Exception,)):
    def deco(fn):
        @functools.wraps(fn)
        def wrapper(*args, **kwargs):
            delay = backoff
            for attempt in range(1, times + 1):
                try:
                    return fn(*args, **kwargs)
                except exceptions as e:
                    if attempt == times:
                        raise
                    sleep(delay)
                    delay *= 2
        return wrapper
    return deco
```

### Idempotency and Caching
```python
# Simple in-memory cache
from functools import lru_cache

@lru_cache(maxsize=256)
def get_user(user_id: str):
    # expensive call
    ...
```

### Logging Decorator
```python
import logging

def log_call(fn):
    @functools.wraps(fn)
    def wrapper(*args, **kwargs):
        logging.info("Calling %s args=%s kwargs=%s", fn.__name__, args, kwargs)
        res = fn(*args, **kwargs)
        logging.info("%s -> %r", fn.__name__, res)
        return res
    return wrapper
```

## 🔐 DevOps Patterns
- Use retry decorator for flaky network I/O (timeouts, 5xx)
- Use timing + logging to observe slow steps in pipelines
- Use generators to create streaming ETL from logs to metrics

## 🧪 Example: Streaming log error counter with backoff API
```python
@retry(times=5, backoff=0.25, exceptions=(TimeoutError, ConnectionError))
def push_metric(name: str, value: int):
    # call monitoring API
    ...

def count_errors(path):
    for batch in chunks(stream_lines(path), 1000):
        count = sum(1 for l in batch if "ERROR" in l)
        push_metric("app_errors", count)
```

## 🛡️ Best Practices
- Always preserve function metadata with functools.wraps
- Limit cache size for lru_cache to avoid memory bloat
- Emit structured logs inside decorators for better observability
- Keep decorators pure; avoid global side effects

---
Generators and decorators help you build efficient, observable, and resilient DevOps automation.
