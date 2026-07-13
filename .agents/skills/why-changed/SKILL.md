---
name: why-changed
description: Pull upstream, then read what arrived and why the author changed it — one HTML page per updated skill, diff plus intent.
disable-model-invocation: true
---

# Why Changed

Upstream moved. You want to know what landed in your fork, and why the author did it.

You care about **arrivals** — what this pull dragged in — not what upstream authored today. A fork merges upstream in batches, so a commit written last week arrives today. Filter by the calendar and you get an empty report on the very day a week of work lands.

## 1. Gate on a clean tree

Run `git status --porcelain`. If it prints anything, stop: list the dirty paths and say the pull needs a clean tree. Do not stash, do not commit, do not merge.

This skill reads. Its only write is HTML.

**Done when** the tree is clean, or you have stopped and handed the dirty paths back.

## 2. Pull, and capture the range

1. `git rev-parse HEAD` → `BEFORE`
2. `git pull`
3. `git rev-parse HEAD` → `AFTER`

If `BEFORE` equals `AFTER`, report that nothing arrived and stop. Write no files, touch nothing on disk.

`BEFORE..AFTER` is **the range**. Every later step reads only from it.

**Done when** you hold a non-empty range, or have reported no arrivals.

## 3. Map the range onto skills

`git diff --name-only BEFORE..AFTER -- skills/` gives the changed paths. Each `skills/<bucket>/<name>/**` path belongs to skill `<name>`; that set of names is the work.

A skill's **scope** — the only paths whose diff belongs on its page:

- `skills/<bucket>/<name>/**`
- `docs/<bucket>/<name>.md`

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

Write `.why-changed/<name>.html` for each changed skill, overwriting any file already there. Add `.why-changed/` to `.gitignore` if it is not already listed.

Pages from earlier runs stay on disk, so every page states **when it was generated**, in UTC+8. That timestamp is what tells the reader the page in front of them is this run's.

The page heads with the skill name, the range as short hashes, and that generation time. Its body is one section per update, **oldest first**, each section carrying four things in order:

1. **Commit** — time in UTC+8, subject, and a link to `<repository.url from package.json>/commit/<sha>`.
2. **The account** — the changeset body, verbatim, in the author's English.
3. **Intent (简体中文)** — ① what problem existed before the change ② how this change solves it.
4. **The diff** — `git show --format= <sha> -- <scope>` for each of the update's commits, oldest first, concatenated into one diff2html view.

The intent analysis is the only thing on the page that reading the raw changeset would not already give the reader. So do not paraphrase the account back at them — it sits directly above. Reach for what the account leaves out: an account states the change as a *conclusion*, while the problem that provoked it lives in the diff's lines. Read them, and say what the author was fighting.

**Done when** every changed skill has a page, and every update on it has all four parts.

## The HTML

diff2html from CDN, one view per update. Raw diffs ride in `<script type="text/plain">` so nothing needs escaping — except a literal `</script>` inside a diff, which becomes `<\/script>`.

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/diff2html/bundles/css/diff2html.min.css" />
<script src="https://cdn.jsdelivr.net/npm/diff2html/bundles/js/diff2html-ui.min.js"></script>

<script type="text/plain" id="diff-1">
diff --git a/... (raw unified diff)
</script>
<div id="view-1"></div>

<script>
  document.querySelectorAll('[id^="diff-"]').forEach((src) => {
    const view = document.getElementById(src.id.replace("diff-", "view-"));
    new Diff2HtmlUI(view, src.textContent, {
      drawFileList: false,
      matching: "lines",
      outputFormat: "side-by-side",
    }).draw();
  });
</script>
```

Style the page yourself: readable prose column, the account visually distinct from the intent, sections separated so a reader scrolling down feels the chronology.
