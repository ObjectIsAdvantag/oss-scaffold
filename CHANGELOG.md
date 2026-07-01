# Changelog

All notable changes to `oss-scaffold` are documented here. The format is based
on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

`oss-scaffold` dogfoods its own methodology: releases go through a branch + PR,
CI, and a `v*` tag on `main`.

> **Pre-1.0:** while on `0.x`, the methodology and file layout may change in
> backward-incompatible ways between minor versions as it stabilizes.

## [Unreleased]

## [0.1.0] - 2026-07-01

### Added

- Initial methodology extracted from `@cisco_open/mcptoolkit-contract`.
- Vendored skill `skill/oss-scaffold/` with `init`/`check`/`update`/`settings`.
- Profiles: `node-npm-lib`, `node-cli`, `generic`.
- Shared governance + methodology docs (`RELEASING.md`, `OSS-PRACTICES.md`).
- `check.mjs` (offline drift), `apply.mjs`, `staleness.mjs`, `settings.mjs`.
- `MANIFEST.json` describing managed vs templated files per profile.

[Unreleased]: https://github.com/ObjectIsAdvantag/oss-scaffold/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/ObjectIsAdvantag/oss-scaffold/releases/tag/v0.1.0
