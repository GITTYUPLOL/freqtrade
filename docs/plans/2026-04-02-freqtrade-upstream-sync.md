# Freqtrade Upstream Sync Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Configure the local `freqtrade` fork so upstream `stable` updates land automatically on a dedicated sync branch without rewriting the active work branch.

**Architecture:** Use a fork on GitHub with an `upstream` remote and a scheduled GitHub Actions workflow to force-update `upstream-sync` from `freqtrade/stable`. Separately, install a local macOS LaunchAgent that only fetches remote refs so the local clone stays aware of upstream changes without mutating working branches.

**Tech Stack:** Git, GitHub Actions, macOS `launchd`, `zsh`

---

### Task 1: Bootstrap the fork-backed repository

**Files:**
- Modify: `.git/config`

**Step 1: Verify upstream branches**

Run: `git ls-remote --heads https://github.com/freqtrade/freqtrade.git`
Expected: `stable` appears in the branch list.

**Step 2: Fork the repository**

Run: `gh repo fork freqtrade/freqtrade --clone=false --remote=false`
Expected: fork URL under the authenticated GitHub account.

**Step 3: Clone the fork into the workspace root**

Run: `gh repo clone GITTYUPLOL/freqtrade .`
Expected: local clone with `origin` pointing to the fork.

**Step 4: Verify remotes**

Run: `git remote -v`
Expected: `origin` points to the fork and `upstream` points to `freqtrade/freqtrade`.

### Task 2: Add fork sync automation

**Files:**
- Create: `.github/workflows/upstream-sync.yml`

**Step 1: Write the workflow**

Add a workflow that fetches `upstream/stable` and force-pushes it to `origin/upstream-sync` on `workflow_dispatch` and a schedule.

**Step 2: Verify workflow syntax**

Run: `sed -n '1,200p' .github/workflows/upstream-sync.yml`
Expected: workflow has `workflow_dispatch`, `schedule`, and `git push --force origin refs/remotes/upstream/stable:refs/heads/upstream-sync`.

**Step 3: Commit the workflow**

Run: `git add .github/workflows/upstream-sync.yml && git commit -m "chore: add upstream sync automation"`
Expected: new commit on the control branch.

### Task 3: Create the sync and work branches

**Files:**
- Modify: refs under `.git/`

**Step 1: Fetch upstream stable**

Run: `git fetch upstream stable`
Expected: `upstream/stable` exists locally.

**Step 2: Create and push `upstream-sync`**

Run: `git branch upstream-sync upstream/stable && git push -u origin upstream-sync`
Expected: fork now contains `upstream-sync`.

**Step 3: Create and push the work branch**

Run: `git switch -c daniel-dev upstream-sync && git push -u origin daniel-dev`
Expected: local work branch tracks `origin/daniel-dev`.

### Task 4: Install local fetch automation

**Files:**
- Create: `/Users/daniel/Library/Application Support/Cryptobot/auto-fetch-freqtrade.sh`
- Create: `/Users/daniel/Library/LaunchAgents/com.cryptobot.freqtrade-fetch.plist`

**Step 1: Write the fetch script**

Add a script that enters `/Users/daniel/Code/Cryptobot` and fetches `origin` and `upstream` with pruning.

**Step 2: Install the LaunchAgent**

Load a LaunchAgent that runs the script hourly and at login.

**Step 3: Verify the agent**

Run: `launchctl print gui/$(id -u)/com.cryptobot.freqtrade-fetch`
Expected: loaded service with the configured script path.

### Task 5: Verify the final state

**Files:**
- Modify: local git refs and GitHub fork branches

**Step 1: Verify branch tracking**

Run: `git branch -vv`
Expected: `upstream-sync` and `daniel-dev` exist with the expected upstreams.

**Step 2: Verify fork branches**

Run: `git ls-remote --heads origin`
Expected: `upstream-sync` and `daniel-dev` exist on the fork.

**Step 3: Verify launchd status**

Run: `launchctl print gui/$(id -u)/com.cryptobot.freqtrade-fetch`
Expected: service is loaded.
