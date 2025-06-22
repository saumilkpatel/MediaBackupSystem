# Media Backup System

## Overview
An automated system that detects when a phone connects to the home WiFi network and automatically backs up new media (photos and videos) to a physical hard drive. This provides a seamless, hands-free backup solution for mobile media content.

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
  - Mounted at `/backup/saumil`
  - Mount configuration: `uid=1000,gid=1000`
  - Serves as the backup storage destination
- **iPhone**
  - Source device for media content
  - Connects to home WiFi network
  - Uses PhotoSync app for automated transfer
  - Configured with SFTP for secure file transfer

## Key Features (Planned)
- Automatic phone detection on WiFi connection
- Selective media backup (photos and videos)
- Incremental backup (only new media)
- Local storage on physical hard drive
- Zero-interaction backup process

## System Configuration
### Storage Setup
```bash
# View block devices and their mount points
lsblk

# Mount external drive with user permissions
sudo mount -o uid=1000,gid=1000 /dev/sda1 /backup/saumil
```
- Filesystem: exFAT (supports large files and is compatible with multiple operating systems)
- Mount point: `/backup/saumil`
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
```bash
# Install SSH server for remote access and file transfer
sudo apt install openssh-server
```
- OpenSSH server installed for:
  - Secure remote access
  - Secure file transfer capabilities
  - Remote administration

### iPhone Configuration
1. **PhotoSync App Installation**
   - Download PhotoSync from App Store
   - Log into the app
   - Configure SFTP connection

2. **SFTP Configuration**
   - Protocol: SFTP
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

3. **Special Media Handling**
   - Live Photos Configuration:
     - Enabled "Live Photo MOV -> Photos Folder"
     - MOV components of Live Photos will be stored in the photos directory
     - Keeps Live Photo components together for better organization
     - Prevents MOV files from being split into the videos folder

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
