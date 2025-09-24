# Section 2: Setting Up and Running Python - DevOps Cheat Sheet

## 🐍 Python Installation Essentials

### System Installation Methods
```bash
# macOS (using Homebrew)
brew install python@3.11

# Linux (Ubuntu/Debian)
sudo apt update && sudo apt install python3.11

# CentOS/RHEL
sudo yum install python3.11

# Windows (Chocolatey)
choco install python
```

### Version Management with pyenv
```bash
# Install pyenv
curl https://pyenv.run | bash

# List available versions
pyenv install --list

# Install specific version
pyenv install 3.11.5

# Set global version
pyenv global 3.11.5

# Set local version for project
pyenv local 3.10.8
```

## 📦 Virtual Environment Management

### Built-in venv (Recommended for DevOps)
```bash
# Create virtual environment
python -m venv devops-env

# Activate (Linux/macOS)
source devops-env/bin/activate

# Activate (Windows)
devops-env\Scripts\activate

# Deactivate
deactivate

# Remove environment
rm -rf devops-env
```

### Advanced: conda for Data-Heavy DevOps
```bash
# Create environment with specific Python version
conda create -n devops-tools python=3.11

# Activate environment
conda activate devops-tools

# Install packages
conda install requests pyyaml

# Export environment
conda env export > environment.yml

# Create from file
conda env create -f environment.yml
```

## 🔧 Package Management Mastery

### pip Essentials
```bash
# Upgrade pip
python -m pip install --upgrade pip

# Install packages
pip install requests pyyaml click

# Install from requirements
pip install -r requirements.txt

# Generate requirements
pip freeze > requirements.txt

# Install development dependencies
pip install -e .[dev]

# Install from git repository
pip install git+https://github.com/user/repo.git
```

### Modern Alternative: pipenv
```bash
# Install pipenv
pip install pipenv

# Initialize project
pipenv --python 3.11

# Install packages
pipenv install requests
pipenv install pytest --dev

# Activate shell
pipenv shell

# Run commands in environment
pipenv run python script.py

# Generate requirements
pipenv requirements > requirements.txt
```

## 🚀 Python Execution Methods

### Interactive Python Sessions
```bash
# Standard Python REPL
python

# Enhanced IPython (install: pip install ipython)
ipython

# Jupyter for exploratory work
jupyter notebook
```

### Script Execution
```bash
# Direct execution
python script.py

# Module execution
python -m module_name

# With arguments
python script.py arg1 arg2 --flag

# One-liner execution
python -c "print('Hello DevOps!')"
```

## 🔍 Python Path and Import System

### Understanding sys.path
```python
import sys
print(sys.path)  # Current Python path

# Add custom path
sys.path.insert(0, '/custom/path')
```

### Environment Variables
```bash
# Set PYTHONPATH
export PYTHONPATH="/path/to/modules:$PYTHONPATH"

# Disable .pyc files
export PYTHONDONTWRITEBYTECODE=1

# Enable development mode
export PYTHONDEVMODE=1
```

## 🛠️ DevOps-Specific Setup Patterns

### Dockerfile Python Setup
```dockerfile
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy requirements first (layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Set Python path
ENV PYTHONPATH=/app

# Run application
CMD ["python", "app.py"]
```

### CI/CD Python Setup
```yaml
# GitHub Actions example
steps:
  - uses: actions/setup-python@v4
    with:
      python-version: '3.11'
      cache: 'pip'
  
  - name: Install dependencies
    run: |
      pip install --upgrade pip
      pip install -r requirements.txt
```

## 📋 Version Checking and Diagnostics

### System Information Commands
```bash
# Python version
python --version
python -V

# Detailed version info
python -c "import sys; print(sys.version)"

# Installation location
which python
whereis python

# Site packages location
python -c "import site; print(site.getsitepackages())"

# Installed packages
pip list
pip show package_name
```

