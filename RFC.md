# RFC: `oss-scaffold` — consistent OSS setup, management & publishing via a vendored skill/agent

- **Status:** Draft (for review)
- **Owner:** Steve (`github.com/ObjectIsAdvantag`)
- **Source of truth:** `github.com/ObjectIsAdvantag/oss-scaffold` (public)
- **Date:** 2026-07-01

---

## 1. Problem & goal

Steve owns multiple open-source repositories across **several GitHub orgs and a
personal account**, mostly **Node.js published to npm** (a few other languages).
Today the release methodology — proven in `@cisco_open/mcptoolkit-contract` — is
excellent but lives in one repo. The goal is **one canonical methodology**,
applied identically to every repo, for:

- repository **setup** (governance files, CI, publish, dependabot, templates),
- repository **management** (settings, branch protection, security features),
- **CI + tag-driven npm publish** (provenance, dist-tag policy, tag/version check),

with a **hard consistency guarantee** and a **single, simple UX**: *run one
agent to init a fresh repo, or to check/update an existing one.*

### Non-goals
- Forcing npm-specific pieces onto non-Node repos (publish is per-profile/opt-in).
- A hosted platform or central bot with cross-org write tokens. Everything is
  **vendored + pull/agent-driven**, so it works across org boundaries.

## 2. Locked decisions

| Decision | Choice |
|---|---|
| Source-of-truth repo | `github.com/ObjectIsAdvantag/oss-scaffold`, **public** |
| Distribution | **Vendor** the skill into each repo (portable, offline) |
| Versioning | Skill is SemVer-tagged; vendored copy records its version + source and can detect staleness vs upstream |
| Enforcement | **Both**: agent for authoring/repair **+** required **CI drift-check** gate |
| npm auth | **Per-profile option**: OIDC trusted publishing **or** `NPM_TOKEN` |
| Repo settings | Skill **proposes `gh` commands**; applies **only on explicit confirm** |
| Sync | **Agent-run on demand** (no scheduled bot in v1) |
| DCO sign-off | **Universal** (`git commit -s`) across all repos |

## 3. Core idea: the methodology *is* a vendored skill + agent

"Steve's Methodology" ships as a self-contained **skill package** (instructions +
artifacts + scripts) with a thin **agent** that drives it. Onboarding or
realigning any repo = running that agent.

```
oss-scaffold/                          # source of truth (this repo)
  RFC.md                               # this document
  CHANGELOG.md                         # scaffold's own SemVer history (dogfoods the methodology)
  VERSION                              # current scaffold version, e.g. 1.0.0
  MANIFEST.json                        # per-profile file list; managed vs templated; markers
  skill/oss-scaffold/                  # the distributable skill (vendored into targets)
    SKILL.md                           # methodology + compare/apply/settings logic
    VERSION                            # version of THIS skill snapshot
    artifacts/
      shared/                          # all profiles
        RELEASING.md                   # canonical release methodology (the "why")
        OSS-PRACTICES.md               # setup + management conventions
        CONTRIBUTING.md  CODE_OF_CONDUCT.md  SECURITY.md  SUPPORT.md
        .github/PULL_REQUEST_TEMPLATE.md
        .github/ISSUE_TEMPLATE/*
      profiles/
        node-npm-lib/                  # library published to npm
          .github/workflows/ci.yml     # build+test matrix + doc-links + drift-check job
          .github/workflows/publish.yml
          .github/dependabot.yml
          .github/skills/release/SKILL.md
          .github/agents/release-manager.agent.md
          package.scripts.json         # prerelease/test:links/sync-badge fragments
        node-cli/                      # extends node-npm-lib (bin, shell completions)
        generic/                       # non-Node / governance + settings only
    scripts/
      check.mjs                        # offline: managed files vs vendored artifacts → drift report
      apply.mjs                        # write/update files, respect <!-- scaffold:* --> markers
      staleness.mjs                    # online: vendored skill version vs upstream latest tag
      settings.mjs                     # MCP→gh→checklist repo-settings audit + proposed fixes
  agent/oss-scaffold.agent.md          # "init / check / update / settings" entry point
```

When applied to a target repo, the skill is vendored at:
```
<repo>/.github/skills/oss-scaffold/     # copy of skill/oss-scaffold/ at a pinned version
<repo>/.scaffoldrc.json                 # profile, pinned version, overrides, package name
<repo>/.github/workflows/*.yml          # managed files installed from the chosen profile
```

## 4. The agent: four verbs

| Verb | Purpose |
|---|---|
| **init** | Fresh repo: detect language/registry, pick profile, write `.scaffoldrc.json`, vendor the skill, install managed files + templated sections, open a PR. |
| **check** | Compare repo ↔ methodology (offline drift) **and** report skill **staleness** vs upstream. No writes. |
| **update** | Apply the diff from `check` (idempotent; preserves local content between markers); re-vendor skill if stale; open a PR. |
| **settings** | Audit GitHub repo configuration (see §7); propose `gh` fixes; apply only on explicit confirmation. |

Because the skill runs in the maintainer's Copilot session, it is the **author /
repair** surface. The **enforcement** surface is the CI drift-check gate it
installs (§6). Agent = the hands; CI job = the ratchet.

## 5. Managed vs. templated files

