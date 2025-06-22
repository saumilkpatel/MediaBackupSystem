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

### Development Log

#### 2025-06-22
- iPhone Integration Setup:
  - Installed PhotoSync app from App Store
  - Configured SFTP connection in PhotoSync
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
- System Configuration:
  - Confirmed pre-installed Linux OS on Raspberry Pi 5
  - Established ethernet connection for network access
- Storage Configuration:
  - Formatted external drive with exFAT filesystem for cross-platform compatibility
  - Created mount point at `/backup/saumil`
  - Configured drive mounting with user permissions using command:
    ```bash
    sudo mount -o uid=1000,gid=1000 /dev/sda1 /backup/saumil
    ```
  - Mount options set for proper user access (uid=1000,gid=1000)
- Hardware Architecture Decisions:
  - Selected Raspberry Pi 5 as the system controller
  - Planned ethernet connection to router for stable network access
  - Decided on USB-connected external hard drive for storage
  - Identified iPhone as the source device for media content
- Project Vision Defined: Automated media backup system that triggers when phone connects to home WiFi
- Core Features Identified:
  - WiFi-based phone detection
  - Automatic media (photos/videos) backup
  - Incremental backup support
  - Local hard drive storage
  - Zero-interaction design goal
- Project initialized
- User requested to start a README file and maintain a detailed log of all activities
- Created initial README.md with project structure and vision
- Set up CHANGELOG.md for tracking changes and conversations
- Updated README.md with planned features and detailed project overview
- Added system architecture section to README.md with hardware components
- Added system configuration section with storage setup details
- Added network services section with SSH server configuration
- Added storage management commands and documentation
- Added iPhone configuration details with PhotoSync setup
- Added Live Photos handling configuration details
