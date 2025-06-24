# Media Backup System

## Overview
An automated system that detects when phones connect to the home WiFi network and automatically backs up new media (photos and videos) to a physical hard drive. This provides a seamless, hands-free backup solution for mobile media content.

## System Architecture
### Hardware Components
- **Raspberry Pi 5**
  - Pre-installed Linux operating system
  - Connected to router via ethernet
  - Provides both internet and local network access
  - Acts as the system controller
- **External Hard Drive**
  - Connected to Raspberry Pi via USB
  - Formatted with exFAT filesystem
  - Mounted at `/backup`
  - Separate directories for each user:
    - `/backup/saumil`
    - `/backup/vaishnavi`
  - Mount configuration: `uid=1000,gid=1000`
  - Serves as the backup storage destination
- **iPhones**
  - Multiple source devices for media content
  - Connect to home WiFi network
  - Use PhotoSync app for automated transfer
  - Configured with SMB for high-speed file transfer

## Key Features (Planned)
- Automatic phone detection on WiFi connection
- Selective media backup (photos and videos)
- Incremental backup (only new media)
- Local storage on physical hard drive
- Zero-interaction backup process
- High-speed SMB file transfer
- Web interface for viewing photos (PhotoPrism)
- Multi-user support with separate backup spaces

## System Configuration
### Storage Setup
```bash
# View block devices and their mount points
lsblk

# Mount external drive with user permissions
sudo mount -o uid=1000,gid=1000 /dev/sda1 /backup

# Create user-specific backup directories
sudo mkdir -p /backup/saumil
sudo mkdir -p /backup/vaishnavi

# Set permissions for backup directories
sudo chown -R photosync:photosync /backup/saumil
sudo chown -R photosync:photosync /backup/vaishnavi
sudo chmod -R 777 /backup/saumil
sudo chmod -R 777 /backup/vaishnavi
```
- Filesystem: exFAT (supports large files and is compatible with multiple operating systems)
- Mount point: `/backup`
- User directories:
  - `/backup/saumil` - Saumil's media backup
  - `/backup/vaishnavi` - Vaishnavi's media backup
- Mount options:
  - `uid=1000,gid=1000`: Sets ownership to user ID 1000 and group ID 1000
  - Device path: `/dev/sda1`
- Use `lsblk` to:
  - Verify drive detection
  - Check mount points
  - View drive partitions
  - Confirm successful mounting

### Operating System
- Linux OS (pre-installed on Raspberry Pi 5)
- Connected via ethernet for network access

### Network Services
#### SMB File Sharing
```bash
# Install Samba packages
sudo apt install samba samba-common-bin smbclient

# Backup original configuration
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.backup

# Create and configure photosync user
sudo useradd -m photosync
sudo usermod -aG users photosync
sudo usermod -aG sudo photosync
sudo passwd photosync

# Set directory permissions
sudo chown -R photosync:photosync /backup/saumil
sudo chown -R photosync:photosync /backup/vaishnavi
sudo chmod -R 777 /backup/saumil
sudo chmod -R 777 /backup/vaishnavi

# Set up Samba user and password
sudo smbpasswd -a photosync

# Restart Samba services
sudo systemctl restart smbd
sudo systemctl restart nmbd
```

#### Samba Configuration
File: `/etc/samba/smb.conf`
```ini
[Saumil]
   path = /backup/saumil
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

[Vaishnavi]
   path = /backup/vaishnavi
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

### iPhone Configuration
1. **PhotoSync App Installation**
   - Download PhotoSync from App Store on each iPhone
   - Log into the app
   - Configure SMB connection

2. **SMB Configuration for Saumil's iPhone**
   - Protocol: SMB (Windows Share)
   - Server: Raspberry Pi's IP address
   - Share Name: Saumil
   - Authentication:
     - Username: photosync
     - Password: (Samba password)
   - Directory Structure:
     ```
     <mediaType>/<year>/<filename>
     ```
   - Example paths:
     ```
     photos/2025/IMG_1234.jpg
     videos/2025/VID_5678.mp4
     ```
   - Automatic organization by:
     - Media type (photos/videos)
     - Year
     - Original filename

3. **SMB Configuration for Vaishnavi's iPhone**
   - Protocol: SMB (Windows Share)
   - Server: Raspberry Pi's IP address
   - Share Name: Vaishnavi
   - Authentication:
     - Username: photosync
     - Password: (Samba password)
   - Directory Structure:
     ```
     <mediaType>/<year>/<filename>
     ```
   - Example paths:
     ```
     photos/2025/IMG_1234.jpg
     videos/2025/VID_5678.mp4
     ```
   - Automatic organization by:
     - Media type (photos/videos)
     - Year
     - Original filename

4. **Special Media Handling**
   - Live Photos Configuration (for both iPhones):
     - Enabled "Live Photo MOV -> Photos Folder"
     - MOV components of Live Photos will be stored in the photos directory
     - Keeps Live Photo components together for better organization
     - Prevents MOV files from being split into the videos folder

### Web Interface (PhotoPrism)
PhotoPrism Community Edition provides a web-based interface for viewing and managing backed up photos.

1. **Access**
   - Web Interface: `http://raspberry-pi-ip:2342`
   - Mobile-friendly responsive design
   - Secure login required

2. **Key Features**
   - Photo viewing and organization
   - Face detection
   - Location mapping
   - Auto-tagging
   - Search capabilities
   - Mobile-friendly interface

3. **Directory Structure**
   ```
   /docker/photoprisim/
   ├── storage/          # PhotoPrism database and cache
   └── originals/        # Symbolic link to backup directory
       └── -> /backup    # All user directories accessible here
```

4. **Integration**
   - Direct access to existing backup structure
   - No file duplication
   - Automatic monitoring of new photos
   - Preserves original folder organization

## Project Status
🚧 Initial Development - Project setup phase

## Getting Started
*Documentation will be added as the project develops*

## Development
- IDE: JetBrains IntelliJ IDEA
- Version Control: Git

## Project Structure
```
.
└── .idea/           # IntelliJ IDEA configuration
```

## License
*To be determined*

---
For detailed development history and changes, please see [CHANGELOG.md](CHANGELOG.md)
