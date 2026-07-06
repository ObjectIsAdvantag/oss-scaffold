# AGENTS.md — working in `oss-scaffold`

This repository is the **source of truth** for ObjectIsAdvantag's open-source
methodology. It is not an application; it is a **vendored skill + agent** plus the
artifacts they install into other repositories.

Read [README.md](README.md) for the overview and [RFC.md](RFC.md) for the full
design rationale.

## What this repo contains

```
skill/oss-scaffold/     # the distributable skill (this is what gets vendored)
  SKILL.md              # methodology + how the verbs work
  VERSION               # skill snapshot version (matches the repo VERSION on release)
  MANIFEST.json         # per-profile file list; managed vs templated
  artifacts/            # the files installed into target repos
    shared/             # governance + methodology docs (templated / opt-out)
    profiles/           # node-npm-lib, node-cli, generic
  scripts/              # check.mjs, apply.mjs, staleness.mjs, settings.mjs (Node built-ins only)
agent/oss-scaffold.agent.md   # init / check / update / settings entry point
VERSION                 # current scaffold version (SemVer)
CHANGELOG.md            # Keep a Changelog; follows the same branch + PR + tag release flow
```

## Core concepts (do not break these)

- **Managed files are byte-identical across every target repo.** They must
  contain **no per-repo values** (no package names, no owners) so the offline
  drift check (`check.mjs`) can compare them verbatim. Anything repo-specific
  belongs in a **templated** file (between `<!-- scaffold:NAME:start/end -->`
  markers) or in the target's `.scaffoldrc.json`.
- **The vendored unit is `skill/oss-scaffold/`.** Scripts resolve paths relative
  to their own location: `scripts/` → skill dir → repo root is
  `../../..`. `MANIFEST.json` and `VERSION` therefore live **inside** the skill
  dir so they travel with it when vendored.
- **`MANIFEST.json` drives everything.** It maps each profile's `managed` and
  `templated` entries to source artifacts, supports `variants` (e.g. `npmAuth`:
  `token` | `oidc`), and profile `extends`.

## Common tasks

### Change a managed artifact (workflow, template, release skill…)
1. Edit the file under `skill/oss-scaffold/artifacts/…`.
2. Keep it **generic** — no per-repo strings. If you need a per-repo value, make
   it templated instead and add markers.
3. Validate scripts still parse: `node --check skill/oss-scaffold/scripts/*.mjs`.
4. Bump the version and add a CHANGELOG entry.

### Add or change a profile
1. Edit `skill/oss-scaffold/MANIFEST.json` (`managed`, `templated`, `extends`,`
   `packageScripts`).
2. Add the referenced artifacts under `artifacts/…`.
3. Test against a temp repo for that profile (see **Test harness** below).

### Test harness (always run before releasing)
```bash
# Simulate a target repo, apply, and confirm zero drift.
rm -rf /tmp/testrepo && mkdir -p /tmp/testrepo/.github/skills
cp -r skill/oss-scaffold /tmp/testrepo/.github/skills/oss-scaffold
cp skill/oss-scaffold/artifacts/.scaffoldrc.example.json /tmp/testrepo/.scaffoldrc.json
printf '{\n  "name": "@x/y",\n  "version": "0.1.0",\n  "scripts": {}\n}\n' > /tmp/testrepo/package.json
node /tmp/testrepo/.github/skills/oss-scaffold/scripts/apply.mjs
node /tmp/testrepo/.github/skills/oss-scaffold/scripts/check.mjs --ci   # must exit 0
```

## Release process

This repo follows the same branch + PR + tag flow the scaffold ships, but
**without** npm publish (oss-scaffold is not published to npm; it is vendored
via `degit`). The CI workflow runs on the PR branch; the `v*` tag on `main`
is the release record, not a publish trigger.

1. Branch `release/X.Y.Z`; bump **both** `VERSION` and
   `skill/oss-scaffold/VERSION` to the same value; roll `CHANGELOG.md`.
2. Commit signed off (`git commit -s`); open a PR; wait for CI + review.
3. Merge with a **merge commit**; tag `vX.Y.Z` on `main` and push.

**Pre-1.0:** minor versions may include backward-incompatible methodology
changes until the format stabilizes.

## Invariants / guardrails

- Never put per-repo values in managed files.
- Keep `VERSION` and `skill/oss-scaffold/VERSION` in lockstep.
- Scripts use **Node built-ins only** (no dependencies) so they run in any repo's
  CI without `npm install`.
- Governance in `shared/` is **opt-out** and only applied if already present in a
  target (so org-mandated files are never clobbered).
