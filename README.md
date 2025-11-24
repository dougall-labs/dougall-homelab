# homelab
Repo for misc homelab config and IaC.

## Getting Started

**New to this repository?** Start with the [SETUP.md](SETUP.md) guide for environment setup instructions.

## Secret Detection

This repository uses [detect-secrets](https://github.com/Yelp/detect-secrets) to prevent secrets from being committed to version control. A pre-commit hook automatically scans files for potential secrets before each commit.

### Quick Setup

1. Install detect-secrets and pre-commit:
   ```bash
   brew install detect-secrets pre-commit
   ```

2. Install the git hooks:
   ```bash
   pre-commit install
   ```

3. Test the setup:
   ```bash
   pre-commit run --all-files
   ```

For detailed setup instructions, see [SETUP.md](SETUP.md).

### Baseline File

The `.secrets.baseline` file contains known false positives and should be committed to the repository. When detect-secrets finds a new potential secret, you can update the baseline:

```bash
detect-secrets scan --update .secrets.baseline
```

**Note**: Only update the baseline if you're certain the detected value is not a secret (e.g., example values, test data, etc.).
