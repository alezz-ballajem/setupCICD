# Sonod Docker Image API Documentation

## Overview

The Sonod Docker image is a specialized Ubuntu 22.04-based container designed for running Odoo CI workloads. It provides a complete environment with all necessary dependencies, tools, and configurations for Odoo development and continuous integration.

## Docker Image API

### Base Image
- **Base**: `ubuntu:22.04`
- **Architecture**: `amd64`
- **User**: `gitlab-runner` (non-privileged)

### Environment Variables

#### Core Environment
```bash
LANG=C.UTF-8                    # System locale
LC_ALL=C.UTF-8                  # System locale (all categories)
DEBIAN_FRONTEND=noninteractive  # Prevents interactive prompts during package installation
```

#### PIPX Configuration
```bash
PIPX_BIN_DIR=/home/gitlab-runner/.local/bin     # PIPX binary installation directory
PIPX_HOME=/home/gitlab-runner/.local/pipx       # PIPX home directory
PATH=/home/gitlab-runner/.local/bin:$PATH       # Extended PATH for user binaries
```

### Installed Dependencies

#### System Packages
| Package | Version | Purpose |
|---------|---------|---------|
| `git` | Latest | Version control system |
| `mercurial` | Latest | Alternative version control |
| `openssh-client` | Latest | SSH client for secure connections |
| `rsync` | Latest | File synchronization utility |
| `make` | Latest | Build automation tool |
| `postgresql-client` | Latest | PostgreSQL database client |
| `wkhtmltopdf` | 0.12.6.1-2 | HTML to PDF converter |
| `tcl` + `expect` | Latest | Provides `unbuffer` utility |
| `gettext` | Latest | Internationalization tools for .po/.pot files |

#### Python Versions
- Python 3.8 (with dev packages and venv)
- Python 3.9 (with dev packages and venv)
- Python 3.10 (with dev packages and venv)
- Python 3.11 (with dev packages and venv)
- Default Python 3 (system default)

#### Odoo-Specific Dependencies
| Package | Purpose |
|---------|---------|
| `graphviz` | Graph visualization for Odoo reports |
| `node-clean-css` | CSS minification |
| `node-less` | LESS CSS preprocessor |
| `poppler-utils` | PDF manipulation utilities |
| `antiword` | Microsoft Word document processing |
| `libreoffice-writer` | Document processing for py3o |
| `libreoffice-calc` | Spreadsheet processing for py3o |

#### Python Tools (via PIPX)
| Tool | Version | Purpose |
|------|---------|---------|
| `virtualenv` | Latest | Python virtual environment creation |
| `git-autoshare` | >=1.0.0b4 | Git repository sharing and caching |
| `manifestoo` | >=0.4.0 | Odoo manifest management tool |

### Volumes and Directories

#### Recommended Mounts
For optimal performance, mount these volumes:

```bash
# Pip cache for faster package installations
/home/gitlab-runner/.cache/pip

# Git autoshare cache for repository sharing
/home/gitlab-runner/.cache/git-autoshare

# Legacy acsoo wheel cache (if using deprecated acsoo wheel command)
/home/gitlab-runner/.cache/acsoo-wheel
```

#### Pre-created Directories
```bash
/home/gitlab-runner/.local/share/Odoo/addons    # Odoo addons directory
/home/gitlab-runner/.local/share/Odoo/filestore # Odoo file storage
/home/gitlab-runner/.local/share/Odoo/sessions  # Odoo session storage
/home/gitlab-runner/.cache                      # General cache directory
/home/gitlab-runner/.config                     # Configuration directory
/home/gitlab-runner/.ssh                        # SSH configuration (mode 700)
```

### Exposed Functionality

#### Git Integration
- **Git Wrapper**: Custom git wrapper that automatically uses `git-autoshare` for clone operations
- **Pre-configured Git User**: 
  - Name: `GitLab`
  - Email: `gitlab@opsivist.io`
- **Safe Directory**: All directories marked as safe (`*`)
- **Autoshare Configuration**: Configured for GitHub.com repositories