### Health Check Script
```python
#!/usr/bin/env python3
import sys
import subprocess

def health_check():
    print(f"Python Version: {sys.version}")
    print(f"Python Path: {sys.executable}")
    
    # Check critical packages
    packages = ['requests', 'pyyaml', 'click']
    for pkg in packages:
        try:
            __import__(pkg)
            print(f"✅ {pkg} - OK")
        except ImportError:
            print(f"❌ {pkg} - Missing")

if __name__ == "__main__":
    health_check()
```

## ⚡ DevOps Quick Wins

### 1. Automation Script Template
```python
#!/usr/bin/env python3
"""
DevOps Automation Script Template
"""
import argparse
import logging
import sys
from pathlib import Path

def setup_logging():
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s - %(levelname)s - %(message)s'
    )

def main():
    parser = argparse.ArgumentParser(description='DevOps Tool')
    parser.add_argument('--config', type=Path, help='Config file path')
    parser.add_argument('--dry-run', action='store_true', help='Show what would be done')
    
    args = parser.parse_args()
    
    setup_logging()
    logging.info("Starting automation...")
    
    # Your automation logic here
    
if __name__ == "__main__":
    main()
```

### 2. Environment Setup Script
```bash
#!/bin/bash
# setup-devops-python.sh

set -e

echo "Setting up Python DevOps environment..."

# Create project directory
mkdir -p devops-project
cd devops-project

# Create virtual environment
python -m venv venv
source venv/bin/activate

# Install essential packages
cat > requirements.txt << EOF
requests>=2.28.0
pyyaml>=6.0
click>=8.0.0
pytest>=7.0.0
black>=22.0.0
flake8>=5.0.0
EOF

pip install -r requirements.txt

echo "✅ DevOps Python environment ready!"
echo "Activate with: source venv/bin/activate"
```

### 3. Common DevOps Packages
```txt
# requirements-devops.txt
# Core automation
requests>=2.28.0          # HTTP requests
pyyaml>=6.0               # YAML processing
click>=8.0.0              # CLI interfaces
rich>=12.0.0              # Beautiful terminal output

# Infrastructure
boto3>=1.26.0             # AWS SDK
ansible>=6.0.0            # Configuration management
terraform-compliance>=1.3  # Terraform testing

# Monitoring & Logging
prometheus-client>=0.15.0  # Metrics
structlog>=22.0.0         # Structured logging

# Testing
pytest>=7.0.0             # Testing framework
pytest-cov>=4.0.0         # Coverage

# Code Quality
black>=22.0.0             # Code formatting
flake8>=5.0.0             # Linting
mypy>=0.991               # Type checking
```

## 🔧 Troubleshooting Common Issues

### Permission Issues
```bash
# Fix pip permissions (avoid sudo pip)
pip install --user package_name

# Or use virtual environment
python -m venv venv && source venv/bin/activate
```

### SSL Certificate Issues
```bash
# Upgrade certificates (macOS)
/Applications/Python\ 3.x/Install\ Certificates.command

# Trust pip with corporate proxy
pip install --trusted-host pypi.org --trusted-host pypi.python.org package_name
```

### Import Path Issues
```python
# Add current directory to path
import sys
from pathlib import Path
sys.path.insert(0, str(Path(__file__).parent))
```

## 🎯 DevOps Best Practices

1. **Always use virtual environments** - Isolate project dependencies
2. **Pin package versions** - Use specific versions in requirements.txt
3. **Use .python-version files** - For consistent team environments
4. **Automate environment setup** - Create setup scripts for new team members
5. **Test multiple Python versions** - Use CI/CD matrix builds
6. **Monitor security vulnerabilities** - Use `pip audit` (Python 3.10+)

## 🚀 Next Steps

After mastering Python setup:
- Learn Python fundamentals (variables, data structures)
- Practice with real DevOps scenarios
- Integrate Python tools into your CI/CD pipeline
- Build your first automation script

---
*This cheat sheet covers Python installation, environment management, and DevOps-specific setup patterns for reliable automation development.*
