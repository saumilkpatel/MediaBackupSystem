# SOP: Complete Media Backup System Setup

## Overview
This procedure describes how to set up the entire Media Backup System from scratch, including hardware setup, software configuration, and iPhone integration.

## Prerequisites
### Hardware Requirements
- Raspberry Pi 5
- External Hard Drive
- Ethernet cable
- Power supply for Raspberry Pi
- Power supply for external drive (if needed)
- MicroSD card (16GB+ recommended)

### Software Requirements
- Raspberry Pi OS image
- Raspberry Pi Imager software
- iPhones with PhotoSync app
- Network with WiFi

### Tools Required
- MicroSD card reader
- Basic tools for mounting/connecting hardware
- Computer for initial setup

## Steps

### Phase 1: Initial Hardware Setup
1. **Prepare Raspberry Pi**
   ```bash
   # Download and install Raspberry Pi Imager on your computer
   # Insert MicroSD card
   # Use Imager to write Raspberry Pi OS
   # Configure in Imager:
   # - Set hostname (e.g., mediabackup)
   # - Enable SSH
   # - Set username and password
   # - Configure WiFi (optional, we'll use ethernet)
   ```

2. **Physical Setup**
   - Insert MicroSD card into Raspberry Pi
   - Connect ethernet cable to router
   - Connect external drive to USB port
   - Connect power supply
   - Power on Raspberry Pi

3. **Initial Network Connection**
   - Find Raspberry Pi's IP address (check router or use network scanner)
   - Connect via SSH:
   ```bash
   ssh username@raspberry-pi-ip
   ```

### Phase 2: System Configuration
1. **Update System**
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

2. **Create Service User**
   ```bash
   # Create photosync user
   sudo useradd -m photosync
   sudo usermod -aG users photosync
   sudo usermod -aG sudo photosync
   sudo passwd photosync
   ```

3. **Prepare Storage**
   ```bash
   # Identify drive
   lsblk

   # Format drive with exFAT (replace sdX with correct drive letter)
   sudo apt install exfat-fuse exfat-utils
   sudo mkfs.exfat /dev/sdX1

   # Create mount point
   sudo mkdir -p /backup

   # Mount drive (use photosync's UID/GID)
   sudo mount -o uid=1001,gid=1001 /dev/sdX1 /backup

   # Create user directories
   sudo mkdir -p /backup/user1
   sudo mkdir -p /backup/user2

   # Set permissions
   sudo chown -R photosync:photosync /backup
   sudo chmod -R 777 /backup
   ```

4. **Make Mount Permanent**
   ```bash
   # Add to fstab
   echo "/dev/sdX1  /backup  exfat  defaults,uid=1001,gid=1001  0  0" | sudo tee -a /etc/fstab
   ```

### Phase 3: Network Services Setup
1. **Install Required Packages**
   ```bash
   # Install Samba and tools
   sudo apt install -y samba samba-common-bin smbclient
   ```

2. **Configure Samba**
   ```bash
   # Backup original config
   sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.backup

   # Create new config
   sudo nano /etc/samba/smb.conf
   ```

   Add at the end of file:
   ```ini
   [User1]
   path = /backup/user1
   browseable = yes
   read only = no
   create mask = 0777
   directory mask = 0777
   valid users = photosync
   force user = photosync
   force group = photosync
   guest ok = no
   writeable = yes
   public = no
   follow symlinks = yes
   wide links = yes
   unix extensions = no

   [User2]
   path = /backup/user2
   browseable = yes
   read only = no
   create mask = 0777
   directory mask = 0777
   valid users = photosync
   force user = photosync
   force group = photosync
   guest ok = no
   writeable = yes
   public = no
   follow symlinks = yes
   wide links = yes
   unix extensions = no
   ```

3. **Set Up Samba User**
   ```bash
   # Set Samba password for photosync
   sudo smbpasswd -a photosync

   # Restart Samba services
   sudo systemctl restart smbd
   sudo systemctl restart nmbd
   ```

### Phase 4: PhotoPrism Setup
1. **Install Docker**
   ```bash
   # Install Docker
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh

   # Add user to docker group
   sudo usermod -aG docker $USER

   # Install Docker Compose
   sudo apt install -y docker-compose

   # Start and enable Docker
   sudo systemctl enable docker
   sudo systemctl start docker
   ```

