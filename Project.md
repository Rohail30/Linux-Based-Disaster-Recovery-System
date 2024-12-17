
#### 1. Check if SSH Is Installed
On both VM A and VM B:
```bash
sudo systemctl status sshd
```
- If it says **"active (running)"**, SSH is installed and running.
- If not installed:
  ```bash
  sudo yum install -y openssh-server
  ```

#### 2. Start and Enable SSH Service
On both VMs:
```bash
sudo systemctl start sshd
sudo systemctl enable sshd
```

#### 3. Allow SSH Through the Firewall
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

---

### **Add the Script to Crontab**
Once passwordless SSH is configured, you can safely run your script via `crontab`.

1. Open the crontab editor:
   ```bash
   crontab -e
   ```

2. Add an entry to run your script at the desired time. For example, to run the script daily at 2 AM:
   ```bash
   0 2 * * * /path/to/your/backup_script.sh
   ```

3. Save and exit the editor.

---

### **Verify Crontab Execution**
1. Check the `cron` log to ensure the script runs as scheduled:
   ```bash
   grep CRON /var/log/syslog
   ```

2. Check your `backup.log` file for successful backups:
   ```bash
   cat /home/user/SNA-Project/backup/backup.log
   ```

---

By setting up passwordless SSH, your script will execute seamlessly in `crontab` without requiring manual password entry.
