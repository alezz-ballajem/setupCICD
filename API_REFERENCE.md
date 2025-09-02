# Complete API Reference

## Overview

This document provides a comprehensive API reference for the Sonod Docker image, including all public interfaces, commands, configurations, and integration points.

## Docker Image API

### Image Specification

#### Registry Information
```
Registry: ghcr.io
Repository: opsivist/sonod
Tags: 22.04, 20.04, latest
Full Image Name: ghcr.io/opsivist/sonod:22.04
```

#### Image Metadata
```bash
# Get image information
docker inspect ghcr.io/opsivist/sonod:22.04

# Key metadata
Base Image: ubuntu:22.04
Architecture: linux/amd64
User: gitlab-runner (UID: 1000)
Working Directory: /home/gitlab-runner
Shell: /bin/bash
```

### Environment Variables API

#### Required Environment Variables
| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `PGHOST` | string | - | PostgreSQL host address |
| `PGUSER` | string | - | PostgreSQL username |
| `PGPASSWORD` | string | - | PostgreSQL password |
| `PGDATABASE` | string | - | PostgreSQL database name |

#### Optional Environment Variables
| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `PGPORT` | integer | 5432 | PostgreSQL port |
| `ODOO_RC` | string | - | Path to Odoo configuration file |
| `PYTHONPATH` | string | - | Additional Python module paths |
| `PIP_INDEX_URL` | string | - | Custom PyPI index URL |
| `PIP_EXTRA_INDEX_URL` | string | - | Additional PyPI index URLs |

#### System Environment Variables (Pre-configured)
| Variable | Value | Purpose |
|----------|-------|---------|
| `LANG` | `C.UTF-8` | System locale |
| `LC_ALL` | `C.UTF-8` | System locale (all categories) |
| `PIPX_BIN_DIR` | `/home/gitlab-runner/.local/bin` | PIPX binary directory |
| `PIPX_HOME` | `/home/gitlab-runner/.local/pipx` | PIPX home directory |
| `PATH` | `/home/gitlab-runner/.local/bin:$PATH` | Extended PATH |

### Volume API

#### Recommended Volume Mounts
```bash
# Performance optimization volumes
-v pip-cache:/home/gitlab-runner/.cache/pip
-v git-cache:/home/gitlab-runner/.cache/git-autoshare
-v acsoo-cache:/home/gitlab-runner/.cache/acsoo-wheel

# Application volumes
-v $(pwd):/workspace                                    # Source code
-v odoo-data:/home/gitlab-runner/.local/share/Odoo     # Odoo data
-v logs:/workspace/logs                                 # Application logs

# Configuration volumes
-v custom-ssh:/home/gitlab-runner/.ssh                 # Custom SSH config
-v custom-git:/home/gitlab-runner/.config/git-autoshare # Custom git-autoshare config
```

#### Volume Specifications
| Mount Point | Type | Purpose | Recommended Size |
|-------------|------|---------|------------------|
| `/home/gitlab-runner/.cache/pip` | Cache | Python package cache | 1-5 GB |
| `/home/gitlab-runner/.cache/git-autoshare` | Cache | Git repository cache | 5-20 GB |
| `/home/gitlab-runner/.local/share/Odoo` | Data | Odoo filestore and sessions | 1-10 GB |
| `/workspace` | Source | Project source code | Variable |

### Port API

#### Default Ports
| Port | Protocol | Service | Purpose |
|------|----------|---------|---------|
| 8069 | HTTP | Odoo Web | Main Odoo web interface |
| 8072 | HTTP | Odoo Longpolling | Real-time notifications |
| 5432 | TCP | PostgreSQL | Database connection (external) |

#### Port Mapping Examples
```bash
# Basic port mapping
docker run -p 8069:8069 ghcr.io/opsivist/sonod:22.04

# Multiple services
docker run \
  -p 8069:8069 \
  -p 8072:8072 \
  -p 5432:5432 \
  ghcr.io/opsivist/sonod:22.04

# Custom port mapping
docker run -p 9000:8069 ghcr.io/opsivist/sonod:22.04
```

