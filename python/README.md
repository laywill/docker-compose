# Python Development Environment

A secure, configurable Python development container for REPL work, scripting, experimentation, and project development.

**🚨 Important**: The `.env` file is REQUIRED - the WORKSPACE_PATH variable must be set. Copy `.env.example` to `.env` and configure it for your system.

## ⚠️ Required Configuration

### 1. Environment Variables (MANDATORY)

```bash
cp .env.example .env
# Edit .env with your workspace path - this is required!
```

**Minimum required configuration:**

```bash
# The only required variable - no fallback provided
WORKSPACE_PATH=/path/to/your/development/directory
```

**Optional configuration with defaults:**

```bash
PYTHON_VERSION=3.11          # Python version (default: 3.11)
CONTAINER_NAME=python-dev    # Container name (default: python-dev-container)
PUID=1000                    # User ID (default: 1000)
PGID=1000                    # Group ID (default: 1000)  
TZ=UTC                       # Timezone (default: UTC)

# Resource limits (prevents runaway processes)
MEMORY_LIMIT=6400M           # Memory limit (default: ~6.4GB)
CPU_LIMIT=3.2               # CPU limit (default: 3.2 cores)
```

### 2. Create Workspace Directory

```bash
# Create and set permissions for your workspace
mkdir -p /path/to/your/development
sudo chown -R $USER:$USER /path/to/your/development
```

## Dependencies

- Docker Engine or Docker Desktop
- A development workspace directory on your host system

## Usage

### Basic Usage

```bash
# Start the Python development container
docker compose run --rm python-dev

# Container starts with interactive bash shell in /workspace
# Your host workspace directory is mounted and accessible
```

### Example Session

```bash
# Start container
docker compose run --rm python-dev

# Inside container:
python --version        # Check Python version
python                  # Start Python REPL
pip install requests    # Install packages
python script.py        # Run your scripts
exit                    # Exit container
```

### Python Versions

Supported Python versions (configure via `PYTHON_VERSION`):

- `3.8`, `3.9`, `3.10`, `3.11`, `3.12`, `3.13`
- Slim variants: `3.11-slim`, `3.12-slim`
- Alpine variants: `3.11-alpine`, `3.12-alpine`

```bash
# Use Python 3.12
PYTHON_VERSION=3.12 docker compose run --rm python-dev

# Use Python 3.11 slim
PYTHON_VERSION=3.11-slim docker compose run --rm python-dev
```

## Git Integration (Optional)

For seamless Git operations, uncomment the git volume mounts in `compose.yaml`:

```yaml
# Uncomment these lines in compose.yaml:
# - ${HOME}/.gitconfig:/home/python/.gitconfig:ro
# - ${HOME}/.git-credentials:/home/python/.git-credentials:ro  
# - ${HOME}/.ssh:/home/python/.ssh:ro
```

This provides:

- **Git configuration**: Your identity and settings
- **Credentials**: Access to private repositories
- **SSH keys**: Authentication for Git operations

**SSH Key Permissions:**

```bash
chmod 600 ~/.ssh/id_rsa
chmod 700 ~/.ssh
```

## Security Features

- **Minimal privileges**: Runs as non-root user (UID/GID 1000:1000)
- **Capability dropping**: Only essential capabilities enabled
- **No privilege escalation**: `no-new-privileges:true`
- **Health monitoring**: Automatic Python interpreter health checks
- **Configurable capabilities**: Documented options for additional permissions
- **Resource limits**: Prevents runaway processes from consuming all system resources

### Available Capabilities

The container includes only `CHOWN` by default. Uncomment additional capabilities in `compose.yaml` if needed:

- **DAC_OVERRIDE**: Override file access permissions
- **FOWNER**: File ownership operations
- **SETUID/SETGID**: Change user/group ID (some package installations)
- **NET_BIND_SERVICE**: Bind to privileged ports (<1024)
- **SYS_PTRACE**: Debug/trace processes (debuggers like pdb)

## Common Workflows

### 1. Script Development

```bash
# Start container with your project directory
docker compose run --rm python-dev

# Inside container - your files are in /workspace
cd /workspace
python my_script.py
```

### 2. Package Experimentation

```bash
# Start container
docker compose run --rm python-dev

# Install and test packages
pip install pandas numpy
python -c "import pandas; print(pandas.__version__)"
```

### 3. Jupyter Notebooks

```bash
# Install and run Jupyter inside container
docker compose run --rm python-dev

# Inside container:
pip install jupyter
jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser --allow-root
```

### 4. Interactive Development

```bash
# Long-running container for development session
docker compose up -d python-dev
docker compose exec python-dev bash

# Your development session...

# When finished:
docker compose down
```

## Environment Variables

- **PYTHONUNBUFFERED=1**: Force Python output to be unbuffered
- **PYTHONDONTWRITEBYTECODE=1**: Prevent Python from writing .pyc files
- **TZ**: Timezone for proper timestamp handling

## Health Monitoring

The container includes health checks that verify:

- Python interpreter is accessible
- Python version can be determined
- Basic Python functionality works

Check health status:

```bash
docker compose ps
```

## File Permissions

The container runs as UID/GID 1000:1000 by default. If you need different user IDs:

```bash
# Check your user ID
id

# Set in .env file
PUID=1001
PGID=1001
```

## Troubleshooting

### Permission Issues

```bash
# Ensure workspace is owned by your user
sudo chown -R $USER:$USER /path/to/workspace

# Check container user matches your user
id
```

### Git Operations Fail

```bash
# Ensure SSH key permissions
chmod 600 ~/.ssh/id_rsa
chmod 700 ~/.ssh

# Test SSH connection
ssh -T git@github.com
```

### Python Package Installation Issues

Uncomment additional capabilities in `compose.yaml` if package installation fails:

```yaml
cap_add:
  - CHOWN
  - SETUID    # Uncomment for package installations
  - SETGID    # Uncomment for package installations
```

### Memory/CPU Exhaustion

Python processes can consume large amounts of resources. Adjust limits in `.env`:

```bash
# For memory-intensive tasks (ML, data processing)
MEMORY_LIMIT=12G
CPU_LIMIT=6.0

# For resource-constrained systems
MEMORY_LIMIT=2G
CPU_LIMIT=1.0
```

Auto-detect your system limits:

```bash
# Memory (80% of total)
echo "MEMORY_LIMIT=$(($(grep MemTotal /proc/meminfo | awk '{print $2}') * 80 / 100 / 1024))M"

# CPU (80% of total)
echo "CPU_LIMIT=$(echo "$(nproc) * 0.8" | bc)"
```

## Example Projects

### Data Analysis Setup

```bash
# .env configuration
WORKSPACE_PATH=/home/user/data-analysis
PYTHON_VERSION=3.11

# Start container and install data stack
docker compose run --rm python-dev
pip install pandas numpy matplotlib jupyter seaborn
```

### Web Development Setup  

```bash
# .env configuration
WORKSPACE_PATH=/home/user/web-projects
PYTHON_VERSION=3.12

# Enable port binding capability in compose.yaml
# Uncomment: NET_BIND_SERVICE

# Start container and install web framework
docker compose run --rm python-dev
pip install flask fastapi uvicorn
```
