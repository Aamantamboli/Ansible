### To configure passwordless SSH authentication (SSH key-based authentication) between your control node and manage node in AWS using **Ubuntu-based EC2 instances**, follow these steps:

---

### Prerequisites:
- You need to create **two EC2 instances** on AWS using an **Ubuntu AMI**: one will act as the **control node** and the other as the **manage node**.
- You should have access to both EC2 instances using SSH and can log in with a password or key-based authentication.

---

### Steps to configure passwordless SSH authentication:

---

### **Step 1: Create Two EC2 Instances on AWS (Control Node and Manage Node)**

#### 1. **Log into AWS Console**:
   - Navigate to the [AWS Management Console](https://aws.amazon.com/console/).
   - Open the **EC2 Dashboard**.

#### 2. **Launch the Control Node EC2 Instance**:
   - Click on **Launch Instance**.
   - In the **Choose an Amazon Machine Image (AMI)** step, search for and select an **Ubuntu AMI** (e.g., `Ubuntu Server 20.04 LTS`).
   - Choose an instance type (e.g., `t2.micro` for the free tier).
   - Configure the instance (network, security settings).
   - Create or select a key pair for SSH access.
   - Open **port 22 (SSH)** in the Security Group settings to allow remote SSH access.
   - Click **Launch** to create the control node EC2 instance.

#### 3. **Launch the Manage Node EC2 Instance**:
   - Repeat the same steps to launch another EC2 instance for the **manage node**, using the same or a different Ubuntu AMI.
   - Ensure SSH access is allowed (port 22) and a key pair is selected.
   - Note the **Public IP addresses** for both the control node and manage node. You'll use these IPs to SSH into the instances.

---

### **Step 2: Generate SSH Key Pair on the Control Node**

1. **Log into your control node** (the node from which you will connect to the manage node):
   ```bash
   ssh -i /path/to/control-node-key.pem ubuntu@control-node-ip
   ```

2. **Check if you already have an SSH key pair**:
   ```bash
   ls ~/.ssh
   ```

   If there is an existing key pair (e.g., `id_rsa`), you can use that. If not, generate a new SSH key pair.

3. **Generate a new SSH key pair** (if needed):
   ```bash
   ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa
   ```

   - `-t rsa`: Specifies the key type (RSA).
   - `-b 2048`: Specifies the key length (2048 bits).
   - `-f ~/.ssh/id_rsa`: Specifies the file name for the key (it will create `id_rsa` and `id_rsa.pub`).

   Press `Enter` to confirm the file location and skip the passphrase (or provide one if you prefer).

4. **Check the generated keys**:
   ```bash
   ls ~/.ssh
   ```

   You should see two files: `id_rsa` (private key) and `id_rsa.pub` (public key).

---

### **Step 3: Copy the Public Key to the Manage Node**

5. **Copy the public key to the manage node** using `ssh-copy-id` or manually via `scp`. We'll cover both methods:

#### Method 1: Using `ssh-copy-id`

   If you're using `ssh-copy-id` (a simpler method), use the following command:
   ```bash
   ssh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@manage-node-ip
   ```

   Replace `manage-node-ip` with the actual IP address of your manage node. This command will copy the public key to the `~/.ssh/authorized_keys` file on the manage node.

   - If `ssh-copy-id` is not installed, you can manually copy the key using the next method.

#### Method 2: Manually Copy the Public Key

1. **Log into the manage node**:
   ```bash
   ssh -i /path/to/manage-node-key.pem ubuntu@manage-node-ip
   ```

2. **Create the `~/.ssh` directory if it doesn't exist**:
   ```bash
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   ```

3. **Copy the public key manually**:
   - On the control node, copy the content of your public key:
     ```bash
     cat ~/.ssh/id_rsa.pub
     ```

   - On the manage node, paste the public key into the `~/.ssh/authorized_keys` file:
     ```bash
     echo "your-public-key-content" >> ~/.ssh/authorized_keys
     ```

   - Ensure correct permissions for the `authorized_keys` file:
     ```bash
     chmod 600 ~/.ssh/authorized_keys
     ```

4. **Verify the SSH config**:
   - Ensure that the SSH daemon is configured to allow key-based authentication by editing `/etc/ssh/sshd_config` (on the manage node):
     ```bash
     sudo vim /etc/ssh/sshd_config
     ```

   - Ensure the following settings are present (uncomment and modify if necessary):
     ```bash
     PubkeyAuthentication yes
     AuthorizedKeysFile .ssh/authorized_keys
     ```

   - After making any changes, restart the SSH service:
     ```bash
     sudo systemctl restart ssh
     ```

---

### **Step 4: Test Passwordless Authentication**

5. **Test SSH access from the control node to the manage node**:
   ```bash
   ssh ubuntu@manage-node-ip
   ```

   You should be able to SSH into the manage node without entering a password. If it works, then passwordless SSH authentication is successfully configured.

---

### **Step 5: (Optional) Troubleshooting**

- If you encounter any issues, check the following:
  - **File permissions**: Make sure the `~/.ssh` directory has `700` permissions and the `authorized_keys` file has `600` permissions.
  - **SSH configuration**: Ensure that `PubkeyAuthentication yes` is enabled in the `sshd_config` file on the manage node.
  - **SSH agent**: If you're using an SSH agent to manage keys, ensure it's properly configured to use the correct key.
  - **Security groups**: Ensure that the security groups associated with both EC2 instances allow inbound SSH (port 22) traffic.

---

### **Additional Notes**:
- If you want to enable this for multiple users, repeat the steps for each user, copying their public keys to the appropriate `authorized_keys` file on the manage node.
- Make sure your private key (`id_rsa`) is kept secure. Never share it or expose it publicly.

---

Once this is done, the control node can SSH into the manage node without requiring a password.

