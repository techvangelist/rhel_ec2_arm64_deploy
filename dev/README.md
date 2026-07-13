# Developer Tools

This directory contains tools for local development and testing.

## Makefile

The Makefile provides convenient commands for testing the automation locally before committing to the repository.

### Prerequisites

- Ansible installed locally
- ansible-builder installed (`pip install ansible-builder`)
- Podman or Docker installed
- AWS credentials configured

### Available Commands

```bash
# Install dependencies locally
make -f dev/Makefile install

# Build the Execution Environment locally
make -f dev/Makefile build-ee

# Test deployment locally
make -f dev/Makefile deploy

# Test configuration (requires INSTANCE_IP)
make -f dev/Makefile configure INSTANCE_IP=<ip-address>

# Test verification (requires INSTANCE_IP)
make -f dev/Makefile verify INSTANCE_IP=<ip-address>

# Run complete workflow
make -f dev/Makefile all

# Lint the playbooks and roles
make -f dev/Makefile lint

# Clean temporary files
make -f dev/Makefile clean
```

### Testing Before Commit

Before committing changes:

1. Test locally:
   ```bash
   make -f dev/Makefile install
   make -f dev/Makefile lint
   make -f dev/Makefile build-ee
   ```

2. Optionally test deployment (if you have AWS access):
   ```bash
   make -f dev/Makefile deploy
   ```

3. Commit and push - GitHub Actions will handle the rest

## Note

**These tools are for local development only.** For production deployments, use:
- **GitHub Actions**: Automatically builds and pushes the EE on commit
- **AAP 2.7**: Push-button deployment via job templates

See `../AAP_SETUP.md` for production setup instructions.