## Command Line API

### Git Wrapper API

#### Function Signature
```bash
git <command> [arguments...]
```

#### Commands
| Command | Behavior | Implementation |
|---------|----------|----------------|
| `git clone <url>` | Uses git-autoshare | `/usr/bin/git autoshare-clone <url>` |
| `git <other>` | Standard git | `/usr/bin/git <other>` |

#### Examples
```bash
# Enhanced clone (uses autoshare)
git clone https://github.com/odoo/odoo.git

# Standard git commands
git status
git add .
git commit -m "message"
git push origin main

# Direct autoshare usage
/usr/bin/git autoshare-clone https://github.com/odoo/odoo.git
```

#### Return Codes
| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General git error |
| 128 | Git repository error |

### PIPX Tools API

#### Manifestoo API
```bash
# Command structure
manifestoo <command> [options] <path>

# Available commands
manifestoo list [--separator=SEP] <path>              # List addons
manifestoo list-depends [--separator=SEP] <path>      # List dependencies
manifestoo list-external-depends <path>               # List external dependencies
manifestoo check-depends <path>                       # Validate dependencies
manifestoo create-addon <name>                        # Create addon structure
```

#### Examples and Return Values
```bash
# List addons (returns comma-separated addon names)
$ manifestoo list --separator=, addons/
addon1,addon2,addon3

# Check dependencies (returns 0 if valid, 1 if issues found)
$ manifestoo check-depends addons/
✓ All dependencies satisfied
$ echo $?
0

# List with tree format
$ manifestoo list-depends --format=tree addons/
addon1
├── base
└── mail
addon2
├── base
└── web
```

#### Git Autoshare API
```bash
# Command structure
git autoshare <command> [options]

# Available commands
git autoshare status                    # Show cache status
git autoshare clean [--dry-run]        # Clean cache
git autoshare gc                        # Garbage collection
git autoshare-clone <url> [options]    # Clone with caching
```

#### Status Information
```bash
$ git autoshare status
Git autoshare status:
  Cache directory: /home/gitlab-runner/.cache/git-autoshare
  Total repositories: 15
  Total size: 2.3 GB
  Last cleanup: 2024-01-15 10:30:00
```

## Configuration API

### Git Autoshare Configuration Schema

#### YAML Structure
```yaml
# Schema definition
<host>:                          # Git host (e.g., github.com)
  enterprise:                    # Enterprise/private repositories
    orgs:                       # List of organizations
      - <org-name>              # Organization name
    private: <boolean>          # Mark as private repositories
    token_env: <env-var>        # Environment variable for auth token
  "*":                          # Wildcard for other repositories
    orgs:                       # List of organizations
      - <org-name>              # Organization name
```

#### Configuration Examples
```yaml
# Basic configuration
github.com:
  "*":
    orgs:
      - myorg
      - opensource-org

# Advanced configuration with authentication
github.com:
  enterprise:
    orgs:
      - private-org
    private: true
    token_env: GITHUB_TOKEN
  "*":
    orgs:
      - public-org

gitlab.com:
  enterprise:
    orgs:
      - company
    private: true
    token_env: GITLAB_TOKEN
```

#### Validation
```bash
# Validate configuration syntax
python3 -c "
import yaml
with open('/home/gitlab-runner/.config/git-autoshare/repos.yml') as f:
    config = yaml.safe_load(f)
    print('✅ Configuration is valid YAML')
    print(f'Configured hosts: {list(config.keys())}')
"
```

### SSH Configuration API

#### Configuration Format
```ssh
# SSH client configuration format
Host <pattern>
  <option> <value>
  <option> <value>
```

