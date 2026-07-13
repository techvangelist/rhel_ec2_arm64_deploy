# CI/CD with GitHub Actions

This repository uses GitHub Actions to automate the build and deployment of the Execution Environment.

## Workflows

### 1. Test Ansible Content (`test.yml`)

**Triggers:**
- On every pull request to `main`
- On every push to `main`

**Jobs:**
- **ansible-lint**: Validates Ansible playbooks and roles
- **yaml-lint**: Validates YAML syntax
- **syntax-check**: Checks Ansible syntax
- **validate-ee-definition**: Validates the Execution Environment definition

**Purpose:** Ensures code quality before merging and deployment.

### 2. Build and Push Execution Environment (`build-and-push-ee.yml`)

**Triggers:**
- On push to `main` when EE-related files change:
  - `execution-environment.yml`
  - `requirements.yml`
  - `requirements.txt`
  - `bindep.txt`
  - `ansible.cfg`
- Manual trigger via workflow dispatch

**Jobs:**
- **lint**: Runs ansible-lint on all playbooks
- **build-and-push**: Builds and pushes the EE to Quay.io

**Tags Created:**
- `latest`: Always points to the most recent build
- `<git-sha>`: Specific commit SHA for version tracking

**Purpose:** Automatically builds and publishes the Execution Environment when dependencies or configuration changes.

## Setup

### Required Secrets

Add the following secrets to your GitHub repository:

1. **QUAY_USERNAME**: Your Quay.io username
2. **QUAY_TOKEN**: Your Quay.io token (or robot account token)
3. **QUAY_ORGANIZATION**: Your Quay.io organization or username

#### Creating Secrets

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add each secret with the values above

#### Getting Quay.io Credentials

**Option 1: Personal Account**
1. Log into [quay.io](https://quay.io)
2. Go to **Account Settings** → **Generate Encrypted Password**
3. Use your username and the encrypted password

**Option 2: Robot Account (Recommended for CI/CD)**
1. Log into [quay.io](https://quay.io)
2. Go to your organization
3. Navigate to **Robot Accounts**
4. Click **Create Robot Account**
5. Give it write permissions to the repository
6. Use the robot account credentials in GitHub secrets

### Repository Configuration

**Using Quay.io (Recommended):**
```
QUAY_USERNAME: your-username
QUAY_TOKEN: your-token-or-encrypted-password
QUAY_ORGANIZATION: your-org-name
```

**Using Docker Hub (Alternative):**

Modify `.github/workflows/build-and-push-ee.yml`:
1. Replace `quay.io` with `docker.io`
2. Change secrets to `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`
3. Update the `podman-login` step accordingly

**Using Private Registry (Alternative):**

Modify `.github/workflows/build-and-push-ee.yml`:
1. Replace `quay.io` with your registry URL
2. Update secrets appropriately
3. Add `REGISTRY_URL` secret if needed

## Workflow Usage

### Automatic Build on Push

When you push changes to `main` that affect the EE:

```bash
git add execution-environment.yml requirements.yml
git commit -m "Update collection versions"
git push origin main
```

GitHub Actions will automatically:
1. Run tests
2. Build the new EE
3. Push to Quay.io with `latest` and commit SHA tags

### Manual Build

Trigger a manual build with a specific version tag:

1. Go to **Actions** → **Build and Push Execution Environment**
2. Click **Run workflow**
3. Optionally specify a tag version (e.g., `v1.0.0`)
4. Click **Run workflow**

### Monitoring Builds

1. Go to the **Actions** tab in your repository
2. Click on a workflow run to see details
3. View logs for each job
4. Check the summary for image tags and pull commands

## Integration with AAP

After GitHub Actions builds and pushes the EE:

1. The image is available at: `quay.io/<org>/rhel10-ec2-ee:latest`
2. AAP will automatically pull the new image on next job run (if pull policy is set to "Always")
3. Or manually sync the EE in AAP:
   - Go to **Execution Environments**
   - Find your EE
   - Click the sync/refresh button

## Version Management

### Tagging Strategy

- **latest**: Always the most recent build from `main`
- **Git SHA (short)**: Specific commit for traceability (e.g., `abc1234`)
- **Manual tags**: Version tags from workflow dispatch (e.g., `v1.0.0`)

### Updating AAP to Use a Specific Version

In AAP Execution Environment settings:
- For latest: `quay.io/<org>/rhel10-ec2-ee:latest`
- For specific commit: `quay.io/<org>/rhel10-ec2-ee:abc1234`
- For specific version: `quay.io/<org>/rhel10-ec2-ee:v1.0.0`

### Creating a Release

To create a versioned release:

1. Update your code
2. Create a git tag:
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```
3. Manually trigger the workflow:
   - Go to **Actions** → **Build and Push Execution Environment**
   - Run workflow with tag `v1.0.0`

## Troubleshooting

### Build Fails

**Check the logs:**
1. Go to **Actions** tab
2. Click the failed workflow
3. Expand failed job
4. Review error messages

**Common issues:**
- Invalid YAML syntax: Run `yamllint` locally
- Missing collections: Verify `requirements.yml`
- Invalid EE definition: Run `ansible-builder create` locally
- Registry authentication: Check secrets are correct

### Push Fails

**Common issues:**
- Incorrect credentials: Verify `QUAY_USERNAME` and `QUAY_TOKEN`
- Insufficient permissions: Ensure robot account has write access
- Repository doesn't exist: Create the repository in Quay.io first
- Network issues: Check GitHub Actions network status

### AAP Not Pulling New Image

**Solutions:**
1. Check pull policy in AAP EE settings (set to "Always")
2. Manually sync the EE in AAP
3. Verify the image tag in AAP matches what was pushed
4. Check AAP can access the registry (network/firewall)

## Best Practices

1. **Test locally first**: Use `dev/Makefile` to test before pushing
2. **Use pull requests**: Let tests run before merging to `main`
3. **Version tags**: Use manual workflow dispatch for production releases
4. **Monitor builds**: Check Actions tab after pushing changes
5. **Keep secrets secure**: Never commit credentials to the repository
6. **Document changes**: Update README and CHANGELOG for major changes

## Local Development

For local testing before committing:

```bash
# Lint your code
make -f dev/Makefile lint

# Build EE locally
make -f dev/Makefile build-ee

# Test syntax
ansible-playbook --syntax-check site.yml
```

## Disabling Workflows

To temporarily disable a workflow:

1. Edit the workflow file
2. Comment out the `on:` triggers
3. Or go to **Actions** → **Workflows** → Select workflow → **Disable workflow**

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Quay.io Documentation](https://docs.quay.io/)
- [Ansible Builder Documentation](https://ansible-builder.readthedocs.io/)
- [AAP Execution Environments](https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.7/html/creating_and_consuming_execution_environments/)
