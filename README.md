# GitHub Actions - EC2 Management Workflows
This repository demonstrates how to use GitHub Actions to manage and monitor AWS EC2 instances with three automated workflows.

## 🎯 What This Does

This project includes three workflows:

### Workflow 1: EC2 Connectivity Check (Automatic)
Runs automatically on every push to main branch:
- Tests SSH connectivity to your EC2 instance
- Securely connects using SSH key authentication
- Extracts and displays EC2 metadata including:
  - Public IP address
  - Private IP address
  - Hostname
  - Instance ID
  - Availability Zone
  - Instance Type
  - System uptime

### Workflow 2: Fix The Hostname (Manual)
Manual execution to manage EC2 hostname:
- Checks current hostname on EC2 instance
- Sets hostname to "HelloWorld" if it's different
- Updates configuration for persistence across reboots
- Verifies the hostname change was successful
- Skips if hostname is already correct

### Workflow 3: Execute Metadata Script (Manual)
Manual execution to run custom script on EC2:
- Connects to EC2 instance via SSH
- Checks if `/home/ubuntu/metadata.sh` exists
- Executes the script and displays output
- Shows results directly in GitHub Actions console

## 🚀 Quick Start Guide

### Prerequisites

1. An AWS EC2 instance with SSH access enabled
2. A GitHub account
3. SSH key pair for EC2 access

### Step 1: Initial Setup on Your Local Machine

Navigate to your project directory:
```bash
cd "c:\CLOUD\OneDrive - Hogarth Worldwide\Documents\Ostad\Batch-08\GiT\github-actions-hello-world"
```

### Step 2: Initialize Git Repository

```bash
# Initialize git repository
git init

# Add all files
git add .

# Create initial commit
git commit -m "Initial commit: EC2 connectivity check workflow"
```

### Step 3: Create GitHub Repository

