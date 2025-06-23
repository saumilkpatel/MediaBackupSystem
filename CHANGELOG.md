# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial project setup with IntelliJ IDEA
- Created README.md with basic project structure
- Created CHANGELOG.md for tracking project history
- Defined project vision and core features
- Specified hardware architecture and components
- Configured external drive mounting with exFAT filesystem
- Installed OpenSSH server for remote access and file transfer
- Added drive management commands and documentation
- Configured iPhone PhotoSync app with SFTP setup
- Configured Live Photos handling in PhotoSync
- Migrated from SFTP to SMB for improved transfer speeds
- Added Samba configuration for file sharing
- Added multi-user support with separate backup directories

### Changed
- Switched file transfer protocol from SFTP to SMB for better performance
- Updated PhotoSync configuration to use SMB instead of SFTP
- Modified Samba configuration to use 'photosync' user instead of 'saumil'
- Changed mount point structure to support multiple users
- Reorganized backup directories for multiple users (Saumil and Vaishnavi)

### Development Log

#### Next Steps
- Swap in new hard drive when delivered

#### 2025-06-22
- Multi-User Support:
  - Reorganized mount structure to support multiple users
  - Created separate backup directories:
    - /backup/saumil for Saumil's media
    - /backup/vaishnavi for Vaishnavi's media
  - Updated Samba configuration with separate shares
  - Configured permissions for both backup directories
- Permission Configuration:
  - Created system user 'photosync'
  - Added 'photosync' to users group
  - Added 'photosync' to sudo group for administrative access
  - Set proper ownership of backup directories
  - Updated Samba share permissions
  - Fixed STATUS_ACCESS_DENIED error with proper permissions
- Performance Optimization:
  - Identified SFTP performance bottleneck
  - Researched alternative transfer protocols
  - Selected SMB for improved transfer speeds
  - Installed and configured Samba:
    ```bash
    sudo apt install samba samba-common-bin
    sudo apt install smbclient  # For testing and verification
    ```
  - Created Samba share configurations for both users
  - Set up Samba user authentication with username 'photosync'
  - Updated PhotoSync to use SMB protocol
  - Fixed permission issues:
    ```bash
    sudo useradd -m photosync
    sudo usermod -aG users photosync
    sudo chown -R photosync:photosync /backup/saumil
    sudo chown -R photosync:photosync /backup/vaishnavi
    sudo chmod -R 777 /backup/saumil
    sudo chmod -R 777 /backup/vaishnavi
    ```
- iPhone Integration Setup:
  - Installed PhotoSync app from App Store
  - Configured SMB connection in PhotoSync
  - Set up automatic directory structure:
    ```
    <mediaType>/<year>/<filename>
    ```
  - Enabled automatic media organization by type and year
  - Configured Live Photos handling:
    - Enabled "Live Photo MOV -> Photos Folder" option
    - Ensures Live Photo components stay together in photos directory
- Storage Management:
  - Used `lsblk` command to:
    - Verify external drive detection
    - Confirm mount point location
    - View drive partition information
- Network Services Setup:
  - Installed OpenSSH server using command:
    ```bash
    sudo apt install openssh-server
    ```
  - Enabled secure remote access and file transfer capabilities
  - Installed and configured Samba for high-speed file sharing
  - Created dedicated 'photosync' Samba user for file transfers
- System Configuration:
  - Confirmed pre-installed Linux OS on Raspberry Pi 5
  - Established ethernet connection for network access
- Storage Configuration:
  - Formatted external drive with exFAT filesystem for cross-platform compatibility
  - Created mount point at `/backup`
  - Created user-specific directories:
    - `/backup/saumil`
    - `/backup/vaishnavi`
  - Configured drive mounting with user permissions using command:
    ```bash
    sudo mount -o uid=1000,gid=1000 /dev/sda1 /backup
    ```
  - Mount options set for proper user access (uid=1000,gid=1000)
- Hardware Architecture Decisions:
  - Selected Raspberry Pi 5 as the system controller
  - Planned ethernet connection to router for stable network access
  - Decided on USB-connected external hard drive for storage
  - Identified iPhones as source devices for media content
- Project Vision Defined: Automated media backup system that triggers when phones connect to home WiFi
- Core Features Identified:
  - WiFi-based phone detection
  - Automatic media (photos/videos) backup
  - Incremental backup support
  - Local storage on physical hard drive
  - Zero-interaction design goal
  - High-speed file transfer using SMB
  - Multi-user support with separate backup spaces
- Project initialized
- User requested to start a README file and maintain a detailed log of all activities
- Created initial README.md with project structure and vision
- Set up CHANGELOG.md for tracking changes and conversations
- Updated README.md with planned features and detailed project overview
- Added system architecture section to README.md with hardware components
- Added system configuration section with storage setup details
- Added network services section with SSH and Samba configuration
- Added storage management commands and documentation
- Added iPhone configuration details with PhotoSync setup
- Added Live Photos handling configuration details
- Added multi-user support documentation
