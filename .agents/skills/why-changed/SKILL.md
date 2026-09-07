---
name: why-changed
description: Explain the skill changes delivered by an upstream sync.
disable-model-invocation: true
---

# Why Changed

Create local HTML reports that explain what changed in each skill delivered by an upstream sync, and why it changed.

## 1. Sync and capture arrivals

Start with `git status --porcelain`. If the working tree is dirty, stop and list the paths that need the user's attention.

Record `BEFORE` from `HEAD`. Confirm this GitHub repository is a fork with an upstream parent, then sync upstream into the fork and pull the fork into the current local branch. Record `AFTER` from `HEAD`. If they match, report that no skill changes arrived and stop. Otherwise, `BEFORE..AFTER` is this run's range.

## 2. Show what changed and why

Find every changed `skills/<bucket>/<name>/` directory in the range. For each skill, call the Skill tool with "show-me" and create `.why-changed/<bucket>-<name>.html`.

Use HTML to make clear, in Simplified Chinese, what changed and why it changed. Read the relevant diff and history, then present the evidence needed to support or inspect the explanation. Name the skill, identify `BEFORE..AFTER`, and state when the report was generated.
