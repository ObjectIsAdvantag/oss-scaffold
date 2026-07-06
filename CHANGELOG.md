# Changelog

All notable changes to `oss-scaffold` are documented here. The format is based
on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

`oss-scaffold` follows its own branch + PR + tag release flow (without npm
publish — the repo itself uses the `node-npm-lib` profile with the publish
workflow overridden).

> **Pre-1.0:** while on `0.x`, the methodology and file layout may change in
> backward-incompatible ways between minor versions as it stabilizes.

<!-- toc -->

- [[Unreleased]](#unreleased)
- [[0.2.2] - 2026-07-06](#022---2026-07-06)
- [[0.2.1] - 2026-07-02](#021---2026-07-02)
- [[0.2.0] - 2026-07-01](#020---2026-07-01)
- [[0.1.0] - 2026-07-01](#010---2026-07-01)

<!-- tocstop -->

## [Unreleased]

## [0.2.2] - 2026-07-06

### Added

- README: new "What `init` does" section documenting the interview questions
  (`profile`, `packageName`, `owner`, `repoUrl`, `npmAuth`), the files installed
  per profile, and the managed-vs-templated distinction.

### Changed

- README bootstrap section now clarifies that the `OSS Scaffold` agent mode comes
  from this source repo and that `init` installs it as a managed file
  (`.github/agents/oss-scaffold.agent.md`) in the target repo, making it
  self-contained for future runs.

## [0.2.1] - 2026-07-02

### Fixed

- Add `oss-scaffold.agent.md` to the managed artifacts installed into target
  repos (`.github/agents/oss-scaffold.agent.md`). The agent was present in the
  source repo but missing from the MANIFEST, so target repos had the skill but
  no agent mode to invoke it. Added to both `node-npm-lib` and `generic` profiles
  (`node-cli` inherits it via `extends: node-npm-lib`).

## [0.2.0] - 2026-07-01

### Added

- Placeholder rendering in `apply.mjs`: templated files may use `{{owner}}`,
  `{{repoUrl}}`, `{{packageName}}`, `{{version}}`, `{{npmAuth}}`, substituted from
  `.scaffoldrc.json` (unresolved placeholders are reported, never emptied).
- `owner` and `repoUrl` fields in `.scaffoldrc.json` (see the example config).
- Preflight in `settings.mjs`: detects the repo host from the git remote
  (normalizing SSH aliases) and runs `gh auth status -h <host>` first, failing
  early with actionable guidance when the GitHub API side isn't authenticated.
- "Working across multiple GitHub accounts (git + gh)" section in
  `OSS-PRACTICES.md` (SSH host aliases, `includeIf`, `gh auth switch`, direnv,
  and the `GH_TOKEN`/`GITHUB_ENTERPRISE_TOKEN` override caveat).
- Bootstrap/vendoring instructions (`npx degit …#v0.2.0`) in the README and skill
  `SKILL.md`, clarifying that oss-scaffold is vendored into repos rather than
  used as a GitHub template repository.

### Changed

- The **release skill** and **release-manager agent** are now **templated**
  (previously managed): a synced core lives between
  `<!-- scaffold:release-core -->` / `<!-- scaffold:release-agent -->` markers,
  with a project-specific section outside the markers for repo-unique steps
  (e.g. schema bumps). Repos no longer need a full override to add release steps.
- Issue `config.yml` is now templated with a `{{repoUrl}}` placeholder instead of
  a hard-coded owner URL.

## [0.1.0] - 2026-07-01

### Added

- Initial methodology extracted from `@cisco_open/mcptoolkit-contract`.
- Vendored skill `skill/oss-scaffold/` with `init`/`check`/`update`/`settings`.
- Profiles: `node-npm-lib`, `node-cli`, `generic`.
- Shared governance + methodology docs (`RELEASING.md`, `OSS-PRACTICES.md`).
- `check.mjs` (offline drift), `apply.mjs`, `staleness.mjs`, `settings.mjs`.
- `MANIFEST.json` describing managed vs templated files per profile.

[Unreleased]: https://github.com/ObjectIsAdvantag/oss-scaffold/compare/v0.2.2...HEAD
[0.2.2]: https://github.com/ObjectIsAdvantag/oss-scaffold/compare/v0.2.1...v0.2.2
[0.2.0]: https://github.com/ObjectIsAdvantag/oss-scaffold/compare/v0.1.0...v0.2.0
[0.2.1]: https://github.com/ObjectIsAdvantag/oss-scaffold/compare/v0.2.0...v0.2.1
[0.1.0]: https://github.com/ObjectIsAdvantag/oss-scaffold/releases/tag/v0.1.0
