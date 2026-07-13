# Quick Start Guide

This guide will help you quickly deploy and configure a RHEL 10 ARM64 EC2 instance.

## Prerequisites

1. AWS account with appropriate permissions
2. AWS credentials configured
3. SSH key pair in AWS
4. Security group allowing SSH access

## Step 1: Set Environment Variables

```bash
export AWS_ACCESS_KEY_ID="your-access-key-id"
export AWS_SECRET_ACCESS_KEY="your-secret-access-key"
export AWS_REGION="us-east-2"
export AWS_KEYPAIR_NAME="your-keypair-name"
export AWS_DEFAULT_SG_ID="sg-xxxxxxxxxxxxx"
export AWS_SSH_KEY_PATH="~/.ssh/your-key.pem"
```

## Step 2: Install Dependencies

```bash
# Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# Install Python dependencies
pip install -r requirements.txt
```

## Step 3: Run the Complete Workflow

```bash
ansible-playbook site.yml
```

This single command will:
1. Deploy a new RHEL 10 ARM64 EC2 instance
2. Configure it with your admin user and packages
3. Verify the configuration

## Step 4: SSH to Your Instance

After the playbook completes, you can SSH to your instance:

```bash
ssh asnell@<INSTANCE_IP>
```

Replace `<INSTANCE_IP>` with the IP address shown in the playbook output.

## Individual Steps (Alternative)

If you prefer to run each step separately:

### Deploy Only

```bash
ansible-playbook deploy.yml
```

Note the instance IP from the output.

### Configure Only

```bash
ansible-playbook -i <INSTANCE_IP>, configure.yml
```

### Verify Only

```bash
ansible-playbook -i <INSTANCE_IP>, verify.yml
```

## Customization

To customize the deployment, edit the variables in:
- `roles/rhel10_ec2_deploy/defaults/main.yml` - EC2 instance settings
- `roles/rhel10_ec2_configure/defaults/main.yml` - Configuration settings

## Troubleshooting

### SSH Connection Fails

- Ensure your security group allows SSH (port 22) from your IP
- Verify the correct SSH key path is set in `AWS_SSH_KEY_PATH`
- Check that the instance is in "running" state

### Deployment Fails

- Verify all required environment variables are set
- Check AWS credentials have necessary permissions
- Ensure the subnet and security group IDs are correct

### Configuration Fails

- Wait a few moments after deployment for SSH to be ready
- Verify you're using the correct instance IP address
- Check that the SSH key has correct permissions (chmod 600)

## Next Steps

- Review the [README.md](README.md) for detailed documentation
- Customize bashrc.d files in `roles/rhel10_ec2_configure/files/`
- Add additional packages to `baseline_packages` variable
- Set up the Execution Environment for AAP 2.7
