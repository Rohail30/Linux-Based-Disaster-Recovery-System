Here are the step-by-step instructions to connect two CentOS VMs, A and B, using SSH. We'll start from the basics, assuming the VMs are freshly set up.

---

### **Step 1: Configure the Network**
#### 1. Set Networking to Enable Communication
- **For VirtualBox**:
  - Go to the settings of both VMs A and B.
  - Navigate to the **Network** tab.
  - Set the **Adapter Type**:
    - **Bridged Adapter**: Makes the VMs accessible on the same physical network.
    - **Host-Only Adapter**: If you want them to communicate only with each other.

- **For VMware**:
  - Use **NAT** or **Host-Only** mode for internal communication.

#### 2. Check the IP Address of Each VM
On both VM A and VM B, run:
```bash
ip a
```
- Note the IP address of each VM (e.g., `192.168.1.101` for A and `192.168.1.102` for B).

#### 3. Test Network Connectivity
From VM A, ping VM B:
```bash
ping <IP_of_VM_B>
```
From VM B, ping VM A:
```bash
ping <IP_of_VM_A>
```
If pings fail:
- Ensure both VMs are on the same network/subnet.
- Disable firewalls temporarily to test connectivity:
  ```bash
  sudo systemctl stop firewalld
  sudo systemctl disable firewalld
  ```

---

### **Step 2: Install and Start SSH Services**

SSH is typically pre-installed on CentOS, but confirm it on both VMs.

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

---

### **Step 3: Connect from VM A to VM B**

#### 1. SSH into VM B from VM A
On VM A, connect to VM B using its IP address:
```bash
ssh <username>@<IP_of_VM_B>
```
- Replace `<username>` with the user account on VM B.
- Example:
  ```bash
  ssh root@192.168.1.102
  ```

#### 2. Accept the SSH Key Fingerprint
When prompted, type **yes** to accept the SSH key.

#### 3. Enter the Password
Provide the password for the user on VM B.

---

### **Step 4: Set Up Passwordless SSH (Optional)**

For convenience, configure passwordless SSH from VM A to VM B.

#### 1. Generate an SSH Key on VM A
Run this on VM A:
```bash
ssh-keygen -t rsa -b 2048
```
- Press **Enter** to accept the default file location (`~/.ssh/id_rsa`).
- Leave the passphrase blank by pressing **Enter** twice.

#### 2. Copy the Public Key to VM B
From VM A, copy the SSH key to VM B:
```bash
ssh-copy-id <username>@<IP_of_VM_B>
```
- Replace `<username>` and `<IP_of_VM_B>` with the appropriate values.
- Enter the password for the user on VM B when prompted.

#### 3. Test Passwordless SSH
Try connecting from VM A to VM B again:
```bash
ssh <username>@<IP_of_VM_B>
```
This time, it should not ask for a password.

---

### **Step 5: Reverse the Connection (VM B to VM A)**

To connect from VM B to VM A:
- Repeat **Step 3** and **Step 4** on VM B to establish an SSH connection to VM A.

---

### **Troubleshooting**

1. **SSH Connection Refused**:
   - Ensure the SSH service is running:
     ```bash
     sudo systemctl status sshd
     ```
   - Confirm the port (default: 22) is open:
     ```bash
     sudo netstat -tuln | grep 22
     ```

2. **Ping Fails**:
   - Ensure both VMs are on the same network/subnet.
   - Temporarily disable firewalls:
     ```bash
     sudo systemctl stop firewalld
     ```

3. **Key-Based Authentication Fails**:
   - Verify the key is copied to `~/.ssh/authorized_keys` on VM B:
     ```bash
     cat ~/.ssh/authorized_keys
     ```

---

Let me know if you encounter any issues!
