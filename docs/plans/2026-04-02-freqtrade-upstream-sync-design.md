# Freqtrade Upstream Sync Design

## Goal

Turn `/Users/daniel/Code/Cryptobot` into a fork-backed `freqtrade` workspace that can receive upstream `stable` updates automatically without touching the user's working branch.

## Approved Decisions

- `/Users/daniel/Code/Cryptobot` is the canonical repository root for the forked `freqtrade` codebase.
- `/Users/daniel/Desktop/Cryptobot` is a deprecated stub and should not be used for active development.
- `origin` points to the user's GitHub fork.
- `upstream` points to `https://github.com/freqtrade/freqtrade.git`.
- `upstream-sync` mirrors `upstream/stable`.
- Active development happens on a separate branch, not on `upstream-sync`.
- Local automation may fetch updates automatically, but it must not rewrite the active branch.

## Branch Model

- `develop`: fork default branch and control branch for GitHub Actions configuration.
- `upstream-sync`: force-synced mirror of `freqtrade/stable`.
- `daniel-dev`: user-facing working branch based on `upstream-sync`.

## Automation Model

- GitHub Actions in the fork fetches `upstream/stable` on a schedule and on manual dispatch.
- The action force-pushes the fetched commit to `origin/upstream-sync`.
- A local macOS LaunchAgent periodically fetches both `origin` and `upstream` so the clone sees fresh refs without altering checked out work.

## Safety Constraints

- Only `upstream-sync` is force-updated automatically.
- The local background job only performs fetch operations.
- Any merge, rebase, or reset from `origin/upstream-sync` into the working branch remains an explicit user action.
