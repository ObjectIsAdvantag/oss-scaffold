# OSS Practices

> Repository setup and management conventions for ObjectIsAdvantag open-source
> projects. Distributed and kept in sync by
> [oss-scaffold](https://github.com/ObjectIsAdvantag/oss-scaffold).

## Repository files

Every repo carries:

- `README.md`, `LICENSE` (Apache-2.0 unless noted), `CHANGELOG.md`
  (Keep a Changelog format).
- `CONTRIBUTING.md` (DCO), `CODE_OF_CONDUCT.md` (Contributor Covenant 2.1),
  `SECURITY.md`, `SUPPORT.md`.
- `.github/` — CI + publish workflows, `dependabot.yml`, PR + issue templates,
  the release skill and release-manager agent.
- `RELEASING.md` — the release methodology (see that file).
- `.github/skills/oss-scaffold/` — the vendored methodology skill (versioned).
- `.scaffoldrc.json` — profile, pinned scaffold version, overrides, package name.

## GitHub repository settings

Audited and enforced by the `oss-scaffold` agent's `settings` verb (proposes
`gh` commands; applies only on explicit confirmation):

### Merge strategy (critical)

- `allow_merge_commit = true`
- `allow_squash_merge = false`
- `allow_rebase_merge = false`

Squash/rebase create **new SHAs** at merge time that were never reviewed or
CI-verified, which breaks the tag-on-merge-commit release contract.

### Branch protection (`main`)

- Require a pull request before merging (≥ 1 approval).
- Require status checks to pass: CI build/test, doc-links, and **Scaffold Drift
  Check**.
- Require branches to be up to date before merging.
- Disallow force pushes and deletions.

### Security features

- Dependabot **alerts** + **security updates** enabled.
- **Secret scanning** + **push protection** enabled.
- Dependabot version updates via `.github/dependabot.yml` (low-noise: monthly,
  grouped minor/patch, cooldown; majors get individual PRs).

### Metadata

- Default branch `main`; description, topics, and license set.
- Actions enabled with `id-token` permitted (provenance / OIDC).

## Consistency model

- **Managed** files (workflows, dependabot, templates, release skill/agent) are
  byte-identical across repos and enforced by the **Scaffold Drift Check** CI
  job. Editing them locally fails CI.
- **Templated** files (README, CONTRIBUTING, CHANGELOG) keep local content
  between `<!-- scaffold:NAME:start -->` / `<!-- scaffold:NAME:end -->` markers;
  the scaffold owns only the marked regions.
- **Overrides** in `.scaffoldrc.json` let a repo intentionally opt a managed file
  out (e.g. org-mandated governance under a different org).

## Staying current

Run the `oss-scaffold` agent's `check` verb to see drift and whether the
vendored skill is behind the upstream source; `update` re-vendors and re-applies.