- **Managed** (byte-identical, drift-enforced): `ci.yml`, `publish.yml`,
  `dependabot.yml`, `release/SKILL.md`, `release-manager.agent.md`, issue/PR
  templates. Header on each:
  `# Managed by oss-scaffold — edit upstream, not here.`
- **Templated** (local content preserved between markers): `README.md`,
  `AGENTS.md`, `CHANGELOG.md`, `CONTRIBUTING.md`. Convention:
  `<!-- scaffold:release-process:start --> … <!-- scaffold:release-process:end -->`.
- **Overrides:** `.scaffoldrc.json → overrides[]` lets a repo intentionally opt a
  managed file out; the drift-check ignores it (documented exception).

`.scaffoldrc.json`:
```json
{
  "source": "github:ObjectIsAdvantag/oss-scaffold",
  "version": "1.0.0",
  "profile": "node-npm-lib",
  "npmAuth": "oidc",
  "overrides": [],
  "packageName": "@scope/name"
}
```

## 6. Two-tier versioning & guarantee

Vendoring lets us separate the **hard gate** from the **freshness signal**, and
keeps the required CI check **offline and deterministic**:

1. **Drift check (CI gate, offline).** `check.mjs` compares the repo's managed
   files against the **vendored** `.github/skills/oss-scaffold/artifacts/`. No
   network, no token. A PR that hand-edits `publish.yml` **fails CI**. Wired as a
   **required status check** via branch protection (installed by the `settings`
   verb). This is the "100% consistent behavior" guarantee.
2. **Staleness check (agent, online).** The vendored skill records its `version`
   and `source`. `staleness.mjs` fetches the upstream latest SemVer tag via
   `gh`/git and reports e.g. *"vendored 1.0.0, upstream 1.2.0 → run update."*
   `update` re-vendors and re-applies.

`oss-scaffold` is itself SemVer-tagged with a moving major channel; breaking
changes to the methodology → new major; repos migrate deliberately.

## 7. Repo-settings audit (`settings` verb)

Checks configuration that code can't express. Strategy: **GitHub MCP if present
→ `gh` CLI → manual checklist** with exact UI paths for anything not covered or
where auth lacks admin. Per operational safety, it **proposes** the exact `gh`
command per gap and **applies only on explicit confirmation** (shared-infra,
hard to reverse).

Audited (most readable/settable via `gh api`):
- Dependabot **security updates** + **vulnerability alerts**
- **Secret scanning** + **push protection**
- Default branch `main`; **branch protection**: required PR review, required
  status checks (CI + **drift-check**), no force-push
- **Merge button policy: allow merge commits only — disable squash and rebase
  merging.** The release flow tags the reviewed, CI-green SHA on `main`; squash
  and rebase create new SHAs at merge time that were never reviewed/CI-verified,
  breaking the tag-on-merge-commit contract. The `settings` verb enforces:
  `allow_merge_commit=true`, `allow_squash_merge=false`, `allow_rebase_merge=false`.
- Actions permissions + `id-token` allowed (provenance / OIDC)
- License / description / topics present

## 8. Profiles (v1)

- **`node-npm-lib`** — the full methodology: CI matrix (Node 20/22/24) +
  doc-links + drift-check, tag-driven `publish.yml` (provenance, tag/version
  verify, `next`/`latest` dist-tag policy), dependabot, release skill + agent,
  governance, DCO. `npmAuth` = `oidc` | `token`.
- **`node-cli`** — extends `node-npm-lib` with `bin` + shell completions.
- **`generic`** — governance + settings audit only (for non-Node repos), no
  publish; reuses the same `RELEASING.md` process language where applicable.

## 9. Rollout plan

0. **Extract & generalize (dogfood).** Turn `mcptoolkit-contract`'s
   CI/publish/release-skill/agent/docs into the `node-npm-lib`/`node-cli`
   profiles + `shared/`; write `RELEASING.md` + `OSS-PRACTICES.md`. Re-vendor the
   skill back into `mcptoolkit-contract` to prove parity.
1. **Tooling.** `check.mjs`, `apply.mjs`, `staleness.mjs`, `settings.mjs`,
   `MANIFEST.json`, `.scaffoldrc.json` schema, the agent.
2. **Pilot.** `init`/`check`/`update`/`settings` on 1–2 repos across different
   owners; iterate.
3. **Fleet rollout.** Onboard remaining repos; make drift-check a required check.
4. **Hardening.** Non-Node profiles, release-notes automation, optional
   scheduled sync if desired later.

## 10. Risks & open items

- **Skill-in-CI boundary.** Skills run in Copilot, not CI — mitigated by the
  offline drift-check job (no dependency on the agent at merge time).
- **`gh` admin scope.** Some settings need admin; skill degrades to a checklist.
- **Templated-marker discipline.** Local edits outside markers in templated files
  can be clobbered; `apply.mjs` must diff and warn, never silently overwrite.
- **Multi-language creep.** Keep publish logic strictly per-profile so non-Node
  repos aren't burdened.

## 11. Next step

On approval of this RFC: execute **Phase 0** — extract the current repo's proven
setup into `oss-scaffold` profiles + `shared/`, author `SKILL.md` and the agent,
and dogfood by re-vendoring into `mcptoolkit-contract`.
