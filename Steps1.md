Building a **basic disaster recovery system with backups synced to multiple machines** is an excellent way to get started while adding robustness from the beginning. Let’s break it down into manageable steps.

---

## **Basic Features Overview**
1. **Local Backup**: Back up files and directories on your local machine.
2. **Sync Backups to Remote Machines**: Automatically transfer backups to other machines for redundancy.

---

### **Step 1: Set Up the Local Environment**

#### 1. Create Required Directories
- Local data directory (data to back up):
  ```bash
  mkdir -p /home/user/test_data
  ```
- Local backup directory:
  ```bash
  mkdir -p /home/user/backup
  ```

#### 2. Add Sample Data
- Create some mock files to simulate real data:
  ```bash
  echo "File 1 content" > /home/user/test_data/file1.txt
  echo "File 2 content" > /home/user/test_data/file2.txt
  ```

---

### **Step 2: Write the Local Backup Script**

#### 1. Create a Script File
- Save the script in `/home/user/scripts/backup.sh`:
  ```bash
  mkdir -p /home/user/scripts
  nano /home/user/scripts/backup.sh
  ```

#### 2. Write the Script
Add the following lines to the file:
```bash
#!/bin/bash

# Define directories
SRC="/home/user/test_data"         # Source directory
DEST="/home/user/backup"           # Backup destination directory
DATE=$(date +'%Y-%m-%d_%H-%M-%S')  # Current timestamp for naming backups

# Step 1: Create a compressed archive of the data
tar -czvf $DEST/backup_$DATE.tar.gz $SRC

# Step 2: Log backup completion
echo "Backup completed on $(date)" >> $DEST/backup.log
```

#### 3. Make the Script Executable
```bash
chmod +x /home/user/scripts/backup.sh
```

#### 4. Test the Script
Run the script to verify it works:
```bash
/home/user/scripts/backup.sh
```

- **Check Results**:
  - The `/home/user/backup` directory should contain a `.tar.gz` file.
  - A `backup.log` file will log the completion time.

---

### **Step 3: Sync Backups to a Remote Machine**

#### 1. Set Up SSH Access
To transfer files securely, configure **passwordless SSH** access between the local and remote machines:
1. **Generate an SSH Key on the Local Machine**:
   ```bash
   ssh-keygen -t rsa -b 2048
   ```
2. **Copy the Key to the Remote Machine**:
   Replace `remote_user` and `remote_host` with the remote machine's username and IP address:
   ```bash
   ssh-copy-id remote_user@remote_host
   ```

#### 2. Update the Backup Script
Modify `/home/user/scripts/backup.sh` to include remote syncing:
```bash
#!/bin/bash

# Define directories
SRC="/home/user/test_data"
DEST="/home/user/backup"
DATE=$(date +'%Y-%m-%d_%H-%M-%S')

REMOTE_USER="remote_user"
REMOTE_HOST="remote_host"
REMOTE_DEST="/remote/backup"

# Step 1: Create a compressed archive
tar -czvf $DEST/backup_$DATE.tar.gz $SRC

# Step 2: Sync the backup to the remote machine
rsync -avz $DEST/backup_$DATE.tar.gz $REMOTE_USER@$REMOTE_HOST:$REMOTE_DEST

# Step 3: Log backup completion
echo "Backup completed and synced on $(date)" >> $DEST/backup.log
```

#### 3. Test the Script
Run the updated script:
```bash
/home/user/scripts/backup.sh
```

- **Verify Results**:
  - Check `/remote/backup` on the remote machine for the backup file.
  - Ensure the `backup.log` reflects the sync.

---

### **Step 4: Automate the Backup with Cron**

#### 1. Open Cron Editor
```bash
crontab -e
```

#### 2. Schedule the Script
Add this line to run the backup daily at 2:00 AM:
```bash
0 2 * * * /home/user/scripts/backup.sh
```

#### 3. Verify Cron Job
List scheduled jobs to ensure the cron job is active:
```bash
crontab -l
```

---

### **Step 5: Set Up Sync to a Second Remote Machine**
Repeat the **SSH Setup** for a second remote machine, then modify the script to sync to both machines.

#### Update the Backup Script
```bash
#!/bin/bash

# Define directories
SRC="/home/user/test_data"
DEST="/home/user/backup"
DATE=$(date +'%Y-%m-%d_%H-%M-%S')

REMOTE_USER1="remote_user1"
REMOTE_HOST1="remote_host1"
REMOTE_DEST1="/remote1/backup"

REMOTE_USER2="remote_user2"
REMOTE_HOST2="remote_host2"
REMOTE_DEST2="/remote2/backup"

# Step 1: Create a compressed archive
tar -czvf $DEST/backup_$DATE.tar.gz $SRC

# Step 2: Sync the backup to the first remote machine
rsync -avz $DEST/backup_$DATE.tar.gz $REMOTE_USER1@$REMOTE_HOST1:$REMOTE_DEST1

# Step 3: Sync the backup to the second remote machine
rsync -avz $DEST/backup_$DATE.tar.gz $REMOTE_USER2@$REMOTE_HOST2:$REMOTE_DEST2

# Step 4: Log backup completion
echo "Backup completed and synced to two remote locations on $(date)" >> $DEST/backup.log
```

---

### **Step 6: Restore Backup (Optional for Testing)**
To restore a backup:
1. Copy the `.tar.gz` file from the local or remote backup directory.
2. Extract the archive:
   ```bash
   tar -xzvf /path/to/backup.tar.gz -C /desired/restore/location
   ```

---

### **Summary of What You’ve Built**
1. A script that creates a local backup of a directory.
2. Automated syncing of backups to one or more remote machines.
3. Automated scheduling of backups using `cron`.

---

### **Next Steps**
- **Test Disaster Scenarios**: Simulate data loss by deleting files and restoring from backups.
- **Enhance Security**: Encrypt the backups using tools like `gpg` before syncing.
- **Add Logging and Alerts**: Email notifications about backup status or errors.

Let me know if you’d like more help at any step!
