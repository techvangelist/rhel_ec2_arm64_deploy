# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial release of RHEL 10 ARM64 EC2 automation
- Three Ansible roles: deploy, configure, verify
- Execution Environment definition for AAP 2.7
- GitHub Actions CI/CD workflows for automated EE builds
- Comprehensive documentation for AAP 2.7 setup
- Local development tools in `dev/` directory

### Features

#### Deployment Role (rhel10_ec2_deploy)
- Automatic AMI discovery for latest RHEL 10 ARM64
- EC2 instance deployment with configurable instance types
- Tagging and resource management
- Dynamic inventory integration

#### Configuration Role (rhel10_ec2_configure)
- Admin user creation with passwordless sudo
- Baseline package installation (git, tmux, sysstat)
- Custom bashrc.d configuration system
- SSH key management
- System updates via RHUI

#### Verification Role (rhel10_ec2_verify)
- Comprehensive configuration verification
- Service status validation
- User and permission checks
- Package installation verification
- Architecture validation

#### CI/CD
- Automated Execution Environment builds on commit
- Automated testing (lint, syntax check, validation)
- Multi-registry support (Quay.io, Docker Hub, private)
- Version tagging (latest, commit SHA, manual versions)

## Version History

### [1.0.0] - YYYY-MM-DD (Initial Release)

First stable release for AAP 2.7 integration.

---

## How to Use This Changelog

- Keep an "Unreleased" section at the top for upcoming changes
- Add new versions in reverse chronological order
- Group changes into: Added, Changed, Deprecated, Removed, Fixed, Security
- Reference issues and PRs when applicable
- Update the version date when releasing
