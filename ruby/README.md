# Ruby Development Environment

A secure, configurable Ruby development container for REPL work, scripting, web development, and project development with Rails, Sinatra, and other Ruby frameworks.

**🚨 Important**: The `.env` file is REQUIRED - the WORKSPACE_PATH variable must be set. Copy `.env.example` to `.env` and configure it for your system.

## ⚠️ Required Configuration

### 1. Environment Variables (MANDATORY)

```bash
cp .env.example .env
# Edit .env with your workspace path - this is required!
```

**Minimum required configuration:**

```bash
# The only required variable - current directory (.) used as fallback
# Linux/macOS examples:
WORKSPACE_PATH=/home/user/development
# WORKSPACE_PATH=/Users/username/projects

# Windows examples:
# WORKSPACE_PATH=C:/Users/username/development
# WORKSPACE_PATH=C:/dev/projects
# WORKSPACE_PATH=D:/code
```

**Optional configuration with defaults:**

```bash
RUBY_VERSION=3.1             # Ruby version (default: 3.1)
CONTAINER_NAME=ruby-dev      # Container name (default: ruby-dev-container)
PUID=1000                    # User ID (default: 1000)
PGID=1000                    # Group ID (default: 1000)
TZ=UTC                       # Timezone (default: UTC)

# Resource limits (prevents runaway processes)
MEMORY_LIMIT=6400M           # Memory limit (default: ~6.4GB)
CPU_LIMIT=3.2               # CPU limit (default: 3.2 cores)
```

### 2. Create Workspace Directory

**Linux/macOS:**

```bash
# Create and set permissions for your workspace
mkdir -p /home/user/development
sudo chown -R $USER:$USER /home/user/development
```

**Windows (PowerShell):**

```powershell
# Create workspace directory
New-Item -ItemType Directory -Force -Path "C:\Users\$env:USERNAME\development"

# Windows handles permissions automatically for user directories
# Ensure Docker Desktop has access to the drive in Settings > Resources > File Sharing
```

## Dependencies

- Docker Engine or Docker Desktop
- A development workspace directory on your host system

## Usage

### Basic Usage

```bash
# Start the Ruby development container
docker compose run --rm ruby-dev

# Container starts with interactive bash shell in /workspace
# Your host workspace directory is mounted and accessible
```

### Example Session

```bash
# Start container
docker compose run --rm ruby-dev

# Inside container:
ruby --version         # Check Ruby version
ruby -e 'puts "Hello World"'  # Run Ruby code
gem install sinatra   # Install gems
ruby app.rb            # Run your Ruby scripts
irb                    # Start Interactive Ruby
exit                   # Exit container
```

### Ruby Versions

Supported Ruby versions (configure via `RUBY_VERSION`):

- `2.7`, `3.0`, `3.1`, `3.2`, `3.3`
- Slim variants: `3.1-slim`, `3.2-slim`
- Alpine variants: `3.1-alpine`, `3.2-alpine`

```bash
# Use Ruby 3.2
RUBY_VERSION=3.2 docker compose run --rm ruby-dev

# Use Ruby 3.1 slim
RUBY_VERSION=3.1-slim docker compose run --rm ruby-dev
```

## Git Integration (Optional)

For seamless Git operations, uncomment the git volume mounts in `compose.yaml`:

```yaml
# Uncomment these lines in compose.yaml:
# - ${HOME}/.gitconfig:/home/ruby/.gitconfig:ro
# - ${HOME}/.git-credentials:/home/ruby/.git-credentials:ro
# - ${HOME}/.ssh:/home/ruby/.ssh:ro
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
- **Health monitoring**: Automatic Ruby interpreter health checks
- **Configurable capabilities**: Documented options for additional permissions
- **Resource limits**: Prevents runaway processes from consuming all system resources

### Available Capabilities

The container includes only `CHOWN` by default. Uncomment additional capabilities in `compose.yaml` if needed:

- **DAC_OVERRIDE**: Override file access permissions
- **FOWNER**: File ownership operations
- **SETUID/SETGID**: Change user/group ID (some gem installations)
- **NET_BIND_SERVICE**: Bind to privileged ports (<1024)
- **SYS_PTRACE**: Debug/trace processes (debuggers like byebug)

## Common Workflows

### 1. Script Development

```bash
# Start container with your project directory
docker compose run --rm ruby-dev

# Inside container - your files are in /workspace
cd /workspace
ruby my_script.rb
```

### 2. Gem Experimentation

```bash
# Start container
docker compose run --rm ruby-dev

# Install and test gems
gem install rails
ruby -e "require 'rails'; puts Rails::VERSION::STRING"
```

### 3. Rails Application Development

```bash
# Start container
docker compose run --rm ruby-dev

# Inside container:
gem install rails
rails new myapp --database=postgresql
cd myapp
bundle install
rails server -b 0.0.0.0
```

### 4. Sinatra Web Applications

```bash
# Start container
docker compose run --rm ruby-dev

# Inside container:
gem install sinatra
# Create and run your Sinatra app
ruby app.rb -o 0.0.0.0
```

### 5. Interactive Development

```bash
# Long-running container for development session
docker compose up -d ruby-dev
docker compose exec ruby-dev bash

# Your development session...

