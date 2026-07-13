# RHEL 10 ARM64 EC2 Ansible Automation

[![Build and Push EE](https://github.com/<your-org>/<your-repo>/actions/workflows/build-and-push-ee.yml/badge.svg)](https://github.com/<your-org>/<your-repo>/actions/workflows/build-and-push-ee.yml)
[![Test](https://github.com/<your-org>/<your-repo>/actions/workflows/test.yml/badge.svg)](https://github.com/<your-org>/<your-repo>/actions/workflows/test.yml)

Ansible roles and playbooks for deploying, configuring, and verifying RHEL 10 ARM64 EC2 instances on AWS.

**Push-button automation for Ansible Automation Platform 2.7** with automated Execution Environment builds via GitHub Actions.

## Overview

This project provides three main Ansible roles:

1. **rhel10_ec2_deploy** - Deploys RHEL 10 ARM64 EC2 instances
2. **rhel10_ec2_configure** - Configures instances with users, packages, and system settings
3. **rhel10_ec2_verify** - Verifies the instance configuration

## Quick Start

### For AAP 2.7 Users (Recommended)

1. **Fork/clone this repository**
2. **Set up GitHub Secrets** for automated EE builds (see [CI/CD Setup](.github/CICD.md))
3. **Follow the [AAP Setup Guide](AAP_SETUP.md)** for one-time configuration
4. **Click "Launch"** on your job template - that's it!

The Execution Environment is automatically built and pushed to your registry on every commit via GitHub Actions.

### For Local Development/Testing

See [dev/README.md](dev/README.md) for local development tools and testing.

## Prerequisites

### AWS Credentials

Set the following environment variables:

```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_REGION="us-east-2"
export AWS_KEYPAIR_NAME="your-keypair-name"
export AWS_DEFAULT_SG_ID="sg-xxxxxxxxxxxxx"
export AWS_DEFAULT_SUBNET_ID="subnet-xxxxxxxxxxxxx"  # Optional
export AWS_SSH_KEY_PATH="~/.ssh/your-key.pem"
```

### Ansible Collections

Install required collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

### Python Dependencies

Install required Python packages:

```bash
pip install -r requirements.txt
```

## Usage

### Option 1: Complete Workflow (Deploy, Configure, Verify)

Run the complete workflow that deploys, configures, and verifies an instance:

```bash
ansible-playbook site.yml
```

### Option 2: Individual Steps

#### Deploy Instance

```bash
ansible-playbook deploy.yml
```

This will output the instance IP address for the next steps.

#### Configure Instance

```bash
ansible-playbook -i <INSTANCE_IP>, configure.yml
```

Note: The comma after the IP is required for Ansible to treat it as a list.

#### Verify Configuration

```bash
ansible-playbook -i <INSTANCE_IP>, verify.yml
```

## Roles

### rhel10_ec2_deploy

Deploys RHEL 10 ARM64 EC2 instances.

**Variables** (see `roles/rhel10_ec2_deploy/defaults/main.yml`):
- `ec2_instance_type`: Instance type (default: `t4g.xlarge`)
- `ec2_instance_name`: Instance name tag (default: `rhel10-spark-build`)
- `ec2_root_volume_size`: Root volume size in GB (default: `100`)
- `ec2_root_volume_type`: EBS volume type (default: `gp3`)

### rhel10_ec2_configure

Configures RHEL 10 instances with:
- System updates
- Baseline packages (git, tmux, sysstat)
- Admin user creation with passwordless sudo
- SSH key setup
- Custom bashrc.d configuration files

**Variables** (see `roles/rhel10_ec2_configure/defaults/main.yml`):
- `admin_user`: Admin username (default: `asnell`)
- `admin_comment`: User's full name (default: `Abraham Snell`)
- `baseline_packages`: List of packages to install

### rhel10_ec2_verify

Verifies the instance configuration by checking:
- SSH connectivity
- User and group membership
- Passwordless sudo
- Package installation
- Service status (sysstat)
- Bashrc.d configuration
- OS and architecture

**Variables** (see `roles/rhel10_ec2_verify/defaults/main.yml`):
- `verify_user`: User to verify (default: `asnell`)
- `expected_packages`: List of expected packages
- `expected_bashrc_files`: List of expected bashrc.d files

## CI/CD with GitHub Actions

This repository uses GitHub Actions to automate the build and deployment of the Execution Environment.

### Automated Workflows

1. **Test Workflow** - Runs on every PR and push:
   - Ansible lint
   - YAML lint
   - Syntax checking
   - EE definition validation

2. **Build and Push EE** - Runs when EE files change:
   - Automatically builds the Execution Environment
   - Pushes to your container registry (Quay.io, Docker Hub, etc.)
   - Tags with `latest` and commit SHA

### Setup GitHub Actions

1. **Add GitHub Secrets** (Settings → Secrets → Actions):
   - `QUAY_USERNAME`: Your Quay.io username
   - `QUAY_TOKEN`: Your Quay.io token
   - `QUAY_ORGANIZATION`: Your Quay.io organization

2. **Push to repository** - GitHub Actions handles the rest!

For detailed CI/CD setup and configuration, see [.github/CICD.md](.github/CICD.md).

## Execution Environment for AAP 2.7

This repository includes an Execution Environment definition for Ansible Automation Platform 2.7.

### Automated Build (Recommended)

**The Execution Environment is automatically built via GitHub Actions** when you push changes. No manual build needed!

The built image will be available at: `quay.io/<your-org>/rhel10-ec2-ee:latest`

### Manual Build (Optional)

For local testing, install ansible-builder:

```bash
pip install ansible-builder
```

Build the execution environment:

```bash
ansible-builder build -t rhel10-ec2-ee:latest -v 3
```

### Push to Container Registry

Push to your container registry for use with AAP:

```bash
# Tag for your registry
podman tag rhel10-ec2-ee:latest <your-registry>/rhel10-ec2-ee:latest

# Login to your registry
podman login <your-registry>

# Push the image
podman push <your-registry>/rhel10-ec2-ee:latest
```

### Using in AAP 2.7

1. In AAP, navigate to **Execution Environments**
2. Click **Add**
3. Provide:
   - **Name**: `RHEL 10 EC2 Execution Environment`
   - **Image**: `<your-registry>/rhel10-ec2-ee:latest`
   - **Pull**: Always or Missing
4. Save the execution environment

5. Create a new **Project** pointing to this Git repository
6. Create a **Job Template** using:
   - **Inventory**: Your AAP inventory or create a new one
   - **Project**: The project created in step 5
   - **Playbook**: `site.yml` (or `deploy.yml`, `configure.yml`, `verify.yml`)
   - **Execution Environment**: The execution environment created in step 4
   - **Credentials**: AWS credential type with your AWS access keys

7. Add environment variables to your Job Template:
   - `AWS_REGION`
   - `AWS_KEYPAIR_NAME`
   - `AWS_DEFAULT_SG_ID`
   - `AWS_DEFAULT_SUBNET_ID`
   - `AWS_SSH_KEY_PATH`

## Customization

### Modifying Admin User

Edit `roles/rhel10_ec2_configure/defaults/main.yml`:

```yaml
admin_user: youruser
admin_comment: "Your Name"
```

### Adding Packages

Edit `roles/rhel10_ec2_configure/defaults/main.yml`:

```yaml
baseline_packages:
  - git
  - tmux
  - sysstat
  - vim
  - htop
```

### Customizing Bashrc Configuration

Add or modify files in `roles/rhel10_ec2_configure/files/` and update the `bashrc_d_files` list.

## Directory Structure

```
.
├── ansible.cfg
├── deploy.yml
├── configure.yml
├── verify.yml
├── site.yml
├── requirements.yml
├── requirements.txt
├── bindep.txt
├── execution-environment.yml
├── roles/
│   ├── rhel10_ec2_deploy/
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── meta/
│   │       └── main.yml
│   ├── rhel10_ec2_configure/
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   ├── files/
│   │   │   ├── 02-aliases
│   │   │   ├── 03-bash_history
│   │   │   └── 05-shell_env
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── meta/
│   │       └── main.yml
│   └── rhel10_ec2_verify/
│       ├── defaults/
│       │   └── main.yml
│       ├── tasks/
│       │   └── main.yml
│       └── meta/
│           └── main.yml
└── README.md
```

## License

Apache-2.0

## Author

Abraham Snell (Red Hat)
