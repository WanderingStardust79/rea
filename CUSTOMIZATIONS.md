# Customizations

Personal fork of [morluto/rea](https://github.com/morluto/rea), tracking upstream, not contributing back. My changes live on the `mattb/main` branch (the fork's default branch, so the scheduled sync runs); `main` stays a clean mirror of `upstream/main`.

`mattb/main` is based on release tag `rea-agents-6.4.0`. Merge release tags into it, not `main` HEAD.

## Changes

_Add entries as you go: date, file, what and why._

- _2026-10-12_ — `CUSTOMIZATIONS.md`, `.github/workflows/sync-upstream.yml` — adoption files. All inherited upstream workflows disabled on the fork.
- _2026-10-12_ — `turbo.json` — `"agentGuidance": false` so turbo stops rewriting the managed block in `AGENTS.md` when it detects an agent (vetting condition).
- _2026-10-12_ — no change needed for Frida: `rea-agents-6.4.0` has no `frida` dependency (it landed on upstream `main` later). Recheck when merging the next tag. 6.4.0 lockfile audited separately: 0 advisories, all registry with integrity, install scripts only esbuild (dev) and fsevents (macOS only).
- _2026-10-12_ — built in WSL Ubuntu (aarch64) at `/home/paperclip/rea`; registered for Claude Code at project scope in `C:/Users/mattb/re-lab/.mcp.json`.

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

- **Isolate.** Run REA only in WSL2 or a VM, never on the Windows host that holds `~/operator` and Yardi or AppFolio sessions. `install.sh` supports only macOS and Linux anyway.
- **Run from the fork only.** Never run `install.sh` or `npx rea-agents...`. Never `--recurse-submodules`, never run anything from `third_party/`.
- **Build.** On `mattb/main`, drop `frida` from `optionalDependencies` and regenerate the lockfile unless Frida is needed (do not use `npm ci --omit=optional`; it drops esbuild and node-pty platform packages). Set `"agentGuidance": false` in `turbo.json`. Export `TURBO_TELEMETRY_DISABLED=1`. Build with `HUSKY=0 npm ci` then `npm run build:cached`.
- **Register by hand, one client.** Preview with `node <fork>/scripts/rea.mjs setup --client claude_code --dry-run --json`. Then `claude mcp add --scope project rea -- node <fork>/scripts/rea.mjs mcp`. Deselect every other detected client. Decline setup's skill install.
- **Skill via `~/operator`.** Copy `.agents/skills/reverse-engineer-anything/` into `~/operator/skills/`, replace the `npx -y rea-agents@latest` fallback (SKILL.md lines 34 and 35) with the fork launcher path, deploy with `~/operator/scripts/sync.sh`.
- **GitHub Actions.** Keep every inherited workflow disabled (includes npm publish with `id-token: write`, release-please, Cloudflare deploy). Only `sync-upstream.yml` stays enabled.
- **Browser and targets.** Attach only to a dedicated, logged out browser profile inside the VM. Never a profile signed into Yardi, AppFolio, banking, or Gmail. Analyze only software you own or are authorized to examine (see `DISCLAIMER.md`).
- **Hopper.** Only pass `--install-hopper` if needed; its integrity check comes from the same vendor API it downloads from. Otherwise use your own Ghidra.
