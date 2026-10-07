---
name: why-changed
description: Update your mental models of skills delivered by an upstream sync.
disable-model-invocation: true
---

# Why Changed

Create local HTML reports that help the user build or update their
mental model of each skill delivered by an upstream sync.

## 1. Sync and capture arrivals

Start with `git status --porcelain`. If the working tree is dirty,
stop and list the paths that need the user's attention.

Record `BEFORE` from `HEAD`. Confirm this GitHub repository is a fork
with an upstream parent, then sync upstream into the fork and pull
the fork into the current local branch. Record `AFTER` from `HEAD`.
If they match, report that no skill changes arrived and stop.
Otherwise, `BEFORE..AFTER` is this run's range.

## 2. Build the understanding

Find every changed `skills/<bucket>/<name>/` directory in the range.

For each skill, use its before and after versions, diff, history,
and any relevant context to help the user build or update a mental
model of the skill.

Let the update determine what deserves explanation. Ground the
explanation in evidence, distinguishing documented intent from inference.

## 3. Present the mental model

For each skill, use "show-me" to present the understanding developed
in step 2 as `.why-changed/<bucket>-<name>.html` in Simplified Chinese.

Include the skill name, `BEFORE..AFTER`, and generation time.