# When finished:
docker compose down
```

## Ruby-Specific Environment

The container includes pre-configured Ruby environment variables:

- **BUNDLE_PATH=/usr/local/bundle**: Where gems are installed
- **BUNDLE_BIN=/usr/local/bundle/bin**: Where gem binaries are stored
- **GEM_HOME=/usr/local/bundle**: Primary gem installation directory
- **BUNDLE_SILENCE_ROOT_WARNING=1**: Disable Bundler root warnings
- **PATH**: Includes gem binary paths

### Bundler and Gem Management

```bash
# Inside container:
bundle install          # Install gems from Gemfile
bundle exec ruby app.rb  # Run commands in bundle context
gem install gem_name     # Install specific gems
gem list                 # List installed gems
```

## Health Monitoring

The container includes health checks that verify:

- Ruby interpreter is accessible
- Ruby version can be determined
- Basic Ruby functionality works

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

### Gem Installation Issues

Uncomment additional capabilities in `compose.yaml` if gem installation fails:

```yaml
cap_add:
  - CHOWN
  - SETUID    # Uncomment for gem installations
  - SETGID    # Uncomment for gem installations
```

### Memory/CPU Exhaustion

Ruby processes can consume large amounts of resources, especially Rails applications. Adjust limits in `.env`:

```bash
# For memory-intensive applications (Rails with large datasets)
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

# Windows PowerShell equivalents:
# Memory (80% of total)
$totalMem = [math]::Round((Get-CimInstance Win32_PhysicalMemory | Measure-Object -Property capacity -Sum).sum /1gb * 0.8, 1)
Write-Host "MEMORY_LIMIT=${totalMem}G"

# CPU (80% of total)
$totalCPU = [math]::Round($env:NUMBER_OF_PROCESSORS * 0.8, 1)
Write-Host "CPU_LIMIT=$totalCPU"
```

### Windows-Specific Issues

**Volume Mount Problems:**

```powershell
# Ensure Docker Desktop has drive access
# Settings > Resources > File Sharing > Add your drive (C:, D:, etc.)

# Check if path exists and is accessible
Test-Path "C:\Users\$env:USERNAME\development"

# Use forward slashes in .env file (Docker prefers this)
# WORKSPACE_PATH=C:/Users/username/development  # Good
# WORKSPACE_PATH=C:\Users\username\development  # May cause issues
```

**Permission Issues on Windows:**

```powershell
# Check current user has access
Get-Acl "C:\Users\$env:USERNAME\development" | Format-List

# If using WSL2, ensure path is accessible from WSL
wsl ls -la /mnt/c/Users/$USER/development
```

## Example Projects

### Rails Web Application

**Linux/macOS:**

```bash
# .env configuration
WORKSPACE_PATH=/home/user/rails-projects
RUBY_VERSION=3.2

# Start container and create Rails app
docker compose run --rm ruby-dev
gem install rails
rails new blog --database=postgresql
cd blog
bundle install
```

**Windows:**

```powershell
# .env configuration
# WORKSPACE_PATH=C:/Users/username/rails-projects
# RUBY_VERSION=3.2

# Start container and create Rails app
docker compose run --rm ruby-dev
gem install rails
rails new blog --database=postgresql
cd blog
bundle install
```

### Sinatra Web Application

**Linux/macOS:**

```bash
# .env configuration
WORKSPACE_PATH=/home/user/sinatra-apps
RUBY_VERSION=3.1

# Enable port binding capability in compose.yaml
# Uncomment: NET_BIND_SERVICE

# Start container and create Sinatra app
docker compose run --rm ruby-dev
gem install sinatra
```

**Windows:**

```powershell
# .env configuration
# WORKSPACE_PATH=C:/Users/username/sinatra-apps
# RUBY_VERSION=3.1

# Enable port binding capability in compose.yaml
# Uncomment: NET_BIND_SERVICE

# Start container and create Sinatra app
docker compose run --rm ruby-dev
gem install sinatra
```

### Gem Development

**Linux/macOS:**

```bash
# .env configuration
WORKSPACE_PATH=/home/user/gem-development
RUBY_VERSION=3.1

# Start container and create gem
docker compose run --rm ruby-dev
gem install bundler
bundle gem my_awesome_gem
cd my_awesome_gem
bundle install
bundle exec rspec  # Run tests
```

**Windows:**

```powershell
# .env configuration
# WORKSPACE_PATH=C:/Users/username/gem-development
# RUBY_VERSION=3.1

# Start container and create gem
docker compose run --rm ruby-dev
gem install bundler
bundle gem my_awesome_gem
cd my_awesome_gem
bundle install
bundle exec rspec  # Run tests
```

## Common Ruby Commands

```bash
# Inside the container:

# Check Ruby version and info
ruby --version
ruby -v

# Interactive Ruby (IRB)
irb

# Install gems
gem install gem_name
gem install rails --version=7.0.0

# Bundler operations
bundle install           # Install gems from Gemfile
bundle update            # Update gems
bundle exec command      # Run command in bundle context

# Rails commands (if Rails is installed)
rails new app_name       # Create new Rails app
rails server -b 0.0.0.0  # Start Rails server (accessible from host)
rails console            # Rails console
rails generate model User name:string  # Generate model

# Run Ruby scripts
ruby script.rb           # Run a Ruby script
ruby -e 'puts "Hello"'   # Execute Ruby code directly
```