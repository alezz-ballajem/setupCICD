# Components Documentation

## Git Wrapper Script

### Overview
The `git-wrapper` is a bash script that intercepts git commands to provide enhanced functionality through git-autoshare integration.

### Location
`/usr/local/bin/git`

### Functionality

#### Script Logic
```bash
#!/bin/bash
if [ "$1" == "clone" ]
then
    shift
    /usr/bin/git autoshare-clone "$@"
else
    /usr/bin/git "$@"
fi
```

#### Behavior
- **Clone Operations**: Redirects `git clone` to `git autoshare-clone` for repository caching
- **Other Operations**: Passes all other git commands directly to the standard git binary
- **Transparency**: Acts as a drop-in replacement for git with enhanced clone performance

### Usage Examples

#### Standard Clone (Enhanced)
```bash
# This command automatically uses git-autoshare
git clone https://github.com/odoo/odoo.git
# Internally executes: /usr/bin/git autoshare-clone https://github.com/odoo/odoo.git
```

#### Other Git Commands (Pass-through)
```bash
git status          # → /usr/bin/git status
git add .           # → /usr/bin/git add .
git commit -m "msg" # → /usr/bin/git commit -m "msg"
git push origin     # → /usr/bin/git push origin
```

#### Benefits
- **Faster Clones**: Repository caching reduces clone times
- **Bandwidth Savings**: Shared repositories reduce network usage
- **Transparent**: No changes needed to existing git workflows
- **CI Optimization**: Particularly beneficial in CI/CD environments

### Configuration Dependencies
- Requires `git-autoshare` to be installed (provided via PIPX)
- Uses configuration from `/home/gitlab-runner/.config/git-autoshare/repos.yml`

## Configuration Files

### Git Autoshare Configuration

#### File Location
`/home/gitlab-runner/.config/git-autoshare/repos.yml`

#### Configuration Structure
```yaml
github.com:
  enterprise:           # Enterprise/private repository settings
    orgs:
      - opsivist        # Private repositories from opsivist org
      - odoo           # Private repositories from odoo org
    private: True       # Mark as private repositories
  "*":                 # Wildcard for all other repositories
    orgs:
      - opsivist        # Public repositories from opsivist org
      - OCA            # Open Community Association repositories
      - odoo           # Public repositories from odoo org
```

#### Supported Organizations
| Organization | Access Type | Repository Types |
|--------------|-------------|------------------|
| `opsivist` | Private + Public | All repositories |
| `odoo` | Private + Public | All repositories |
| `OCA` | Public only | Open source Odoo addons |

#### Usage Examples
```bash
# These repositories will use autoshare caching:
git clone https://github.com/opsivist/private-repo.git    # Private
git clone https://github.com/odoo/odoo.git               # Public
git clone https://github.com/OCA/server-tools.git        # Public

# Other repositories will use standard git clone:
git clone https://github.com/other-org/repo.git          # Standard clone
```

### SSH Configuration

#### File Location
`/home/gitlab-runner/.ssh/config`

#### Configuration Content
```ssh
Host *
  StrictHostKeyChecking no
```

#### Purpose
- **CI/CD Optimization**: Prevents interactive prompts for unknown hosts
- **Automation Friendly**: Allows automated SSH connections without manual intervention
- **Security Trade-off**: Disables host key verification for convenience

#### Security Considerations
- **Appropriate for**: Trusted CI/CD environments
- **Not Recommended for**: Production environments or untrusted networks
- **Mitigation**: Use in controlled, secure network environments only

#### Usage Examples
```bash
# SSH connections won't prompt for host key verification
ssh user@unknown-host.example.com

# Git operations over SSH work seamlessly
git clone git@github.com:opsivist/repo.git

# SCP and rsync operations are automated
scp file.txt user@host:/path/
rsync -av local/ user@host:/remote/
```

## Git Configuration

### Global Git Settings

