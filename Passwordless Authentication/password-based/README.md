# SSH Password-Based Authentication between Control Node and Manage Node on AWS

This guide walks you through the process of setting up **password-based SSH authentication** between a **Control Node** and a **Manage Node** using **Ubuntu EC2 instances** in AWS.

---

## Prerequisites

Before you begin, ensure the following:

- You have access to **two EC2 instances** in AWS running **Ubuntu AMI**:
  - **Control Node**: The local machine from which you will SSH.
  - **Manage Node**: The remote machine you want to SSH into.

- Both EC2 instances should have SSH access enabled (port 22 open) and you should have the **private key** for both instances.

---

## Steps to Configure Password-Based SSH Authentication

---

### **Step 1: Create Two EC2 Instances on AWS (Control Node and Manage Node)**

#### 1. **Log into AWS Console**:
   - Navigate to the [AWS Management Console](https://aws.amazon.com/console/).
   - Open the **EC2 Dashboard**.

#### 2. **Launch the Control Node EC2 Instance**:
   - Click **Launch Instance**.
   - Choose an **Ubuntu AMI** (e.g., `Ubuntu Server 20.04 LTS`).
   - Select an instance type (e.g., `t2.micro` for the free tier).
   - Configure the instance (network, security groups, etc.).
   - **Create or select an existing key pair** for SSH access.
   - **Allow port 22 (SSH)** in the Security Group to ensure remote SSH access.
   - Click **Launch**.

#### 3. **Launch the Manage Node EC2 Instance**:
   - Repeat the same steps to launch another EC2 instance for the **Manage Node**. You can use the same or a different Ubuntu AMI.
   - Ensure **SSH access (port 22)** is enabled in the security group and a key pair is selected.
   - **Note down the public IP addresses** for both the Control Node and Manage Node.

---

### **Step 2: Enable Password Authentication on the Manage Node**

By default, many EC2 instances disable password authentication for SSH. To enable password authentication, follow these steps:

1. **Log into the Manage Node**:
   - SSH into the **Manage Node** using the key pair (replace `manage-node-ip` with the actual IP address of your Manage Node):
     ```bash
     ssh -i /path/to/manage-node-key.pem ubuntu@manage-node-ip
     ```

2. **Edit the SSH configuration to enable password authentication**:
   - Open the SSH configuration file in a text editor:
     ```bash
     sudo vim /etc/ssh/sshd_config
     ```

   - Find and modify (or add) the following line to ensure password authentication is enabled:
     ```bash
     PasswordAuthentication yes
     ```

   - Save and exit the editor.

3. **Restart the SSH service** to apply the changes:
   ```bash
   sudo systemctl restart ssh
   ```

---

### **Step 3: Set the Password for the `ubuntu` User on the Manage Node**

To use password-based SSH authentication, you need to set or reset the password for the `ubuntu` user (or whichever user you want to use for SSH login).

1. **Set the password for the `ubuntu` user**:
   ```bash
   sudo passwd ubuntu
   ```

   You will be prompted to enter and confirm a new password.

---

### **Step 4: Test Password Authentication from the Control Node to the Manage Node**

1. **Log into the Control Node** (the machine from which you'll be SSHing into the Manage Node):
   ```bash
   ssh -i /path/to/control-node-key.pem ubuntu@control-node-ip
   ```

2. **Test SSH access to the Manage Node using the `ubuntu` user's password**:
   ```bash
   ssh ubuntu@manage-node-ip
   ```

   At this point, you will be **prompted for a password** like this:

   ```
   ubuntu@manage-node-ip's password:
   ```

3. **Enter the password** that you set earlier for the `ubuntu` user on the Manage Node. You will **not see the characters** as you type, but once you enter the correct password and hit **Enter**, you will be logged into the **Manage Node**.

   **Note**: If you enter the password incorrectly, the system will prompt you again for the correct password. Make sure to enter the exact password you set.

---

### **Step 5: Troubleshooting**

If you encounter issues, here are a few things to check:

1. **Firewall and Security Groups**:
   - Ensure that the security groups associated with both EC2 instances allow inbound SSH (port 22) traffic.
   - If using `ufw` or any firewall on the server, ensure that port 22 is open.

2. **SSH Configuration**:
   - Ensure that the `PasswordAuthentication yes` directive is properly set in `/etc/ssh/sshd_config` on the Manage Node.
   - Ensure that there are no conflicting settings in `sshd_config` that may block password authentication.

3. **Permissions**:
   - Ensure the correct permissions for the SSH configuration files:
     ```bash
     sudo chmod 600 /etc/ssh/sshd_config
     sudo systemctl restart ssh
     ```

4. **Restarting SSH**:
   - If the changes do not take effect immediately, restart the SSH service on the Manage Node:
     ```bash
     sudo systemctl restart ssh
     ```

---

### **Additional Notes**

- Password authentication is less secure than SSH key-based authentication. It is recommended to use SSH keys for authentication whenever possible.
- If you want to configure password authentication for other users, repeat the steps for each user, setting the password and modifying the SSH configuration as necessary.

---

Once this process is complete, you will be able to SSH into the **Manage Node** from the **Control Node** using password authentication.

---

### **Summary of Commands**

#### Manage Node
```bash
# Log into the manage node
ssh -i /path/to/manage-node-key.pem ubuntu@manage-node-ip

# Enable password authentication in SSH configuration
sudo vim /etc/ssh/sshd_config
# Ensure PasswordAuthentication yes is set
sudo systemctl restart ssh

# Set the password for the `ubuntu` user
sudo passwd ubuntu
```

#### Control Node
```bash
# Log into the control node
ssh -i /path/to/control-node-key.pem ubuntu@control-node-ip

# Test SSH login to the manage node
ssh ubuntu@manage-node-ip
```

---

This guide provides a clear and structured method for setting up **password-based SSH authentication** on AWS EC2 instances running Ubuntu. Let me know if you need any further assistance!

---

### **Summary of Behavior**

1. **On the Control Node**: After running `ssh ubuntu@manage-node-ip`, the terminal will prompt you for the password:
   
   ```bash
   ubuntu@manage-node-ip's password:
   ```

2. **Enter the Password**: Type the password you set for the `ubuntu` user on the **Manage Node** (the characters won't be visible as you type) and press **Enter**. Once the correct password is entered, you'll be logged into the Manage Node.

---