1. Go to [GitHub](https://github.com) and log in
2. Click the **"+"** icon in the top-right corner
3. Select **"New repository"**
4. Fill in the details:
   - **Repository name**: `github-actions-hello-world` (or your preferred name)
   - **Description**: "EC2 connectivity check using GitHub Actions"
   - **Visibility**: Choose Public or Private
   - **DO NOT** initialize with README, .gitignore, or license (we already have files)
5. Click **"Create repository"**

### Step 4: Link and Push to GitHub

GitHub will show you commands to push an existing repository. Use these:

```bash
# Add GitHub remote (replace YOUR_USERNAME with your GitHub username)
git remote add origin https://github.com/YOUR_USERNAME/github-actions-hello-world.git

# Rename branch to main (if needed)
git branch -M main

# Push to GitHub
git push -u origin main
```

### Step 5: Configure GitHub Secrets

This is the most important step! Your workflow needs three secrets to connect to your EC2 instance.

1. Go to your repository on GitHub
2. Click **Settings** (top navigation bar)
3. In the left sidebar, click **Secrets and variables** → **Actions**
4. Click **"New repository secret"** button
5. Add the following three secrets:

#### Secret 1: EC2_HOST
- **Name**: `EC2_HOST`
- **Value**: Your EC2 instance's public IP address (e.g., `54.123.45.67`)
- Click **"Add secret"**

#### Secret 2: EC2_USER
- **Name**: `EC2_USER`
- **Value**: Your SSH username (e.g., `ubuntu`, `ec2-user`, `admin`)
- Click **"Add secret"**

#### Secret 3: EC2_SSH_KEY
- **Name**: `EC2_SSH_KEY`
- **Value**: Your complete private SSH key
- To get your private key content:
  ```bash
  # On Windows (PowerShell)
  Get-Content "path\to\your\key.pem"
  
  # Copy the entire output including:
  # -----BEGIN RSA PRIVATE KEY-----
  # (all the key content)
  # -----END RSA PRIVATE KEY-----
  ```
- Paste the entire key content (including BEGIN and END lines)
- Click **"Add secret"**

## 🔄 How It Works

### Workflow 1: EC2 Connectivity Check

**Trigger:** Automatic on every push to `main` branch
```yaml
on:
  push:
    branches:
      - main
```

**Workflow Steps:**
1. **Checkout Repository**: Gets the latest code from your repo
2. **Setup SSH Key**: 
   - Creates SSH directory structure
   - Securely writes the SSH key from secrets
   - Sets proper permissions (600)
   - Adds EC2 host to known_hosts
3. **Test EC2 Connectivity**: Performs a simple SSH connection test
4. **Extract EC2 Metadata**: 
   - Connects to EC2 instance
   - Uses AWS IMDSv2 (Instance Metadata Service v2) to fetch instance details
   - Displays comprehensive system information
5. **Cleanup SSH Key**: Removes the SSH key file for security (always runs)

### Workflow 2: Fix The Hostname

**Trigger:** Manual execution only via GitHub UI
```yaml
on:
  workflow_dispatch:
```

**Workflow Steps:**
1. **Checkout Repository**: Gets the latest code
2. **Setup SSH Key**: Configures secure SSH connection
3. **Check Current Hostname**: Reads and displays current hostname
4. **Set Hostname to HelloWorld** (conditional): 
   - Only runs if hostname is NOT already "HelloWorld"
   - Uses `hostnamectl` to set hostname
   - Updates `/etc/hostname` for persistence
   - Updates `/etc/hosts` for proper resolution
5. **Verify Hostname Change**: Confirms the change was successful
6. **Cleanup SSH Key**: Removes SSH key for security

### Workflow 3: Execute Metadata Script

**Trigger:** Manual execution only via GitHub UI
```yaml
on:
  workflow_dispatch:
```

**Workflow Steps:**
1. **Checkout Repository**: Gets the latest code
2. **Setup SSH Key**: Configures secure SSH connection
3. **Check if metadata.sh exists**: Verifies the script file is present
4. **Execute metadata.sh script** (conditional):
   - Makes script executable
   - Runs the script on EC2 instance
   - Displays all output in GitHub Actions console
5. **File Not Found** (conditional): Shows helpful error if script doesn't exist
6. **Cleanup SSH Key**: Removes SSH key for security

## 📋 Viewing Workflow Results

### EC2 Connectivity Check Workflow

After pushing to GitHub:
1. Go to your repository on GitHub
2. Click the **Actions** tab (top navigation)
3. You'll see workflow runs for "EC2 Connectivity Check"
4. Click on any run to see detailed logs
5. Expand the "Extract EC2 Metadata" step to view all the information

### Fix The Hostname Workflow

To run manually:
1. Go to your repository on GitHub
2. Click the **Actions** tab (top navigation)
3. Click **"Fix The Hostname"** workflow in the left sidebar
4. Click **"Run workflow"** button (top right)
5. Select branch (usually `main`)
6. Click the green **"Run workflow"** button
7. Wait a few seconds and refresh to see the run appear
8. Click on the run to view execution details

### Execute Metadata Script Workflow

To run manually:
1. Go to your repository on GitHub
2. Click the **Actions** tab (top navigation)
3. Click **"Execute Metadata Script"** workflow in the left sidebar
4. Click **"Run workflow"** button (top right)
5. Select branch (usually `main`)
6. Click the green **"Run workflow"** button
7. Wait a few seconds and refresh to see the run appear
8. Click on the run to view script output

**Note:** Make sure `/home/ubuntu/metadata.sh` exists on your EC2 instance before running this workflow.

## 🔒 Security Best Practices

✅ **DO:**
- Store sensitive information (SSH keys, IPs, usernames) in GitHub Secrets
- Use SSH key authentication instead of passwords
- Set proper file permissions for SSH keys (600)
- Clean up SSH keys after use

❌ **DON'T:**
- Commit SSH keys or credentials to the repository
- Expose IP addresses or sensitive data in code
- Use weak authentication methods

## 🛠️ Customization

### EC2 Connectivity Check Workflow

**Change Trigger to Other Branches:**
Edit [.github/workflows/ec2-connectivity-check.yml](.github/workflows/ec2-connectivity-check.yml):
```yaml
on:
  push:
    branches:
      - main
      - develop  # Add more branches
```

**Add Schedule (Run at Specific Times):**
```yaml
on:
  push:
    branches:
      - main
  schedule:
    - cron: '0 0 * * *'  # Runs daily at midnight UTC
```

**Add Manual Trigger:**
```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:  # Allows manual trigger from GitHub UI
```

### Fix The Hostname Workflow

**Change Hostname to Something Else:**
Edit [.github/workflows/fix-hostname.yml](.github/workflows/fix-hostname.yml) and replace "HelloWorld" with your desired hostname in:
- The comparison check
- The `hostnamectl` command
- The `/etc/hostname` update

**Add Input Parameter for Custom Hostname:**
```yaml
on:
  workflow_dispatch:
    inputs:
      hostname:
        description: 'New hostname for EC2'
        required: true
        default: 'HelloWorld'
```

Then use `${{ inputs.hostname }}` in the workflow steps.

## 📁 Repository Structure

```
.
├── .github/
│   └── workflows/
│       ├── ec2-connectivity-check.yml    # Automatic EC2 monitoring workflow
│       ├── fix-hostname.yml              # Manual hostname management workflow
│       └── execute-metadata-script.yml   # Manual script execution workflow
├── .gitignore                            # Files to ignore in git
└── README.md                             # This file
```

## 🐛 Troubleshooting

### EC2 Connectivity Check Workflow

**Workflow Fails with "Permission denied (publickey)"**
- Verify that `EC2_SSH_KEY` contains the complete private key
- Check that `EC2_USER` matches your EC2 instance's SSH user
- Ensure the corresponding public key is in `~/.ssh/authorized_keys` on your EC2

**Workflow Fails with "Connection timed out"**
- Verify `EC2_HOST` has the correct IP address
- Check EC2 security group allows SSH (port 22) from GitHub Actions IPs
- Ensure your EC2 instance is running

**Metadata Service Returns Errors**
- Ensure your EC2 instance has the metadata service enabled
- This workflow uses IMDSv2 (token-based authentication)
- Check that IMDSv2 is not disabled in instance settings

### Fix The Hostname Workflow

**Workflow Fails with "sudo: command not found" or Permission Errors**
- Ensure your EC2 user has sudo privileges
- For Ubuntu instances, the `ubuntu` user has sudo by default
- For Amazon Linux, use `ec2-user`
- Test manually: `ssh user@host "sudo -n true"` should succeed

**Hostname Changes Don't Persist After Reboot**
- The workflow updates both `/etc/hostname` and `/etc/hosts`
- Verify the user has write permissions with sudo
- Check if your instance uses a different hostname configuration system

**Workflow Shows "No changes needed" but Hostname is Wrong**
- Check the comparison logic in the workflow
- Hostname comparison is case-sensitive
- Verify with: `ssh user@host "hostname"` to see actual value

## 📚 Learn More

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [AWS EC2 Instance Metadata Service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
- [GitHub Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## � Workflows Summary

| Workflow | Trigger | Purpose | File |
|----------|---------|---------|------|
| EC2 Connectivity Check | Automatic (Push to main) | Monitor EC2 and extract metadata | [ec2-connectivity-check.yml](.github/workflows/ec2-connectivity-check.yml) |
| Fix The Hostname | Manual (workflow_dispatch) | Set EC2 hostname to HelloWorld | [fix-hostname.yml](.github/workflows/fix-hostname.yml) || Execute Metadata Script | Manual (workflow_dispatch) | Run custom script on EC2 | [execute-metadata-script.yml](.github/workflows/execute-metadata-script.yml) |
---

## 🧑‍💻 Author
**Md. Sarowar Alam**  
Lead DevOps Engineer, Hogarth Worldwide  
📧 Email: sarowar@hotmail.com  
🔗 LinkedIn: [linkedin.com/in/sarowar](https://www.linkedin.com/in/sarowar/)
