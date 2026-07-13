# Ansible Automation Platform 2.7 Setup Guide

This guide walks you through setting up this project in Ansible Automation Platform (AAP) 2.7.

## Overview

This project provides automation for deploying, configuring, and verifying RHEL 10 ARM64 EC2 instances. The following steps will help you integrate it with AAP 2.7.

## Prerequisites

- Ansible Automation Platform 2.7 installed and accessible
- Access to a container registry (Quay.io, Docker Hub, or private registry)
- Red Hat Container Registry credentials (for pulling base EE images)
- AWS credentials with EC2 permissions
- ansible-builder installed on your build machine

## Step 1: Build the Execution Environment

### Install ansible-builder

On your build machine:

```bash
pip install ansible-builder
```

### Build the Execution Environment

Navigate to the project directory and build:

```bash
cd rhel10-ec2-ansible
ansible-builder build -t rhel10-ec2-ee:latest -v 3
```

This will:
- Use the AAP 2.7 minimal EE as base
- Install required Ansible collections (amazon.aws, community.general)
- Install Python dependencies (boto3, botocore)
- Configure the environment for AWS operations

### Tag and Push to Registry

Tag for your container registry:

```bash
# For Quay.io
podman tag rhel10-ec2-ee:latest quay.io/<your-org>/rhel10-ec2-ee:latest

# For private registry
podman tag rhel10-ec2-ee:latest registry.example.com/rhel10-ec2-ee:latest
```

Login to your registry:

```bash
# For Quay.io
podman login quay.io

# For private registry
podman login registry.example.com
```

Push the image:

```bash
# For Quay.io
podman push quay.io/<your-org>/rhel10-ec2-ee:latest

# For private registry
podman push registry.example.com/rhel10-ec2-ee:latest
```

## Step 2: Configure AAP - Execution Environment

1. Log into AAP web interface
2. Navigate to **Execution Environments** (left sidebar)
3. Click **Add**
4. Fill in:
   - **Name**: `RHEL 10 EC2 Execution Environment`
   - **Image**: Your registry path (e.g., `quay.io/<your-org>/rhel10-ec2-ee:latest`)
   - **Description**: `Execution environment for RHEL 10 ARM64 EC2 automation`
   - **Organization**: Select your organization
   - **Registry Credential**: Select or create registry credential if using private registry
   - **Pull**: `Always` or `If not present`
5. Click **Save**

## Step 3: Configure AAP - Credentials

### AWS Credential

1. Navigate to **Credentials** (left sidebar)
2. Click **Add**
3. Fill in:
   - **Name**: `AWS EC2 Credentials`
   - **Description**: `AWS credentials for EC2 operations`
   - **Organization**: Select your organization
   - **Credential Type**: `Amazon Web Services`
   - **Access Key**: Your AWS access key ID
   - **Secret Key**: Your AWS secret access key
4. Click **Save**

### SSH Credential (for EC2 instances)

1. Navigate to **Credentials**
2. Click **Add**
3. Fill in:
   - **Name**: `AWS EC2 SSH Key`
   - **Description**: `SSH key for EC2 instances`
   - **Organization**: Select your organization
   - **Credential Type**: `Machine`
   - **Username**: `ec2-user`
   - **SSH Private Key**: Paste your AWS key pair private key
   - **Privilege Escalation Method**: `sudo`
4. Click **Save**

## Step 4: Configure AAP - Project

1. Navigate to **Projects** (left sidebar)
2. Click **Add**
3. Fill in:
   - **Name**: `RHEL 10 EC2 Automation`
   - **Description**: `Ansible roles for RHEL 10 ARM64 EC2 deployment`
   - **Organization**: Select your organization
   - **Default Execution Environment**: Select `RHEL 10 EC2 Execution Environment`
   - **Source Control Type**: `Git`
   - **Source Control URL**: Your Git repository URL
   - **Source Control Branch/Tag/Commit**: `main`
   - **Options**: Check `Update Revision on Launch`
4. Click **Save**

## Step 5: Configure AAP - Inventory

### Option 1: Static Inventory (for existing instances)

1. Navigate to **Inventories**
2. Click **Add** → **Add inventory**
3. Fill in:
   - **Name**: `RHEL 10 EC2 Instances`
   - **Description**: `RHEL 10 ARM64 EC2 instances`
   - **Organization**: Select your organization