#### Supported Options
| Option | Type | Purpose | Example |
|--------|------|---------|---------|
| `StrictHostKeyChecking` | yes/no | Host key verification | `no` |
| `UserKnownHostsFile` | path | Known hosts file | `/dev/null` |
| `IdentityFile` | path | Private key file | `~/.ssh/id_rsa` |
| `User` | string | Default username | `git` |
| `Port` | integer | Default port | `22` |

#### Example Configurations
```ssh
# Disable host key checking (current default)
Host *
  StrictHostKeyChecking no

# Host-specific configuration
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github_key
  StrictHostKeyChecking no

Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/gitlab_key
  StrictHostKeyChecking no

# Proxy configuration
Host internal-git
  HostName git.internal.com
  ProxyCommand ssh -W %h:%p proxy.internal.com
  User git
```

## Python API

### Virtual Environment API

#### Creation Functions
```bash
# Using virtualenv (recommended)
virtualenv [options] <env-name>

# Options
-p, --python <python-exe>       # Python interpreter to use
--system-site-packages          # Include system packages
--clear                         # Clear existing environment
--no-pip                        # Don't install pip
--no-setuptools                 # Don't install setuptools
```

#### Management Commands
```bash
# Activation
source <env-name>/bin/activate

# Deactivation
deactivate

# Environment information
which python                    # Show active Python
which pip                       # Show active pip
pip list                        # Show installed packages
pip freeze > requirements.txt   # Export requirements
```

#### Python Version API
```bash
# Available Python executables
/usr/bin/python3                # System default (3.10)
/usr/bin/python3.8              # Python 3.8
/usr/bin/python3.9              # Python 3.9
/usr/bin/python3.10             # Python 3.10
/usr/bin/python3.11             # Python 3.11

# Version checking
python3.11 --version            # Returns: Python 3.11.x
python3.11 -c "import sys; print(sys.version_info)"
```

### Package Management API

#### Pip Configuration
```bash
# Configuration locations
/home/gitlab-runner/.config/pip/pip.conf    # User configuration
/home/gitlab-runner/.cache/pip/             # Cache directory

# Environment variables
PIP_CACHE_DIR=/home/gitlab-runner/.cache/pip
PIP_CONFIG_FILE=/home/gitlab-runner/.config/pip/pip.conf
PIP_INDEX_URL=https://pypi.org/simple/
```

#### Package Installation Patterns
```bash
# Basic installation
pip install package-name

# Version constraints
pip install "package-name>=1.0,<2.0"
pip install package-name==1.2.3

# From requirements file
pip install -r requirements.txt

# Development installation
pip install -e .

# Binary vs source
pip install --only-binary=all package-name    # Force binary
pip install --no-binary :all: package-name    # Force source compilation
```

## Database API

### PostgreSQL Client API

#### Connection Parameters
```bash
# Environment variables
PGHOST=<hostname>               # Database host
PGPORT=<port>                   # Database port (default: 5432)
PGUSER=<username>               # Database user
PGPASSWORD=<password>           # Database password
PGDATABASE=<database>           # Database name
```

#### Command Line Tools
```bash
# Connection testing
pg_isready -h <host> -p <port>

# Database operations
psql -h <host> -U <user> -d <database>
pg_dump -h <host> -U <user> <database> > backup.sql
psql -h <host> -U <user> -d <database> < backup.sql

# Database management
createdb -h <host> -U <user> <database>
dropdb -h <host> -U <user> <database>
```

#### Connection Examples
```bash
# Interactive connection
psql -h postgres -U odoo -d odoo_dev

# Non-interactive query
psql -h postgres -U odoo -d odoo_dev -c "SELECT version();"

# Backup and restore
pg_dump -h postgres -U odoo odoo_dev | gzip > backup.sql.gz
gunzip -c backup.sql.gz | psql -h postgres -U odoo odoo_test
```

## Odoo API Integration

### Odoo Command Line API

