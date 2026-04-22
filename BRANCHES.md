# Branch Model

The Factory uses a three-branch model optimized for unattended updates via the updater service (landing in Phase 3). Contributors coming from Git-flow or trunk-based development should read this before opening a PR.

## Branches

### `main`
The integration branch. All feature work merges here first. Day-to-day development, code review, and CI run against `main`. `main` is expected to build and pass tests at every commit, but it is not the branch end users run.

### `rolling`
What the updater pulls for users who want the latest features and are willing to hit occasional rough edges. Tracks `main` closely — fast-forwarded to `main` after each merge once CI is green (automated in Phase 3).

### `stable`
What the updater pulls for users who want battle-tested code. Only promoted from `rolling` after a soak period and manual smoke testing on a clean VM. Hotfixes may be cherry-picked directly into `stable` if a regression is caught post-release.

## Workflow

```
feature/xyz ──► main ──► rolling ──► stable
                 │                       ▲
                 └──── hotfix/* ─────────┘
```

1. Branch from `main` for features or fixes: `feature/<name>` or `fix/<name>`.
2. Open a PR to `main`. Tests must pass.
3. After merge, `rolling` is fast-forwarded to `main` (manual today, CI-driven in Phase 3).
4. After a soak period on `rolling` with no new bug reports, `rolling` is promoted to `stable`.
5. Regressions on `stable` are handled with `hotfix/*` branches cut from `stable`, merged back to both `stable` and `main`.

## Which branch should I run?

- **Developers / contributors:** `main`
- **Users who want new features fast:** `rolling` (set by default when you install from scratch today)
- **Users who want minimal surprises:** `stable` (the updater's production channel)

The updater service (Phase 3) will pick a channel via a single `.env` variable: `UPDATE_CHANNEL=rolling` or `UPDATE_CHANNEL=stable`.

## Why not Git-flow or trunk-based?

The updater needs two distinct publishing streams — one conservative, one aggressive — that users can switch between without re-cloning. Git-flow's `develop` / `main` split doesn't model this cleanly; pure trunk-based gives only one stream. `stable` and `rolling` are explicit about the contract each branch offers to end-user installations, which is the primary thing the updater needs to reason about.

## Tagging

Releases on `stable` are tagged `vMAJOR.MINOR.PATCH` (semantic versioning). `rolling` is not tagged — users on `rolling` get whatever is at the tip.
