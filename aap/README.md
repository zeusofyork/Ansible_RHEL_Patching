# AAP wiring — pre-Wiz OS update

The first play in `site.yml` ("Pre-Wiz OS package update") refreshes the OS package
repos (apt/dnf/yum) and upgrades installed packages before the Wiz collection runs.
It is gated by survey answers delivered as `extra_vars`. Defaults keep it a safe
no-op: `run_system_update=false`, and when enabled it defaults to a dry run.

## Survey variables

| Survey field                 | Variable             | Values              | Default   |
|------------------------------|----------------------|---------------------|-----------|
| Run OS package update first  | `run_system_update`  | `false` / `true`    | `false`   |
| Dry run                      | `dry_run`            | `true` / `false`    | `true`    |
| Skip packages / applications | `skip_packages`      | free text (space/comma separated) | `""` |
| Reboot if required           | `reboot_if_required` | `no` / `yes`        | `no`      |

- **Dry run** uses Ansible `check_mode`, so it reports what *would* upgrade without
  changing anything (and never reboots). `dry_run` also gates the Wiz remediation play.
- **Skip** excludes packages from the upgrade (dnf/yum `exclude`; on apt the named
  packages are `hold`/`unhold` around the upgrade).
- **Reboot** happens only when `reboot_if_required=yes`, the run is live, and the OS
  actually reports a pending reboot.
- The pre-update targets every host in the inventory; use the Job Template **Limit**
  field to narrow it.

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

Start with `run_system_update=true` + `dry_run=true` to preview, then re-launch with
`dry_run=false` to apply.
