# Section 5: Error Handling and Context Managers - Cheat Sheet

## 🎯 Section Overview
**Duration:** 1hr 32min  
**Focus:** Robust error handling and resource management for production automation

## 📋 Key Concepts

### Exception Handling Fundamentals

#### Basic Try-Except Structure
```python
try:
    result = risky_operation()
except SpecificError as e:
    print(f"Specific error occurred: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
else:
    print("Operation successful")
finally:
    print("Always executes")
```

#### Multiple Exception Types
```python
try:
    file_operations()
except FileNotFoundError:
    print("File not found")
except PermissionError:
    print("Permission denied")
except (IOError, OSError) as e:
    print(f"System error: {e}")
```

### Custom Exceptions for DevOps

#### Infrastructure-Specific Exceptions
```python
class InfrastructureError(Exception):
    """Base exception for infrastructure operations"""
    pass

class ServiceUnavailableError(InfrastructureError):
    """Raised when a service is unavailable"""
    def __init__(self, service_name, status_code=None):
        self.service_name = service_name
        self.status_code = status_code
        super().__init__(f"Service {service_name} unavailable (status: {status_code})")

class ConfigurationError(InfrastructureError):
    """Raised when configuration is invalid"""
    pass
```

#### Usage in Automation Scripts
```python
def check_service_health(service_url):
    try:
        response = requests.get(service_url, timeout=10)
        if response.status_code != 200:
            raise ServiceUnavailableError(
                service_url, 
                response.status_code
            )
    except requests.RequestException as e:
        raise ServiceUnavailableError(service_url) from e
```

### Context Managers for Resource Management

#### Understanding the `with` Statement
```python
# Without context manager (risky)
file = open('config.txt', 'r')
data = file.read()
file.close()  # Might not execute if error occurs

# With context manager (safe)
with open('config.txt', 'r') as file:
    data = file.read()
# File automatically closed even if error occurs
```

#### Built-in Context Managers
```python
# File operations
with open('log.txt', 'a') as log_file:
    log_file.write(f"Process started at {datetime.now()}\n")

# Threading locks
import threading
lock = threading.Lock()
with lock:
    # Critical section
    shared_resource += 1

# Database connections
import sqlite3
with sqlite3.connect('database.db') as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
```

### Creating Custom Context Managers

#### Class-Based Context Manager
```python
class DatabaseConnection:
    def __init__(self, db_name):
        self.db_name = db_name
        self.connection = None
    
    def __enter__(self):
        print(f"Connecting to {self.db_name}")
        self.connection = sqlite3.connect(self.db_name)
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.connection:
            if exc_type is None:
                self.connection.commit()
            else:
                self.connection.rollback()
            self.connection.close()
            print(f"Disconnected from {self.db_name}")

# Usage
with DatabaseConnection('app.db') as db:
    cursor = db.cursor()
    cursor.execute("INSERT INTO logs VALUES (?, ?)", (datetime.now(), "System check"))
```

#### Function-Based Context Manager (@contextmanager)
```python
from contextlib import contextmanager
import os

@contextmanager
def change_directory(path):
    """Context manager to temporarily change directory"""
    original_path = os.getcwd()
    try:
        os.chdir(path)
        yield path
    finally:
        os.chdir(original_path)

# Usage
with change_directory('/tmp'):
    # Work in /tmp directory
    with open('temp_file.txt', 'w') as f:
        f.write('Temporary data')
# Automatically returns to original directory
```

### DevOps Context Manager Examples

#### Service Management Context Manager
```python
@contextmanager
def service_maintenance(service_name):
    """Context manager for service maintenance operations"""
    print(f"Stopping service: {service_name}")
    subprocess.run(['systemctl', 'stop', service_name], check=True)
    try:
        yield
    except Exception as e:
        print(f"Maintenance failed: {e}")
        raise
    finally:
        print(f"Starting service: {service_name}")
        subprocess.run(['systemctl', 'start', service_name], check=True)

# Usage
with service_maintenance('nginx'):
    # Perform maintenance tasks
    update_config_files()
    backup_logs()
```

#### Environment Variable Context Manager
```python
@contextmanager
def environment_variables(**env_vars):
    """Temporarily set environment variables"""
    old_environ = dict(os.environ)
    os.environ.update(env_vars)
    try:
        yield
    finally:
        os.environ.clear()
        os.environ.update(old_environ)

# Usage
with environment_variables(DEBUG='true', LOG_LEVEL='debug'):
    # Run operations with specific environment
    run_tests()
```

## 🛠️ Production Error Handling Patterns

### Retry Mechanism with Exponential Backoff
```python
import time
import random

def retry_with_backoff(func, max_retries=3, base_delay=1):
    """Retry function with exponential backoff"""
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise e
            delay = base_delay * (2 ** attempt) + random.uniform(0, 1)
            print(f"Attempt {attempt + 1} failed, retrying in {delay:.2f}s")
            time.sleep(delay)

# Usage
def unreliable_api_call():
    response = requests.get('https://api.example.com/data')
    response.raise_for_status()
    return response.json()

data = retry_with_backoff(unreliable_api_call)
```

