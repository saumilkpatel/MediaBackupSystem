# SOP: Adding a New iPhone User

## Overview
This procedure describes how to add a new iPhone user to the Media Backup System.

## Prerequisites
- Access to Raspberry Pi
- Sudo privileges
- New user's iPhone with PhotoSync app installed
- Existing Samba configuration

## Steps

### 1. Create Backup Directory
```bash
# Create new user directory (replace NEW_USER with the user's name)
sudo mkdir -p /backup/NEW_USER

# Set directory permissions
sudo chown -R photosync:photosync /backup/NEW_USER
sudo chmod -R 777 /backup/NEW_USER
```

### 2. Add Samba Share
1. Edit Samba configuration:
```bash
sudo nano /etc/samba/smb.conf
```

2. Add new share configuration (replace NEW_USER with the user's name):
```ini
[NEW_USER]
   path = /backup/NEW_USER
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

3. Restart Samba services:
```bash
sudo systemctl restart smbd
sudo systemctl restart nmbd
```

### 3. Configure PhotoSync on New iPhone
1. Install PhotoSync app from App Store
2. Open PhotoSync settings
3. Configure new destination:
   - Protocol: SMB (Windows Share)
   - Server: Raspberry Pi's IP address
   - Share Name: NEW_USER (exactly as configured in smb.conf)
   - Authentication:
     - Username: photosync
     - Password: (Samba password)
   - Directory Structure:
     ```
     <mediaType>/<year>/<filename>
     ```
   - Enable "Live Photo MOV -> Photos Folder"

### 4. Verify Setup
1. Test Samba share access:
```bash
smbclient -L localhost -U photosync
```

2. Test directory creation:
```bash
smbclient //localhost/NEW_USER -U photosync
smb: \> mkdir test
smb: \> ls
```

3. Test PhotoSync connection:
   - Open PhotoSync on iPhone
   - Try syncing a single photo
   - Verify file appears in correct directory structure

## Troubleshooting
1. Permission Issues:
   - Verify directory ownership: `ls -la /backup/NEW_USER`
   - Verify Samba user: `sudo pdbedit -L -v | grep photosync`

2. Connection Issues:
   - Check Samba status: `sudo systemctl status smbd`
   - Verify network connectivity: `ping [Raspberry Pi IP]`

3. PhotoSync Errors:
   - Verify share name matches exactly (case-sensitive)
   - Confirm correct password
   - Check network connection

## Notes
- Keep consistent naming convention for directories and shares
- Document new user addition in system documentation
- Consider backup space requirements for new user
