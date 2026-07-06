# oss-scaffold

Canonical setup, management, and release methodology for
[@ObjectIsAdvantag](https://github.com/ObjectIsAdvantag)'s open-source
repositories — packaged as a **vendored skill + agent** so any repo can be
initialized, checked, and kept consistent by running one agent.

This repository is the **single source of truth**. The methodology (the "why")
plus the concrete artifacts (workflows, templates, governance, scripts) live
here, are SemVer-tagged, and are **vendored** into each target repo under
`.github/skills/oss-scaffold/`.

## What it guarantees

- Identical **CI** and **tag-driven npm publish** behavior (provenance,
  dist-tag policy, tag/version verification) across every repo.
- A **required CI drift-check** so managed files can't silently diverge.
- Consistent **governance**, **dependabot**, **branch protection**, and repo
  **settings** (merge-commit-only, security features on).
- Universal **DCO** sign-off and **Semantic Versioning**.

## How it's used

Run the `oss-scaffold` agent against any repo:

| Verb | Purpose |
|------|---------|
| `init` | Scaffold a fresh repo from a profile and open a PR. |
| `check` | Report drift (offline) + skill **staleness** vs this source (online). |
| `update` | Apply fixes / re-vendor a newer skill; open a PR. |
| `settings` | Audit GitHub repo settings; propose `gh` fixes (apply on confirm). |

### What `init` does

`init` walks through a short interview, writes `.scaffoldrc.json`, then calls
`apply.mjs` to install all files for the chosen profile.

**Questions asked:**

| Field | Values | Notes |
|-------|--------|-------|
| `profile` | `node-npm-lib` · `node-cli` · `generic` | `node-cli` extends `node-npm-lib`; `generic` is governance-only |
| `packageName` | e.g. `@myorg/mylib` | npm package name |
| `owner` | e.g. `ObjectIsAdvantag` | GitHub user or org |
| `repoUrl` | e.g. `https://github.com/owner/repo` | Used in governance docs |
| `npmAuth` | `oidc` (preferred) · `token` | Selects the publish workflow variant |

**What gets installed** (for `node-npm-lib` / `node-cli`):

- `.github/workflows/ci.yml` — build + test matrix + scaffold drift-check job *(managed)*
- `.github/workflows/publish.yml` — tag-driven npm publish with provenance *(managed)*
- `.github/dependabot.yml` — dependency update schedule *(managed)*
- `.github/agents/oss-scaffold.agent.md` — this agent, for future `check`/`update`/`settings` runs *(managed)*
- `.github/skills/release/SKILL.md` + `.github/agents/release-manager.agent.md` — release workflow *(templated)*
- `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, `SUPPORT.md` — governance docs *(templated, opt-out)*
- `.scaffoldrc.json` — profile, pinned skill version, and the fields above

`generic` skips the workflows and publish artifacts; everything else applies.
After `apply.mjs`, `init` opens a PR and runs the `settings` verb.

### Bootstrap: vendor the skill into a repo

**Prerequisite:** Node.js 18+ (for `npx`). If the target repo has no Node.js yet,
use the git fallback below.

Vendor the skill from this repo at a tag (pin the version you want):

```bash
npx degit ObjectIsAdvantag/oss-scaffold/skill/oss-scaffold#v0.2.2 \
  .github/skills/oss-scaffold
```

**No Node.js yet?** Use git instead:

```bash
git clone --depth 1 --branch v0.2.2 \
  https://github.com/ObjectIsAdvantag/oss-scaffold /tmp/oss-scaffold
cp -r /tmp/oss-scaffold/skill/oss-scaffold .github/skills/oss-scaffold
```

Then run the `oss-scaffold` agent's `init` verb. The agent mode is available from
this source repo — open it alongside the target repo in VS Code, or select it from
any workspace that has already been scaffolded. `init` installs the agent as a
managed file (`.github/agents/oss-scaffold.agent.md`) in the target repo so it is
self-contained for future `check`/`update`/`settings` runs. `update` re-runs the
vendoring step at a newer tag. This is **not** a GitHub *template repository*: the
skill is copied *into* existing repos rather than used to stamp out new ones.

## Layout

```
skill/oss-scaffold/     # the distributable skill (vendored into targets)
  SKILL.md              # methodology + compare/apply/settings logic
  VERSION               # version of this skill snapshot
  MANIFEST.json         # per-profile file list; managed vs templated
  artifacts/            # files installed into target repos
    shared/             # governance + methodology docs (templated / opt-out)
    profiles/           # node-npm-lib, node-cli, generic
  scripts/              # check.mjs, apply.mjs, staleness.mjs, settings.mjs
agent/oss-scaffold.agent.md
VERSION                 # current scaffold version
```

See [RFC.md](RFC.md) for the full design.

## License

Apache-2.0.