#### User Identity
```bash
user.name = "GitLab"
user.email = "gitlab@opsivist.io"
```
- **Purpose**: Provides identity for git operations in CI/CD
- **Use Case**: Automated commits, tags, and other git operations

#### Safe Directory Configuration
```bash
safe.directory = "*"
```
- **Purpose**: Allows git operations when repository owner differs from current user
- **Scenario**: GitLab CI checks out as root, but container runs as gitlab-runner
- **Security**: Wildcard setting - use only in controlled environments

### Integration Benefits

#### Repository Caching
- **First Clone**: Downloads and caches repository
- **Subsequent Clones**: Uses cached version for faster access
- **Cache Location**: `/home/gitlab-runner/.cache/git-autoshare`
- **Performance**: Significant speedup in CI/CD pipelines

#### Bandwidth Optimization
- **Shared Objects**: Git objects shared between repositories
- **Incremental Updates**: Only downloads changes, not full repositories
- **Network Efficiency**: Reduces bandwidth usage in CI environments

## Component Dependencies

### Git Wrapper Dependencies
- `/usr/bin/git` (standard git binary)
- `git-autoshare` package (installed via PIPX)
- Git autoshare configuration file

### Configuration File Dependencies
- `git-autoshare` package for repository configuration
- SSH client for SSH configuration
- Proper file permissions (600 for SSH config, 644 for others)

## Troubleshooting

### Git Wrapper Issues

#### Problem: Clone fails with autoshare error
```bash
# Check git-autoshare installation
pipx list | grep git-autoshare

# Verify configuration
cat /home/gitlab-runner/.config/git-autoshare/repos.yml

# Test autoshare directly
/usr/bin/git autoshare-clone https://github.com/odoo/odoo.git
```

#### Problem: Standard git behavior needed
```bash
# Bypass wrapper by calling git directly
/usr/bin/git clone https://github.com/example/repo.git

# Or temporarily rename wrapper
sudo mv /usr/local/bin/git /usr/local/bin/git.bak
git clone https://github.com/example/repo.git
sudo mv /usr/local/bin/git.bak /usr/local/bin/git
```

### Configuration Issues

#### Problem: SSH host key verification fails
```bash
# Verify SSH config is loaded
ssh -F /home/gitlab-runner/.ssh/config -T git@github.com

# Check file permissions
ls -la /home/gitlab-runner/.ssh/config

# Test SSH connection
ssh -o StrictHostKeyChecking=no user@host
```

#### Problem: Git autoshare not working
```bash
# Check configuration syntax
python3 -c "import yaml; yaml.safe_load(open('/home/gitlab-runner/.config/git-autoshare/repos.yml'))"

# Verify git-autoshare status
git autoshare status

# Clear cache if corrupted
rm -rf /home/gitlab-runner/.cache/git-autoshare
```

## Advanced Configuration

### Custom Git Autoshare Setup
```yaml
# Example custom configuration
your-git-server.com:
  "*":
    orgs:
      - your-organization
    private: False

github.com:
  enterprise:
    orgs:
      - your-private-org
    private: True
  "*":
    orgs:
      - your-public-org
```

### Custom SSH Configuration
```ssh
# Example enhanced SSH config
Host github.com
  HostName github.com
  User git
  IdentityFile /home/gitlab-runner/.ssh/github_key
  StrictHostKeyChecking no

Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile /home/gitlab-runner/.ssh/gitlab_key
  StrictHostKeyChecking no

Host *
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```

### Environment Customization
```bash
# Custom environment variables for specific workflows
export ODOO_RC=/home/gitlab-runner/.odoorc
export PGHOST=localhost
export PGPORT=5432
export PGUSER=odoo
export PGDATABASE=odoo_test

# Python-specific settings
export PYTHONPATH=/home/gitlab-runner/custom-modules:$PYTHONPATH
export PIP_INDEX_URL=https://your-private-pypi.com/simple/
```