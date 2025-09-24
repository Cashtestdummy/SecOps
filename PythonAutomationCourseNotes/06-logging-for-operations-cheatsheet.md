# Section 6: Logging for Operations - Cheat Sheet

## 🎯 Section Overview
Duration: 1hr 57min
Focus: Production-grade logging patterns for DevOps automation and observability

## 📋 Key Concepts

### Why Logging Matters in DevOps
- Traceability: Who/what/when for actions and changes
- Troubleshooting: Context-rich records for incidents
- Observability: Metrics, logs, traces triad
- Compliance: Audit trails and retention

### Logging vs. Printing
```python
import logging

# Bad
print("Starting deployment...")

# Good
logging.info("Starting deployment", extra={"component": "deploy", "env": "prod"})
```

## 🧰 Python logging Basics
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s %(message)s",
    datefmt="%Y-%m-%dT%H:%M:%S%z",
)
logger = logging.getLogger("deploy")

logger.debug("Debug details")
logger.info("Starting deployment")
logger.warning("Low disk space", extra={"server": "web-1"})
logger.error("Failed to restart service")
logger.critical("Outage detected")
```

### Logger Hierarchy and Names
```python
app_logger = logging.getLogger("app")
app_logger_db = logging.getLogger("app.db")  # inherits handlers unless propagate=False
```

## 🧱 Handlers, Formatters, Filters

### File and Console Handlers
```python
import logging

logger = logging.getLogger("ops")
logger.setLevel(logging.INFO)

# Console handler
ch = logging.StreamHandler()
ch.setLevel(logging.INFO)

# File handler with rotation
from logging.handlers import RotatingFileHandler
fh = RotatingFileHandler(
    "ops.log", maxBytes=5_000_000, backupCount=5
)
fh.setLevel(logging.INFO)

# Formatters
fmt = logging.Formatter(
    "%(asctime)s %(levelname)s %(name)s %(process)d %(threadName)s %(message)s"
)
ch.setFormatter(fmt)
fh.setFormatter(fmt)

# Attach
logger.addHandler(ch)
logger.addHandler(fh)

logger.info("Pipeline started")
```

### Timed Rotation (daily logs)
```python
from logging.handlers import TimedRotatingFileHandler
handler = TimedRotatingFileHandler(
    filename="/var/log/app/pipeline.log", when="midnight", interval=1, backupCount=14
)
```

### Contextual Logging with Filters
```python
class ContextFilter(logging.Filter):
    def __init__(self, **ctx):
        super().__init__()
        self.ctx = ctx
    def filter(self, record):
        for k, v in self.ctx.items():
            setattr(record, k, v)
        return True

logger = logging.getLogger("deploy")
logger.addFilter(ContextFilter(env="prod", team="platform"))

fmt = logging.Formatter(
    "%(asctime)s %(levelname)s env=%(env)s team=%(team)s %(message)s"
)
```

## 📦 JSON Logging for Structured Observability
```python
import json, logging

class JsonFormatter(logging.Formatter):
    def format(self, record):
        payload = {
            "timestamp": self.formatTime(record, "%Y-%m-%dT%H:%M:%S%z"),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "pid": record.process,
            "thread": record.threadName,
        }
        # include extra fields if present
        for key in ("env", "team", "service", "request_id"):
            if hasattr(record, key):
                payload[key] = getattr(record, key)
        return json.dumps(payload)

logger = logging.getLogger("api")
handler = logging.StreamHandler()
handler.setFormatter(JsonFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)

logger.info("User login", extra={"service": "auth", "request_id": "abc-123"})
```

## 🔄 Integrations: Syslog, HTTP, and Cloud
```python
from logging.handlers import SysLogHandler, HTTPHandler

# Syslog (local or remote)
syslog = SysLogHandler(address=("127.0.0.1", 514))
syslog.setLevel(logging.WARNING)
logger.addHandler(syslog)

# HTTP to log collector (not for high-throughput)
http = HTTPHandler(
    host="logs.internal:8080",
    url="/ingest",
    method="POST",
)
http.setLevel(logging.ERROR)
logger.addHandler(http)
```

- Cloud: use vendor SDKs/agents for CloudWatch, Stackdriver, Azure Monitor

## 🧪 Logging with Exception Context
```python
try:
    risky()
except Exception:
    logger.exception("Deployment step failed")  # includes traceback
```

## 🧵 Multiprocess/Thread-Safe Patterns

### Queue-based Logging (recommended for concurrency)
```python
import logging, logging.handlers, multiprocessing

def configure_listener(log_file):
    root = logging.getLogger()
    root.setLevel(logging.INFO)
    fh = logging.handlers.RotatingFileHandler(log_file, maxBytes=10_000_000, backupCount=3)
    fmt = logging.Formatter("%(asctime)s %(levelname)s %(processName)s %(name)s %(message)s")
    fh.setFormatter(fmt)
    root.addHandler(fh)

if __name__ == "__main__":
    queue = multiprocessing.Queue(-1)
    listener = logging.handlers.QueueListener(queue)
    listener.start()

    # workers use QueueHandler
    qh = logging.handlers.QueueHandler(queue)
    logger = logging.getLogger("worker")
    logger.addHandler(qh)
    logger.setLevel(logging.INFO)
    logger.info("Work item started")
```

## ⏱️ Performance and Sampling
- Use appropriate levels (DEBUG off in prod)
- Sample high-volume events (e.g., 1 in N)
- Avoid logging in tight loops; batch where possible
- Prefer lazy formatting: logger.debug("val=%s", expensive())

## 🔐 Security and Compliance
- Never log secrets, tokens, passwords, PII
- Mask sensitive fields before logging
- Enforce retention policies and secure log paths/permissions
- Time sync (NTP) to align timestamps across systems

## 🧭 Configuration via dictConfig
```python
import logging.config

LOGGING = {
  "version": 1,
  "disable_existing_loggers": False,
  "formatters": {
    "json": {"()": "__main__.JsonFormatter"},
    "default": {"format": "%(asctime)s %(levelname)s %(name)s %(message)s"}
  },
  "handlers": {
    "console": {"class": "logging.StreamHandler", "formatter": "default", "level": "INFO"},
    "file": {
      "class": "logging.handlers.TimedRotatingFileHandler",
      "formatter": "default", "filename": "/var/log/app/app.log",
      "when": "midnight", "backupCount": 7
    }
  },
  "loggers": {
    "": {"handlers": ["console", "file"], "level": "INFO"},
    "api": {"level": "INFO", "propagate": True}
  }
}

logging.config.dictConfig(LOGGING)
logger = logging.getLogger("api")
logger.info("Service up", extra={"env": "prod"})
```

## ⚡ Quick Reference
- Levels: DEBUG < INFO < WARNING < ERROR < CRITICAL
- Use one logger per module: logger = logging.getLogger(__name__)
- Structure: prefer JSON for ingestion pipelines
- Rotation: TimedRotatingFileHandler or RotatingFileHandler
- Context: extra= and Filters for enrichments
- Exceptions: logger.exception captures traceback

## 🚀 DevOps Use Cases
- Deployment pipelines: step logs, context, and durations
- Incident response: correlate request_id/trace_id across services
- Auditing: record config changes and user actions
- API gateways: access logs with user, method, status, latency
- Cron/agents: log start/end, exit codes, and metrics
