# Comprehensive Usage Examples

## Table of Contents
1. [Quick Start](#quick-start)
2. [Development Workflows](#development-workflows)
3. [CI/CD Integration](#cicd-integration)
4. [Advanced Use Cases](#advanced-use-cases)
5. [Troubleshooting Examples](#troubleshooting-examples)

## Quick Start

### Basic Container Usage

#### Running Interactive Shell
```bash
# Start interactive session
docker run -it --rm ghcr.io/opsivist/sonod:22.04 /bin/bash

# With volume mounts for persistence
docker run -it --rm \
  -v $(pwd):/workspace \
  -v pip-cache:/home/gitlab-runner/.cache/pip \
  -w /workspace \
  ghcr.io/opsivist/sonod:22.04 /bin/bash
```

#### One-off Commands
```bash
# Check Python version
docker run --rm ghcr.io/opsivist/sonod:22.04 python3 --version

# List installed tools
docker run --rm ghcr.io/opsivist/sonod:22.04 pipx list

# Test git functionality
docker run --rm ghcr.io/opsivist/sonod:22.04 git --version
```

### Simple Odoo Setup

#### Basic Odoo Installation
```bash
# Start container with database connection
docker run -it --rm \
  -e PGHOST=host.docker.internal \
  -e PGPORT=5432 \
  -e PGUSER=odoo \
  -e PGPASSWORD=odoo \
  -v $(pwd):/workspace \
  -w /workspace \
  ghcr.io/opsivist/sonod:22.04 /bin/bash

# Inside container: Setup Odoo
virtualenv -p python3.11 odoo-env
source odoo-env/bin/activate
git clone https://github.com/odoo/odoo.git --branch 16.0 --depth 1
pip install -r odoo/requirements.txt

# Run Odoo
cd odoo
python odoo-bin --init=base --stop-after-init
```

## Development Workflows

### Odoo Addon Development

#### Complete Development Environment
```bash
#!/bin/bash
# setup-odoo-dev.sh

# Configuration
ODOO_VERSION="16.0"
PYTHON_VERSION="3.11"
PROJECT_NAME="my-odoo-project"

# Create project structure
mkdir -p $PROJECT_NAME/{addons,config,data}
cd $PROJECT_NAME

# Start development container
docker run -it --rm \
  --name odoo-dev \
  -v $(pwd):/workspace \
  -v pip-cache:/home/gitlab-runner/.cache/pip \
  -v git-cache:/home/gitlab-runner/.cache/git-autoshare \
  -e PGHOST=postgres \
  -e PGUSER=odoo \
  -e PGPASSWORD=odoo \
  -e PGDATABASE=odoo_dev \
  -p 8069:8069 \
  -w /workspace \
  ghcr.io/opsivist/sonod:22.04 /bin/bash

# Inside container setup
cat << 'EOF' > setup-env.sh
#!/bin/bash
set -e

echo "Setting up Odoo development environment..."

# Create virtual environment
virtualenv -p python$PYTHON_VERSION venv
source venv/bin/activate

# Clone Odoo
if [ ! -d "odoo" ]; then
    git clone https://github.com/odoo/odoo.git --branch $ODOO_VERSION --depth 1
fi

# Install Odoo dependencies
pip install -r odoo/requirements.txt

# Install development tools
pip install \
    pytest \
    pytest-odoo \
    black \
    flake8 \
    isort \
    pre-commit

# Setup pre-commit hooks
pre-commit install

echo "Environment setup complete!"
echo "To start Odoo: python odoo/odoo-bin --addons-path=addons,odoo/addons"
EOF

chmod +x setup-env.sh
./setup-env.sh
```

#### Addon Development Workflow
```bash
# Create new addon structure
manifestoo create-addon my_custom_addon

# Addon structure
my_custom_addon/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── my_model.py
├── views/
│   └── my_model_views.xml
├── security/
│   └── ir.model.access.csv
└── tests/
    ├── __init__.py
    └── test_my_model.py

# Development cycle
source venv/bin/activate

# Code formatting
black my_custom_addon/
isort my_custom_addon/

# Linting
flake8 my_custom_addon/

# Testing
python odoo/odoo-bin -i my_custom_addon --test-enable --stop-after-init

# Run specific tests
python odoo/odoo-bin -i my_custom_addon --test-tags=my_custom_addon
```

### Multi-Addon Project Management

#### Project Structure
```
my-odoo-project/
├── addons/
│   ├── addon1/
│   ├── addon2/
│   └── addon3/
├── requirements.txt
├── odoo.conf
├── docker-compose.yml
└── .gitlab-ci.yml
```

#### Docker Compose Development Setup
```yaml
# docker-compose.yml
version: '3.8'

services:
  odoo:
    image: ghcr.io/opsivist/sonod:22.04
    container_name: odoo-dev
    volumes:
      - .:/workspace
      - pip-cache:/home/gitlab-runner/.cache/pip
      - git-cache:/home/gitlab-runner/.cache/git-autoshare
      - odoo-data:/home/gitlab-runner/.local/share/Odoo
    working_dir: /workspace
    environment:
      - PGHOST=db
      - PGUSER=odoo
      - PGPASSWORD=odoo
      - PGDATABASE=odoo
    ports:
      - "8069:8069"
    depends_on:
      - db
    command: /bin/bash -c "
      virtualenv -p python3.11 venv &&
      source venv/bin/activate &&
      git clone https://github.com/odoo/odoo.git --branch 16.0 --depth 1 &&
      pip install -r odoo/requirements.txt &&
      python odoo/odoo-bin --addons-path=addons,odoo/addons --config=odoo.conf
      "

  db:
    image: postgres:13
    container_name: odoo-db
    environment:
      - POSTGRES_DB=odoo
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=odoo
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  pip-cache:
  git-cache:
  odoo-data:
  postgres-data:
```

#### Usage Commands
```bash
# Start development environment
docker-compose up -d

# Access Odoo container
docker-compose exec odoo /bin/bash

# View logs
docker-compose logs -f odoo

# Stop environment
docker-compose down

# Reset database
docker-compose down -v
docker-compose up -d
```

## CI/CD Integration

### GitLab CI Examples

#### Basic Odoo Testing Pipeline
```yaml
# .gitlab-ci.yml
image: ghcr.io/opsivist/sonod:22.04

variables:
  PIP_CACHE_DIR: /home/gitlab-runner/.cache/pip
  POSTGRES_DB: test_odoo
  POSTGRES_USER: odoo
  POSTGRES_PASSWORD: odoo

services:
  - postgres:13

cache:
  paths:
    - /home/gitlab-runner/.cache/pip/
    - /home/gitlab-runner/.cache/git-autoshare/

stages:
  - test
  - lint
  - build
  - deploy

before_script:
  - virtualenv -p python3.11 venv
  - source venv/bin/activate
  - git clone https://github.com/odoo/odoo.git --branch 16.0 --depth 1
  - pip install -r odoo/requirements.txt

test_addons:
  stage: test
  script:
    - source venv/bin/activate
    # Initialize database
    - python odoo/odoo-bin -i base --stop-after-init --log-level=warn
    # Test custom addons
    - python odoo/odoo-bin -i $(manifestoo list --separator=, addons/) --test-enable --stop-after-init
  coverage: '/TOTAL.*\s+(\d+%)$/'

lint_code:
  stage: lint
  script:
    - source venv/bin/activate
    - pip install flake8 black isort
    - black --check addons/
    - isort --check-only addons/
    - flake8 addons/ --max-line-length=88

build_wheel:
  stage: build
  script:
    - source venv/bin/activate
    - pip install setuptools wheel
    - python setup.py bdist_wheel
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

deploy_staging:
  stage: deploy
  script:
    - echo "Deploying to staging..."
    # Add deployment commands here
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main

deploy_production:
  stage: deploy
  script:
    - echo "Deploying to production..."
    # Add deployment commands here
  environment:
    name: production
    url: https://production.example.com
  only:
    - tags
  when: manual
```

#### Advanced Multi-Version Testing
```yaml
# .gitlab-ci.yml
.test_template: &test_template
  image: ghcr.io/opsivist/sonod:22.04
  services:
    - postgres:13
  variables:
    POSTGRES_DB: test_odoo
    POSTGRES_USER: odoo
    POSTGRES_PASSWORD: odoo
  cache:
    paths:
      - /home/gitlab-runner/.cache/pip/
  before_script:
    - virtualenv -p $PYTHON_VERSION venv
    - source venv/bin/activate
    - git clone https://github.com/odoo/odoo.git --branch $ODOO_VERSION --depth 1
    - pip install -r odoo/requirements.txt
  script:
    - source venv/bin/activate
    - python odoo/odoo-bin -i base --stop-after-init
    - python odoo/odoo-bin -i $(manifestoo list --separator=, addons/) --test-enable --stop-after-init

# Test matrix
test_py38_odoo16:
  <<: *test_template
  variables:
    PYTHON_VERSION: python3.8
    ODOO_VERSION: "16.0"

test_py39_odoo16:
  <<: *test_template
  variables:
    PYTHON_VERSION: python3.9
    ODOO_VERSION: "16.0"

test_py310_odoo16:
  <<: *test_template
  variables:
    PYTHON_VERSION: python3.10
    ODOO_VERSION: "16.0"

test_py311_odoo16:
  <<: *test_template
  variables:
    PYTHON_VERSION: python3.11
    ODOO_VERSION: "16.0"
```

### GitHub Actions Examples

#### Comprehensive Testing Workflow
```yaml
# .github/workflows/test.yml
name: Test Odoo Addons

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/opsivist/sonod:22.04
      volumes:
        - pip-cache:/home/gitlab-runner/.cache/pip
        - git-cache:/home/gitlab-runner/.cache/git-autoshare
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: odoo
          POSTGRES_USER: odoo
          POSTGRES_DB: test_odoo
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    strategy:
      matrix:
        python-version: [3.9, 3.10, 3.11]
        odoo-version: [16.0]

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Python ${{ matrix.python-version }}
        run: |
          virtualenv -p python${{ matrix.python-version }} venv
          source venv/bin/activate
          python --version

      - name: Cache Odoo
        uses: actions/cache@v3
        with:
          path: odoo
          key: odoo-${{ matrix.odoo-version }}-${{ hashFiles('**/requirements.txt') }}

      - name: Install Odoo
        run: |
          source venv/bin/activate
          if [ ! -d "odoo" ]; then
            git clone https://github.com/odoo/odoo.git --branch ${{ matrix.odoo-version }} --depth 1
          fi
          pip install -r odoo/requirements.txt

      - name: Install addon dependencies
        run: |
          source venv/bin/activate
          if [ -f "requirements.txt" ]; then
            pip install -r requirements.txt
          fi

      - name: Run addon tests
        env:
          PGHOST: postgres
          PGUSER: odoo
          PGPASSWORD: odoo
          PGDATABASE: test_odoo
        run: |
          source venv/bin/activate
          # Initialize database
          python odoo/odoo-bin -i base --stop-after-init --log-level=warn
          
          # Test addons
          ADDONS=$(manifestoo list --separator=, .)
          python odoo/odoo-bin -i $ADDONS --test-enable --stop-after-init --log-level=test

      - name: Generate coverage report
        run: |
          source venv/bin/activate
          pip install coverage
          coverage run --source=. odoo/odoo-bin -i $(manifestoo list --separator=, .) --test-enable --stop-after-init
          coverage xml

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
```

## Development Workflows

### Local Development Setup

#### Complete Development Environment
```bash
#!/bin/bash
# dev-setup.sh - Complete development environment setup

set -e

PROJECT_NAME=${1:-"my-odoo-project"}
ODOO_VERSION=${2:-"16.0"}
PYTHON_VERSION=${3:-"3.11"}

echo "Setting up Odoo development environment: $PROJECT_NAME"

# Create project directory
mkdir -p $PROJECT_NAME
cd $PROJECT_NAME

# Create docker-compose for development
cat << EOF > docker-compose.dev.yml
version: '3.8'

services:
  odoo-dev:
    image: ghcr.io/opsivist/sonod:22.04
    container_name: ${PROJECT_NAME}-odoo
    volumes:
      - .:/workspace
      - pip-cache:/home/gitlab-runner/.cache/pip
      - git-cache:/home/gitlab-runner/.cache/git-autoshare
      - odoo-data:/home/gitlab-runner/.local/share/Odoo
    working_dir: /workspace
    environment:
      - PGHOST=db
      - PGUSER=odoo
      - PGPASSWORD=odoo
      - PGDATABASE=odoo_dev
    ports:
      - "8069:8069"
      - "8072:8072"  # longpolling
    depends_on:
      - db
    tty: true
    stdin_open: true

  db:
    image: postgres:13
    container_name: ${PROJECT_NAME}-db
    environment:
      - POSTGRES_DB=odoo_dev
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=odoo
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  pip-cache:
  git-cache:
  odoo-data:
  postgres-data:
EOF

# Create development configuration
cat << EOF > odoo.conf
[options]
addons_path = addons,odoo/addons
admin_passwd = admin
db_host = db
db_port = 5432
db_user = odoo
db_password = odoo
db_name = odoo_dev
logfile = /tmp/odoo.log
log_level = info
dev_mode = reload,qweb,werkzeug,xml
EOF

# Create addon requirements
cat << EOF > requirements.txt
# Add your custom addon requirements here
# Example:
# requests>=2.25.0
# beautifulsoup4>=4.9.0
EOF

# Create development scripts
cat << 'EOF' > scripts/dev-start.sh
#!/bin/bash
source venv/bin/activate
python odoo/odoo-bin --config=odoo.conf --dev=reload,qweb,werkzeug,xml
EOF

cat << 'EOF' > scripts/dev-test.sh
#!/bin/bash
source venv/bin/activate
ADDONS=$(manifestoo list --separator=, addons/)
python odoo/odoo-bin -i $ADDONS --test-enable --stop-after-init --config=odoo.conf
EOF

cat << 'EOF' > scripts/dev-shell.sh
#!/bin/bash
source venv/bin/activate
python odoo/odoo-bin shell --config=odoo.conf
EOF

chmod +x scripts/*.sh

echo "Development environment created!"
echo "Next steps:"
echo "1. docker-compose -f docker-compose.dev.yml up -d"
echo "2. docker-compose -f docker-compose.dev.yml exec odoo-dev /bin/bash"
echo "3. ./setup-env.sh (inside container)"
echo "4. ./scripts/dev-start.sh (to start Odoo)"
```

#### Hot Reload Development
```bash
# Inside development container
source venv/bin/activate

# Start Odoo with hot reload
python odoo/odoo-bin \
  --config=odoo.conf \
  --dev=reload,qweb,werkzeug,xml \
  --log-level=debug

# In another terminal: make changes and see them reload automatically
# Edit addon files - changes are automatically detected and reloaded
```

### Testing Workflows

#### Unit Testing Setup
```bash
#!/bin/bash
# test-setup.sh

# Setup test environment
virtualenv -p python3.11 test-env
source test-env/bin/activate

# Install testing dependencies
pip install \
    pytest \
    pytest-odoo \
    pytest-cov \
    pytest-mock \
    factory-boy

# Install Odoo in test mode
git clone https://github.com/odoo/odoo.git --branch 16.0 --depth 1
pip install -e odoo/

# Run tests
pytest addons/ -v --cov=addons --cov-report=html
```

#### Integration Testing
```bash
#!/bin/bash
# integration-test.sh

source venv/bin/activate

# Start with clean database
dropdb --if-exists test_integration
createdb test_integration

# Install base modules
python odoo/odoo-bin \
  -d test_integration \
  -i base \
  --stop-after-init \
  --log-level=warn

# Install and test custom addons
ADDONS=$(manifestoo list --separator=, addons/)
python odoo/odoo-bin \
  -d test_integration \
  -i $ADDONS \
  --test-enable \
  --stop-after-init \
  --log-level=test

# Cleanup
dropdb test_integration
```

## Advanced Use Cases

### Multi-Environment Management

#### Environment Configuration
```bash
#!/bin/bash
# env-manager.sh

ENVIRONMENT=${1:-"development"}

case $ENVIRONMENT in
  "development")
    export PGHOST=localhost
    export PGDATABASE=odoo_dev
    export ODOO_LOG_LEVEL=debug
    export ODOO_DEV_MODE=reload,qweb,werkzeug,xml
    ;;
  "testing")
    export PGHOST=test-db
    export PGDATABASE=odoo_test
    export ODOO_LOG_LEVEL=test
    export ODOO_DEV_MODE=""
    ;;
  "staging")
    export PGHOST=staging-db
    export PGDATABASE=odoo_staging
    export ODOO_LOG_LEVEL=info
    export ODOO_DEV_MODE=""
    ;;
  "production")
    export PGHOST=prod-db
    export PGDATABASE=odoo_prod
    export ODOO_LOG_LEVEL=warn
    export ODOO_DEV_MODE=""
    ;;
esac

echo "Environment: $ENVIRONMENT"
echo "Database: $PGHOST/$PGDATABASE"
echo "Log level: $ODOO_LOG_LEVEL"
```

#### Usage
```bash
# Source environment configuration
source env-manager.sh development
python odoo/odoo-bin --config=odoo.conf

source env-manager.sh testing
./scripts/run-tests.sh

source env-manager.sh production
python odoo/odoo-bin --config=odoo.prod.conf
```

### Performance Testing

#### Load Testing Setup
```bash
#!/bin/bash
# load-test.sh

source venv/bin/activate

# Install load testing tools
pip install locust

# Create load test script
cat << 'EOF' > locustfile.py
from locust import HttpUser, task, between

class OdooUser(HttpUser):
    wait_time = between(1, 3)
    
    def on_start(self):
        # Login to Odoo
        self.client.post("/web/login", data={
            "login": "admin",
            "password": "admin",
            "csrf_token": self.get_csrf_token()
        })
    
    def get_csrf_token(self):
        response = self.client.get("/web/login")
        # Extract CSRF token from response
        return "dummy-token"  # Implement actual extraction
    
    @task(3)
    def view_dashboard(self):
        self.client.get("/web")
    
    @task(2)
    def view_contacts(self):
        self.client.get("/web#action=base.action_partner_form")
    
    @task(1)
    def create_contact(self):
        self.client.post("/web/dataset/call_kw/res.partner/create", json={
            "method": "create",
            "args": [{"name": "Test Contact"}]
        })
EOF

# Run load test
echo "Starting Odoo for load testing..."
python odoo/odoo-bin --config=odoo.conf &
ODOO_PID=$!

sleep 30  # Wait for Odoo to start

echo "Running load test..."
locust -f locustfile.py --host=http://localhost:8069 --users=10 --spawn-rate=2 --run-time=5m --html=load-test-report.html

# Cleanup
kill $ODOO_PID
```

### Custom Image Extensions

#### Creating Specialized Images
```dockerfile
# Dockerfile.custom
FROM ghcr.io/opsivist/sonod:22.04

USER root

# Install additional system dependencies
RUN apt-get update && apt-get install -y \
    redis-tools \
    nginx \
    supervisor \
    && rm -rf /var/lib/apt/lists/*

# Install additional Python tools
USER gitlab-runner
RUN pipx install \
    pre-commit \
    black \
    isort \
    flake8

# Install project-specific requirements
COPY requirements.txt /tmp/
RUN virtualenv -p python3.11 /home/gitlab-runner/project-venv && \
    /home/gitlab-runner/project-venv/bin/pip install -r /tmp/requirements.txt

# Copy custom configurations
COPY --chown=gitlab-runner custom-configs/ /home/gitlab-runner/.config/

# Set default virtual environment
ENV PATH="/home/gitlab-runner/project-venv/bin:$PATH"

# Custom entrypoint
COPY entrypoint.sh /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
```

#### Custom Entrypoint Script
```bash
#!/bin/bash
# entrypoint.sh

set -e

# Activate project virtual environment
source /home/gitlab-runner/project-venv/bin/activate

# Setup Odoo if not present
if [ ! -d "/workspace/odoo" ]; then
    echo "Setting up Odoo..."
    cd /workspace
    git clone https://github.com/odoo/odoo.git --branch 16.0 --depth 1
fi

# Install additional requirements if present
if [ -f "/workspace/requirements.txt" ]; then
    pip install -r /workspace/requirements.txt
fi

# Execute command or start Odoo
if [ $# -eq 0 ]; then
    # Default: start Odoo
    cd /workspace
    python odoo/odoo-bin --addons-path=addons,odoo/addons
else
    # Execute provided command
    exec "$@"
fi
```

## Production Deployment

### Kubernetes Deployment

#### Deployment Configuration
```yaml
# k8s/odoo-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: odoo-ci
  labels:
    app: odoo-ci
spec:
  replicas: 3
  selector:
    matchLabels:
      app: odoo-ci
  template:
    metadata:
      labels:
        app: odoo-ci
    spec:
      containers:
      - name: odoo-ci
        image: ghcr.io/opsivist/sonod:22.04
        ports:
        - containerPort: 8069
        env:
        - name: PGHOST
          value: "postgres-service"
        - name: PGUSER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: username
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        volumeMounts:
        - name: pip-cache
          mountPath: /home/gitlab-runner/.cache/pip
        - name: git-cache
          mountPath: /home/gitlab-runner/.cache/git-autoshare
        - name: addon-source
          mountPath: /workspace/addons
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /web/health
            port: 8069
          initialDelaySeconds: 60
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /web/health
            port: 8069
          initialDelaySeconds: 30
          periodSeconds: 10
      volumes:
      - name: pip-cache
        persistentVolumeClaim:
          claimName: pip-cache-pvc
      - name: git-cache
        persistentVolumeClaim:
          claimName: git-cache-pvc
      - name: addon-source
        configMap:
          name: addon-source-config
```

#### Service Configuration
```yaml
# k8s/odoo-service.yml
apiVersion: v1
kind: Service
metadata:
  name: odoo-ci-service
spec:
  selector:
    app: odoo-ci
  ports:
  - name: http
    port: 80
    targetPort: 8069
  - name: longpolling
    port: 8072
    targetPort: 8072
  type: LoadBalancer
```

### Docker Swarm Deployment

#### Stack Configuration
```yaml
# docker-stack.yml
version: '3.8'

services:
  odoo:
    image: ghcr.io/opsivist/sonod:22.04
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    environment:
      - PGHOST=postgres
      - PGUSER=odoo
      - PGPASSWORD_FILE=/run/secrets/postgres_password
    secrets:
      - postgres_password
    volumes:
      - pip-cache:/home/gitlab-runner/.cache/pip
      - git-cache:/home/gitlab-runner/.cache/git-autoshare
    networks:
      - odoo-network
    command: |
      /bin/bash -c "
      source /home/gitlab-runner/project-venv/bin/activate &&
      python odoo/odoo-bin --addons-path=addons,odoo/addons
      "

  postgres:
    image: postgres:13
    environment:
      - POSTGRES_DB=odoo_prod
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD_FILE=/run/secrets/postgres_password
    secrets:
      - postgres_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - odoo-network
    deploy:
      placement:
        constraints:
          - node.role == manager

secrets:
  postgres_password:
    external: true

volumes:
  pip-cache:
  git-cache:
  postgres-data:

networks:
  odoo-network:
    driver: overlay
```

#### Deployment Commands
```bash
# Deploy stack
docker stack deploy -c docker-stack.yml odoo-prod

# Scale services
docker service scale odoo-prod_odoo=5

# Update service
docker service update --image ghcr.io/opsivist/sonod:22.04 odoo-prod_odoo

# Monitor deployment
docker stack ps odoo-prod
docker service logs -f odoo-prod_odoo
```

## Troubleshooting Examples

### Common Development Issues

#### Database Connection Problems
```bash
#!/bin/bash
# debug-db-connection.sh

echo "=== Database Connection Debug ==="

# Check environment variables
echo "PGHOST: $PGHOST"
echo "PGPORT: ${PGPORT:-5432}"
echo "PGUSER: $PGUSER"
echo "PGDATABASE: $PGDATABASE"

# Test PostgreSQL client
echo "Testing psql connection..."
if psql -h $PGHOST -U $PGUSER -d $PGDATABASE -c "SELECT version();" 2>/dev/null; then
    echo "✅ Database connection successful"
else
    echo "❌ Database connection failed"
    
    # Troubleshooting steps
    echo "Troubleshooting steps:"
    echo "1. Check if PostgreSQL is running:"
    pg_isready -h $PGHOST -p ${PGPORT:-5432}
    
    echo "2. Test network connectivity:"
    nc -zv $PGHOST ${PGPORT:-5432}
    
    echo "3. Check credentials:"
    echo "   - Verify PGUSER and PGPASSWORD"
    echo "   - Check database exists: psql -h $PGHOST -U $PGUSER -l"
fi
```

#### Python Environment Issues
```bash
#!/bin/bash
# debug-python-env.sh

echo "=== Python Environment Debug ==="

# Check Python installation
echo "Available Python versions:"
ls /usr/bin/python* | grep -E "python[0-9]"

# Check virtual environment
if [ -n "$VIRTUAL_ENV" ]; then
    echo "✅ Virtual environment active: $VIRTUAL_ENV"
    echo "Python executable: $(which python)"
    echo "Python version: $(python --version)"
    echo "Pip version: $(pip --version)"
else
    echo "❌ No virtual environment active"
    echo "System Python: $(which python3)"
    echo "System Python version: $(python3 --version)"
fi

# Check package installation
echo "Checking critical packages:"
python -c "
packages = ['lxml', 'psycopg2', 'werkzeug', 'jinja2']
for pkg in packages:
    try:
        __import__(pkg)
        print(f'✅ {pkg}: OK')
    except ImportError as e:
        print(f'❌ {pkg}: {e}')
"

# Check Odoo installation
if [ -d "odoo" ]; then
    echo "✅ Odoo directory exists"
    cd odoo
    python -c "import odoo; print(f'✅ Odoo import: {odoo.release.version}')" 2>/dev/null || echo "❌ Odoo import failed"
else
    echo "❌ Odoo directory not found"
fi
```

#### Git and Repository Issues
```bash
#!/bin/bash
# debug-git.sh

echo "=== Git Configuration Debug ==="

# Check git wrapper
echo "Git executable: $(which git)"
echo "Git version: $(git --version)"

# Check git configuration
echo "Git user configuration:"
git config --global --list | grep user

echo "Git safe directory configuration:"
git config --global --list | grep safe.directory

# Test git autoshare
echo "Testing git autoshare:"
if command -v git >/dev/null 2>&1; then
    git autoshare status 2>/dev/null || echo "❌ Git autoshare not working"
    
    # Check autoshare configuration
    if [ -f "/home/gitlab-runner/.config/git-autoshare/repos.yml" ]; then
        echo "✅ Git autoshare config exists"
        cat /home/gitlab-runner/.config/git-autoshare/repos.yml
    else
        echo "❌ Git autoshare config missing"
    fi
else
    echo "❌ Git not found"
fi

# Test SSH configuration
echo "SSH configuration:"
if [ -f "/home/gitlab-runner/.ssh/config" ]; then
    echo "✅ SSH config exists"
    cat /home/gitlab-runner/.ssh/config
else
    echo "❌ SSH config missing"
fi

# Test git operations
echo "Testing git clone:"
cd /tmp
if git clone https://github.com/odoo/odoo.git --depth 1 test-clone 2>/dev/null; then
    echo "✅ Git clone successful"
    rm -rf test-clone
else
    echo "❌ Git clone failed"
fi
```

### Performance Optimization Examples

#### Memory-Optimized Odoo Startup
```bash
#!/bin/bash
# memory-optimized-start.sh

source venv/bin/activate

# Set Python memory optimization
export PYTHONMALLOC=malloc
export MALLOC_MMAP_THRESHOLD_=1048576

# Set Odoo worker configuration for container
export ODOO_WORKERS=2
export ODOO_MAX_CRON_THREADS=1

# Start Odoo with optimized settings
python odoo/odoo-bin \
  --config=odoo.conf \
  --workers=$ODOO_WORKERS \
  --max-cron-threads=$ODOO_MAX_CRON_THREADS \
  --limit-memory-soft=671088640 \
  --limit-memory-hard=805306368 \
  --limit-time-cpu=600 \
  --limit-time-real=1200
```

#### Build Performance Optimization
```bash
#!/bin/bash
# optimize-build.sh

# Clean package cache
apt-get clean
rm -rf /var/lib/apt/lists/*

# Clean Python cache
find /home/gitlab-runner -name "*.pyc" -delete
find /home/gitlab-runner -name "__pycache__" -type d -exec rm -rf {} + 2>/dev/null || true

# Optimize pip cache
pip cache purge

# Clean git autoshare cache
git autoshare clean

# Remove unnecessary files
rm -rf /tmp/*
rm -rf /var/tmp/*

echo "Build optimization complete"
```

### Monitoring and Logging

#### Comprehensive Logging Setup
```bash
#!/bin/bash
# setup-logging.sh

# Create log directories
mkdir -p logs/{odoo,nginx,postgres}

# Setup log rotation
cat << 'EOF' > /etc/logrotate.d/odoo
/workspace/logs/odoo/*.log {
    daily
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 644 gitlab-runner gitlab-runner
}
EOF

# Start Odoo with comprehensive logging
source venv/bin/activate
python odoo/odoo-bin \
  --config=odoo.conf \
  --logfile=logs/odoo/odoo.log \
  --log-level=info \
  --log-handler=werkzeug:WARNING \
  --log-handler=odoo.sql_db:WARNING \
  --log-web=True
```

#### Health Monitoring
```bash
#!/bin/bash
# health-monitor.sh

while true; do
    echo "=== $(date) ==="
    
    # Check Odoo process
    if pgrep -f "odoo-bin" >/dev/null; then
        echo "✅ Odoo process running"
    else
        echo "❌ Odoo process not found"
    fi
    
    # Check database connectivity
    if psql -h $PGHOST -U $PGUSER -d $PGDATABASE -c "SELECT 1;" >/dev/null 2>&1; then
        echo "✅ Database connection OK"
    else
        echo "❌ Database connection failed"
    fi
    
    # Check memory usage
    MEMORY_USAGE=$(free | grep Mem | awk '{printf "%.1f", $3/$2 * 100.0}')
    echo "Memory usage: ${MEMORY_USAGE}%"
    
    # Check disk usage
    DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
    echo "Disk usage: ${DISK_USAGE}%"
    
    # Wait before next check
    sleep 60
done
```

## Integration Examples

### Jenkins Pipeline
```groovy
// Jenkinsfile
pipeline {
    agent {
        docker {
            image 'ghcr.io/opsivist/sonod:22.04'
            args '''
                -v pip-cache:/home/gitlab-runner/.cache/pip
                -v git-cache:/home/gitlab-runner/.cache/git-autoshare
                -e PGHOST=postgres
                -e PGUSER=odoo
                -e PGPASSWORD=odoo
            '''
        }
    }
    
    environment {
        PYTHON_VERSION = '3.11'
        ODOO_VERSION = '16.0'
    }
    
    stages {
        stage('Setup') {
            steps {
                sh '''
                    virtualenv -p python${PYTHON_VERSION} venv
                    source venv/bin/activate
                    git clone https://github.com/odoo/odoo.git --branch ${ODOO_VERSION} --depth 1
                    pip install -r odoo/requirements.txt
                '''
            }
        }
        
        stage('Lint') {
            steps {
                sh '''
                    source venv/bin/activate
                    pip install flake8 black isort
                    black --check addons/
                    isort --check-only addons/
                    flake8 addons/ --max-line-length=88
                '''
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    source venv/bin/activate
                    python odoo/odoo-bin -i base --stop-after-init
                    ADDONS=$(manifestoo list --separator=, addons/)
                    python odoo/odoo-bin -i $ADDONS --test-enable --stop-after-init
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'test-reports',
                        reportFiles: 'index.html',
                        reportName: 'Test Report'
                    ])
                }
            }
        }
        
        stage('Build') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    source venv/bin/activate
                    pip install setuptools wheel
                    python setup.py bdist_wheel
                '''
                archiveArtifacts artifacts: 'dist/*.whl', fingerprint: true
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        failure {
            emailext (
                subject: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "Build failed. Check console output at ${env.BUILD_URL}",
                to: "${env.CHANGE_AUTHOR_EMAIL}"
            )
        }
    }
}
```

### Azure DevOps Pipeline
```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main
    - develop
  tags:
    include:
    - v*

pool:
  vmImage: 'ubuntu-latest'

container: ghcr.io/opsivist/sonod:22.04

variables:
  PYTHON_VERSION: '3.11'
  ODOO_VERSION: '16.0'
  PIP_CACHE_DIR: $(Pipeline.Workspace)/.cache/pip

stages:
- stage: Test
  jobs:
  - job: TestAddons
    strategy:
      matrix:
        Python38:
          PYTHON_VERSION: '3.8'
        Python39:
          PYTHON_VERSION: '3.9'
        Python310:
          PYTHON_VERSION: '3.10'
        Python311:
          PYTHON_VERSION: '3.11'
    
    steps:
    - task: Cache@2
      inputs:
        key: 'pip | "$(Agent.OS)" | requirements.txt'
        path: $(PIP_CACHE_DIR)
      displayName: 'Cache pip packages'

    - script: |
        virtualenv -p python$(PYTHON_VERSION) venv
        source venv/bin/activate
        git clone https://github.com/odoo/odoo.git --branch $(ODOO_VERSION) --depth 1
        pip install -r odoo/requirements.txt
      displayName: 'Setup Environment'

    - script: |
        source venv/bin/activate
        python odoo/odoo-bin -i base --stop-after-init
        ADDONS=$(manifestoo list --separator=, addons/)
        python odoo/odoo-bin -i $ADDONS --test-enable --stop-after-init
      displayName: 'Run Tests'

- stage: Build
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - job: BuildArtifacts
    steps:
    - script: |
        source venv/bin/activate
        pip install setuptools wheel
        python setup.py bdist_wheel
      displayName: 'Build Wheel'

    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: 'dist'
        artifactName: 'wheels'
```

## Best Practices Summary

### Development Best Practices
1. **Environment Isolation**: Always use virtual environments
2. **Version Pinning**: Pin Python and package versions for reproducibility
3. **Caching**: Leverage all available caching mechanisms
4. **Testing**: Implement comprehensive testing at multiple levels
5. **Code Quality**: Use automated linting and formatting tools

### CI/CD Best Practices
1. **Pipeline Optimization**: Use parallel jobs and caching
2. **Security**: Properly manage secrets and credentials
3. **Monitoring**: Implement health checks and alerting
4. **Documentation**: Keep documentation updated with code changes
5. **Rollback Strategy**: Plan for quick rollbacks in case of issues

### Production Best Practices
1. **Resource Management**: Set appropriate resource limits
2. **High Availability**: Use multiple replicas and health checks
3. **Monitoring**: Implement comprehensive logging and monitoring
4. **Security**: Regular security updates and vulnerability scanning
5. **Backup Strategy**: Regular database and configuration backups