#### SSH Configuration
- **Host Key Checking**: Disabled (`StrictHostKeyChecking no`)
- **Target Use Case**: CI environments where host key verification is not required

## Usage Examples

### Basic Container Usage

#### Running Interactive Shell
```bash
docker run -it --rm ghcr.io/opsivist/sonod:latest /bin/bash
```

#### Running with Recommended Volumes
```bash
docker run -it --rm \
  -v pip-cache:/home/gitlab-runner/.cache/pip \
  -v git-autoshare-cache:/home/gitlab-runner/.cache/git-autoshare \
  -v acsoo-cache:/home/gitlab-runner/.cache/acsoo-wheel \
  ghcr.io/opsivist/sonod:latest /bin/bash
```

### GitLab CI Integration

#### Basic .gitlab-ci.yml Configuration
```yaml
image: ghcr.io/opsivist/sonod:latest

variables:
  PIP_CACHE_DIR: /home/gitlab-runner/.cache/pip

cache:
  paths:
    - /home/gitlab-runner/.cache/pip/
    - /home/gitlab-runner/.cache/git-autoshare/

before_script:
  - python3 --version
  - git --version

test:
  script:
    - virtualenv venv
    - source venv/bin/activate
    - pip install -r requirements.txt
    - python -m pytest
```

#### Advanced CI Configuration with Multiple Python Versions
```yaml
.test_template: &test_template
  image: ghcr.io/opsivist/sonod:latest
  cache:
    paths:
      - /home/gitlab-runner/.cache/pip/
  script:
    - $PYTHON_VERSION --version
    - virtualenv -p $PYTHON_VERSION venv
    - source venv/bin/activate
    - pip install -r requirements.txt
    - python -m pytest

test_python38:
  <<: *test_template
  variables:
    PYTHON_VERSION: python3.8

test_python39:
  <<: *test_template
  variables:
    PYTHON_VERSION: python3.9

test_python310:
  <<: *test_template
  variables:
    PYTHON_VERSION: python3.10

test_python311:
  <<: *test_template
  variables:
    PYTHON_VERSION: python3.11
```

### Odoo Development Workflow

#### Setting up Odoo Development Environment
```bash
# Clone Odoo repository (uses git-autoshare automatically)
git clone https://github.com/odoo/odoo.git

# Create virtual environment
virtualenv -p python3.11 odoo-venv
source odoo-venv/bin/activate

# Install Odoo dependencies
pip install -r odoo/requirements.txt

# Set up PostgreSQL connection (assuming external DB)
export PGHOST=your-postgres-host
export PGUSER=odoo
export PGPASSWORD=your-password

# Run Odoo
python odoo/odoo-bin --addons-path=odoo/addons
```

#### Using with Manifestoo
```bash
# Analyze Odoo addon manifests
manifestoo --help

# List addon dependencies
manifestoo list-depends --separator=, path/to/addons

# Check for missing dependencies
manifestoo check-depends path/to/addons
```

### Docker Compose Integration

#### Basic docker-compose.yml
```yaml
version: '3.8'

services:
  odoo-ci:
    image: ghcr.io/opsivist/sonod:latest
    volumes:
      - .:/workspace
      - pip-cache:/home/gitlab-runner/.cache/pip
      - git-autoshare-cache:/home/gitlab-runner/.cache/git-autoshare
    working_dir: /workspace
    command: /bin/bash
    environment:
      - PGHOST=postgres
      - PGUSER=odoo
      - PGPASSWORD=odoo
    depends_on:
      - postgres

  postgres:
    image: postgres:13
    environment:
      POSTGRES_DB: odoo
      POSTGRES_USER: odoo
      POSTGRES_PASSWORD: odoo
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  pip-cache:
  git-autoshare-cache:
  postgres-data:
```

## API Reference

### Container Lifecycle

#### Image Build
The image is built using the provided Dockerfile with the following stages:
1. System package installation and updates
2. Python versions installation
3. PIPX setup and tool installation
4. User creation and configuration
5. Git and SSH configuration
6. Directory structure creation

