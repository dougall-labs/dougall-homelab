# Environment Setup Guide

This guide will help you set up your development environment for working with this homelab repository.

## Prerequisites

### macOS Setup

1. **Homebrew** (if not already installed):
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **detect-secrets** - Secret detection tool:
   ```bash
   brew install detect-secrets
   ```

3. **pre-commit** - Git hooks framework:
   ```bash
   brew install pre-commit
   # or alternatively:
   pip install pre-commit
   ```

## Initial Setup

### 1. Clone the Repository

```bash
git clone <repository-url>
cd homelab
```

### 2. Install Pre-commit Hooks

Install the git hooks that will run automatically before each commit:

```bash
pre-commit install
```

This will set up hooks based on `.pre-commit-config.yaml` to:
- Run detect-secrets before each commit to prevent secrets from being committed

### 3. Verify Setup

Test that the pre-commit hooks are working:

```bash
pre-commit run --all-files
```

This will scan all files in the repository. If everything is set up correctly, you should see no errors (or only expected warnings for files that are intentionally excluded).

## Secret Detection

### How It Works

The repository uses [detect-secrets](https://github.com/Yelp/detect-secrets) to scan for potential secrets before each commit. The pre-commit hook will:

1. Scan staged files for potential secrets
2. Compare findings against the `.secrets.baseline` file
3. Block the commit if new secrets are detected
4. Allow the commit if no new secrets are found

### Baseline File

The `.secrets.baseline` file contains known false positives and should be committed to the repository. This file tracks legitimate values that might look like secrets but aren't (e.g., example values, test data, etc.).

### Updating the Baseline

If detect-secrets flags something that is not actually a secret (a false positive), you can update the baseline:

```bash
detect-secrets scan --update .secrets.baseline
```

**Important**: Only update the baseline if you're certain the detected value is not a secret. Examples of safe values to add:
- Example values in `.example` files
- Test data
- Public documentation values
- Placeholder values

### Excluded Files

The following file patterns are automatically excluded from secret detection:
- `*.secrets.baseline` - The baseline file itself
- `*.example` - Example files (e.g., `secrets.yml.example`)
- `*.sample` - Sample files
- `.git/` - Git directory

## Troubleshooting

### Pre-commit Hook Not Running

If the pre-commit hook doesn't run automatically:

1. Verify hooks are installed:
   ```bash
   ls -la .git/hooks/pre-commit
   ```

2. Reinstall if needed:
   ```bash
   pre-commit install
   ```

### False Positives

If detect-secrets flags legitimate values:

1. Review the flagged value to confirm it's not a secret
2. Update the baseline:
   ```bash
   detect-secrets scan --update .secrets.baseline
   ```
3. Commit the updated baseline file

### Manual Secret Scan

To manually scan the entire repository:

```bash
detect-secrets scan
```

To scan specific files:

```bash
detect-secrets scan path/to/file
```

## Additional Tools

### Ansible Setup

If you're working with Ansible playbooks, see `ansible/README.md` for Ansible-specific setup instructions.

### Terraform Setup

If you're working with Terraform configurations, see `cloudflare/terraform/README.md` for Terraform-specific setup instructions.
