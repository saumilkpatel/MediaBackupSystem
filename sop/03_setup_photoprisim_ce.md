# SOP: Setting Up PhotoPrism Community Edition

## Overview
This procedure describes how to install and configure PhotoPrism Community Edition on the Raspberry Pi to provide a web interface for viewing backed up photos.

## Prerequisites
- Raspberry Pi 5 with Linux OS
- Docker and Docker Compose installed
- Existing backup structure at `/backup/{saumil,vaishnavi}`
- At least 4GB RAM available
- Terminal access to Raspberry Pi

## Steps

### 1. Install Docker (if not installed)
```bash
# Update system
sudo apt update
sudo apt upgrade -y

# Install required packages
sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add current user to docker group
sudo usermod -aG docker $USER

# Start and enable Docker
sudo systemctl enable docker
sudo systemctl start docker
```

### 2. Install Docker Compose
```bash
# Install using apt
sudo apt install -y docker-compose
```

### 3. Create PhotoPrism Directory Structure
```bash
# Create directories
sudo mkdir -p /docker/photoprisim/storage

# Create symbolic link to backup directory
sudo ln -s /backup /docker/photoprisim/originals

# Set permissions
sudo chown -R $USER:$USER /docker/photoprisim
```

### 4. Create Docker Compose Configuration
```bash
# Create and edit docker-compose.yml
nano /docker/photoprisim/docker-compose.yml
```

Add the following content:
```yaml
version: '3.5'

services:
  photoprisim:
    image: photoprism/photoprism:latest
    container_name: photoprisim
    restart: unless-stopped
    security_opt:
      - seccomp:unconfined
      - apparmor:unconfined
    ports:
      - "2342:2342" # HTTP port (adjust if needed)
    environment:
      PHOTOPRISM_ADMIN_PASSWORD: "please-change-password"      # Initial admin password
      PHOTOPRISM_SITE_URL: "http://raspberry-pi-ip:2342/"      # Server URL
      PHOTOPRISM_ORIGINALS_LIMIT: 5000                         # Photo file size limit in MB
      PHOTOPRISM_HTTP_COMPRESSION: "gzip"                      # Improves transfer speed
      PHOTOPRISM_DEBUG: "false"                                # Debug mode
      PHOTOPRISM_PUBLIC: "false"                               # Disable public access
      PHOTOPRISM_READONLY: "false"                             # Read-only mode
      PHOTOPRISM_EXPERIMENTAL: "false"                         # Experimental features
      PHOTOPRISM_DISABLE_CHOWN: "false"                        # Disable storage permission updates
      PHOTOPRISM_DISABLE_WEBDAV: "false"                       # Disable built-in WebDAV server
      PHOTOPRISM_DISABLE_SETTINGS: "false"                     # Disable Settings page
      PHOTOPRISM_DISABLE_TENSORFLOW: "false"                   # Disable TensorFlow
      PHOTOPRISM_DETECT_NSFW: "false"                         # Flag NSFW photos
      PHOTOPRISM_UPLOAD_NSFW: "true"                          # Allow uploads that may be NSFW
      PHOTOPRISM_DATABASE_DRIVER: "sqlite"                     # Using SQLite for simplicity
      PHOTOPRISM_SITE_TITLE: "Family Photos"                  # Site title
      PHOTOPRISM_SITE_CAPTION: "AI-Powered Photos App"        # Site caption
      PHOTOPRISM_SITE_DESCRIPTION: ""                         # Site description
      PHOTOPRISM_SITE_AUTHOR: ""                              # Site author
    volumes:
      - "/docker/photoprisim/storage:/photoprism/storage"     # Storage folder
      - "/docker/photoprisim/originals:/photoprism/originals" # Original media files
```

### 5. Start PhotoPrism
```bash
# Navigate to PhotoPrism directory
cd /docker/photoprisim

# Start the container
docker-compose up -d

# Check logs
docker-compose logs -f
```

### 6. Initial Configuration
1. Access PhotoPrism web interface:
   - Open browser and go to `http://raspberry-pi-ip:2342`
   - Replace raspberry-pi-ip with your Pi's IP address

2. Login with default credentials:
   - Username: `admin`
   - Password: `please-change-password`

3. Change admin password:
   - Go to Settings → Account
   - Update password to something secure

4. Configure index:
   - Go to Settings → Library
   - Start initial indexing
   - This may take some time depending on library size

### 7. Verify Setup
1. Check if photos are visible in the library
2. Verify folder structure is preserved
3. Test search functionality
4. Check face detection features
5. Try accessing from mobile device

## Optional Configurations

### Enable HTTPS
1. Set up reverse proxy (nginx/traefik)
2. Configure SSL certificates
3. Update PHOTOPRISM_SITE_URL in docker-compose.yml

### Auto-Start on Boot
```bash
# Enable docker service
sudo systemctl enable docker

# Create systemd service for PhotoPrism
sudo nano /etc/systemd/system/photoprisim.service
```

Add content:
```ini
[Unit]
Description=PhotoPrism Docker Compose
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/docker/photoprisim
ExecStart=/usr/bin/docker-compose up -d
ExecStop=/usr/bin/docker-compose down

[Install]
WantedBy=multi-user.target
```

Enable service:
```bash
sudo systemctl enable photoprisim
sudo systemctl start photoprisim
```

## Troubleshooting

### 1. Container Won't Start
- Check logs: `docker-compose logs`
- Verify port 2342 is not in use: `sudo lsof -i :2342`
- Check disk space: `df -h`

### 2. Photos Not Appearing
- Verify symbolic links: `ls -la /docker/photoprisim/originals`
- Check permissions: `ls -la /backup`
- Restart indexing from web interface

### 3. Performance Issues
- Check RAM usage: `free -h`
- Monitor CPU: `top`
- Consider reducing PHOTOPRISM_ORIGINALS_LIMIT

### 4. Network Access Issues
- Check firewall: `sudo ufw status`
- Verify port forwarding if accessing from outside
- Test local access: `curl http://localhost:2342`

## Maintenance

### Regular Tasks
1. Monitor disk space
2. Check logs periodically
3. Keep Docker images updated:
```bash
docker-compose pull
docker-compose up -d
```

### Backup PhotoPrism Data
```bash
# Backup storage directory
sudo tar -czf photoprisim-backup.tar.gz /docker/photoprisim/storage
```

## Notes
- PhotoPrism may take significant time for initial indexing
- Face recognition requires more CPU resources
- Consider setting up regular backups of the PhotoPrism storage directory
- Monitor resource usage during first few days
- Check PhotoPrism documentation for advanced features
