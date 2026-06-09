# Linux Patch Survey

What to enter in the AAP survey builder for `patch.yml`
(**Templates → your job template → Survey → +Add**). One entry per field, in
this order. The same four fields can be imported in one shot from
`aap/patch_survey_spec.json` (see `aap/README.md`).

---

## 1. Target environment
| Setting | Value |
|---|---|
| **Question** | `Target environment` |
| **Description** | `Environment label / safety gate. A live prod run prints a warning. Make sure the inventory/limit matches.` |
| **Answer variable name** | `target_env` |
| **Answer type** | Multiple Choice (single select) |
| **Multiple Choice Options** | `non-prod` / `prod` (one per line) |
| **Default answer** | `non-prod` |
| **Required** | On |

## 2. Dry run
| Setting | Value |
|---|---|
| **Question** | `Dry run` |
| **Description** | `When true, only reports what would change (check_mode) and never reboots.` |
| **Answer variable name** | `dry_run` |
| **Answer type** | Multiple Choice (single select) |
| **Multiple Choice Options** | `true` / `false` (one per line) |
| **Default answer** | `true` |
| **Required** | On |

## 3. Skip packages / applications
| Setting | Value |
|---|---|
| **Question** | `Skip packages / applications` |
| **Description** | `Space- or comma-separated packages to exclude (e.g. kernel docker-ce nginx). Leave BLANK to fall back to each host's inventory value (skip_packages_default).` |
| **Answer variable name** | `skip_packages` |
| **Answer type** | Text |
| **Minimum length** | `0` |
| **Maximum length** | `1024` |
| **Default answer** | *(leave empty)* |
| **Required** | Off |

## 4. Reboot if required
| Setting | Value |
|---|---|
| **Question** | `Reboot if required` |
| **Description** | `Reboot after upgrade only when the OS reports one is needed. Ignored during a dry run.` |
| **Answer variable name** | `reboot_if_required` |
| **Answer type** | Multiple Choice (single select) |
| **Multiple Choice Options** | `no` / `yes` (one per line) |
| **Default answer** | `no` |
| **Required** | On |

---

## Not survey fields

These are configured outside the survey, so don't look for them there:

- **Which hosts to patch** — set on the template itself: the **Inventory** field
  plus the **Limit** field. Enable **Prompt on Launch** on Limit to choose hosts
  at launch time (e.g. `web01.example.com`, `prod`, or `prod:&web`).
- **Per-host skip lists** — not a survey field; set `skip_packages_default` in the
  inventory's `host_vars` / `group_vars`. It is used only when the survey
  **Skip packages** field is left blank.

## Quick reference — variable summary

| Survey field                 | Variable             | Values              | Default     |
|------------------------------|----------------------|---------------------|-------------|
| Target environment           | `target_env`         | `non-prod` / `prod` | `non-prod`  |
| Dry run                      | `dry_run`            | `true` / `false`    | `true`      |
| Skip packages / applications | `skip_packages`      | free text           | `""`        |
| Reboot if required           | `reboot_if_required` | `no` / `yes`        | `no`        |
