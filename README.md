# RHEL 9 Patching Automation

Ansible-based patching automation for RHEL 9 environments with pre/post validation, rollback support, reboot handling, Subscription Manager integration, and audit reporting.

## Repo Structure

```
rhel-patching/
├── inventory/
│   ├── group_vars/          # Group-level variables
│   │   ├── all.yml          # Global defaults
│   │   └── rhel9.yml        # RHEL 9 specific vars
│   └── host_vars/           # Host-level overrides
├── playbooks/
│   ├── patch.yml            # Main patching entry point
│   ├── rollback.yml         # Manual rollback trigger
│   └── report.yml           # Standalone report generation
├── roles/
│   ├── patch_validation/    # Pre/post health checks
│   ├── rhel_patch/          # Core patching logic
│   ├── patch_report/        # HTML/JSON audit report generation
│   └── rollback/            # DNF history rollback
├── library/                 # Custom Ansible modules
├── filter_plugins/          # Custom Jinja2 filters
├── .github/workflows/       # CI/CD pipelines
└── docs/                    # Runbooks and references
```

## Quick Start

### 1. Configure Inventory

```ini
# inventory/hosts
[rhel9]
server01.example.com
server02.example.com

[rhel9:vars]
ansible_user=ansible
ansible_become=true
```

### 2. Configure Variables

Review and adjust `inventory/group_vars/all.yml` for your environment — especially:
- `patch_reboot_policy`
- `patch_subscription_manager`
- `patch_report_dest`

### 3. Run a Patch Job

```bash
# Dry run (check mode)
ansible-playbook playbooks/patch.yml -i inventory/hosts --check

# Full patch run
ansible-playbook playbooks/patch.yml -i inventory/hosts

# Patch with explicit maintenance window tag
ansible-playbook playbooks/patch.yml -i inventory/hosts -e "patch_window=mw_saturday"

# Limit to specific hosts
ansible-playbook playbooks/patch.yml -i inventory/hosts --limit server01.example.com
```

### 4. Rollback

```bash
# Rollback to last known good DNF transaction
ansible-playbook playbooks/rollback.yml -i inventory/hosts --limit server01.example.com
```

## Variable Reference

| Variable | Default | Description |
|---|---|---|
| `patch_reboot_policy` | `if_needed` | `always`, `never`, `if_needed` |
| `patch_reboot_timeout` | `600` | Seconds to wait for reboot |
| `patch_exclude_packages` | `[]` | Packages to exclude from updates |
| `patch_subscription_manager` | `true` | Enable/disable SubMan integration |
| `patch_report_dest` | `/var/log/patching` | Report output path on target |
| `patch_report_fetch` | `true` | Fetch reports back to controller |
| `patch_report_local_dest` | `./reports` | Local path for fetched reports |
| `patch_rollback_enabled` | `true` | Allow rollback role to execute |
| `patch_pre_checks_fail_fast` | `true` | Abort if pre-checks fail |

## Maintenance Windows

Set `patch_window` at inventory or play level. Only used for tagging reports; actual scheduling is handled externally (cron, AWX, etc.).

## Report Output

Reports are written as JSON + HTML per host:
- `/var/log/patching/<hostname>_<timestamp>.json`
- `/var/log/patching/<hostname>_<timestamp>.html`

Fetched reports land in `./reports/` on the controller.

## Satellite / Subscription Manager

Set `patch_subscription_manager: true` and optionally:
```yaml
patch_subman_auto_attach: true
patch_subman_release_lock: "9.4"   # Lock to a specific minor release
```

## Requirements

- Ansible >= 2.14
- Python >= 3.9 on controller
- `community.general` collection
- `redhat.rhel_system_roles` (optional, for advanced SubMan tasks)

```bash
ansible-galaxy collection install -r requirements.yml
```
