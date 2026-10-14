# CLAUDE.md: Home Server with Voice Access

Read this at the start of every session. Keep it short; detail lives in `docs/`.

## 1. Project Snapshot
- Home Assistant (HA) as the hub for the household's smart-home devices, with voice access.
- 6-month goal: bring everything currently run through Alexa into HA, and move toward local/offline voice (Google Home is an acceptable starting point).
- Users: the owner (software engineer) and spouse. No separate permissions needed.
- Budget: under $500 total. Plan and prices in `docs/roadmap.md`.
- Current phase: see section 13.

## 2. Working Agreement
- Owner describes what they want; Claude drafts; the owner reviews every change before it is committed.
- Every proposal comes as: (a) file path and full contents or a diff, (b) a plain-English summary, (c) a one-line rollback.
- Claude never assumes a step succeeded, and never invents entity IDs, add-on names, or settings it cannot see. If needed, ask the owner to paste the real entity list or output.
- Ask before expanding scope. One task per branch.

## 3. Execution Mode
- **Mode B (active): Claude writes, owner applies.** Claude has no access to the server.
- Each task ends with a "paste me the output" step. A step is unconfirmed until the owner pastes the result.
- Confirmed state of the server is recorded in `docs/applied-changes.md`.

## 4. Session Start Checklist
1. Date of last good backup? (If none or stale, backup comes first.)
2. Is `git status` clean? Any unmerged branches?
3. HA Core / OS version (owner pastes from Settings > About).
4. Re-read section 13 for today's focus.

## 5. Environment
- HA OS on bare metal: Intel i5, 32 GB RAM. Server sits in a closet.
- Bluetooth: single USB dongle on the server (poor range from a closet; see Bermuda note in `docs/voice-roadmap.md` and `docs/roadmap.md`).
- Network: fiber modem (EXOS) plus a simple router. VLAN-capable but not configured. Details in `docs/network.md`.
- Existing platforms: Alexa, Google Home, SmartThings, ecobee, Sonos, Samsung (phone, tablet, refrigerator), TCL TV, Roku, Wyze, Brother laser printer, Google Calendar, MediaMonkey.
- HACS components in use or planned: Mushroom, Bermuda BLE trilateration, LocalThings.

## 6. Repo Layout and Conventions
- Repo root is the HA `/config` directory.
- Config style (hybrid): integrations stay UI-managed. Automations, scripts, helpers, and dashboards are YAML packages tracked in git.
- Never commit: `secrets.yaml`, `.storage/`, `home-assistant_v2.db*`, `*.log`, backups. Keep these in `.gitignore`.
- Entity and helper names: `<room>_<device>_<function>` (e.g. `kitchen_light_main`). Rename existing entities gradually, not in one sweep.
- Secrets go in `secrets.yaml` and are referenced with `!secret`.

## 7. Integration Map
- Full list in `docs/inventory.md`: device, room, protocol, local or cloud, status.
- Rule: prefer local integrations. Flag anything cloud-dependent so we can decide whether to keep it.

## 8. Change Procedure
1. Fresh backup exists (see section 9).
2. New git branch for the task.
3. Claude provides the files; owner applies them.
4. Run Check Configuration; owner pastes the result.
5. Owner reviews the diff.
6. Commit. Reload the specific domain if possible; restart only when required.
7. Verify the behavior; owner pastes confirmation.
8. Merge, and add an entry to `docs/applied-changes.md`.
- Rollback: `git revert` the commit (or restore the file), reload or restart, and restore the backup if config is broken.

## 9. Commands Cheat Sheet
Run in the Terminal & SSH add-on (confirm each tool exists with `--version` or `help` before relying on it).
```
ha core check                    # validate configuration
ha backups new --name "pre-<task>"   # create a backup before changes
ha backups list                  # confirm it exists
ha core restart                  # only when a reload is not enough
git status
git switch -c <task-branch>
git diff
git add <files> && git commit -m "<message>"
git revert <sha>                 # undo a committed change
```
Reloads (automations, scripts, scenes, etc.) are done in Developer Tools > YAML.

## 10. Safety Rules (non-negotiable)
1. **Backups first.** No backups exist today. Set up HA backups plus an off-server copy before any other change.
2. **Validate, version, then restart.** Run Check Configuration and commit to git before restarting. Owner pastes outputs; unconfirmed means not done.
3. **No internet exposure; one network change at a time.** No port forwarding. Remote access only via Tailscale or Nabu Casa. Save current router settings before any router or VLAN change, and plan a rollback.
4. **Custom code runs with full access.** Review the source and maintainers of HACS items (Bermuda, LocalThings, etc.), pin versions, and do not update them on the same day as an HA core update.
5. **Physical-world automations need limits.** Hard bounds on ecobee setpoints, nothing critical (e.g. refrigerator) on a switchable plug, and every automation must fail safe. Test spouse-facing changes before they replace a working manual option.

## 11. Dashboard Standards
- Three main views: **Kitchen** (Sonos, Samsung fridge status, lights, grocery list), **Entertainment** (Sonos, TCL, Roku, MediaMonkey), **Out of House** (away mode, "is anything left on", thermostat setback, calendar, presence).
- Spouse test: anything common is doable in one or two taps without asking the owner.
- Primary screen: the Samsung tablet (wall-mounted later); layouts must also work on phones.
- Main dashboard is YAML in git (Mushroom cards). The visual editor is for experiments only.
- No raw entity lists on main views. Spec in `docs/dashboard-spec.md`.

## 12. Voice Strategy
- Now: Alexa and Google Home remain the voice front-ends.
- Later: local Assist on the server (speech-to-text and text-to-speech run on the i5) with 1-2 satellites. Plan, hardware, and costs in `docs/voice-roadmap.md`.

## 13. Roadmap and Current Focus
- **Current focus: Phase 1.** Backups, git repo and `.gitignore`, `docs/inventory.md` and `docs/network.md` first drafts.
- Later phases (in `docs/roadmap.md`): dashboard views, Bluetooth proxies, tablet mount, voice satellites.

## 14. Decisions and Open Questions
- Decisions go in `docs/decisions.md`.
- Open: which fridge entities SmartThings exposes (owner to paste the list); whether to sync the grocery list with Alexa (default: use HA's built-in list first); router model and VLAN support details.
