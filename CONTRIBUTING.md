# Contributing to RHEL 10 ARM64 EC2 Ansible Automation

Thank you for your interest in contributing! This document provides guidelines for contributing to this project.

## Getting Started

### Prerequisites

- Git
- Ansible (2.15+)
- Python 3.9+
- Podman or Docker (for building EE)
- ansible-builder (for EE builds)
- AWS account with appropriate permissions (for testing)

### Development Setup

1. **Fork and clone the repository:**
   ```bash
   git clone https://github.com/<your-username>/rhel10-ec2-ansible.git
   cd rhel10-ec2-ansible
   ```

2. **Install dependencies:**
   ```bash
   make -f dev/Makefile install
   ```

3. **Set up AWS credentials:**
   ```bash
   export AWS_ACCESS_KEY_ID="your-key"
   export AWS_SECRET_ACCESS_KEY="your-secret"
   export AWS_REGION="us-east-2"
   export AWS_KEYPAIR_NAME="your-keypair"
   export AWS_DEFAULT_SG_ID="sg-xxxxx"
   ```

## Development Workflow

### 1. Create a Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/your-bug-fix
```

Use descriptive branch names:
- `feature/` for new features
- `fix/` for bug fixes
- `docs/` for documentation updates
- `refactor/` for code refactoring

### 2. Make Your Changes

- Follow Ansible best practices
- Keep roles focused and modular
- Update documentation as needed
- Add or update tests

### 3. Test Your Changes

**Lint your code:**
```bash
make -f dev/Makefile lint
```

**Test syntax:**
```bash
ansible-playbook --syntax-check site.yml
```

**Test the build:**
```bash
make -f dev/Makefile build-ee
```

**Test deployment (optional):**
```bash
make -f dev/Makefile deploy
```

### 4. Commit Your Changes

Follow conventional commit messages:

```bash
git commit -m "feat: add support for custom instance tags"
git commit -m "fix: resolve issue with SSH key permissions"
git commit -m "docs: update AAP setup guide"
```

Commit message format:
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation changes
- `refactor:` - Code refactoring
- `test:` - Test additions or changes
- `chore:` - Maintenance tasks

### 5. Push and Create Pull Request

```bash
git push origin feature/your-feature-name
```

Then create a pull request on GitHub with:
- Clear description of changes
- Reference to any related issues
- Test results or screenshots if applicable

## Code Standards

### Ansible Guidelines

1. **Use fully qualified collection names (FQCN):**
   ```yaml
   # Good
   - ansible.builtin.debug:
       msg: "Hello"
   
   # Avoid
   - debug:
       msg: "Hello"
   ```

2. **Follow naming conventions:**
   - Variables: `snake_case`
   - Roles: `snake_case`
   - Files: `kebab-case` or `snake_case`

3. **Document variables:**
   - Add comments in `defaults/main.yml`
   - Update role README if creating new roles

4. **Use meaningful task names:**
   ```yaml
   # Good
   - name: Install baseline packages for administration
   
   # Avoid
   - name: Install packages
   ```

### YAML Style

- Use 2 spaces for indentation
- No trailing whitespace
- Max line length: 120 characters
- Use `---` at the start of YAML files
- Quote strings when needed for clarity

### Directory Structure

```
roles/
  role_name/
    defaults/
      main.yml       # Default variables
    files/           # Static files
    tasks/
      main.yml       # Main task file
    meta/
      main.yml       # Role metadata
    README.md        # Role documentation (if complex)
```

## Testing

### Local Testing

Before submitting a PR:

1. **Lint all playbooks and roles:**
   ```bash
   make -f dev/Makefile lint
   ```

2. **Validate syntax:**
   ```bash
   ansible-playbook --syntax-check *.yml
   ```

3. **Test EE build:**
   ```bash
   make -f dev/Makefile build-ee
   ```

### Automated Testing

GitHub Actions will automatically run:
- ansible-lint
- yamllint
- Syntax validation
- EE definition validation

These must pass before a PR can be merged.

## Updating Dependencies

### Ansible Collections

Update `requirements.yml`:
```yaml
collections:
  - name: amazon.aws
    version: ">=8.2.0"  # Update version
```

### Python Dependencies

Update `requirements.txt`:
```
boto3>=1.35.0  # Update version
```

### Execution Environment

After updating dependencies:
1. Test the build locally
2. Update CHANGELOG.md
3. GitHub Actions will rebuild the EE automatically

## Documentation

### Update Documentation When:

- Adding new variables
- Adding new roles
- Changing playbook behavior
- Updating AAP integration
- Modifying CI/CD workflows

### Documentation Files:

- `README.md` - Main project documentation
- `AAP_SETUP.md` - AAP 2.7 setup guide
- `QUICKSTART.md` - Quick start guide
- `.github/CICD.md` - CI/CD documentation
- `CHANGELOG.md` - Version history
- Role-specific docs in role directories

## Pull Request Process

1. **Ensure tests pass locally**
2. **Update documentation**
3. **Update CHANGELOG.md** (add to "Unreleased" section)
4. **Create PR with clear description**
5. **Respond to review comments**
6. **Squash commits if requested**

### PR Checklist

- [ ] Code follows project standards
- [ ] Tests pass locally
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] Commit messages follow convention
- [ ] No merge conflicts
- [ ] PR description is clear

## Reporting Issues

### Bug Reports

Include:
- Steps to reproduce
- Expected behavior
- Actual behavior
- Environment (OS, Ansible version, AAP version)
- Error messages or logs
- Screenshots if applicable

### Feature Requests

Include:
- Use case description
- Proposed solution
- Alternative solutions considered
- Additional context

## Questions?

- Open an issue for questions
- Check existing issues and PRs
- Review documentation first

## Code of Conduct

- Be respectful and inclusive
- Provide constructive feedback
- Focus on the code, not the person
- Help others learn and grow

## License

By contributing, you agree that your contributions will be licensed under the Apache-2.0 License.

## Thank You!

Your contributions help make this project better for everyone!
