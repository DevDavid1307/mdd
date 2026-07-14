# The report page

Markup for `.why-changed/<name>.html`. What each part *means* — commit, account, intent, diff — is defined in `SKILL.md` step 5; this file only says how to render it.

The page is a **timeline**: a rail down the left, one dot per update oldest-first, each dot carrying a card. The card leads with the commit, gives **intent** the body, folds the account into a `<details>` at its foot, and closes on the diff.

## Width

Everything runs the **full width of the window** — cards, intent, account, diff. The container carries padding (`px-8`) and nothing else.

Long Chinese lines look like they want a cap; they don't. A capped column strands a narrow strip of text beside dead space inside an already-wide card, which reads worse than the long line did. If lines ever get unbearable, reach for `columns-2` and keep filling the width.

## Colour

Page and diff both follow the reader's system dark mode, off one signal: `prefers-color-scheme`. Tailwind's `dark:` variant already keys off it, and `colorScheme: "auto"` makes diff2html do the same — its default is `light`, which is what strands a light diff on a dark page.

Timeline dots cycle a rainbow in time order — `rose → amber → emerald → sky → violet → fuchsia`, wrapping past six updates. The colour is **decoration**; it carries no meaning, so **shape** is what an update's state has to speak with:

- **Accounted** — solid disc.
- **Unaccounted** — dashed hollow ring, keeping whatever rainbow slot it landed in. Its `outline` is filled with the page background so the ring *interrupts* the rail; without that, the rail draws straight through the dot's hollow centre.

## Type

The whole page is monospaced — `<body>` carries `font-mono`, everything inherits it. The stack is two local families, no webfont and no generic fallback:

```
"GeistMono Nerd Font Mono", "LXGW WenKai Mono"
```

Geist holds no CJK glyphs, so the split falls out of CSS's per-character fallback on its own: Latin and digits render in Geist, Chinese in LXGW.

LXGW ships a single face. Chinese therefore never takes a weight — a `font-bold` on it buys nothing but the browser's synthesised smear. The two intent headings separate by **colour and size**; that is already enough to tell them apart at a glance. Latin keeps its real weights, since Geist ships nine.

## Diffs

diff2html loads from CDN, one view per update — an update's commits concatenate into a single diff, rendered `line-by-line`. A raw diff rides in `<script type="text/plain">`, where nothing needs escaping — except a literal `</script>` inside a diff, which becomes `<\/script>`.

Each update `N` pairs a `diff-N` source with a `view-N` mount; the boot script wires every pair on load.

Layout and colour inside the view are diff2html's own: leave its `.d2h-*` markup alone and hang no utility classes on it. **Type is the one exception** — diff2html hardcodes `Menlo` on code lines and a sans stack on file headers, both of which would strand the diff outside the page's typeface, so the `@theme` block hands it `inherit` instead.

## The account

**Verbatim** (SKILL.md step 5) does not mean pasting the changeset as plain text: its body is Markdown, and `**bold**`, backticks and `-` lists must render as `<strong>`, `<code>` and `<ul>`, or the reader eats the raw asterisks.

## Skeleton

Tailwind comes from CDN. The page's only stylesheet is the `@theme` block below — it exists to name the typeface and to hand that typeface to diff2html. Everything else is a utility class.

