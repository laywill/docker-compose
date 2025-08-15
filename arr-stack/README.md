# Arr Stack with VPN

A complete media management and download stack including Sonarr, Radarr, Lidarr, Prowlarr, Bazarr, Flaresolverr, Overseerr, Requestrr, qBittorrent, SABnzbd, and NZBHydra2 with VPN protection via Gluetun and automatic updates via Watchtower.

**⚠️ Important**: This Docker Compose configuration brings the entire stack online with secure defaults. Each service requires individual configuration through their web interfaces to function properly.

**🚨 Critical**: The `.env` file is MANDATORY - the stack will not work without it. There are no fallback paths in the compose file.

## Required Configuration

Before deploying, you **must** configure the following:

### 1. Environment Variables (MANDATORY)

The stack **will not start** without a properly configured `.env` file:

```bash
cp .env.example .env
# Edit .env with your settings - ALL variables must be set!
```

**Required variables in `.env` file (no defaults provided):**

```bash
# VPN Configuration
VPN_PROVIDER=nordvpn          # Examples: nordvpn, surfshark, expressvpn, protonvpn, pia, windscribe
VPN_TYPE=openvpn             # Options: openvpn, wireguard
VPN_USERNAME=your_username    # Your VPN account username
VPN_PASSWORD=your_password    # Your VPN account password

# Path Configuration (customize these base paths)
CONFIG_ROOT=/home/user/docker/configs    # Base path for all app configurations
MEDIA_ROOT=/home/user/media              # Base path for media libraries
DOWNLOADS_ROOT=/home/user/downloads      # Downloads directory

# Timezone Configuration (recommended to match VPN exit location)
TZ=Europe/London                         # Default: Europe/London

# Optional VPN Settings
VPN_REGION=                  # Leave empty for auto-selection
VPN_COUNTRY=                 # Leave empty for auto-selection
```

### 2. Directory Structure

The configuration uses three main path variables to organize all directories:

- **CONFIG_ROOT**: Base directory for all application configurations
- **MEDIA_ROOT**: Base directory for media libraries (music, movies, tv)
- **DOWNLOADS_ROOT**: Downloads directory shared by all download clients

Each application automatically creates its subdirectory under CONFIG_ROOT:

```
${CONFIG_ROOT}/
├── gluetun/
├── sonarr/
├── radarr/
├── lidarr/
├── prowlarr/
├── bazarr/
├── overseerr/
├── requestrr/
├── qbittorrent/
├── sabnzbd/
└── nzbhydra2/

${MEDIA_ROOT}/
├── music/
├── movies/
└── tv/

${DOWNLOADS_ROOT}/
├── incomplete/      # SABnzbd incomplete downloads
└── [completed files]
```

### 3. Directory Permissions

Ensure all directories are owned by UID/GID 1000:1000:

```bash
# Using the path variables from your .env file
sudo chown -R 1000:1000 ${CONFIG_ROOT}
sudo chown -R 1000:1000 ${MEDIA_ROOT}
sudo chown -R 1000:1000 ${DOWNLOADS_ROOT}

# Example with actual paths:
sudo chown -R 1000:1000 /home/user/docker/configs
sudo chown -R 1000:1000 /home/user/media
sudo chown -R 1000:1000 /home/user/downloads
```

### 4. Timezone Configuration

**Important**: Set your timezone to match your VPN exit node location for optimal functionality:

- **Netherlands VPN**: `TZ=Europe/Amsterdam`
- **Germany VPN**: `TZ=Europe/Berlin`
- **US East VPN**: `TZ=America/New_York`
- **US West VPN**: `TZ=America/Los_Angeles`
- **Canada VPN**: `TZ=America/Toronto`
- **Default**: `TZ=Europe/London` (if not specified)

This ensures proper scheduling, logging timestamps, and optimal integration with indexers and trackers in your VPN region.

## Dependencies

- Docker Engine or Docker Desktop
- VPN provider account (NordVPN, Surfshark, ProtonVPN, PIA, Windscribe, etc.)

## Network Architecture

All media services and downloaders (Sonarr, Radarr, Lidarr, Prowlarr, Bazarr, Flaresolverr, Overseerr, Requestrr, qBittorrent, SABnzbd, NZBHydra2) route their traffic through the Gluetun VPN container:

- **Gluetun**: Acts as VPN gateway, exposes all service ports
- **Media Services**: Use `network_mode: "service:gluetun"` for VPN routing
- **Watchtower**: Runs on separate network to maintain update capability

## Accessing Services

All services are accessible through Gluetun's IP on these ports:

