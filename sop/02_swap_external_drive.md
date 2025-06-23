# SOP: Swapping External Hard Drive

## Overview
This procedure describes how to safely replace the external hard drive in the Media Backup System.

## Prerequisites
- New external hard drive
- Sudo privileges
- Backup of critical data (if replacing a failing drive)

## Steps

### 1. Prepare for Drive Swap
1. Stop Samba services:
```bash
sudo systemctl stop smbd
sudo systemctl stop nmbd
```

2. Change directory to home to ensure we're not in the mount point:
```bash
cd ~
```

3. Unmount existing drive:
```bash
# Check if drive is busy
sudo lsof /backup

# Force close any processes using the mount
sudo fuser -km /backup

# Unmount the drive
sudo umount -f /backup
```

### 2. Physical Drive Swap
1. Power down Raspberry Pi:
```bash
sudo shutdown -h now
```

2. Once powered off:
   - Disconnect old drive
   - Connect new drive
   - Power on Raspberry Pi
   - Wait for system to boot

### 3. Prepare New Drive
1. Identify new drive:
```bash
lsblk
```

2. Format drive with exFAT (replace sdX with correct drive letter):
```bash
sudo mkfs.exfat /dev/sdX1
```

### 4. Mount New Drive
1. Create mount point if it doesn't exist:
```bash
sudo mkdir -p /backup
```

2. Mount the drive:
```bash
sudo mount -o uid=1000,gid=1000 /dev/sdX1 /backup
```

### 5. Create User Directories
```bash
# Create user directories
sudo mkdir -p /backup/saumil
sudo mkdir -p /backup/vaishnavi

# Set permissions
sudo chown -R photosync:photosync /backup/saumil
sudo chown -R photosync:photosync /backup/vaishnavi
sudo chmod -R 777 /backup/saumil
sudo chmod -R 777 /backup/vaishnavi
```

### 6. Restart Services
```bash
sudo systemctl start smbd
sudo systemctl start nmbd
```

### 7. Verify Setup
1. Check mount:
```bash
df -h /backup
```

2. Verify Samba shares:
```bash
smbclient -L localhost -U photosync
```

3. Test directory creation:
```bash
smbclient //localhost/Saumil -U photosync
smb: \> mkdir test
smb: \> ls
```

4. Test PhotoSync connections:
   - Try syncing a single photo from each iPhone
   - Verify files appear in correct directories

## Optional: Make Mount Permanent
Add to /etc/fstab (replace sdX with correct drive letter):
```bash
sudo nano /etc/fstab
```
Add line:
```
/dev/sdX1  /backup  exfat  defaults,uid=1000,gid=1000  0  0
```

## Troubleshooting
1. Drive Not Detected:
   - Check USB connection
   - Verify drive powers up
   - Check `dmesg` for USB errors

2. Mount Issues:
   - Verify partition exists: `sudo fdisk -l`
   - Check filesystem: `sudo fsck.exfat /dev/sdX1`

3. Permission Problems:
   - Verify mount options: `mount | grep backup`
   - Check directory ownership: `ls -la /backup`

## Notes
- Always verify data integrity before removing old drive
- Consider implementing regular SMART monitoring for drive health
- Document new drive details (size, model, serial number)
- Consider periodic backup verification procedures
