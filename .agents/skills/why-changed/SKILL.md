---
name: why-changed
description: Sync the fork and pull, then read what arrived and why the author changed it — one HTML page per updated skill, diff plus intent.
disable-model-invocation: true
---

# Why Changed

You care about **arrivals** — what this pull dragged in — not what upstream authored today. A fork merges upstream in batches, so a commit written last week arrives today. Filter by the calendar and you get an empty report on the very day a week of work lands.

## 1. Gate on a clean tree

This skill is **read-only** on your repo, but for the two things it owns: the HTML pages, and the one `.gitignore` line that hides them. A dirty tree is the human's to resolve.

Run `git status --porcelain`. If it prints anything, stop: list the dirty paths and say the pull needs a clean tree.

**Done when** the tree is clean, or you have stopped and handed the dirty paths back.

## 2. Sync the fork, pull, and capture the range

`origin` is a **fork**, so upstream's commits reach this clone in **two hops**: upstream → fork on GitHub, then fork → local. `git pull` walks only the second one. Walk the first yourself, or the pull arrives empty however much upstream has written.

1. `git rev-parse HEAD` → `BEFORE`
2. `gh repo view --json owner,name,parent` → the fork, `owner.login/name`, and its upstream, `parent.owner.login/parent.name`
3. `gh repo sync <fork> --source <upstream>` — naming the fork routes this through GitHub's merge-upstream, which fast-forwards the fork, or merges upstream into it when the fork carries commits of its own. (Bare `gh repo sync` targets the local repo instead, and only ever fast-forwards.)
4. `git pull`
5. `git rev-parse HEAD` → `AFTER`

Three conditions end the run right here, each handed back with the failing command's own output: `gh` absent or unauthenticated; `parent` null, so there is no upstream to sync from; a sync refused because upstream and the fork's own commits touch the same lines. That last one is a merge conflict, and it stays the human's to resolve — `resolving-merge-conflicts` is the skill for it. Forcing a refused sync would throw the fork's own commits away, so a refusal is a stop.

If `BEFORE` equals `AFTER`, report that nothing arrived and stop.

`BEFORE..AFTER` is **the range**. Every later step reads only from it.

**Done when** the fork sits on top of upstream and you hold a non-empty range — or you have stopped, handing back the sync's refusal or the empty pull.

## 3. Map the range onto skills

`git diff --name-only BEFORE..AFTER -- skills/` gives the changed paths. Each `skills/<bucket>/<name>/**` path belongs to skill `<name>`; that set of names is the work.

A skill's **scope** — the only paths whose diff belongs on its page:

- `skills/<bucket>/<name>/**`
- `docs/<bucket>/<name>.md`

Discovery runs on `skills/` alone: a change under a skill's own folder is what makes it an arrival. `docs/<bucket>/<name>.md` never triggers a page on its own — it is supplementary, and rides along on the diff of a skill already in the work.

Aggregate files (`README.md`, `.claude-plugin/plugin.json`, `CLAUDE.md`) stay out. A cross-cutting commit would otherwise repeat the same README lines on every skill's page.

**Done when** every changed path under `skills/` is either mapped to a skill name or deliberately dropped.

## 4. Attach the author's account

A changeset file (`.changeset/*.md`) is the **author's account** of a change: prose, no diff, no structured skill name. Accounts attach to skills many-to-many.

- A commit in the range touching both `.changeset/X.md` and a skill's scope makes `X` an account of that skill's change.
- One account can cover many skills — a cross-cutting commit editing eighteen `agents/openai.yaml` files carries one changeset.
- One skill can carry many accounts, when the range holds more than one update to it.
- A skill change no commit ties to an account: search the range's changesets for one naming that skill, in its filename or body. Still nothing, and the update stands **unaccounted** — say so on the page and read intent from the diff alone.
- An account landing on no skill at all (a manifest- or ADR-only change) is dropped.

An **update** is one account plus every commit in the range that carries it, so a three-commit PR is one update, not three. An unaccounted commit is its own update.

**Done when** every changed skill carries its accounts, or is marked unaccounted.

## 5. Write one page per skill

Ensure `.gitignore` lists `.why-changed/`, then write `.why-changed/<name>.html` for each changed skill, overwriting any file already there.

Pages from earlier runs stay on disk, so every page states **when it was generated**, in UTC+8. That timestamp is what tells the reader the page in front of them is this run's.

One section per update, **oldest first**, each carrying four things:

1. **Commit** — time in UTC+8, subject, and a link to `<repository.url from package.json>/commit/<sha>`.
2. **The account** — the changeset body, verbatim, in the author's English.
3. **Intent (简体中文)** — ① what problem existed before the change ② how this change solves it.
4. **The diff** — `git show --format= <sha> -- <scope>` for each of the update's commits, oldest first.

The intent analysis is the only thing on the page that reading the raw changeset would not already give the reader. So do not paraphrase the account back at them — it sits directly above. Reach for what the account leaves out: an account states the change as a *conclusion*, while the problem that provoked it lives in the diff's lines. Read them, and say what the author was fighting.

Render all four through the skeleton in [HTML-REPORT.md](HTML-REPORT.md) — read it before writing the first page.

**Done when** every changed skill has a page carrying its generation timestamp, every update on it has all four parts, and `.gitignore` lists `.why-changed/`.
