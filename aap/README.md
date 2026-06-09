# AAP wiring — OS patching (`patch.yml`)

`patch.yml` refreshes the OS package repos (apt/dnf/yum) and upgrades installed
packages, with an optional reboot. It is driven by a Job Template survey whose
answers arrive as `extra_vars`. Defaults keep it safe: a dry run, non-prod, no
skips, no reboot.

> `site.yml` contains its own integrated pre-update play (gated by
> `run_system_update`) for the Wiz workflow. This survey targets the standalone
> `patch.yml`.

## Survey variables

| Survey field                 | Variable             | Values              | Default     |
|------------------------------|----------------------|---------------------|-------------|
| Target environment           | `target_env`         | `non-prod` / `prod` | `non-prod`  |
| Dry run                      | `dry_run`            | `true` / `false`    | `true`      |
| Skip packages / applications | `skip_packages`      | free text (space/comma separated) | `""` |
| Reboot if required           | `reboot_if_required` | `no` / `yes`        | `no`        |

- **Target environment** is a label / safety gate; a live prod run prints a
  prominent warning. It does not select hosts — see host targeting below.
- **Dry run** uses Ansible `check_mode`, so it reports what *would* upgrade
  without changing anything (and never reboots).
- **Skip** excludes packages from the upgrade (dnf/yum `exclude`; on apt the
  named packages are `hold`/`unhold` around the upgrade). **Leave it blank to
  fall back to each host's inventory value** — see below.
- **Reboot** happens only when `reboot_if_required=yes`, the run is live, and the
  OS actually reports a pending reboot.

## Choosing which hosts to patch

Host selection is done on the Job Template, not the survey:

- **Inventory** attached to the template = the universe of hosts.
- **Limit** field narrows it: a single host (`web01.example.com`), a comma list,
  a group (`prod`), or an intersection (`prod:&web`). Enable **Prompt on Launch**
  on Limit so the operator chooses hosts at launch.

Survey answers apply to the **whole job run** — every targeted host gets the same
`dry_run` / `target_env` / `reboot_if_required`.

## Per-host skip lists (inventory fallback)

When the survey `skip_packages` field is **blank**, each host uses its own
inventory variable `skip_packages_default`. When the survey field is filled, it
overrides inventory for every host in that run.

```yaml
# inventory host_vars/web01.yml  (or group_vars/prod.yml)
skip_packages_default: "kernel nginx"
```

| `skip_packages` (survey) | `skip_packages_default` (inventory) | Effective excludes |
|--------------------------|-------------------------------------|--------------------|
| blank                    | `kernel nginx`                      | `kernel nginx`     |
| `docker-ce`              | `kernel nginx`                      | `docker-ce`        |
| blank                    | unset                               | none               |

## Attach the survey

Either build it from the table above, or import the spec:

```bash
# via awx/AAP CLI
awx job_templates modify "<template name>" --survey_enabled true
awx job_templates modify "<template name>" --survey_spec @aap/patch_survey_spec.json
```

Or against the controller API directly:

```bash
curl -sk -u "$USER:$PASS" \
  -H "Content-Type: application/json" \
  -X POST "https://<controller>/api/v2/job_templates/<id>/survey_spec/" \
  -d @aap/patch_survey_spec.json
# then enable the survey on the template (survey_enabled: true)
```

Start with `dry_run=true` to preview, then re-launch with `dry_run=false` to apply.