2. **Prepare PhotoPrism Directories**
   ```bash
   # Create directories
   sudo mkdir -p /docker/photoprisim/storage

   # Create symbolic link
   sudo ln -s /backup /docker/photoprisim/originals

   # Set permissions
   sudo chown -R $USER:$USER /docker/photoprisim
   ```

3. **Configure PhotoPrism**
   ```bash
   # Create docker-compose.yml
   nano /docker/photoprisim/docker-compose.yml
   ```

   Add content:
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
         - "2342:2342"
       environment:
         PHOTOPRISM_ADMIN_PASSWORD: "please-change-password"
         PHOTOPRISM_SITE_URL: "http://raspberry-pi-ip:2342/"
         PHOTOPRISM_ORIGINALS_LIMIT: 5000
         PHOTOPRISM_HTTP_COMPRESSION: "gzip"
         PHOTOPRISM_DEBUG: "false"
         PHOTOPRISM_PUBLIC: "false"
         PHOTOPRISM_READONLY: "false"
         PHOTOPRISM_EXPERIMENTAL: "false"
         PHOTOPRISM_DISABLE_CHOWN: "false"
         PHOTOPRISM_DISABLE_WEBDAV: "false"
         PHOTOPRISM_DISABLE_SETTINGS: "false"
         PHOTOPRISM_DISABLE_TENSORFLOW: "false"
         PHOTOPRISM_DETECT_NSFW: "false"
         PHOTOPRISM_UPLOAD_NSFW: "true"
         PHOTOPRISM_DATABASE_DRIVER: "sqlite"
         PHOTOPRISM_SITE_TITLE: "Family Photos"
       volumes:
         - "/docker/photoprisim/storage:/photoprism/storage"
         - "/docker/photoprisim/originals:/photoprism/originals"
   ```

4. **Start PhotoPrism**
   ```bash
   cd /docker/photoprisim
   docker-compose up -d
   ```

### Phase 5: iPhone Setup
1. **Install PhotoSync**
   - Install PhotoSync app on each iPhone
   - Open app settings

2. **Configure First iPhone**
   - Add new destination
   - Select SMB (Windows Share)
   - Enter Raspberry Pi's IP address
   - Share Name: User1
   - Username: photosync
   - Password: (Samba password)
   - Directory Structure: `<mediaType>/<year>/<filename>`
   - Enable "Live Photo MOV -> Photos Folder"

3. **Configure Second iPhone**
   - Repeat above steps but use Share Name: User2

### Phase 6: Verification
1. **Check Storage**
   ```bash
   # Verify mount
   df -h /backup
   ls -la /backup
   ```

2. **Test Samba Shares**
   ```bash
   # List shares
   smbclient -L localhost -U photosync

   # Test connections
   smbclient //localhost/User1 -U photosync
   smbclient //localhost/User2 -U photosync
   ```

3. **Test PhotoSync**
   - Try syncing a test photo from each iPhone
   - Verify files appear in correct directories

4. **Test PhotoPrism**
   - Access web interface: http://raspberry-pi-ip:2342
   - Login with default credentials
   - Change admin password
   - Verify photos are visible

## Troubleshooting

### Mount Issues
- Check `dmesg` for USB errors
- Verify drive is properly powered
- Check filesystem with `sudo fsck.exfat /dev/sdX1`

### Permission Problems
- Verify photosync UID/GID: `id photosync`
- Check directory permissions: `ls -la /backup`
- Verify Samba user: `sudo pdbedit -L -v`

### Network Issues
- Test network connectivity: `ping raspberry-pi-ip`
- Check Samba status: `sudo systemctl status smbd`
- Verify firewall settings: `sudo ufw status`

### PhotoPrism Issues
- Check logs: `docker-compose logs -f`
- Verify symbolic links: `ls -la /docker/photoprisim/originals`
- Check disk space: `df -h`

## Maintenance

### Regular Tasks
1. Monitor disk space
2. Check system logs
3. Update system:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```
4. Update PhotoPrism:
   ```bash
   cd /docker/photoprisim
   docker-compose pull
   docker-compose up -d
   ```

### Backup Considerations
- Consider backing up PhotoPrism database
- Document system configuration
- Keep passwords secure
- Monitor drive health

## Notes
- Replace user1/user2 with actual usernames
- Adjust paths and names as needed
- Consider security implications of permissions
- Monitor system during first few days
- Keep documentation updated with changes