```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="utf-8" />
    <meta name="color-scheme" content="light dark" />
    <title>why-changed · SKILL_NAME</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <link
      rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/diff2html/bundles/css/diff2html.min.css"
    />
    <style type="text/tailwindcss">
      @theme {
        --font-mono: "GeistMono Nerd Font Mono", "LXGW WenKai Mono";
      }
      /* The diff joins the page's typeface; its layout and colour stay its own. */
      .d2h-wrapper,
      .d2h-wrapper * {
        font-family: inherit;
      }
    </style>
  </head>
  <body
    class="min-h-screen w-full bg-slate-50 pb-32 font-mono text-slate-900 dark:bg-slate-950 dark:text-slate-100"
  >
    <div class="w-full px-8 pt-12">
      <header class="pb-4">
        <h1 class="text-2xl font-bold tracking-tight">
          SKILL_NAME
          <span class="ml-2 align-middle text-sm font-normal text-slate-500">
            这次 pull 带进来 N 个变更
          </span>
        </h1>
        <p class="mt-1 text-xs text-slate-500 dark:text-slate-400">
          BEFORE_SHORT..AFTER_SHORT · 生成于 YYYY-MM-DD HH:MM (UTC+8)
        </p>
      </header>

      <!-- The rail. One <section> per update, oldest first. -->
      <div
        class="relative mt-8 border-l-2 border-slate-200 pl-8 dark:border-slate-800"
      >
        <!-- An accounted update. Drop `pb-12` on the last section. -->
        <section class="relative pb-12">
          <!-- Rainbow slot: bg-rose|amber|emerald|sky|violet|fuchsia-500 by position. -->
          <span
            class="absolute top-8 -left-[41px] h-4 w-4 rounded-full border-4 border-slate-50 bg-rose-500 dark:border-slate-950"
          ></span>

          <div
            class="rounded-xl bg-white p-8 shadow-sm ring-1 ring-slate-200 dark:bg-slate-900/60 dark:ring-slate-800"
          >
            <div class="flex flex-wrap items-baseline gap-x-3 gap-y-1">
              <a
                class="text-sm text-blue-600 hover:underline dark:text-blue-400"
                href="REPO_URL/commit/SHA"
                >SHORT_SHA</a
              >
              <span class="text-sm font-medium">COMMIT_SUBJECT</span>
              <span class="ml-auto text-xs text-slate-400 dark:text-slate-500"
                >YYYY-MM-DD HH:MM (UTC+8)</span
              >
            </div>

            <!-- Colour and size carry these two, never weight. -->
            <div class="mt-6 space-y-5">
              <div>
                <h3 class="text-lg text-rose-600 dark:text-rose-400">改之前的问题</h3>
                <p class="mt-1.5 text-slate-700 dark:text-slate-300">…</p>
              </div>
              <div>
                <h3 class="text-lg text-emerald-600 dark:text-emerald-400">
                  这次怎么解决
                </h3>
                <p class="mt-1.5 text-slate-700 dark:text-slate-300">…</p>
              </div>
            </div>

            <details
              class="mt-6 border-t border-slate-200 pt-3 dark:border-slate-800"
            >
              <summary
                class="cursor-pointer text-xs text-slate-500 select-none hover:text-slate-800 dark:hover:text-slate-200"
              >
                作者原话（English）
              </summary>
              <div class="mt-2 text-sm text-slate-600 italic dark:text-slate-400">
                <!-- The changeset body, its Markdown rendered: <strong>,
                     <code class="rounded bg-slate-100 px-1 dark:bg-slate-800">,
                     <ul class="mt-2 list-disc space-y-1 pl-5">. -->
                CHANGESET_BODY_VERBATIM
              </div>
            </details>

            <script type="text/plain" id="diff-1">
RAW_UNIFIED_DIFF
            </script>
            <div class="mt-6" id="view-1"></div>
          </div>
        </section>

        <!-- An unaccounted update: hollow dashed dot, and the account block
             gives way to the amber note. Everything else is identical. -->
        <section class="relative pb-12">
          <span
            class="absolute top-8 -left-[41px] box-content h-3.5 w-3.5 rounded-full border-2 border-dashed border-amber-500 bg-slate-50 outline-4 outline-slate-50 dark:bg-slate-950 dark:outline-slate-950"
          ></span>

          <div
            class="rounded-xl bg-white p-8 shadow-sm ring-1 ring-slate-200 dark:bg-slate-900/60 dark:ring-slate-800"
          >
            <!-- … commit header and intent, exactly as above … -->

            <p
              class="mt-6 border-t border-dashed border-amber-300 pt-3 text-xs text-amber-700 dark:border-amber-700/60 dark:text-amber-500"
            >
              这次变更没有 changeset —— 上面的意图是从 diff 反推的，不是作者自己的说法。
            </p>

            <script type="text/plain" id="diff-2">
RAW_UNIFIED_DIFF
            </script>
            <div class="mt-6" id="view-2"></div>
          </div>
        </section>
      </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/diff2html/bundles/js/diff2html-ui.min.js"></script>
    <script>
      document.querySelectorAll('[id^="diff-"]').forEach((src) => {
        const view = document.getElementById(src.id.replace("diff-", "view-"));
        new Diff2HtmlUI(view, src.textContent, {
          drawFileList: false,
          matching: "lines",
          outputFormat: "line-by-line",
          colorScheme: "auto",
        }).draw();
      });
    </script>
  </body>
</html>
```
