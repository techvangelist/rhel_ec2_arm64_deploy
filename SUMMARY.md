# RHEL 10 ARM64 EC2 Automation - Project Summary

## What This Repository Provides

A complete, production-ready automation solution for deploying and managing RHEL 10 ARM64 EC2 instances using Ansible Automation Platform 2.7 with automated Execution Environment builds via GitHub Actions.

## Key Features

### ✅ Push-Button Deployment
- One-click deployment from AAP 2.7 job templates
- No manual EE builds required - GitHub Actions handles it
- Automated testing on every commit

### ✅ Three Modular Ansible Roles

1. **rhel10_ec2_deploy**
   - Automatically finds latest RHEL 10 ARM64 AMI
   - Deploys EC2 instance with configurable specs
   - Adds instance to dynamic inventory

2. **rhel10_ec2_configure**
   - Creates admin user with passwordless sudo
   - Installs baseline packages (git, tmux, sysstat)
   - Sets up custom shell environment
   - Copies SSH keys

3. **rhel10_ec2_verify**
   - Comprehensive configuration validation
   - Checks user, packages, services, SSH access
   - Verifies ARM64 architecture

### ✅ Automated CI/CD

**GitHub Actions workflows:**
- Automatic EE builds on code changes
- Automated testing (lint, syntax, validation)
- Multi-tag versioning (latest, SHA, custom)
- Push to container registry (Quay.io, Docker Hub, private)

### ✅ Complete Documentation

- **README.md** - Overview and usage
- **QUICKSTART.md** - Get started in 5 minutes
- **AAP_SETUP.md** - Detailed AAP 2.7 integration guide
- **.github/CICD.md** - CI/CD setup and troubleshooting
- **CONTRIBUTING.md** - Contribution guidelines
- **CHANGELOG.md** - Version history

## Repository Structure

```
rhel10-ec2-ansible/
├── .github/
│   ├── workflows/
│   │   ├── build-and-push-ee.yml    # Auto-build EE on commit
│   │   └── test.yml                  # Run tests on PR/push
│   └── CICD.md                       # CI/CD documentation
├── dev/
│   ├── Makefile                      # Local development tools
│   └── README.md                     # Developer guide
├── inventory/
│   └── example.yml                   # Example inventory
├── roles/
│   ├── rhel10_ec2_deploy/           # EC2 deployment role
│   ├── rhel10_ec2_configure/        # Instance configuration role
│   └── rhel10_ec2_verify/           # Verification role
├── ansible.cfg                       # Ansible configuration
├── bindep.txt                        # System dependencies for EE
├── requirements.yml                  # Ansible collections
├── requirements.txt                  # Python dependencies
├── execution-environment.yml         # EE definition for AAP 2.7
├── deploy.yml                        # Deploy playbook
├── configure.yml                     # Configure playbook
├── verify.yml                        # Verify playbook
├── site.yml                          # Complete workflow playbook
├── README.md                         # Main documentation
├── QUICKSTART.md                     # Quick start guide
├── AAP_SETUP.md                      # AAP integration guide
├── CONTRIBUTING.md                   # Contribution guide
├── CHANGELOG.md                      # Version history
└── LICENSE                           # Apache 2.0 license
```

## Workflow Overview

### Traditional Manual Process (Before)
1. ❌ Manually write deployment script
2. ❌ SSH to instance and run configuration commands
3. ❌ Manually verify configuration
4. ❌ Rebuild EE when dependencies change
5. ❌ Push EE to registry manually
6. ❌ Update AAP to use new EE

### Automated Process (Now)

#### One-Time Setup
1. Fork/clone repository
2. Set up GitHub secrets (Quay.io credentials)
3. Configure AAP 2.7 (credentials, project, EE, job template)

#### Every Deployment (Push-Button)
1. Click "Launch" in AAP job template
2. ✅ Instance deployed
3. ✅ Instance configured
4. ✅ Configuration verified
5. ✅ Ready to use

#### Every Code Change (Automated)
1. Commit and push changes
2. ✅ GitHub Actions runs tests
3. ✅ GitHub Actions builds new EE
4. ✅ EE pushed to registry
5. ✅ AAP pulls new EE on next run

## Getting Started

### For AAP Users (Production)

1. **Set up repository:**
   ```bash
   git clone https://github.com/<your-org>/rhel10-ec2-ansible.git
   ```

2. **Configure GitHub Secrets:**
   - `QUAY_USERNAME`
   - `QUAY_TOKEN`
   - `QUAY_ORGANIZATION`