#### Basic Commands
```bash
# Odoo binary location (after git clone)
./odoo/odoo-bin [options]

# Common options
--addons-path=<paths>           # Comma-separated addon paths
--config=<file>                 # Configuration file
--database=<name>               # Database name
--init=<modules>                # Modules to initialize
--update=<modules>              # Modules to update
--test-enable                   # Enable testing
--stop-after-init               # Stop after initialization
--log-level=<level>             # Logging level
```

#### Database Management
```bash
# Initialize database with base modules
python odoo/odoo-bin -d <database> -i base --stop-after-init

# Install custom addons
python odoo/odoo-bin -d <database> -i addon1,addon2 --stop-after-init

# Update existing modules
python odoo/odoo-bin -d <database> -u addon1,addon2 --stop-after-init

# Run with testing
python odoo/odoo-bin -d <database> -i addon1 --test-enable --stop-after-init
```

#### Development Mode Options
```bash
# Development mode flags
--dev=reload                    # Auto-reload on file changes
--dev=qweb                      # QWeb template debugging
--dev=werkzeug                  # Werkzeug debugger
--dev=xml                       # XML validation
--dev=reload,qweb,werkzeug,xml  # Multiple options
```

### Manifestoo API

#### Command Reference

##### `manifestoo list`
```bash
# Syntax
manifestoo list [options] <path>

# Options
--separator=<sep>               # Output separator (default: newline)
--format=<format>               # Output format (simple, tree)

# Examples
manifestoo list addons/                     # List all addons
manifestoo list --separator=, addons/      # Comma-separated output
manifestoo list --format=tree addons/      # Tree format
```

##### `manifestoo list-depends`
```bash
# Syntax
manifestoo list-depends [options] <path>

# Options
--separator=<sep>               # Output separator
--format=<format>               # Output format
--transitive                    # Include transitive dependencies

# Examples
manifestoo list-depends addons/my_addon    # Direct dependencies
manifestoo list-depends --transitive addons/my_addon  # All dependencies
```

##### `manifestoo check-depends`
```bash
# Syntax
manifestoo check-depends [options] <path>

# Return codes
0                               # All dependencies satisfied
1                               # Missing dependencies found

# Examples
manifestoo check-depends addons/           # Check all addons
manifestoo check-depends addons/my_addon  # Check specific addon
```

##### `manifestoo create-addon`
```bash
# Syntax
manifestoo create-addon <name>

# Generated structure
<name>/
├── __init__.py
├── __manifest__.py
├── models/
│   └── __init__.py
├── views/
├── security/
└── tests/
    └── __init__.py
```

## File System API

### Directory Structure

#### Standard Paths
```bash
# User directories
/home/gitlab-runner/                    # User home directory
/home/gitlab-runner/.local/bin/         # User binaries
/home/gitlab-runner/.local/share/       # User data
/home/gitlab-runner/.config/            # User configuration
/home/gitlab-runner/.cache/             # User cache

# System directories
/usr/local/bin/                         # System-wide binaries
/opt/pipx/                              # PIPX installation
```

#### Configuration Files
```bash
# Git configuration
/home/gitlab-runner/.gitconfig          # Git user configuration
/home/gitlab-runner/.config/git-autoshare/repos.yml  # Git autoshare config

# SSH configuration
/home/gitlab-runner/.ssh/config         # SSH client configuration
/home/gitlab-runner/.ssh/known_hosts    # Known SSH hosts

# Python configuration
/home/gitlab-runner/.config/pip/pip.conf  # Pip configuration
```

#### Cache Directories
```bash
# Package caches
/home/gitlab-runner/.cache/pip/         # Python package cache
/home/gitlab-runner/.cache/git-autoshare/  # Git repository cache
/home/gitlab-runner/.cache/acsoo-wheel/ # Legacy acsoo cache

# Application data
/home/gitlab-runner/.local/share/Odoo/addons/     # Odoo addons
/home/gitlab-runner/.local/share/Odoo/filestore/  # Odoo file storage
/home/gitlab-runner/.local/share/Odoo/sessions/   # Odoo sessions
```

