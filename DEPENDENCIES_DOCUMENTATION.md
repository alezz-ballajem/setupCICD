# Dependencies Documentation

## Overview

This document provides comprehensive information about all dependencies installed in the Sonod Docker image, their purposes, versions, and usage instructions.

## System Dependencies

### Core System Packages

#### Essential Build Tools
| Package | Version | Purpose | Usage |
|---------|---------|---------|-------|
| `build-essential` | Latest | C/C++ compilation tools | Building Python packages with C extensions |
| `make` | Latest | Build automation | Running Makefiles, build scripts |
| `ca-certificates` | Latest | SSL/TLS certificate authorities | Secure HTTPS connections |
| `wget` | Latest | HTTP/HTTPS file downloader | Downloading files, packages |

**Usage Examples**:
```bash
# Compile Python packages with C extensions
pip install psycopg2-binary

# Run build scripts
make build

# Download files securely
wget https://example.com/file.tar.gz
```

#### Version Control Systems
| Package | Version | Purpose | Usage |
|---------|---------|---------|-------|
| `git` | Latest | Distributed version control | Repository management, CI/CD |
| `mercurial` | Latest | Alternative version control | Legacy project support |

**Usage Examples**:
```bash
# Git operations (enhanced with autoshare)
git clone https://github.com/odoo/odoo.git
git pull origin main
git push origin feature-branch

# Mercurial operations
hg clone https://bitbucket.org/example/repo
hg pull
hg commit -m "Update"
```

#### Network and Transfer Tools
| Package | Version | Purpose | Usage |
|---------|---------|---------|-------|
| `openssh-client` | Latest | SSH client and tools | Secure remote connections |
| `rsync` | Latest | File synchronization | Efficient file transfers |

**Usage Examples**:
```bash
# SSH connections (with disabled host key checking)
ssh user@remote-host

# Secure file transfers
scp file.txt user@host:/remote/path/
rsync -av local/ user@host:/remote/

# SSH tunneling
ssh -L 8080:localhost:80 user@host
```

### Database Tools

#### PostgreSQL Support
| Package | Version | Purpose | Usage |
|---------|---------|---------|-------|
| `postgresql-client` | Latest | PostgreSQL database client | Database operations, backups |

**Usage Examples**:
```bash
# Connect to PostgreSQL database
psql -h localhost -U odoo -d odoo_db

# Database operations
pg_dump -h localhost -U odoo odoo_db > backup.sql
psql -h localhost -U odoo -d odoo_db < backup.sql

# Check connection
pg_isready -h localhost -p 5432
```

### Utility Tools

#### Process and Output Management
| Package | Version | Purpose | Usage |
|---------|---------|---------|-------|
| `tcl` | Latest | Tool Command Language | Scripting, automation |
| `expect` | Latest | Automation of interactive programs | Provides `unbuffer` utility |

**Usage Examples**:
```bash
# Remove output buffering for real-time logs
unbuffer python script.py | tee output.log

# Automate interactive programs
expect -c "
spawn ssh user@host
expect 'password:'
send 'mypassword\r'
interact
"
```

#### Internationalization
| Package | Version | Purpose | Usage |
|---------|---------|---------|-------|
| `gettext` | Latest | Internationalization utilities | Managing .po/.pot translation files |

**Usage Examples**:
```bash
# Extract translatable strings
xgettext --keyword=_ --language=Python --output=messages.pot *.py

# Update translation files
msgmerge --update fr.po messages.pot

# Compile translations
msgfmt fr.po -o fr.mo

# Validate translation files
msgfmt --check fr.po
```

## Python Environment

### Python Versions and Packages

#### Available Python Versions
| Version | Package | Development Headers | Virtual Environment |
|---------|---------|-------------------|-------------------|
| 3.8 | `python3.8` | `python3.8-dev` | `python3.8-venv` |
| 3.9 | `python3.9` | `python3.9-dev` | `python3.9-venv` |
| 3.10 | `python3.10` | `python3.10-dev` | `python3.10-venv` |
| 3.11 | `python3.11` | `python3.11-dev` | `python3.11-venv` |
| Default | `python3` | `python3-dev` | `python3-venv` |