4. Click **Save**
5. Click **Hosts** tab → **Add**
6. Add your instance:
   - **Name**: Instance IP or hostname
   - **Variables** (in YAML):
     ```yaml
     ansible_user: ec2-user
     admin_user: asnell
     ```
7. Click **Save**

### Option 2: Dynamic Inventory (for automated deployment)

For dynamic inventory with AWS EC2 plugin, create an inventory source:

1. Create inventory as above
2. Click **Sources** tab → **Add**
3. Fill in:
   - **Name**: `AWS EC2 Dynamic Inventory`
   - **Source**: `Amazon EC2`
   - **Credential**: Select `AWS EC2 Credentials`
   - **Regions**: Select your AWS region(s)
   - **Instance Filters**: Add filters as needed (e.g., `tag:ManagedBy=Ansible`)
   - **Options**: Check `Update on Launch`, `Overwrite`, `Overwrite Variables`
4. Click **Save**

## Step 6: Create Job Templates

### Job Template 1: Deploy Instance

1. Navigate to **Templates** → **Add** → **Add job template**
2. Fill in:
   - **Name**: `Deploy RHEL 10 EC2 Instance`
   - **Description**: `Deploy a new RHEL 10 ARM64 EC2 instance`
   - **Job Type**: `Run`
   - **Inventory**: `RHEL 10 EC2 Instances` (or create a simple localhost inventory)
   - **Project**: `RHEL 10 EC2 Automation`
   - **Execution Environment**: `RHEL 10 EC2 Execution Environment`
   - **Playbook**: `deploy.yml`
   - **Credentials**:
     - Add `AWS EC2 Credentials`
   - **Variables** (Extra Variables):
     ```yaml
     ec2_instance_name: rhel10-demo-instance
     ec2_instance_type: t4g.xlarge
     ec2_root_volume_size: 100
     ```
   - **Options**: Check `Prompt on launch` for Variables if you want to customize per run
3. Add environment variables by clicking **Add** under Extra Variables:
   ```yaml
   AWS_REGION: us-east-2
   AWS_KEYPAIR_NAME: your-keypair-name
   AWS_DEFAULT_SG_ID: sg-xxxxxxxxxxxxx
   AWS_DEFAULT_SUBNET_ID: subnet-xxxxxxxxxxxxx
   ```
4. Click **Save**

### Job Template 2: Configure Instance

1. Navigate to **Templates** → **Add** → **Add job template**
2. Fill in:
   - **Name**: `Configure RHEL 10 EC2 Instance`
   - **Description**: `Configure RHEL 10 instance with users and packages`
   - **Job Type**: `Run`
   - **Inventory**: `RHEL 10 EC2 Instances`
   - **Project**: `RHEL 10 EC2 Automation`
   - **Execution Environment**: `RHEL 10 EC2 Execution Environment`
   - **Playbook**: `configure.yml`
   - **Credentials**:
     - Add `AWS EC2 SSH Key`
   - **Options**: Check `Privilege Escalation`
3. Click **Save**

### Job Template 3: Verify Configuration

1. Navigate to **Templates** → **Add** → **Add job template**
2. Fill in:
   - **Name**: `Verify RHEL 10 EC2 Configuration`
   - **Description**: `Verify instance configuration is correct`
   - **Job Type**: `Run`
   - **Inventory**: `RHEL 10 EC2 Instances`
   - **Project**: `RHEL 10 EC2 Automation`
   - **Execution Environment**: `RHEL 10 EC2 Execution Environment`
   - **Playbook**: `verify.yml`
   - **Credentials**:
     - Add `AWS EC2 SSH Key`
3. Click **Save**

### Job Template 4: Complete Workflow (Deploy, Configure, Verify)