### File Permissions

#### Permission Scheme
```bash
# User directories
drwxr-xr-x gitlab-runner gitlab-runner /home/gitlab-runner/
drwx------ gitlab-runner gitlab-runner /home/gitlab-runner/.ssh/
drwxr-xr-x gitlab-runner gitlab-runner /home/gitlab-runner/.config/
drwxr-xr-x gitlab-runner gitlab-runner /home/gitlab-runner/.cache/

# Configuration files
-rw-r--r-- gitlab-runner gitlab-runner /home/gitlab-runner/.ssh/config
-rw-r--r-- gitlab-runner gitlab-runner /home/gitlab-runner/.config/git-autoshare/repos.yml

# Executable files
-rwxr-xr-x root root /usr/local/bin/git
```

## Network API

### Outbound Connectivity

#### Required Network Access
| Destination | Port | Protocol | Purpose |
|-------------|------|----------|---------|
| `github.com` | 443 | HTTPS | Git repository access |
| `pypi.org` | 443 | HTTPS | Python package downloads |
| `archive.ubuntu.com` | 80/443 | HTTP/HTTPS | System package updates |
| `ppa.launchpad.net` | 443 | HTTPS | PPA package access |

#### Optional Network Access
| Destination | Port | Protocol | Purpose |
|-------------|------|----------|---------|
| `gitlab.com` | 443 | HTTPS | GitLab repositories |
| `bitbucket.org` | 443 | HTTPS | Bitbucket repositories |
| Custom Git servers | 22/443 | SSH/HTTPS | Private repositories |

### Inbound Connectivity

#### Service Ports
```bash
# Odoo services
8069/tcp                        # Main web interface
8072/tcp                        # Longpolling service

# Database connections (if database in same container)
5432/tcp                        # PostgreSQL
```

## Integration APIs

### CI/CD Platform Integration

#### GitLab CI Variables
```yaml
# Standard GitLab CI variables available in container
CI=true                         # Indicates CI environment
GITLAB_CI=true                  # Indicates GitLab CI
CI_PROJECT_DIR=/builds/<group>/<project>  # Project directory
CI_COMMIT_SHA=<sha>             # Commit SHA
CI_COMMIT_REF_NAME=<branch>     # Branch name
```

#### GitHub Actions Variables
```yaml
# Standard GitHub Actions variables
GITHUB_ACTIONS=true             # Indicates GitHub Actions
GITHUB_WORKSPACE=/github/workspace  # Workspace directory
GITHUB_SHA=<sha>                # Commit SHA
GITHUB_REF=refs/heads/<branch>  # Git reference
```

### Container Orchestration API

#### Docker Compose Schema
```yaml
# Service definition schema
services:
  <service-name>:
    image: ghcr.io/opsivist/sonod:22.04
    container_name: <name>
    volumes:
      - <host-path>:<container-path>
    environment:
      - <VAR>=<value>
    ports:
      - "<host-port>:<container-port>"
    depends_on:
      - <dependency-service>
    command: <command>
    working_dir: <directory>
```

#### Kubernetes Resource Schema
```yaml
# Deployment specification
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: odoo-ci
        image: ghcr.io/opsivist/sonod:22.04
        env:
        - name: <ENV_VAR>
          value: <value>
        volumeMounts:
        - name: <volume-name>
          mountPath: <mount-path>
        resources:
          requests:
            memory: <memory>
            cpu: <cpu>
          limits:
            memory: <memory>
            cpu: <cpu>
```

## Security API

### User and Permission Model

#### User Information
```bash
# User details
Username: gitlab-runner
UID: 1000
GID: 1000
Home: /home/gitlab-runner
Shell: /bin/bash
Groups: gitlab-runner
```

#### Sudo Access
```bash
# No sudo access by default
# Switch to root if needed (not recommended)
docker run --user root ghcr.io/opsivist/sonod:22.04
```

### File System Security

