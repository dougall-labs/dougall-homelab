# Ansible Homelab Configuration

> **Note**: This repository is a work in progress. Some roles may be incomplete or contain hardcoded values that need to be moved to variables.

This directory contains Ansible playbooks and roles for managing the homelab infrastructure. This repository is designed to be open-source while keeping secrets secure and out of version control.

## Structure

```
ansible/
├── ansible.cfg              # Ansible configuration
├── hosts.ini                # Inventory file (host definitions)
├── playbooks/               # Playbooks for deploying service stacks
│   └── unraid.yml          # Unraid server services deployment
├── roles/                   # Reusable Ansible roles for each service
│   ├── immich/             # Immich photo management service
│   │   ├── tasks/
│   │   │   └── main.yml    # Role tasks
│   │   └── templates/
│   │       └── docker-compose.yml
│   ├── watchtower/         # Watchtower container auto-updater
│   │   ├── defaults/
│   │   │   └── main.yml    # Default variables
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── templates/
│   │       └── docker-compose.yml
│   └── unifipoller/        # UniFi network poller
│       └── templates/
│           └── docker-compose.yml
└── group_vars/             # Group-specific variables
    ├── all/                # Variables for all hosts
    │   └── vars.yml
    ├── unraid/             # Unraid host group variables
    │   ├── vars.yml
    │   └── secrets.yml.example  # Example secrets file (safe to commit)
    └── watchtower/         # Watchtower-specific variables
        └── secrets.yml.example  # Example secrets file (safe to commit)
```

## Prerequisites