1. Navigate to **Templates** → **Add** → **Add job template**
2. Fill in:
   - **Name**: `RHEL 10 EC2 Complete Workflow`
   - **Description**: `Deploy, configure, and verify RHEL 10 instance`
   - **Job Type**: `Run`
   - **Inventory**: `RHEL 10 EC2 Instances`
   - **Project**: `RHEL 10 EC2 Automation`
   - **Execution Environment**: `RHEL 10 EC2 Execution Environment`
   - **Playbook**: `site.yml`
   - **Credentials**:
     - Add `AWS EC2 Credentials`
     - Add `AWS EC2 SSH Key`
   - **Options**: Check `Privilege Escalation`
   - **Variables**:
     ```yaml
     AWS_REGION: us-east-2
     AWS_KEYPAIR_NAME: your-keypair-name
     AWS_DEFAULT_SG_ID: sg-xxxxxxxxxxxxx
     AWS_DEFAULT_SUBNET_ID: subnet-xxxxxxxxxxxxx
     ec2_instance_name: rhel10-demo-instance
     ```
3. Click **Save**

## Step 7: Create Workflow Template (Optional)

For more control, create a workflow that chains the jobs:

1. Navigate to **Templates** → **Add** → **Add workflow template**
2. Fill in:
   - **Name**: `RHEL 10 EC2 Deployment Workflow`
   - **Description**: `Automated workflow for EC2 deployment`
   - **Organization**: Select your organization
   - **Inventory**: `RHEL 10 EC2 Instances`
3. Click **Save**
4. Click **Visualizer** to build the workflow:
   - Add node: `Deploy RHEL 10 EC2 Instance` → Save
   - Add node after deploy (on success): `Configure RHEL 10 EC2 Instance` → Save
   - Add node after configure (on success): `Verify RHEL 10 EC2 Configuration` → Save
5. Click **Save**

## Step 8: Test Your Setup

1. Navigate to the job template or workflow you created
2. Click **Launch**
3. If you set variables to prompt on launch, customize as needed
4. Click **Launch** to start the job
5. Monitor the job output in real-time

## Customization for Your Environment

### Custom Admin User

To change the admin user from default (`asnell`):

1. Edit the job template
2. In **Variables**, add:
   ```yaml
   admin_user: youruser
   admin_comment: "Your Name"
   ```

### Additional Packages

To install additional packages:

1. Edit the job template
2. In **Variables**, add:
   ```yaml
   baseline_packages:
     - git
     - tmux
     - sysstat
     - vim
     - htop
     - python3
   ```

### Different Instance Type

To use a different EC2 instance type:

1. Edit the deployment job template
2. In **Variables**, modify:
   ```yaml
   ec2_instance_type: t4g.2xlarge  # or any other ARM64 instance type
   ec2_root_volume_size: 200       # Adjust as needed
   ```

## Troubleshooting

### Execution Environment Pull Fails

- Verify your registry credentials in AAP
- Ensure the EE image is accessible from AAP
- Check the image tag is correct

### AWS Credentials Not Working

- Verify AWS credentials are correctly entered
- Check IAM permissions include EC2 operations
- Test credentials outside AAP first

### Playbook Fails to Find Collections

- Ensure collections are installed in the EE
- Rebuild the EE if collection versions were updated
- Check `requirements.yml` has correct collection versions

### SSH Connection Fails

- Verify security group allows SSH from AAP
- Check the SSH credential has the correct private key
- Ensure key pair name matches in variables

## Maintenance

### Updating the Execution Environment

When updating collections or dependencies:

1. Update `requirements.yml`, `requirements.txt`, or `bindep.txt`
2. Rebuild the EE: `ansible-builder build -t rhel10-ec2-ee:<new-version> -v 3`
3. Push to registry
4. Update the EE definition in AAP with new tag
5. Projects using the EE will pull the new version on next run

### Updating the Project

AAP will automatically update the project if "Update Revision on Launch" is enabled. To manually update:

1. Navigate to **Projects**
2. Find `RHEL 10 EC2 Automation`
3. Click the update icon

## Best Practices

1. **Use Surveys**: Add surveys to job templates for user-friendly variable input
2. **Notifications**: Set up notifications for job success/failure
3. **Schedules**: Create schedules for recurring deployments or verifications
4. **RBAC**: Use AAP's role-based access control to limit who can deploy instances
5. **Vault**: Use Ansible Vault for sensitive data in variables
6. **Logging**: Enable job output logging for audit purposes

## Support

For issues with this automation:
- Check the GitHub repository for updates
- Review AAP logs: `/var/log/tower/` on AAP server
- Review execution environment build logs
- Test playbooks locally before running in AAP