#### Container Startup
- Default user: `gitlab-runner`
- Working directory: `/home/gitlab-runner`
- Shell: `/bin/bash`

### Tool APIs

#### Git Wrapper (`/usr/local/bin/git`)
**Purpose**: Intercepts git clone commands to use git-autoshare for repository caching.

**Usage**:
```bash
git clone <repository-url>  # Automatically uses git-autoshare
git <other-commands>        # Passes through to standard git
```

**Implementation**:
- Detects `clone` command and redirects to `git autoshare-clone`
- All other git commands pass through unchanged
- Transparent to users - works as drop-in git replacement

#### Git Autoshare Configuration
**Configuration File**: `/home/gitlab-runner/.config/git-autoshare/repos.yml`

**Supported Repositories**:
- GitHub.com repositories from organizations:
  - `opsivist` (private and public)
  - `odoo` (private and public)
  - `OCA` (public only)

**Benefits**:
- Reduced clone times through repository caching
- Bandwidth optimization for CI/CD pipelines
- Automatic handling of enterprise vs public repositories

### Security Configuration

#### SSH Settings
**Configuration File**: `/home/gitlab-runner/.ssh/config`

**Settings**:
- `StrictHostKeyChecking no`: Disables host key verification
- **Use Case**: CI environments where host keys cannot be pre-verified
- **Security Note**: Only suitable for trusted network environments

#### Git Configuration
- **Safe Directory**: All directories marked as safe (`git config --global --add safe.directory '*'`)
- **Reason**: Handles GitLab CI checkout as root, execution as gitlab-runner user
- **User Identity**: Pre-configured for CI commits

## Best Practices

### Performance Optimization
1. **Use Volume Mounts**: Always mount cache directories for better performance
2. **Layer Caching**: Leverage Docker layer caching in CI/CD pipelines
3. **Git Autoshare**: Automatically enabled for supported repositories

### Security Considerations
1. **Non-Privileged User**: Container runs as `gitlab-runner` user
2. **SSH Host Checking**: Disabled for CI use - ensure trusted network environment
3. **Secrets Management**: Use proper secret management for sensitive data

### CI/CD Integration
1. **GitLab CI**: Optimized for GitLab CI/CD pipelines
2. **GitHub Actions**: Compatible with GitHub Actions workflows
3. **Multi-Python**: Supports testing across multiple Python versions
4. **Caching**: Implement proper caching strategies for dependencies

### Troubleshooting

#### Common Issues
1. **Permission Errors**: Ensure volumes are properly mounted with correct permissions
2. **Git Clone Failures**: Check git-autoshare configuration and network connectivity
3. **Python Version Issues**: Specify exact Python version in virtualenv creation
4. **PostgreSQL Connection**: Verify database connection parameters

#### Debug Commands
```bash
# Check installed Python versions
ls /usr/bin/python*

# Verify git-autoshare status
git autoshare status

# Check PIPX installations
pipx list

# Verify PostgreSQL client
psql --version
```

## Version Compatibility

### Supported Odoo Versions
- **Primary**: Odoo 16
- **Note**: Odoo is not pre-installed, install as needed for your specific version

### Python Compatibility
- Python 3.8, 3.9, 3.10, 3.11
- Virtual environment support for all versions
- Development headers included for compiled packages

### Database Support
- PostgreSQL client pre-installed
- Compatible with PostgreSQL 10+ (recommended 13+)

## Contributing

### Building the Image Locally
```bash
docker build -t sonod:local .
```

### Testing Changes
```bash
# Test basic functionality
docker run --rm sonod:local python3 --version
docker run --rm sonod:local git --version
docker run --rm sonod:local pipx list

# Test Odoo-specific tools
docker run --rm sonod:local manifestoo --help
docker run --rm sonod:local wkhtmltopdf --version
```

### Submitting Changes
1. Create feature branch
2. Update documentation if needed
3. Test with sample Odoo project
4. Submit pull request with clear description

## License

This project is licensed under the terms specified in the LICENSE file.