#### Permission Model
```bash
# Owned by gitlab-runner
/home/gitlab-runner/            # User home directory
/home/gitlab-runner/.local/     # User local directory
/home/gitlab-runner/.cache/     # User cache directory
/home/gitlab-runner/.config/    # User configuration

# System-wide (owned by root)
/usr/local/bin/git              # Git wrapper script
/opt/pipx/                      # PIPX installation
```

#### Security Configurations
```bash
# SSH security (disabled for CI)
StrictHostKeyChecking no        # Host key verification disabled

# Git security
safe.directory *                # All directories marked as safe
```

## Error Handling API

### Common Error Codes

#### Git Operations
| Error Code | Meaning | Common Causes |
|------------|---------|---------------|
| 0 | Success | Operation completed successfully |
| 1 | General error | Invalid arguments, network issues |
| 128 | Repository error | Invalid repository, permission issues |
| 129 | Authentication error | Invalid credentials, SSH key issues |

#### Python Operations
| Error Code | Meaning | Common Causes |
|------------|---------|---------------|
| 0 | Success | Operation completed successfully |
| 1 | General error | Import errors, syntax errors |
| 2 | Misuse of shell | Invalid command line arguments |
| 126 | Permission denied | File permission issues |
| 127 | Command not found | Missing executable or package |

#### Database Operations
| Error Code | Meaning | Common Causes |
|------------|---------|---------------|
| 0 | Success | Operation completed successfully |
| 1 | Connection error | Database unreachable, invalid credentials |
| 2 | SQL error | Invalid query, constraint violations |
| 3 | Database error | Database corruption, disk space issues |

### Error Recovery Patterns

#### Network Connectivity Issues
```bash
#!/bin/bash
# network-recovery.sh

retry_count=0
max_retries=3

while [ $retry_count -lt $max_retries ]; do
    if git clone https://github.com/odoo/odoo.git --depth 1; then
        echo "✅ Clone successful"
        break
    else
        retry_count=$((retry_count + 1))
        echo "❌ Clone failed, attempt $retry_count/$max_retries"
        sleep $((retry_count * 5))  # Exponential backoff
    fi
done

if [ $retry_count -eq $max_retries ]; then
    echo "❌ Clone failed after $max_retries attempts"
    exit 1
fi
```

#### Database Recovery
```bash
#!/bin/bash
# db-recovery.sh

# Test database connectivity
if ! pg_isready -h $PGHOST -p ${PGPORT:-5432}; then
    echo "❌ Database not ready, waiting..."
    
    # Wait for database with timeout
    timeout=60
    while [ $timeout -gt 0 ]; do
        if pg_isready -h $PGHOST -p ${PGPORT:-5432}; then
            echo "✅ Database ready"
            break
        fi
        sleep 5
        timeout=$((timeout - 5))
    done
    
    if [ $timeout -le 0 ]; then
        echo "❌ Database timeout"
        exit 1
    fi
fi

# Test database connection
if ! psql -h $PGHOST -U $PGUSER -d $PGDATABASE -c "SELECT 1;" >/dev/null 2>&1; then
    echo "❌ Database connection failed"
    echo "Troubleshooting:"
    echo "1. Check credentials: PGUSER=$PGUSER"
    echo "2. Check database exists: psql -h $PGHOST -U $PGUSER -l"
    echo "3. Check network: nc -zv $PGHOST ${PGPORT:-5432}"
    exit 1
fi

echo "✅ Database connection successful"
```

## Performance API

### Resource Monitoring

#### Memory Usage API
```bash
# Container memory limits
docker run --memory=2g ghcr.io/opsivist/sonod:22.04

# Memory monitoring
free -h                         # System memory
ps aux --sort=-%mem | head     # Top memory consumers
```

#### CPU Usage API
```bash
# Container CPU limits
docker run --cpus=2.0 ghcr.io/opsivist/sonod:22.04

# CPU monitoring
top -bn1                        # CPU usage snapshot
htop                           # Interactive CPU monitor (if installed)
```

