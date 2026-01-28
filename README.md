# Media Server Stack

A complete media server setup using Docker Compose with Plex Media Server, Sonarr, Radarr, Prowlarr, Bazarr, and qBittorrent.

## Services

- **Plex Media Server** (Port 32400) - Media streaming server
- **Sonarr** (Port 8989) - TV show automation
- **Radarr** (Port 7878) - Movie automation
- **Prowlarr** (Port 9696) - Indexer manager
- **Bazarr** (Port 6767) - Subtitle manager
- **qBittorrent** (Port 8080) - Torrent download client

## Prerequisites

- Docker and Docker Compose installed
- Basic knowledge of Docker and media server concepts

## Quick Start

1. **Clone or download this repository**

2. **Set up environment variables**

   Copy `.env.example` to `.env` and configure:
   ```bash
   cp .env.example .env
   ```

   Edit `.env` and set:
   - `PUID` and `PGID` - Your user/group IDs (run `id` to find them)
   - `TZ` - Your timezone (e.g., `America/New_York`, `Europe/London`)
   - `PLEX_CLAIM` - Optional Plex claim token from https://www.plex.tv/claim

3. **Create data directories**

   ```bash
   mkdir -p data/movies data/tv data/downloads
   ```

4. **Start the services**

   ```bash
   docker-compose up -d
   ```

5. **Access the services**

   - Plex: http://localhost:32400/web
   - Sonarr: http://localhost:8989
   - Radarr: http://localhost:7878
   - Prowlarr: http://localhost:9696
   - Bazarr: http://localhost:6767
   - qBittorrent: http://localhost:8080

## Configuration

### Pre-configured Settings

The following settings have been pre-configured via config files:

- **API Keys**: All services have secure API keys generated and configured
- **Ports**: Standard ports are configured for each service
- **Root Folders**: 
  - Sonarr: `/data/tv`
  - Radarr: `/data/movies`
- **Download Paths**: All services use `/data/downloads`
- **qBittorrent**: Web UI authentication is disabled by default (change in config if needed)

### Required Manual Configuration

While most settings are pre-configured, you'll need to complete these steps via the web UI:

#### 1. Plex Media Server

1. Access Plex at http://localhost:32400/web
2. Complete the initial setup wizard
3. Add libraries:
   - Movies library pointing to `/data/movies`
   - TV Shows library pointing to `/data/tv`

#### 2. Prowlarr

1. Access Prowlarr at http://localhost:9696
2. Add indexers (torrent trackers/Usenet providers)
3. Configure app connections:
   - **Sonarr**: 
     - Host: `sonarr`
     - Port: `8989`
     - API Key: (found in `config/sonarr/config.xml`)
   - **Radarr**:
     - Host: `radarr`
     - Port: `7878`
     - API Key: (found in `config/radarr/config.xml`)
   - **Bazarr**:
     - Host: `bazarr`
     - Port: `6767`
     - API Key: (found in `config/bazarr/config.ini`)

#### 3. Sonarr

1. Access Sonarr at http://localhost:8989
2. Go to Settings > Download Clients
3. Add qBittorrent:
   - Host: `qbittorrent`
   - Port: `8080`
   - Username: `admin` (or as configured in qBittorrent)
   - Password: (if authentication is enabled)
4. Go to Settings > Indexers
5. Add Prowlarr:
   - Host: `prowlarr`
   - Port: `9696`
   - API Key: (found in `config/prowlarr/config.xml`)

#### 4. Radarr

1. Access Radarr at http://localhost:7878
2. Go to Settings > Download Clients
3. Add qBittorrent:
   - Host: `qbittorrent`
   - Port: `8080`
   - Username: `admin` (or as configured in qBittorrent)
   - Password: (if authentication is enabled)
4. Go to Settings > Indexers
5. Add Prowlarr:
   - Host: `prowlarr`
   - Port: `9696`
   - API Key: (found in `config/prowlarr/config.xml`)

#### 5. Bazarr

1. Access Bazarr at http://localhost:6767
2. Go to Settings > Sonarr
3. Configure Sonarr connection:
   - Hostname: `sonarr`
   - Port: `8989`
   - API Key: (found in `config/sonarr/config.xml`)
4. Go to Settings > Radarr
5. Configure Radarr connection:
   - Hostname: `radarr`
   - Port: `7878`
   - API Key: (found in `config/radarr/config.xml`)

#### 6. qBittorrent

1. Access qBittorrent at http://localhost:8080
2. Authentication is disabled by default
3. Configure download settings as needed
4. Default download path: `/data/downloads`

## Finding API Keys

API keys are pre-generated and stored in the configuration files:

- **Sonarr**: `config/sonarr/config.xml` - Look for `<ApiKey>`
- **Radarr**: `config/radarr/config.xml` - Look for `<ApiKey>`
- **Prowlarr**: `config/prowlarr/config.xml` - Look for `<ApiKey>`
- **Bazarr**: `config/bazarr/config.ini` - Look for `apikey =`

## Service Communication

All services communicate via Docker service names (not localhost):
- Sonarr → qBittorrent: `qbittorrent:8080`
- Radarr → qBittorrent: `qbittorrent:8080`
- Sonarr/Radarr → Prowlarr: `prowlarr:9696`
- Bazarr → Sonarr: `sonarr:8989`
- Bazarr → Radarr: `radarr:7878`

## File Structure

```
media-server/
├── docker-compose.yml      # Main compose file
├── .env                    # Environment variables (create from .env.example)
├── .env.example           # Example environment variables
├── .gitignore             # Git ignore rules
├── config/                # Service configurations
│   ├── plex/
│   ├── sonarr/
│   ├── radarr/
│   ├── prowlarr/
│   ├── bazarr/
│   └── qbittorrent/
└── data/                  # Media storage (excluded from git)
    ├── movies/
    ├── tv/
    └── downloads/
```

## Updating Services

To update all services to their latest versions:

```bash
docker-compose pull
docker-compose up -d
```

## Stopping Services

```bash
docker-compose down
```

To also remove volumes (⚠️ this will delete all configuration):

```bash
docker-compose down -v
```

## Troubleshooting

### Permission Issues

If you encounter permission issues with files:

1. Check your `PUID` and `PGID` in `.env`
2. Ensure the data directories are owned by the correct user:
   ```bash
   sudo chown -R $PUID:$PGID data/
   ```

### Services Can't Communicate

- Ensure all services are using Docker service names (not `localhost` or `127.0.0.1`)
- Check that services are on the same Docker network (they are by default with `network_mode: bridge`)

### Port Conflicts

If ports are already in use, edit `docker-compose.yml` and change the port mappings on the left side (e.g., `"8989:8989"` → `"8990:8989"`).

## Security Notes

- API keys are pre-generated but should be changed in production
- qBittorrent authentication is disabled by default - enable it in production
- Consider setting up authentication for all services in production
- The `.env` file contains sensitive information and is excluded from git

## Additional Resources

- [LinuxServer.io Documentation](https://docs.linuxserver.io/)
- [Sonarr Wiki](https://wiki.servarr.com/sonarr)
- [Radarr Wiki](https://wiki.servarr.com/radarr)
- [Prowlarr Wiki](https://wiki.servarr.com/prowlarr)
- [Bazarr Wiki](https://wiki.bazarr.media/)
- [qBittorrent Documentation](https://github.com/qbittorrent/qBittorrent/wiki)

## License

This configuration is provided as-is for personal use.
