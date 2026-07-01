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
