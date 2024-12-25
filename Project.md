# Linux-Based-Disaster-Recovery-System

## Project Introduction
This Project is a comprehensive and automated solution designed to simplify and secure the management of backup and restoration processes for critical data. This project ensures data integrity, efficient storage utilization, and seamless recovery mechanisms. It is structured into distinct components that work together to automate backups, manage retention policies, and facilitate both local and remote restoration of data.

### 1. Project requirements

#### Tools and Technologies

1. **Shell Scripting** - For automating backup, cleanup, and restoration processes.  
2. **`tar`** - Used for archiving files into a single package.  
3. **`gzip`** - Compresses backup files to save storage space.  
4. **`rsync`** - Facilitates secure and efficient file transfers, especially for remote backups.  
5. **`scp`** - Provides a simple method for secure file transfer to remote servers.  
6. **`sha256sum`** - Ensures data integrity by generating and verifying file checksums.  
7. **Cron Jobs** - Schedules automated tasks for backup creation and cleanup.  
8. **Linux Command Line Utilities** - For file and directory management (`ls`, `rm`, etc.).
9. **Virtual Machines (VMs)** - Two VMs are used for simulating local and remote environments, ensuring robust testing of backup and restoration processes.

#### Project Directory Structure of Local Machine

```
SNA-Project/
├── backup/           # Stores backup files and logs
│   ├── backup_<timestamp>.tar.gz
│   ├── backup.log
├── data/             # Contains source data to be backed up
│   ├── 2012362/
│   ├── 2112242/
├── restored_data/    # Stores restored backup files
│   ├── backup_<timestamp>.tar.gz
│   ├── home/
├── scripts/          # Automation scripts for backup and restore
    ├── backup.sh          # Creates backups
    ├── delete-backup.sh   # Removes older backups
    ├── remote-backup.sh   # Transfers backups to a remote server
    ├── remote-restore.sh  # Restores backups from a remote server
    ├── restore.sh         # Restores local backups

```

#### Project Directory Structure of Remote Machine

```
SNA-Project/
├── backup/           # Stores backup files
│   ├── backup_<timestamp>.tar.gz
├── scripts/          # Automation scripts for cleanup
    ├── delete.sh          # Deletes older backups
```


#### Scripts Used in This Project

### **1. `backup.sh`**
```bash
#!/bin/bash

# Directories
RESTORE_DIR="/home/user/SNA-Project/restored_data"  # Directory to restore data
BACKUP_FILE="$1"                                   # Backup file to restore
CHECKSUM_FILE="${BACKUP_FILE}.sha256"              # Corresponding checksum file

# Ensure the backup file and checksum file exist
if [ ! -f "$BACKUP_FILE" ] || [ ! -f "$CHECKSUM_FILE" ]; then
    echo "Error: Backup file or checksum file missing."
    exit 1
fi

# Validate the backup file
if ! sha256sum -c "$CHECKSUM_FILE"; then
    echo "Error: Checksum validation failed."
    exit 1
fi

# Create the restore directory if it doesn't exist
mkdir -p "$RESTORE_DIR"

# Extract the backup
tar -xzf "$BACKUP_FILE" -C "$RESTORE_DIR"

# Completion message
echo "Backup restored successfully to $RESTORE_DIR."

```

---

### **2. `delete-backup.sh`**
```bash
#!/bin/bash

# Define backup directory
BACKUP_DIR="/home/user/SNA-Project/backup"

# Retain the 3 most recent backups, delete older ones
ls -t "$BACKUP_DIR"/*.tar.gz | tail -n +4 | xargs -d '\n' -I {} rm -f {}

# Log the cleanup
echo "$(date): Deleted older backups, retaining 3 most recent." >> "$BACKUP_DIR/backup.log"
```

---

### **3. `remote-backup.sh`**
```bash
#!/bin/bash

# Define local and remote directories
BACKUP_DIR="/home/user/SNA-Project/backup"
REMOTE_USER="server"
REMOTE_HOST="192.168.1.100"
REMOTE_DIR="/home/server/SNA-Project/backup"

# Sync backups to remote server
rsync -avz "$BACKUP_DIR/" "$REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR"

# Log the transfer
echo "$(date): Backups synced to remote server $REMOTE_HOST" >> "$BACKUP_DIR/backup.log"
```

---

### **4. `remote-restore.sh`**
```bash
#!/bin/bash

# Define local and remote directories
RESTORED_DIR="/home/user/SNA-Project/restored_data"
REMOTE_USER="server"
REMOTE_HOST="192.168.1.100"
REMOTE_DIR="/home/server/SNA-Project/backup"

# Fetch the latest backup from remote server
LATEST_BACKUP=$(ssh "$REMOTE_USER@$REMOTE_HOST" "ls -t $REMOTE_DIR | head -n 1")
scp "$REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/$LATEST_BACKUP" "$RESTORED_DIR/"

# Log the restore operation
echo "$(date): Restored backup $LATEST_BACKUP from remote server $REMOTE_HOST" >> "$RESTORED_DIR/restore.log"
```

