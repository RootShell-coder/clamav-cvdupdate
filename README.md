# ClamAV Database Mirror

A containerized solution for maintaining and serving ClamAV antivirus databases using Docker and Nginx. This project automatically downloads and updates ClamAV signature databases from official mirrors and serves them via HTTP for local network distribution.

[![Docker](https://img.shields.io/badge/Docker-20.10%2B-blue)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Build and check image](https://github.com/RootShell-coder/clamav-cvdupdate/actions/workflows/docker.yml/badge.svg?branch=master)](https://github.com/RootShell-coder/clamav-cvdupdate/actions/workflows/docker.yml)

## Features

- 🔄 **Automatic Updates**: Periodic database updates using `cvdupdate` tool
- 🌐 **HTTP Distribution**: Nginx-based web server for serving database files
- 🐳 **Containerized**: Fully containerized solution with Docker Compose
- 🔧 **Configurable Mirrors**: Support for multiple ClamAV mirror sources
- 📊 **Health Checks**: Built-in health monitoring for both services
- 🚀 **Production Ready**: Optimized configurations for production use
- 📁 **CVD Format**: Supports ClamAV signature databases in CVD format

## Quick Start

1. **Clone the repository**:

   ```bash
   git clone <repository-url>
   cd clamav_cvdupdate
   ```

2. **Configure mirror (optional)**:

   ```bash
   cp .env.example .env
   # Edit .env to uncomment your preferred mirror
   ```

3. **Start the services**:

   ```bash
   docker-compose up -d
   ```

4. **Access the database files**:
   - Web interface: <http://localhost>
   - Database files: <http://localhost/{filename}.cvd>

## Architecture

```text
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   ClamAV        │    │   cvdupdate     │    │     Nginx       │
│   Official      │───▶│   Container     │───▶│   Web Server    │
│   Mirrors       │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │                        │
                              ▼                        ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │  Shared Volume  │    │  HTTP Clients   │
                       │   /database     │    │ (ClamAV nodes)  │
                       └─────────────────┘    └─────────────────┘
```

## Configuration

### Mirror Selection

Edit the `.env` file to choose your preferred ClamAV mirror:

```bash
# Official ClamAV mirror (recommended)
CLAMAV_MIRROR=https://database.clamav.net

# Alternative mirrors
# CLAMAV_MIRROR=https://db.local.clamav.net
# CLAMAV_MIRROR=https://clamav-mirror.ru
```

**Important**: Only uncomment ONE mirror at a time.

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CLAMAV_MIRROR` | ClamAV mirror URL | `https://database.clamav.net` |

### Docker Compose Configuration

The `docker-compose.yml` includes:

- **cvdupdate service**: Downloads and maintains ClamAV databases
- **nginx service**: Serves database files via HTTP
- **Shared volume**: `/database` for file sharing between containers
- **Health checks**: Monitoring for both services
- **Network**: Isolated Docker network for services

## Database Files

The following ClamAV database files are automatically maintained:

| File | Description | Format |
|------|-------------|--------|
| `main.cvd` | Main virus signatures | CVD |
| `daily.cvd` | Daily updates | CVD |
| `bytecode.cvd` | Bytecode signatures | CVD |

## Usage Examples

### ClamAV Configuration

Configure your ClamAV instances to use the local mirror:

```bash
# In clamd.conf or freshclam.conf
DatabaseMirror http://your-server-ip/
```

### Testing Database Access

```bash
# Download a database file
curl -O http://localhost/main.cvd

# Check available files
curl http://localhost/

# Verify file integrity
sigtool --info main.cvd
```

### Health Check

```bash
# Check service status
docker-compose ps

# View logs
docker-compose logs cvdupdate
docker-compose logs nginx
```

## Monitoring

### Health Checks

The setup includes built-in health checks:

- **cvdupdate**: Checks for presence of `index.html`
- **nginx**: Verifies web server responsiveness

### Logs

Monitor the services:

```bash
# Follow all logs
docker-compose logs -f

# Follow specific service
docker-compose logs -f cvdupdate
docker-compose logs -f nginx
```

## Maintenance

### Manual Database Update

```bash
# Trigger immediate update
docker-compose exec cvdupdate /usr/local/bin/update
```

### Update Container Images

```bash
# Pull latest images
docker-compose pull

# Restart with new images
docker-compose up -d
```

### Backup Database

```bash
# Create backup
tar -czf clamav-db-backup-$(date +%Y%m%d).tar.gz database/
```

## Troubleshooting

### Common Issues

1. **Database download fails**:
   - Check internet connectivity
   - Verify mirror URL in `.env`
   - Check logs: `docker-compose logs cvdupdate`

2. **Nginx not serving files**:
   - Ensure database files exist in `./database/`
   - Check nginx logs: `docker-compose logs nginx`

3. **Permission issues**:
   - Verify volume permissions
   - Check container user permissions

### Debug Commands

```bash
# Check container status
docker-compose ps

# Inspect volumes
docker volume ls
docker volume inspect clamav_cvdupdate_clamav-data

# Check network connectivity
docker-compose exec cvdupdate ping database.clamav.net

# Validate configuration
docker-compose config
```

## Performance Tuning

### Nginx Optimization

The Nginx configuration includes:

- **Gzip compression**: Reduces bandwidth usage
- **Caching headers**: Improves client-side caching
- **Optimized worker processes**: Better performance

### Update Frequency

Database updates run every 2 hours by default. Adjust in the update script if needed.

## Security Considerations

- **Network isolation**: Services run in isolated Docker network
- **Read-only volumes**: Nginx serves files as read-only
- **No privileged access**: Containers run with minimal privileges
- **Health monitoring**: Built-in checks for service availability

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [ClamAV Project](https://www.clamav.net/) - Open-source antivirus engine
- [cvdupdate](https://pypi.org/project/cvdupdate/) - ClamAV database update tool
- [Docker](https://www.docker.com/) - Containerization platform
- [Nginx](https://nginx.org/) - High-performance web server

---

**Note**: This project is not officially affiliated with the ClamAV project. It's a community-maintained solution for ClamAV database distribution.