#### Disk Usage API
```bash
# Container disk limits
docker run --storage-opt size=10g ghcr.io/opsivist/sonod:22.04

# Disk monitoring
df -h                          # Disk space usage
du -sh /home/gitlab-runner/.cache/*  # Cache directory sizes
```

### Performance Optimization API

#### Cache Management
```bash
# Pip cache operations
pip cache info                  # Cache information
pip cache list                  # List cached packages
pip cache purge                 # Clear all cache

# Git autoshare cache operations
git autoshare status           # Cache status
git autoshare clean            # Clean cache
git autoshare gc               # Garbage collection
```

#### Build Optimization
```bash
# Parallel pip installations
pip install --upgrade pip setuptools wheel
pip install -r requirements.txt --find-links /home/gitlab-runner/.cache/pip

# Parallel git operations
git clone --jobs 4 https://github.com/odoo/odoo.git

# Optimized Python compilation
export MAKEFLAGS="-j$(nproc)"
pip install --no-binary :all: package-with-c-extensions
```

## Logging and Monitoring API

### Logging Configuration

#### Log Levels
| Level | Numeric | Purpose |
|-------|---------|---------|
| `CRITICAL` | 50 | Critical errors only |
| `ERROR` | 40 | Error messages |
| `WARNING` | 30 | Warning messages |
| `INFO` | 20 | Informational messages |
| `DEBUG` | 10 | Debug information |

#### Odoo Logging API
```bash
# Configure Odoo logging
python odoo/odoo-bin \
  --log-level=info \
  --log-handler=werkzeug:WARNING \
  --log-handler=odoo.sql_db:WARNING \
  --logfile=/workspace/logs/odoo.log

# Custom log handlers
--log-handler=<logger>:<level>
```

#### System Logging
```bash
# System log access
journalctl -f                   # Follow system logs
dmesg                          # Kernel messages
tail -f /var/log/syslog        # System log file
```

### Monitoring Endpoints

#### Health Check API
```bash
# Container health check
docker run --health-cmd="python3 -c 'import requests; requests.get(\"http://localhost:8069/web/health\")'" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  ghcr.io/opsivist/sonod:22.04
```

#### Metrics Collection
```bash
# System metrics
cat /proc/meminfo               # Memory information
cat /proc/cpuinfo               # CPU information
cat /proc/loadavg               # Load average

# Process metrics
ps aux                          # Process list
pstree                         # Process tree
```

## Extension API

### Custom Tool Installation

#### PIPX Extension API
```bash
# Install additional tools
pipx install <tool-name>
pipx install --pip-args="--no-cache-dir" <tool-name>

# Tool management
pipx list                       # List installed tools
pipx upgrade <tool-name>        # Upgrade specific tool
pipx upgrade-all               # Upgrade all tools
pipx uninstall <tool-name>     # Remove tool
```

#### System Package Extension
```bash
# Switch to root for system packages
USER root
RUN apt-get update && apt-get install -y <packages>
USER gitlab-runner
```

### Configuration Extension

#### Custom Git Autoshare Configuration
```yaml
# Extended configuration example
your-gitlab.com:
  enterprise:
    orgs:
      - your-company
    private: true
    token_env: GITLAB_TOKEN
    cache_ttl: 3600
  "*":
    orgs:
      - public-org
    cache_ttl: 7200

bitbucket.org:
  "*":
    orgs:
      - your-bitbucket-org
    auth_method: ssh
```

#### Custom SSH Configuration
```ssh
# Extended SSH configuration
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github_ed25519
  IdentitiesOnly yes
  StrictHostKeyChecking no

Host *.internal.com
  User deployer
  IdentityFile ~/.ssh/internal_key
  ProxyCommand ssh -W %h:%p bastion.internal.com
  StrictHostKeyChecking yes
```

## Compatibility API

### Version Compatibility Matrix

