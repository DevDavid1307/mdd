---
name: why-changed
description: Understand what upstream skill updates mean for you as a user.
disable-model-invocation: true
---

# Why Changed

Create a local HTML report for each skill changed by an upstream sync.

## 1. Sync and capture arrivals

Start with `git status --porcelain`. If the working tree is dirty,
stop and list the paths that need the user's attention.

Record `BEFORE` from `HEAD`. Confirm this GitHub repository is a fork
with an upstream parent, then sync upstream into the fork and pull
the fork into the current local branch. Record `AFTER` from `HEAD`.
If they match, report that no skill changes arrived and stop.
Otherwise, `BEFORE..AFTER` is this run's range.

## 2. Understand the update as a user

Find every changed `skills/<bucket>/<name>/` directory in the range.
If none changed, report that no skill changes arrived and stop.

Read the before and after versions, diff, and history from a user's
perspective. Help the user update their mental model.
Ground explanations in evidence.

## 3. Show what it means

For each skill, use "show-me" to present the understanding developed
in step 2 as `.why-changed/<bucket>-<name>.html` in Simplified Chinese.

Include the skill name, `BEFORE..AFTER`, and generation time.