**Usage Examples**:
```bash
# Create virtual environments with specific Python versions
virtualenv -p python3.8 venv38
virtualenv -p python3.9 venv39
virtualenv -p python3.10 venv310
virtualenv -p python3.11 venv311

# Activate and use
source venv311/bin/activate
python --version  # Shows Python 3.11.x
pip install odoo

# Test compatibility across versions
for version in 3.8 3.9 3.10 3.11; do
    echo "Testing Python $version"
    virtualenv -p python$version test-env
    source test-env/bin/activate
    pip install -r requirements.txt
    python -m pytest
    deactivate
    rm -rf test-env
done
```

### PIPX Environment

#### Global PIPX Configuration
```bash
# System-wide installation (as root)
PIPX_BIN_DIR=/usr/local/bin
PIPX_HOME=/opt/pipx

# User-specific installation (as gitlab-runner)
PIPX_BIN_DIR=/home/gitlab-runner/.local/bin
PIPX_HOME=/home/gitlab-runner/.local/pipx
```

#### Installed PIPX Tools
| Tool | Version | Purpose | Documentation |
|------|---------|---------|---------------|
| `virtualenv` | Latest | Virtual environment creation | [virtualenv docs](https://virtualenv.pypa.io/) |
| `git-autoshare` | >=1.0.0b4 | Git repository caching | [git-autoshare docs](https://pypi.org/project/git-autoshare/) |
| `manifestoo` | >=0.4.0 | Odoo manifest management | [manifestoo docs](https://pypi.org/project/manifestoo/) |

**Usage Examples**:
```bash
# List installed PIPX tools
pipx list

# Install additional tools
pipx install black
pipx install flake8
pipx install pre-commit

# Upgrade tools
pipx upgrade git-autoshare
pipx upgrade-all

# Uninstall tools
pipx uninstall tool-name
```

## Odoo-Specific Dependencies

### Document Processing Tools

#### PDF Generation
| Tool | Version | Purpose | Usage |
|------|---------|---------|-------|
| `wkhtmltopdf` | 0.12.6.1-2 | HTML to PDF conversion | Odoo report generation |

**Installation Details**:
- Downloaded from official GitHub releases
- SHA256 verified: `ee88d74834bdec650f7432c7d3ef1c981e42ae7a762a75a01f7f5da59abc18d5`
- Package: `wkhtmltox_0.12.6.1-2.jammy_amd64.deb`

**Usage Examples**:
```bash
# Basic PDF generation
wkhtmltopdf https://example.com output.pdf

# Advanced options for Odoo reports
wkhtmltopdf \
  --page-size A4 \
  --orientation Portrait \
  --margin-top 0.75in \
  --margin-right 0.75in \
  --margin-bottom 0.75in \
  --margin-left 0.75in \
  --encoding UTF-8 \
  --quiet \
  input.html output.pdf

# Test installation
wkhtmltopdf --version
```

#### Document Processing
| Package | Purpose | Supported Formats |
|---------|---------|------------------|
| `poppler-utils` | PDF manipulation | PDF reading, text extraction |
| `antiword` | Microsoft Word processing | .doc files |
| `libreoffice-writer` | Document processing | .docx, .odt, .rtf |
| `libreoffice-calc` | Spreadsheet processing | .xlsx, .ods, .csv |

**Usage Examples**:
```bash
# PDF text extraction
pdftotext document.pdf output.txt
pdftoppm document.pdf image-prefix

# Word document conversion
antiword document.doc > output.txt

# LibreOffice conversions
libreoffice --headless --convert-to pdf document.docx
libreoffice --headless --convert-to csv spreadsheet.xlsx

# Batch processing
for file in *.docx; do
    libreoffice --headless --convert-to pdf "$file"
done
```

### CSS and Web Asset Processing

#### CSS Tools
| Package | Purpose | Usage |
|---------|---------|-------|
| `node-clean-css` | CSS minification | Web asset optimization |
| `node-less` | LESS preprocessing | CSS compilation |

**Usage Examples**:
```bash
# CSS minification
cleancss -o styles.min.css styles.css

# LESS compilation
lessc styles.less styles.css

# Watch mode for development
lessc --watch styles.less styles.css
```

#### Visualization Tools
| Package | Purpose | Usage |
|---------|---------|-------|
| `graphviz` | Graph generation | Odoo workflow diagrams, reports |

**Usage Examples**:
```bash
# Generate workflow diagrams
dot -Tpng workflow.dot -o workflow.png
dot -Tsvg workflow.dot -o workflow.svg

# Create network diagrams
neato -Tpng network.dot -o network.png

# Odoo model relationship diagrams
# (typically generated by Odoo itself using graphviz)
```

## Python Package Management

### Virtual Environment Best Practices

#### Environment Creation
```bash
# Create environment with specific Python version
virtualenv -p python3.11 myproject-env

# Alternative using venv module
python3.11 -m venv myproject-env

# Create with system site packages
virtualenv --system-site-packages global-env
```

#### Environment Management
```bash
# Activate environment
source myproject-env/bin/activate

# Verify active environment
which python
python --version

# Install packages
pip install -r requirements.txt

# Generate requirements
pip freeze > requirements.txt

# Deactivate environment
deactivate
```

### Package Installation Strategies

#### Odoo Dependencies
```bash
# Standard Odoo requirements
pip install \
    Babel \
    chardet \
    cryptography \
    decorator \
    docutils \
    ebaysdk \
    freezegun \
    gevent \
    greenlet \
    idna \
    jinja2 \
    libsass \
    lxml \
    Markupsafe \
    num2words \
    ofxparse \
    passlib \
    Pillow \
    polib \
    psutil \
    psycopg2 \
    pydot \
    pypdf2 \
    PyPDF4 \
    pyserial \
    python-dateutil \
    python-stdnum \
    pytz \
    pyusb \
    qrcode \
    reportlab \
    requests \
    urllib3 \
    vobject \
    Werkzeug \
    xlrd \
    XlsxWriter \
    xlwt \
    zeep
```

#### Development Tools
```bash
# Testing frameworks
pip install pytest pytest-cov pytest-mock

# Code quality tools
pip install black flake8 isort mypy

# Odoo development tools
pip install odoo-test-helper click-odoo-contrib

# Documentation tools
pip install sphinx sphinx-rtd-theme
```

## Tool-Specific Documentation

### Git Autoshare

#### Configuration Management
```bash
# View current configuration
cat /home/gitlab-runner/.config/git-autoshare/repos.yml

# Check autoshare status
git autoshare status

# Manual cache management
git autoshare clean
git autoshare gc
```

#### Advanced Configuration
```yaml
# Custom configuration example
your-gitlab.com:
  enterprise:
    orgs:
      - your-company
    private: True
    token_env: GITLAB_TOKEN
  "*":
    orgs:
      - open-source-org

bitbucket.org:
  "*":
    orgs:
      - your-bitbucket-org
```

### Manifestoo

#### Odoo Manifest Analysis
```bash
# List addon dependencies
manifestoo list-depends --separator=, /path/to/addons

# Check for dependency issues
manifestoo check-depends /path/to/addons

# List external dependencies
manifestoo list-external-depends /path/to/addons

# Generate dependency tree
manifestoo list-depends --format=tree /path/to/addons
```

#### CI Integration
```bash
# Validate all manifests in CI
manifestoo check-depends .
if [ $? -ne 0 ]; then
    echo "Manifest validation failed"
    exit 1
fi

# Generate addon list for testing
ADDONS=$(manifestoo list --separator=, .)
python odoo-bin -i $ADDONS --test-enable --stop-after-init
```

## Performance Considerations

### Caching Strategies

#### Package Caching
```bash
# Pip cache configuration
export PIP_CACHE_DIR=/home/gitlab-runner/.cache/pip
export PIP_NO_CACHE_DIR=0

# Check cache size
du -sh /home/gitlab-runner/.cache/pip

# Clean cache if needed
pip cache purge
```

#### Git Repository Caching
```bash
# Git autoshare cache location
export GIT_AUTOSHARE_CACHE=/home/gitlab-runner/.cache/git-autoshare

# Monitor cache usage
du -sh /home/gitlab-runner/.cache/git-autoshare

# Cache maintenance
git autoshare gc
git autoshare clean --dry-run
```

### Memory Management

#### Python Memory Optimization
```bash
# Set Python memory limits
export PYTHONMALLOC=malloc
export MALLOC_MMAP_THRESHOLD_=1048576

# Monitor memory usage
python -c "
import psutil
import os
process = psutil.Process(os.getpid())
print(f'Memory usage: {process.memory_info().rss / 1024 / 1024:.2f} MB')
"
```

#### System Resource Monitoring
```bash
# Check available resources
free -h
df -h
top -bn1 | head -20

# Monitor during builds
watch -n 1 'free -h; echo "---"; df -h'
```

## Dependency Compatibility Matrix

### Python Version Compatibility

#### Odoo Version Support
| Odoo Version | Python 3.8 | Python 3.9 | Python 3.10 | Python 3.11 |
|--------------|-------------|-------------|--------------|--------------|
| 16.0 | ✅ | ✅ | ✅ | ✅ |
| 15.0 | ✅ | ✅ | ✅ | ⚠️ |
| 14.0 | ✅ | ✅ | ⚠️ | ❌ |
| 13.0 | ✅ | ⚠️ | ❌ | ❌ |

**Legend**:
- ✅ Fully supported
- ⚠️ Limited support or requires specific versions
- ❌ Not supported

#### Package Compatibility
```bash
# Check package compatibility
python3.11 -c "
import sys
print(f'Python version: {sys.version}')

# Test critical packages
try:
    import lxml
    print('✅ lxml: OK')
except ImportError:
    print('❌ lxml: Failed')

try:
    import psycopg2
    print('✅ psycopg2: OK')
except ImportError:
    print('❌ psycopg2: Failed')
"
```

### System Library Dependencies

#### Graphics and Document Libraries
| Library | Dependent Packages | Purpose |
|---------|-------------------|---------|
| `libxml2` | lxml, beautifulsoup4 | XML/HTML processing |
| `libxslt` | lxml | XSLT transformations |
| `libjpeg` | Pillow | JPEG image processing |
| `libpng` | Pillow | PNG image processing |
| `libfreetype` | matplotlib, reportlab | Font rendering |
| `libpq` | psycopg2 | PostgreSQL connectivity |

**Verification Commands**:
```bash
# Check library availability
ldconfig -p | grep libxml2
ldconfig -p | grep libpq
ldconfig -p | grep libjpeg

# Test Python package compilation
pip install --no-binary :all: lxml
pip install --no-binary :all: psycopg2
```

## Security Dependencies

### SSL/TLS Support
```bash
# Verify SSL certificate handling
python3 -c "
import ssl
import requests

# Test HTTPS connectivity
response = requests.get('https://github.com')
print(f'HTTPS connection: {response.status_code}')

# Check SSL context
context = ssl.create_default_context()
print(f'SSL context: {context.protocol}')
"
```

### Cryptographic Libraries
```bash
# Test cryptographic functionality
python3 -c "
import hashlib
import hmac
import secrets

# Test hashing
data = b'test data'
print(f'SHA256: {hashlib.sha256(data).hexdigest()}')

# Test secure random
token = secrets.token_hex(16)
print(f'Secure token: {token}')
"
```

## Development Dependencies

### Code Quality Tools

#### Linting and Formatting
```bash
# Install development tools
pipx install black
pipx install flake8
pipx install isort
pipx install mypy

# Usage examples
black --check .
flake8 --max-line-length=88 .
isort --check-only .
mypy --strict .
```

#### Testing Frameworks
```bash
# Install testing tools
pip install pytest pytest-cov pytest-mock pytest-xdist

# Run tests with coverage
pytest --cov=src --cov-report=html tests/

# Parallel testing
pytest -n auto tests/

# Generate test reports
pytest --junitxml=test-results.xml tests/
```

### Documentation Tools
```bash
# Install documentation tools
pipx install sphinx
pip install sphinx-rtd-theme sphinx-autodoc-typehints

# Generate documentation
sphinx-quickstart docs
sphinx-build -b html docs docs/_build/html

# Auto-generate API docs
sphinx-apidoc -o docs/api src/
```

## Troubleshooting Dependencies

### Common Issues and Solutions

#### Package Installation Failures
```bash
# Update package lists
sudo apt-get update

# Fix broken packages
sudo apt-get install -f

# Install missing development headers
sudo apt-get install python3-dev libpq-dev libxml2-dev libxslt1-dev

# Check disk space
df -h
```

#### Python Package Compilation Issues
```bash
# Install compilation dependencies
sudo apt-get install gcc g++ make

# Set compilation flags
export CFLAGS="-I/usr/include/python3.11"
export LDFLAGS="-L/usr/lib/python3.11"

# Use pre-compiled wheels when available
pip install --only-binary=all package-name

# Force compilation from source
pip install --no-binary :all: package-name
```

#### Virtual Environment Issues
```bash
# Recreate corrupted environment
rm -rf venv
virtualenv -p python3.11 venv
source venv/bin/activate

# Fix pip in environment
python -m ensurepip --upgrade
python -m pip install --upgrade pip

# Verify environment
which python
which pip
pip --version
```

### Dependency Verification

#### Health Check Script
```bash
#!/bin/bash
# dependency-check.sh

echo "=== System Dependencies ==="
for cmd in git hg ssh rsync make psql wkhtmltopdf; do
    if command -v $cmd >/dev/null 2>&1; then
        echo "✅ $cmd: $(command -v $cmd)"
    else
        echo "❌ $cmd: Not found"
    fi
done

echo "=== Python Versions ==="
for version in 3.8 3.9 3.10 3.11; do
    if command -v python$version >/dev/null 2>&1; then
        echo "✅ Python $version: $(python$version --version)"
    else
        echo "❌ Python $version: Not found"
    fi
done

echo "=== PIPX Tools ==="
pipx list --short

echo "=== Git Configuration ==="
git config --global --list | grep -E "(user\.|safe\.)"

echo "=== SSH Configuration ==="
if [ -f /home/gitlab-runner/.ssh/config ]; then
    echo "✅ SSH config exists"
    cat /home/gitlab-runner/.ssh/config
else
    echo "❌ SSH config missing"
fi

echo "=== Git Autoshare ==="
if [ -f /home/gitlab-runner/.config/git-autoshare/repos.yml ]; then
    echo "✅ Git autoshare config exists"
    git autoshare status 2>/dev/null || echo "⚠️ Git autoshare not accessible"
else
    echo "❌ Git autoshare config missing"
fi
```

#### Usage
```bash
# Make script executable
chmod +x dependency-check.sh

# Run health check
./dependency-check.sh

# Use in CI/CD
docker run --rm ghcr.io/opsivist/sonod:22.04 /bin/bash -c "
cat > /tmp/check.sh << 'EOF'
$(cat dependency-check.sh)
EOF
chmod +x /tmp/check.sh
/tmp/check.sh
"
```

## Upgrade and Maintenance

### Dependency Updates

#### System Package Updates
```dockerfile
# In Dockerfile, update base packages
RUN apt-get update && apt-get dist-upgrade -y

# Check for security updates
RUN apt-get update && apt-get upgrade -y --only-upgrade
```

#### Python Tool Updates
```bash
# Update PIPX tools
pipx upgrade-all

# Update specific tools
pipx upgrade git-autoshare
pipx upgrade manifestoo

# Check for outdated packages
pipx list --include-injected
```

#### Version Pinning Strategy
```dockerfile
# Pin critical versions
RUN pipx install --pip-args="--no-cache-dir" "git-autoshare>=1.0.0b4,<2.0.0"
RUN pipx install --pip-args="--no-cache-dir" "manifestoo>=0.4.0,<1.0.0"

# Pin system packages when needed
RUN apt-get install -y python3.11=3.11.0-1ubuntu1
```

### Compatibility Testing

#### Multi-Version Testing
```bash
#!/bin/bash
# test-compatibility.sh

for py_version in 3.8 3.9 3.10 3.11; do
    echo "Testing Python $py_version"
    
    # Create test environment
    virtualenv -p python$py_version test-env-$py_version
    source test-env-$py_version/bin/activate
    
    # Install and test
    pip install odoo-test-requirements.txt
    python -c "import odoo; print('Odoo import: OK')"
    
    # Cleanup
    deactivate
    rm -rf test-env-$py_version
done
```

#### Integration Testing
```bash
# Test complete workflow
git clone https://github.com/odoo/odoo.git --depth 1 --branch 16.0
cd odoo
virtualenv -p python3.11 venv
source venv/bin/activate
pip install -r requirements.txt
python odoo-bin --help
```

## Best Practices

### Dependency Management
1. **Pin Versions**: Use specific versions for reproducible builds
2. **Cache Optimization**: Leverage all available caching mechanisms
3. **Security Updates**: Regularly update dependencies for security patches
4. **Testing**: Test dependency updates in staging before production

### Performance Optimization
1. **Parallel Installation**: Use pip's parallel download features
2. **Wheel Caching**: Prefer binary wheels over source compilation
3. **Layer Optimization**: Order Dockerfile commands for optimal caching
4. **Resource Monitoring**: Monitor memory and disk usage during builds

### Security Considerations
1. **Vulnerability Scanning**: Regularly scan for known vulnerabilities
2. **Minimal Installation**: Only install required packages
3. **User Permissions**: Run with minimal required privileges
4. **Secret Management**: Properly handle sensitive configuration data