# RHEL 9 Patching Runbook

## Pre-Patch Checklist

Before running any patch job:

- [ ] Confirm maintenance window is approved and scheduled
- [ ] Verify target host group in inventory
- [ ] Check Subscription Manager status: `subscription-manager status`
- [ ] Confirm backup or snapshot exists for critical systems
- [ ] Review `patch_exclude_packages` list for any app-specific packages
- [ ] Check disk space manually if in doubt: `df -h /`
- [ ] Verify Ansible connectivity: `ansible <group> -m ping`

---

## Running a Patch Job

### Dry Run First (Always)

```bash
ansible-playbook playbooks/patch.yml -i inventory/hosts --check --diff
```

### Security-only patches

```bash
ansible-playbook playbooks/patch.yml -i inventory/hosts \
  -e "patch_security_only=true"
```

### Skip reboot

```bash
ansible-playbook playbooks/patch.yml -i inventory/hosts \
  -e "patch_reboot_policy=never"
```

### Target a specific host or group

```bash
ansible-playbook playbooks/patch.yml -i inventory/hosts \
  --limit rhel9_non_critical
```

### Tag-based execution

```bash
# Only run pre-patch validation
ansible-playbook playbooks/patch.yml -i inventory/hosts --tags pre_patch

# Only generate report (post-facts must already be set)
ansible-playbook playbooks/patch.yml -i inventory/hosts --tags report
```

---

## Rollback Procedure

### Check DNF history manually

```bash
ansible <host> -m command -a "dnf history list" -i inventory/hosts
```

### Execute rollback

```bash
ansible-playbook playbooks/rollback.yml -i inventory/hosts \
  --limit server01.example.com
```

### Roll back multiple transactions

```bash
ansible-playbook playbooks/rollback.yml -i inventory/hosts \
  --limit server01.example.com \
  -e "patch_rollback_transaction_count=2"
```

---

## Subscription Manager Reference

### Check status

```bash
ansible rhel9 -m command -a "subscription-manager status" -i inventory/hosts
```

### Refresh and list repos

```bash
ansible rhel9 -m command -a "subscription-manager refresh" -i inventory/hosts
ansible rhel9 -m command -a "subscription-manager repos --list-enabled" -i inventory/hosts
```

### Set release lock

```bash
ansible rhel9 -m command -a "subscription-manager release --set=9.4" -i inventory/hosts
```

### Remove release lock

```bash
ansible rhel9 -m command -a "subscription-manager release --unset" -i inventory/hosts
```

---

## Report Locations

- **On target hosts:** `/var/log/patching/<hostname>_<timestamp>.[json|html]`
- **On Ansible controller (fetched):** `./reports/<hostname>_<timestamp>.[json|html]`

---

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Pre-check fails on disk space | `/` partition low | Free space or override `patch_check_disk_min_free_mb` |
| `subscription-manager refresh` fails | Network/proxy issue | Verify connectivity to RHSM/Satellite |
| `dnf check-update` returns 100 but no updates install | Exclude list too broad | Review `patch_exclude_packages` |
| Host unreachable after reboot | Reboot timeout too short | Increase `patch_reboot_timeout` |
| Service check fails post-patch | Service crashed on restart | Check `journalctl -u <service>` on target |
| Rollback fails | Transaction too old or disk full | Run `dnf history` manually and check free space |

---

## Escalation

If automated rollback fails or the system is in an unrecoverable state:

1. Log in directly to the affected host
2. Run `dnf history list` to identify last known good transaction
3. Run `dnf history undo <id>` manually
4. If kernel issue: boot to previous kernel from GRUB menu
5. Escalate to system owner / on-call if not resolved within SLA