---

### **5. `restore.sh`**
```bash
#!/bin/bash

# Define directories
BACKUP_DIR="/home/user/SNA-Project/backup"
DATA_DIR="/home/user/SNA-Project/data"

# Check if the backup file and checksum file are provided as arguments
if [ $# -ne 1 ]; then
    echo "Usage: $0 <backup_file>"
    exit 1
fi

BACKUP_FILE="$1"
CHECKSUM_FILE="$BACKUP_DIR/$(basename "$BACKUP_FILE" .tar.gz).sha256"

# Check if the backup file exists
if [ ! -f "$BACKUP_FILE" ]; then
    echo "Backup file does not exist: $BACKUP_FILE"
    exit 1
fi

# Check if the checksum file exists
if [ ! -f "$CHECKSUM_FILE" ]; then
    echo "Checksum file does not exist: $CHECKSUM_FILE"
    exit 1
fi

# Verify the checksum
echo "Verifying checksum for $BACKUP_FILE..."
sha256sum -c "$CHECKSUM_FILE" --quiet
if [ $? -ne 0 ]; then
    echo "Checksum verification failed! The backup may be corrupted."
    exit 1
fi

# Extract the backup
echo "Restoring backup from $BACKUP_FILE..."
tar -xzf "$BACKUP_FILE" -C "$DATA_DIR"

# Log the restoration
echo "$(date): Backup restored: $BACKUP_FILE" >> "$BACKUP_DIR/backup.log"

echo "Restore completed successfully."

```

---

### **6. `delete.sh` (on Remote Server)**
```bash
#!/bin/bash

# Define backup directory on remote server
BACKUP_DIR="/home/server/SNA-Project/backup"

# Retain the 3 most recent backups, delete older ones
ls -t "$BACKUP_DIR"/*.tar.gz | tail -n +4 | xargs -d '\n' -I {} rm -f {}

# Log the cleanup
echo "$(date): Deleted older backups, retaining 3 most recent on remote server." >> "$BACKUP_DIR/backup.log"
```

---

### **How These Scripts Work Together**
1. **`backup.sh`**: Creates local backups.
2. **`delete-backup.sh`**: Deletes older local backups.
3. **`remote-backup.sh`**: Syncs backups to the remote server.
4. **`delete.sh`**: Manages remote server backup retention.
5. **`restore.sh`**: Restores backups locally.
6. **`remote-restore.sh`**: Fetches and restores backups from the remote server.

## 2. Connect to Server

#### Check if SSH is installed
On both VM A and VM B:
```bash
sudo systemctl status sshd
```
- If it says **"active (running)"**, SSH is installed and running.
- If not installed:
  ```bash
  sudo yum install -y openssh-server
  ```

#### Start and Enable SSH Service
On both VMs:
```bash
sudo systemctl start sshd
sudo systemctl enable sshd
```

#### Allow SSH Through the Firewall
If the firewall is running, allow SSH traffic on both VMs:
```bash
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --reload
```

To run your backup script in `crontab` without being prompted for a password, you need to set up **passwordless SSH authentication** between your local machine and the remote server. Here's how to do it:

---

### **Steps to Enable Passwordless SSH**
1. **Generate an SSH Key Pair**:
   - On your local machine, generate an SSH key pair (if you don't already have one):
     ```bash
     ssh-keygen -t rsa
     ```
   - When prompted, press **Enter** to use the default file location (e.g., `~/.ssh/id_rsa`) and leave the passphrase empty for passwordless authentication.

2. **Copy the Public Key to the Remote Server**:
   - Use the `ssh-copy-id` command to copy your public key to the remote server:
     ```bash
     ssh-copy-id server@192.168.59.134
     ```
   - Enter your password when prompted. This adds your public key to the `~/.ssh/authorized_keys` file on the remote server.

   **If `ssh-copy-id` is unavailable**, you can manually copy the public key:
   - Display your public key:
     ```bash
     cat ~/.ssh/id_rsa.pub
     ```
   - Log in to the remote server and append the public key to the `~/.ssh/authorized_keys` file:
     ```bash
     echo "your-public-key-content" >> ~/.ssh/authorized_keys
     ```
   - Set proper permissions:
     ```bash
     chmod 600 ~/.ssh/authorized_keys
     chmod 700 ~/.ssh
     ```

3. **Test Passwordless SSH**:
   - Verify that you can log in to the remote server without a password:
     ```bash
     ssh server@192.168.59.134
     ```