#### Python-Odoo Compatibility
```python
# Compatibility check function
def check_compatibility(python_version, odoo_version):
    compatibility_matrix = {
        "16.0": ["3.8", "3.9", "3.10", "3.11"],
        "15.0": ["3.8", "3.9", "3.10"],
        "14.0": ["3.8", "3.9"],
        "13.0": ["3.8"]
    }
    
    supported_versions = compatibility_matrix.get(odoo_version, [])
    return python_version in supported_versions

# Usage
print(check_compatibility("3.11", "16.0"))  # True
print(check_compatibility("3.11", "14.0"))  # False
```

#### Package Version Constraints
```bash
# Critical package versions for Odoo 16
lxml>=4.6.5
psycopg2>=2.8.6
Pillow>=8.0.0
reportlab>=3.5.0
werkzeug>=2.0.0,<3.0
```

### Migration API

#### Version Migration Helpers
```bash
#!/bin/bash
# migrate-python-version.sh

OLD_VERSION=${1:-"3.10"}
NEW_VERSION=${2:-"3.11"}

echo "Migrating from Python $OLD_VERSION to $NEW_VERSION"

# Create new environment
virtualenv -p python$NEW_VERSION new-venv

# Export old requirements
if [ -d "venv" ]; then
    source venv/bin/activate
    pip freeze > migration-requirements.txt
    deactivate
fi

# Install in new environment
source new-venv/bin/activate
pip install -r migration-requirements.txt

# Test compatibility
python -c "
import sys
print(f'New Python version: {sys.version}')

# Test critical imports
critical_packages = ['lxml', 'psycopg2', 'werkzeug', 'jinja2']
for pkg in critical_packages:
    try:
        __import__(pkg)
        print(f'✅ {pkg}: Compatible')
    except ImportError as e:
        print(f'❌ {pkg}: {e}')
"

echo "Migration complete. Test thoroughly before switching."
```

## Troubleshooting API

### Diagnostic Commands

#### System Diagnostics
```bash
# System information
uname -a                        # System information
cat /etc/os-release            # OS version
df -h                          # Disk usage
free -h                        # Memory usage
ps aux                         # Running processes
```

#### Python Diagnostics
```bash
# Python environment diagnostics
python3 --version              # Python version
which python3                  # Python location
pip list                       # Installed packages
pip check                      # Check for conflicts
python -m site                 # Site packages info
```

#### Git Diagnostics
```bash
# Git diagnostics
git --version                  # Git version
git config --list             # Git configuration
git autoshare status          # Autoshare status
ssh -T git@github.com         # Test GitHub SSH
```

### Recovery Procedures

#### Environment Recovery
```bash
#!/bin/bash
# recover-environment.sh

echo "Starting environment recovery..."

# Clean corrupted virtual environment
if [ -d "venv" ]; then
    echo "Removing corrupted virtual environment..."
    rm -rf venv
fi

# Recreate virtual environment
echo "Creating new virtual environment..."
virtualenv -p python3.11 venv
source venv/bin/activate

# Reinstall packages
if [ -f "requirements.txt" ]; then
    echo "Reinstalling packages..."
    pip install --upgrade pip
    pip install -r requirements.txt
else
    echo "No requirements.txt found, installing Odoo requirements..."
    if [ -d "odoo" ]; then
        pip install -r odoo/requirements.txt
    fi
fi

# Verify installation
python -c "
import sys
print(f'Python version: {sys.version}')
print('Testing critical imports...')
for pkg in ['lxml', 'psycopg2', 'werkzeug']:
    try:
        __import__(pkg)
        print(f'✅ {pkg}: OK')
    except ImportError:
        print(f'❌ {pkg}: Failed')
"

echo "Environment recovery complete"
```

This comprehensive API reference provides complete documentation for all public interfaces, configurations, and integration points of the Sonod Docker image. Use this reference in conjunction with the other documentation files for complete understanding and implementation guidance.