- **Lidarr** (Music): <http://localhost:8686>
- **Radarr** (Movies): <http://localhost:7878>
- **Sonarr** (TV Shows): <http://localhost:8989>
- **Prowlarr** (Indexers): <http://localhost:9696>
- **Bazarr** (Subtitles): <http://localhost:6767>
- **Flaresolverr** (CloudFlare Bypass): <http://localhost:8191>
- **Overseerr** (Requests): <http://localhost:5055>
- **Requestrr** (Discord Bot): <http://localhost:4545>
- **qBittorrent** (Torrent): <http://localhost:8080>
- **SABnzbd** (Usenet): <http://localhost:8081>
- **NZBHydra2** (NZB Meta Search): <http://localhost:5076>

## Deployment

1. **Copy and configure environment file**: `cp .env.example .env`
2. **Edit `.env`** with your VPN credentials and paths
3. **Set directory permissions** (see above)
4. **Deploy the stack**:

```bash
# Start all services
docker compose up -d

# View logs
docker compose logs -f

# Check service status
docker compose ps
```

## Security Features

- **VPN-only traffic**: All media services route through VPN
- **Minimal privileges**: Services run with dropped capabilities
- **Non-root users**: All services use UID/GID 1000:1000
- **Read-only mounts**: Docker socket mounted read-only for Watchtower
- **No new privileges**: Security hardening enabled
- **Health checks**: All services include health monitoring and automatic recovery
- **Resource limits**: Configurable memory and CPU limits prevent resource exhaustion

## VPN Provider Support

This configuration supports all [Gluetun-compatible VPN providers](https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers):

- NordVPN, Surfshark, ExpressVPN
- ProtonVPN, Private Internet Access (PIA)
- Windscribe, CyberGhost, Mullvad
- And many more...

## Troubleshooting

### Check VPN connectivity

```bash
docker compose logs gluetun
docker compose exec gluetun wget -qO- ifconfig.me
```

### Service can't connect to internet

1. Ensure Gluetun is healthy: `docker compose ps`
2. Check VPN credentials in environment variables
3. Verify VPN provider is correctly configured

### Port access issues

- All services are accessible through Gluetun container ports
- Don't try to access services directly by container name - use localhost with the exposed ports

### Update issues

Watchtower runs outside VPN network to maintain update capability during VPN container restarts.

### Resource exhaustion

```bash
# Check resource usage
docker stats

# Adjust limits in .env for resource-constrained systems
# Example for 8GB RAM system - reduce all limits by 50%:
RADARR_MEMORY_LIMIT=512M
SONARR_MEMORY_LIMIT=512M
QBITTORRENT_MEMORY_LIMIT=1G
```

### High resource usage services

- **Download clients** (qBittorrent, SABnzbd): Highest CPU/memory usage during active downloads
- **Flaresolverr**: Browser automation can be memory intensive
- **Large libraries**: Services managing large media collections need more resources

## Service Descriptions

### Core Media Management

- **Sonarr**: TV show management and monitoring
- **Radarr**: Movie management and monitoring
- **Lidarr**: Music management and monitoring
- **Prowlarr**: Indexer management for all *arr apps

### Additional Services

- **Bazarr**: Subtitle management and downloading
- **Flaresolverr**: CloudFlare protection bypass for indexers
- **Overseerr**: User request management with beautiful web interface
- **Requestrr**: Discord bot for handling media requests

### Download Clients

- **qBittorrent**: BitTorrent client with web interface
- **SABnzbd**: Usenet binary downloader
- **NZBHydra2**: NZB meta search for Usenet indexers

### Configuration Notes

- **Flaresolverr**: No persistent storage needed, runs entirely in memory
- **Requestrr**: Configuration stored in `/root/config` (different from LinuxServer pattern)
- **Overseerr**: Integrates with Radarr/Sonarr for automated request processing
- **Bazarr**: Requires access to same media directories as Radarr/Sonarr for subtitle management
- **qBittorrent**: Default login is `admin/adminadmin` - change immediately after first login
- **SABnzbd**: Configure port 8081 in settings if it defaults to 8080 to avoid conflicts
- **NZBHydra2**: Acts as proxy/aggregator for multiple NZB indexers

## Post-Deployment Configuration Required

**This compose file only brings the services online**. After deployment, you must configure each service individually:

1. **Configure downloaders** (qBittorrent, SABnzbd) with download paths
2. **Set up indexers** in Prowlarr and NZBHydra2
3. **Configure arr apps** (Sonarr, Radarr, Lidarr) with download clients and indexers
4. **Connect services** (Overseerr to Radarr/Sonarr, Bazarr to media libraries)
5. **Set up VPN credentials** in environment variables
6. **Update all file paths** to match your system

Refer to each service's documentation for detailed configuration instructions.