1. **Ansible**: Version >= 2.9
2. **SSH Access**: Configured SSH keys for all target hosts
3. **Python**: On all target hosts (for Ansible modules)
   - On Unraid, install the *Python 3 for UNRAID* plugin via Community Apps (support thread: https://forums.unraid.net/topic/175402-plugin-python-3-for-unraid-611/)
4. **Ansible Collections**: 
   ```bash
   ansible-galaxy collection install community.docker
   ```
   ```bash
   ansible-galaxy collection install community.general  # needed for 1Password lookups
   ```

## Secrets Management

**IMPORTANT**: This repository is designed for open-source deployment. All secrets are stored in `secrets.yml` files that are excluded from version control via `.gitignore`.

### Setting Up Secrets

1. **Copy example files to create your secrets files:**
   ```bash
   # For watchtower
   cp group_vars/watchtower/secrets.yml.example group_vars/watchtower/secrets.yml
   
   # For unraid (if needed)
   cp group_vars/unraid/secrets.yml.example group_vars/unraid/secrets.yml
   ```

2. **Edit the secrets files with your actual values:**
   ```bash
   # Edit watchtower secrets
   nano group_vars/watchtower/secrets.yml
   
   # Edit unraid secrets (if needed)
   nano group_vars/unraid/secrets.yml
   ```

3. **Verify secrets files are ignored by git:**
   ```bash
   git status
   # secrets.yml files should NOT appear in the output
   ```

### Option 1: 1Password CLI (Recommended)

1. Install the 1Password CLI:
   ```bash
   brew install 1password-cli                # macOS with Homebrew
   # or download from https://developer.1password.com/docs/cli/get-started/
   ```
2. Sign in and start a session (saved in your shell environment):
   ```bash
   op signin
   ```
3. Store secrets in 1Password:
   - Create an item (e.g., Secure Note) named `Homelab Secrets`
   - Add fields like `PIHOLE_WEB_PASSWORD`, `WATCHTOWER_WEBHOOK`, etc.
4. Reference 1Password secrets in Ansible using the `community.general.onepassword` lookup:
   ```yaml
   # Example in group_vars/all/secrets.yml (and keep the file vaulted or gitignored)
   pihole_web_password: "{{ lookup('community.general.onepassword', 'item=Homelab Secrets field=PIHOLE_WEB_PASSWORD') }}"
   ```

### Secrets File Locations

- `group_vars/watchtower/secrets.yml` - Watchtower Discord webhook URL
- `group_vars/unraid/secrets.yml` - Unraid-specific secrets (if needed)
- `group_vars/*/secrets.yml` - Any other group-specific secrets

All `secrets.yml` files are automatically excluded from git via `.gitignore` patterns:
- `ansible/group_vars/*/secrets.yml`
- `ansible/host_vars/*/secrets.yml`

### Example Secrets Files

Example files (`.example` suffix) are safe to commit and show the structure:
- `group_vars/watchtower/secrets.yml.example`
- `group_vars/unraid/secrets.yml.example`

## Usage

### Deploy Unraid Services

Deploy all services configured for the Unraid host:
```bash
ansible-playbook playbooks/unraid.yml
```

This will deploy:
- **Watchtower**: Automatic container updates with Discord notifications
- **Immich**: Self-hosted photo management service

### Test Connectivity

Before deploying, test SSH connectivity to all hosts:
```bash
ansible all -m ping
```

### Dry Run (Check Mode)

Preview changes without applying them:
```bash
ansible-playbook playbooks/unraid.yml --check
```

### Deploy Specific Role

Deploy only a specific service:
```bash
ansible-playbook playbooks/unraid.yml --tags watchtower
# or
ansible-playbook playbooks/unraid.yml --tags immich
```

## Hosts

Current hosts defined in `hosts.ini`:
- **harpe** (unraid): `192.168.1.2` - Unraid media server
- **opi5** (ethereum): `192.168.2.30` - Ethereum node
- **rpi5** (pihole): `192.168.1.15` - DNS/networking services

## Roles

### Watchtower

Automatic container update service with Discord notifications.

**Variables** (defined in `roles/watchtower/defaults/main.yml`):
- `watchtower_image`: Container image (default: `nickfedor/watchtower:latest`)
- `watchtower_container_name`: Container name
- `watchtower_interval`: Check interval in seconds (default: 300)
- `watchtower_cleanup`: Remove old images (default: true)
- `watchtower_rolling_restart`: Restart one container at a time (default: true)

**Secrets** (in `group_vars/watchtower/secrets.yml`):
- `watchtower_notification_url`: Discord webhook URL for notifications

### Immich

Self-hosted photo and video management service.

**Note**: The Immich role currently requires a `.env` template file that should be created based on the [official Immich documentation](https://docs.immich.app/install/docker-compose).

**Deployment Path**: `/mnt/cache/appdata/immich`

### UniFi Poller

UniFi network monitoring and metrics collection.

**Note**: This role currently contains hardcoded credentials that should be moved to variables and secrets files before public deployment.

## Making This Repository Public

Before making this repository public, ensure:

1. ✅ **All `secrets.yml` files are excluded** (check `.gitignore`)
2. ✅ **No hardcoded passwords or API keys** in committed files
3. ✅ **Example files exist** for all secrets (`.example` suffix)
4. ✅ **Sensitive data removed** from:
   - `hosts.ini` (if it contains sensitive info)
   - Template files
   - Default variables
   - Any committed configuration files

### Pre-Public Checklist

- [ ] Verify `git status` shows no `secrets.yml` files
- [ ] Review all template files for hardcoded secrets
- [ ] Review `hosts.ini` for sensitive information
- [ ] Ensure all example files are present and documented
- [ ] Test deployment with example files to ensure it fails gracefully
- [ ] Review commit history for any accidentally committed secrets

## Best Practices

1. **Never commit secrets**: All `secrets.yml` files are gitignored
2. **Use variables**: Replace hardcoded values in roles with variables
3. **Provide examples**: Always include `.example` files for secrets
4. **Test playbooks**: Use `--check` flag to dry-run changes
5. **Documentation**: Keep README updated with role requirements
6. **Version control**: Commit only non-sensitive configuration files

## Troubleshooting

### Secrets Not Loading

If secrets aren't being loaded:
1. Verify the secrets file exists: `ls group_vars/*/secrets.yml`
2. Check the playbook includes the correct `vars_files` path
3. Ensure file permissions are correct: `chmod 600 group_vars/*/secrets.yml`

### Template Errors

If you get template errors:
1. Verify all required variables are defined in defaults or group_vars
2. Check that secrets files contain required secret variables
3. Review role templates for missing variable references

## Contributing

When adding new roles:
1. Create role directory structure under `roles/`
2. Add defaults in `roles/<role>/defaults/main.yml`
3. Create tasks in `roles/<role>/tasks/main.yml`
4. Add templates if needed in `roles/<role>/templates/`
5. Create `secrets.yml.example` in appropriate `group_vars/` location
6. Document the role in this README
7. Ensure no hardcoded secrets in templates or defaults
