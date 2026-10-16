# Customizations

Personal fork of [morluto/rea](https://github.com/morluto/rea), tracking upstream, not contributing back. My changes live on the `mattb/main` branch (the fork's default branch, so the scheduled sync runs); `main` stays a clean mirror of `upstream/main`.

`mattb/main` is based on release tag `rea-agents-6.5.0`. Merge release tags into it, not `main` HEAD.

## Changes

_Add entries as you go: date, file, what and why._

- _2026-10-12_ — `CUSTOMIZATIONS.md`, `.github/workflows/sync-upstream.yml` — adoption files. All inherited upstream workflows disabled on the fork.
- _2026-10-12_ — `turbo.json` — `"agentGuidance": false` so turbo stops rewriting the managed block in `AGENTS.md` when it detects an agent (vetting condition).
- _2026-10-12_ — no change needed for Frida: `rea-agents-6.4.0` has no `frida` dependency (it landed on upstream `main` later). Recheck when merging the next tag. 6.4.0 lockfile audited separately: 0 advisories, all registry with integrity, install scripts only esbuild (dev) and fsevents (macOS only).
- _2026-10-12_ — built in an isolated Linux environment; registered for one client at project scope only.
- _2026-10-16_ — merged `rea-agents-6.5.0` (894 commits) into `mattb/main`. `turbo.json`: took upstream's file and kept `"agentGuidance": false` (the only conflict). `package.json`, `package-lock.json`: dropped `frida@17.23.2` from `optionalDependencies` and `allowScripts` (its install script was the only new one) and regenerated the lockfile with `npm install --package-lock-only --ignore-scripts`; only frida's optional subtree left the lockfile and `src/frida/frida-bindings.d.ts` keeps the build typed. Audit flags cleared: setup, launcher and skill diffs read; npm audit 0; the 15 newly code executing tools (incl. `gdb_console`) are inert here because gdb, rizin and cutter are not installed. New workflows disabled on the fork: ci-full-pr, ci-timings (hourly cron), mutation, pr-readiness, pr-title, real-cutter, real-go-binary, real-packet-capture, real-signal-recording, sqlite-runtime. Built in the isolated Linux environment at 2dd73ce05: `npm ci`, `build:cached`, `check:fast` and `test:fast` passed (515 files, 7639 tests, 9 skipped). `main` mirrored by hand to upstream `61e688070`; only Sync upstream is active.

## Syncing upstream

```bash
git fetch upstream --tags
git checkout main && git merge --ff-only upstream/main && git push origin main
git checkout mattb/main && git merge rea-agents-X.Y.Z  # newest release tag; resolve conflicts against your customizations
```

Before every merge into `mattb/main`, diff these paths against the previous tag:

- `package.json` scripts and dependencies, `package-lock.json`
- `install.sh`, `scripts/prepare.mjs`, `scripts/prepack.mjs`, `scripts/rea.mjs`
- `.agents/skills/**`, `AGENTS.md`
- `src/application/SupportedClients.ts`, `SetupSkill.ts`, `SetupHost.ts`
- `.gitmodules`, and any new `https://` host under `src/`

New upstream workflow files arrive enabled after a sync. Disable them on the fork (Actions tab, or `gh workflow disable`).

## License reminder

Upstream license: **MIT** — Permissive. Adapt freely. Attribution required if you redistribute.

## Vetting conditions

Security vetting on 2026-10-12 against commit `8fc2043` returned **CONDITIONAL**. Scout YELLOW (dual use, single npm publisher, young, high velocity), Dependencies SAFE, Code Scan CLEAN, Prompt Injection WARNING (setup writes user scope agent config; skill falls back to `npx -y rea-agents@latest`).

- **Isolate.** Run REA only in WSL2 or a VM, never directly on the primary host. `install.sh` supports only macOS and Linux anyway.
- **Run from the fork only.** Never run `install.sh` or `npx rea-agents...`. Never `--recurse-submodules`, never run anything from `third_party/`.
- **Build.** On `mattb/main`, drop `frida` from `optionalDependencies` and regenerate the lockfile unless Frida is needed (do not use `npm ci --omit=optional`; it drops esbuild and node-pty platform packages). Set `"agentGuidance": false` in `turbo.json`. Export `TURBO_TELEMETRY_DISABLED=1`. Build with `HUSKY=0 npm ci` then `npm run build:cached`.
- **Register by hand, one client.** Preview with `node <fork>/scripts/rea.mjs setup --client claude_code --dry-run --json`. Then `claude mcp add --scope project rea -- node <fork>/scripts/rea.mjs mcp`. Deselect every other detected client. Decline setup's skill install.
- **Skill managed separately.** Keep a reviewed copy of `.agents/skills/reverse-engineer-anything/` in a private skills repo, with every `npx -y rea-agents@latest` line replaced by the fork launcher path (find them with `grep -n npx` on the skill folder; at `rea-agents-6.5.0` they are SKILL.md lines 34, 36, 50, 55, 70 and 78).
- **GitHub Actions.** Keep every inherited workflow disabled (includes npm publish with `id-token: write`, release-please, Cloudflare deploy). Only `sync-upstream.yml` stays enabled.
- **Browser and targets.** Attach only to a dedicated, logged out browser profile inside the VM. Never a profile signed into any real account. Analyze only software you own or are authorized to examine (see `DISCLAIMER.md`).
- **Hopper.** Only pass `--install-hopper` if needed; its integrity check comes from the same vendor API it downloads from. Otherwise use your own Ghidra.