3. **Follow AAP Setup Guide:**
   - See [AAP_SETUP.md](AAP_SETUP.md)
   - Configure Execution Environment, Project, Credentials, Job Templates

4. **Deploy:**
   - Click "Launch" on job template in AAP

### For Developers (Local Testing)

1. **Clone and install:**
   ```bash
   git clone https://github.com/<your-org>/rhel10-ec2-ansible.git
   cd rhel10-ec2-ansible
   make -f dev/Makefile install
   ```

2. **Test locally:**
   ```bash
   make -f dev/Makefile lint
   make -f dev/Makefile build-ee
   make -f dev/Makefile deploy
   ```

3. **See [dev/README.md](dev/README.md) for more tools**

## What Problem Does This Solve?

### Before This Automation

**Manual process for deploying RHEL 10 ARM64 instances:**
- Find the latest AMI ID manually
- Run `aws ec2 run-instances` with many parameters
- Wait for instance to start
- SSH and run multiple configuration commands
- Manually verify each configuration step
- Repeat for every instance
- Document what was done
- Hope nothing was missed

**Execution Environment management:**
- Build EE manually when collections update
- Remember to push to registry
- Update AAP configuration
- Hope EE is consistent across environments

### After This Automation

**Deployment:**
- Click "Launch" in AAP
- Everything happens automatically
- Consistent results every time
- Full verification included
- Complete audit trail

**EE Management:**
- Push code changes
- GitHub Actions builds and pushes EE
- AAP uses updated EE automatically
- Version-tracked via git SHA tags

## Use Cases

### 1. Development Environments
Deploy consistent RHEL 10 ARM64 dev environments on demand.

### 2. Testing
Spin up test instances with known configurations for QA.

### 3. Training/Demos
Quickly provision demo environments for presentations.

### 4. Spark/Analytics Workloads
Deploy instances configured for big data workloads (based on original use case).

### 5. CI/CD Pipeline
Integrate with pipelines to provision test infrastructure.

## Customization

### Change Admin User
Edit `roles/rhel10_ec2_configure/defaults/main.yml`:
```yaml
admin_user: youruser
admin_comment: "Your Name"
```

### Add Packages
Edit `roles/rhel10_ec2_configure/defaults/main.yml`:
```yaml
baseline_packages:
  - git
  - tmux
  - sysstat
  - your-package
```

### Change Instance Type
Edit job template variables in AAP:
```yaml
ec2_instance_type: t4g.2xlarge
ec2_root_volume_size: 200
```

### Add Custom Configuration
Create new tasks in `roles/rhel10_ec2_configure/tasks/main.yml`

## Benefits

### For Operations Teams
- ✅ Consistent deployments
- ✅ Reduced manual errors
- ✅ Audit trail via AAP
- ✅ Self-service for authorized users
- ✅ Automated verification

### For Development Teams
- ✅ Fast environment provisioning
- ✅ Known-good configurations
- ✅ Reproducible environments
- ✅ Version-controlled infrastructure

### For Management
- ✅ Cost visibility (AAP tracking)
- ✅ Compliance (consistent configs)
- ✅ Reduced deployment time
- ✅ Lower operational risk

## Technology Stack

- **Ansible** 2.15+ (automation engine)
- **Ansible Automation Platform** 2.7 (enterprise orchestration)
- **GitHub Actions** (CI/CD)
- **AWS EC2** (cloud infrastructure)
- **Podman/Docker** (container runtime for EE)
- **Quay.io** (container registry)
- **RHEL 10 ARM64** (target OS)

## Support and Maintenance

### Getting Help
- Check [README.md](README.md) for usage
- See [AAP_SETUP.md](AAP_SETUP.md) for AAP issues
- Review [.github/CICD.md](.github/CICD.md) for CI/CD problems
- Open GitHub issue for bugs or questions

### Contributing
- See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines
- Fork and submit pull requests
- Follow code standards
- Add tests and documentation

### Updates
- **Collections**: Update `requirements.yml`
- **Python deps**: Update `requirements.txt`
- **System deps**: Update `bindep.txt`
- GitHub Actions will rebuild EE automatically

## Next Steps

1. **Quick Start**: Follow [QUICKSTART.md](QUICKSTART.md)
2. **AAP Integration**: Follow [AAP_SETUP.md](AAP_SETUP.md)
3. **Customize**: Edit roles to fit your needs
4. **Extend**: Add new roles or playbooks
5. **Share**: Contribute improvements back

## License

Apache-2.0 - See [LICENSE](LICENSE)

## Author

Abraham Snell (Red Hat)

---

**This automation transforms manual, error-prone deployments into push-button, consistent, verified infrastructure.**