### Graceful Error Handling for Automation
```python
def deploy_application(app_name, version):
    """Deploy application with comprehensive error handling"""
    deployment_steps = [
        ('validate_config', validate_configuration),
        ('backup_current', backup_current_version),
        ('deploy_new', deploy_new_version),
        ('run_tests', run_health_checks),
        ('update_load_balancer', update_lb_config)
    ]
    
    completed_steps = []
    
    try:
        for step_name, step_func in deployment_steps:
            print(f"Executing: {step_name}")
            step_func(app_name, version)
            completed_steps.append(step_name)
            print(f"Completed: {step_name}")
            
    except Exception as e:
        print(f"Deployment failed at step: {step_name}")
        print(f"Error: {e}")
        
        # Rollback completed steps
        rollback_deployment(completed_steps, app_name)
        raise
```

### Error Aggregation for Batch Operations
```python
class BatchOperationError(Exception):
    """Exception for collecting multiple operation errors"""
    def __init__(self, errors):
        self.errors = errors
        super().__init__(f"Batch operation failed with {len(errors)} errors")

def process_servers_batch(servers):
    """Process multiple servers, collecting all errors"""
    errors = []
    successful = []
    
    for server in servers:
        try:
            result = process_server(server)
            successful.append((server, result))
        except Exception as e:
            errors.append((server, str(e)))
    
    if errors:
        # Log all errors but continue processing
        for server, error in errors:
            print(f"Failed to process {server}: {error}")
        
        # Optionally raise aggregate error
        if len(errors) > len(successful):
            raise BatchOperationError(errors)
    
    return successful, errors
```

## 🔧 Practical DevOps Applications

### Configuration File Handler
```python
@contextmanager
def config_backup(config_path):
    """Backup configuration file before modification"""
    backup_path = f"{config_path}.backup"
    shutil.copy2(config_path, backup_path)
    try:
        yield config_path
    except Exception:
        # Restore backup on error
        shutil.copy2(backup_path, config_path)
        raise
    finally:
        # Clean up backup file
        if os.path.exists(backup_path):
            os.remove(backup_path)

# Usage
with config_backup('/etc/nginx/nginx.conf') as config_file:
    modify_nginx_config(config_file)
    test_nginx_config()
```

### Resource Monitoring Context Manager
```python
@contextmanager
def monitor_resources(operation_name):
    """Monitor system resources during operation"""
    import psutil
    
    start_time = time.time()
    start_memory = psutil.virtual_memory().used
    start_cpu = psutil.cpu_percent()
    
    try:
        yield
    finally:
        end_time = time.time()
        end_memory = psutil.virtual_memory().used
        end_cpu = psutil.cpu_percent()
        
        duration = end_time - start_time
        memory_diff = end_memory - start_memory
        
        print(f"Operation: {operation_name}")
        print(f"Duration: {duration:.2f}s")
        print(f"Memory change: {memory_diff / 1024 / 1024:.2f} MB")
        print(f"CPU usage: {end_cpu:.1f}%")

# Usage
with monitor_resources("Database migration"):
    run_database_migration()
```

## ⚡ Quick Reference

### Exception Hierarchy
```
BaseException
 ├── Exception
 │   ├── ArithmeticError
 │   ├── LookupError
 │   │   ├── IndexError
 │   │   └── KeyError
 │   ├── OSError
 │   │   ├── FileNotFoundError
 │   │   └── PermissionError
 │   ├── ValueError
 │   └── TypeError
 └── SystemExit
```

### Best Practices
- **Specific Exceptions**: Catch specific exceptions rather than bare `except:`
- **Resource Cleanup**: Always use context managers for resources
- **Error Information**: Include relevant context in exception messages
- **Graceful Degradation**: Handle errors gracefully in production
- **Logging**: Log errors with sufficient detail for debugging
- **Retry Logic**: Implement retry mechanisms for transient failures

### Common Gotchas
- Don't catch `BaseException` or bare `except:`
- Always close resources in `finally` or use context managers
- Be careful with exception chaining using `raise ... from ...`
- Context managers should handle exceptions in `__exit__`
- Use `@contextmanager` for simple cases, classes for complex ones

## 🚀 DevOps Use Cases
- **Service Management**: Graceful service stops/starts with rollback
- **Configuration Management**: Safe config file modifications
- **Database Operations**: Transaction management with rollback
- **Resource Monitoring**: Track resource usage during operations
- **Deployment Automation**: Multi-step deployments with rollback
- **API Integration**: Robust error handling for external services
- **Batch Processing**: Error collection and partial success handling
