# Branch & Release Policy

This is the source of truth for the template's branch model, PR targeting rules, protection expectations, and release promotion flow. Read [`LIFECYCLE.md`](./LIFECYCLE.md) alongside it.

## Branch model

| Branch / prefix | Purpose | Created by | Lifecycle? |
|---|---|---|---|
| `feature/<n>-<slug>` | New capability or enhancement | `issue-start-branch.yml` | yes |
| `bug/<n>-<slug>` | Defect fix | `issue-start-branch.yml` | yes |
| `doc/<n>-<slug>` | Documentation-only change | `issue-start-branch.yml` | yes |
| `chore/<n>-<slug>` | CI, build, refactor, tooling | `issue-start-branch.yml` | yes |
| `dev` | Integration branch | protected long-lived branch | yes |
| `next` | Staging / pre-release branch | protected long-lived branch | yes |
| `main` | Production branch | protected long-lived branch | yes |
| `hotfix/*`, `dependabot/*`, `copilot/*`, `claude/*` | Bot and exception flows | respective bots / maintainers | exempt from branch-tag enforcement |

The lifecycle prefix is chosen from the issue's `type:*` label when `status:in-progress` is applied, so branch prefix and issue type stay aligned.

## PR targeting rules (enforced by `pr-base-policy.yml`)

1. **Lifecycle work targets `dev`.** PRs opened against unsupported base branches are reassigned to `dev`.
2. **Only `dev` promotes into `next`.** Any other head branch targeting `next` is redirected back to `dev`.
3. **Only `next` releases into `main`.** PRs into `main` from any head other than `next` are closed.

## Branch protection and ruleset expectations

`main`, `dev`, and `next` should all be:

1. non-deletable
2. non-force-pushable
3. review-gated
4. status-check gated

This template includes ruleset/bootstrap assets so those protections can be applied consistently.

## Branch-tag enforcement (`pr-branch-enforcement.yml`)

When a PR is opened from a branch that is not one of the lifecycle prefixes and not an exempt automation branch, the workflow:

1. infers work type from the PR title's conventional-commit prefix
2. opens a tracking issue under the `⚙️ EPIC: Workflow` milestone
3. comments on the PR with the tracking issue link

Type inference uses the same conventional-commit vocabulary as `release-tag.yml`, so issue labels and semver signals stay aligned.

## Dynamic issue title prefixes

| Prefix | Label | When |
|---|---|---|
| `[BUG]` | `type:bug` | fix work inferred from `fix:` / `bug:` titles |
| `[FEATURE]` | `type:feature` | feature work inferred from `feat:` titles |
| `[DOC]` | `type:docs` | docs work inferred from `docs:` titles |
| `[CHORE]` | `type:chore` | default for everything else |
| `[ci-failure]` | `ci-failure, bug` | `ci-failure-diagnose.md` |
| `[release]` | `release` | `dev-to-next.md` |
| `[qa]` | `qa, e2e` | `pr-merged-qa-scenarios.md` |

## Release flow: `dev → next → main`

1. **Stage 8 (`dev-to-next.md`)** — when validation passes on `dev`, an agentic workflow opens a `dev → next` PR and updates release-facing docs such as `CHANGELOG.md`, `README.md`, and `CLAUDE.md`.
2. **Stage 11 (`next-to-main-wiki.md`)** — when the `next → main` PR opens, an agentic workflow updates release documentation on the PR branch.
3. **Merge** — a maintainer merges `next → main`.
4. **Stage 12 (`release-tag.yml`)** — on merge, the workflow computes the next semver tag, creates a GitHub Release, and stamps `CHANGELOG.md` when present.

### Semantic versioning (`release-tag.yml`)

| Signal | Bump |
|---|---|
| `BREAKING CHANGE` in any body, or a `<type>!:` subject | **major** |
| any `feat:` / `feature:` subject | **minor** |
| any `fix:` subject (or default) | **patch** |

If no earlier tag exists, versioning starts from `v0.0.0`.

## Cheat sheet

```text
issue (type:feature) ──status:in-progress──▶ feature/123-slug
                                              │
                                              ▼
                                  PR feature/123 → dev
                                              │
                                              ▼  Validation green on dev
                                  PR dev → next
                                              │
                                              ▼
                                  PR next → main
                                              │  merge
                                              ▼
                                  tag vX.Y.Z + Release
```

## Related files

- `.github/LIFECYCLE.md`
- `.github/workflows/pr-base-policy.yml`
- `.github/workflows/pr-branch-enforcement.yml`
- `.github/workflows/dev-to-next.md`
- `.github/workflows/next-to-main-wiki.md`
- `.github/workflows/release-tag.yml`
- `.github/workflows/issue-start-branch.yml`
- `.github/scripts/create-triage-labels.sh`
