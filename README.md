# Docker Compose Collection

A curated collection of Docker Compose configurations for common services.

Inspired by [Docker's awesome-compose](https://github.com/docker/awesome-compose).

## 🚀 Available Stacks

### 📚 [Arr Stack with VPN](./arr-stack/)

Complete media management and download stack with VPN protection:

- **Media Management**: Sonarr, Radarr, Lidarr, Prowlarr
- **Download Clients**: qBittorrent, SABnzbd, NZBHydra2
- **Supporting Services**: Bazarr, Flaresolverr, Overseerr, Requestrr
- **Infrastructure**: Gluetun VPN, Watchtower auto-updater

**Features**: VPN-protected traffic, reverse proxy dashboard, resource limits, health monitoring, security hardening

📖 **[Full Documentation](./arr-stack/README.md)**

### 🐍 [Python Development Environment](./python/)

Secure, configurable Python development container:

- **Multiple Python versions**: 3.8-3.13, slim, alpine variants
- **Development ready**: Git integration, SSH keys, workspace mounting
- **Security hardened**: Minimal privileges, resource limits, health checks
- **Cross-platform**: Windows, Linux, macOS support

**Features**: Predictable user mapping, resource management, VS Code integration

📖 **[Full Documentation](./python/README.md)**

### 💎 [Ruby Development Environment](./ruby/)

Secure, configurable Ruby development container:

- **Multiple Ruby versions**: 2.7-3.3, slim, alpine variants
- **Framework ready**: Rails, Sinatra, gem development
- **Security hardened**: Minimal privileges, resource limits, health checks
- **Cross-platform**: Windows, Linux, macOS support

**Features**: Bundler integration, predictable user mapping, resource management

📖 **[Full Documentation](./ruby/README.md)**

## 🛠️ Quick Start

Each stack is self-contained with its own documentation and configuration:

```bash
# Choose your stack
cd arr-stack/    # For media management
cd python/       # For Python development
cd ruby/         # For Ruby development

# Follow the specific README in each directory
cp .env.example .env
# Edit .env with your configuration
docker compose up -d
```

## 📋 Common Features

All stacks in this collection include:

- ✅ **Environment-driven configuration** - No hardcoded values
- ✅ **Security hardening** - Minimal privileges, capability dropping
- ✅ **Resource management** - Configurable CPU and memory limits
- ✅ **Health monitoring** - Automatic service health checks
- ✅ **Cross-platform support** - Windows, Linux, macOS
- ✅ **Comprehensive documentation** - Setup guides and troubleshooting

## 🔧 Development Setup

This repository includes VS Code configuration for optimal editing:

### Recommended Extensions

- `davidanson.vscode-markdownlint` - Markdown linting and formatting
- `ms-azuretools.vscode-docker` - Docker support
- `redhat.vscode-yaml` - YAML/Docker Compose support

### Auto-formatting

- **Markdown files** format automatically on save
- **Consistent code style** across all configuration files
- **EditorConfig** support for cross-editor consistency

## 📁 Repository Structure

```
docker-compose/
├── arr-stack/                 # Media management stack
│   ├── compose.yaml          # Main configuration
│   ├── .env.example          # Environment template
│   └── README.md             # Complete setup guide
├── python/                   # Python development environment
│   ├── compose.yaml          # Python container config
│   ├── .env.example          # Environment template
│   └── README.md             # Complete setup guide
├── ruby/                     # Ruby development environment
│   ├── compose.yaml          # Ruby container config
│   ├── .env.example          # Environment template
│   └── README.md             # Complete setup guide
├── .vscode/                  # VS Code configuration
│   ├── settings.json         # Editor settings
│   └── extensions.json       # Recommended extensions
├── .editorconfig             # Cross-editor formatting
├── .markdownlint.json        # Markdown linting rules
└── README.md                 # This file
```

## 🤝 Contributing

When adding new stacks:

1. **Create dedicated directory** with descriptive name
2. **Include comprehensive README** with setup instructions
3. **Provide .env.example** with all required variables
4. **Follow security best practices** - non-root users, minimal privileges
5. **Add resource limits** - prevent resource exhaustion
6. **Include health checks** - ensure service reliability

## 📄 License

Licensed under the Apache License, Version 2.0. See [LICENSE](./LICENSE) for details.

## 🔗 Inspiration

This collection draws inspiration from:

- [Docker Awesome Compose](https://github.com/docker/awesome-compose)
- [LinuxServer.io](https://www.linuxserver.io/) containers
- Community best practices for Docker security and configuration
