---
name: ouroboros
description: >-
  Run the full self-documenting build loop: read a source (a YouTube/video transcript, doc,
  article, or notes), install a Claude Code skill from GitHub, hot-load it without restarting
  the session, then invoke that skill to produce an artifact (slide deck, website, report) that
  documents its own creation — and publish it with attribution. Use this whenever the user wants
  to turn a tutorial/transcript/video into a working demo of the very skill it describes, build
  something deliberately "meta" or "recursive" that documents how it was made, reproduce a
  "skill eats itself" / "deck that built itself" workflow, or autonomously go from
  source → installed third-party skill → self-referential artifact → GitHub. Trigger on phrases
  like "make it build itself", "recursive demo", "very meta", "close the loop", "turn this
  transcript into a deck using its own skill", "self-documenting", or "do the whole thing
  autonomously and publish it". Prefer this skill even when the user names only one step (e.g.
  "install this skill and make a deck about installing it") — the value is running the loop end
  to end.
---

# Ouroboros — the self-documenting build loop

A workflow for closing a loop: take a source that *describes* a capability, acquire that
capability, then use it to produce an artifact that documents its own creation. The artifact
becomes evidence of the process that made it. The snake eats its tail.

This is what makes the result feel surprising rather than ordinary: **form mirrors subject.**
A deck about making decks, made by the deck-making skill. A site about a tool, built by the tool.
Keep that mirroring in view at every step — it's the whole point, not a gimmick to bolt on at the end.

## The loop

```
  Source ──▶ Install ──▶ Hot-load ──▶ Invoke ──▶ Publish
 (describes   (acquire    (no restart   (produce a    (ship it with
  a skill)    the skill)   needed)       self-doc       attribution)
                                         artifact)
                              ▲                              │
                              └──────── documents ───────────┘
```

## Phase 0 — Scope the loop (do this first, in one pass)

Pin down the four moving parts before touching anything. Infer from the conversation; only ask if
genuinely blocked. If the user said "autonomous" or "don't stop", **decide these yourself** and proceed.

1. **Source** — what describes the skill? A video URL + transcript, a README, an article, notes.
2. **Skill** — the GitHub repo to install, and the **scope**: personal (`~/.claude/skills/`,
   available everywhere) or project (`./.claude/skills/`, travels with the repo). When unsure,
   personal scope is the safe default; add project scope too if the artifact's repo should be
   reproducible by others.
3. **Artifact** — what the installed skill produces (deck, site, report) *about this very process*.
4. **Publish target** — local only, a GitHub repo, GitHub Pages, etc. Default to local unless asked.

## Phase 1 — Read the source

Read the actual source material, not your memory of it. If it's a video, read the transcript file
and note the URL for later attribution. Extract the substance you'll need: the steps, the claims,
the memorable lines, the *thesis*. The artifact you build later should make the source's argument —
so understand the argument.

## Phase 2 — Install the skill

Clone the repo into the chosen scope. A skill is a directory containing `SKILL.md`; many repos are
*also* structured as plugins (a nested `plugins/.../skills/` tree under a `.claude-plugin/`), but the
top-level `SKILL.md`, if present, works directly as a personal-scope skill.

```bash
git clone --depth 1 https://github.com/OWNER/REPO.git ~/.claude/skills/REPO
rm -rf ~/.claude/skills/REPO/.git     # optional: drop history; for project scope, do this so it isn't a nested repo
```

The skill's folder name becomes the skill name — rename it to the clean name you want
(`~/.claude/skills/REPO` → `~/.claude/skills/clean-name`). Inspect `SKILL.md` so you know its real
workflow before invoking it.

## Phase 3 — Hot-load (no restart)

Dropping a folder into the skills directory triggers a rescan — the agent notices the new skill
**without exiting the session**. This is the same effect `/reload-plugins` forces. You generally
can't fire that slash command yourself, but the rescan that the rename/copy triggers achieves it;
confirm the new skill now appears as available, and tell the user it's loaded.

## Phase 4 — Invoke to produce the self-documenting artifact

Invoke the freshly installed skill and follow *its* workflow — don't reimplement it. Feed it the
content from Phase 1, but bend the subject back on itself: the artifact is **about the loop you are
running right now** (installing the skill, hot-loading it, invoking it to make this artifact).

- If the user asked for autonomy, make the skill's interactive choices yourself (purpose, length,
  style, etc.) instead of stopping to ask. Pick decisively; a strong committed choice beats a timid
  average.
- Lean into the mirror: reference the real commands you ran, the real loop, the real recursion.
- **Verify empirically.** If the artifact renders (HTML, a site), open it in a headless browser,
  screenshot key views, and check for overflow/broken layout/low quality — then fix what you find.
  "Looks right" is not verification; a screenshot is. Match the bar the user set ("jaw-dropping"
  means iterate until it actually is).

## Phase 5 — Publish with attribution

- Write a short **README** that foregrounds the two things that make this interesting: the
  **recursion** (the artifact documents its own creation) and the **autonomy** (it was produced end
  to end from a single instruction). Quote the triggering prompt if there was one.
- **Attribute honestly.** Credit the source (video title + URL) and the installed skill (repo + author).
  The artifact stands on someone else's tool — say so prominently. Don't cite private/raw source
  files the user wants kept out (e.g. scraped transcripts); cite the canonical public source instead.
- Keep secrets and bulky/raw source notes out of the repo (`.gitignore` them).
- If publishing to GitHub, use the `gh` CLI: confirm the active account, `git init` → commit →
  `gh repo create OWNER/NAME --public --source=. --push`. End commit messages and PR bodies with the
  required co-author/attribution trailer if the environment specifies one.
- GitHub serves `.html` as raw source, not a live page. To make a web artifact actually *viewable*
  from the README, enable **GitHub Pages** and link a clickable poster image (a screenshot) to the
  live URL — an inline `<iframe>` won't survive README sanitization. Add an `index.html` redirect so
  the Pages root loads the artifact.

## Principles

- **Recursion is the feature.** If the artifact doesn't visibly document its own making, you've
  built an ordinary thing, not an ouroboros. Make the self-reference explicit.
- **Autonomy when asked.** "Autonomous / don't stop" means decide and proceed through every gate;
  surface results, not a queue of questions.
- **Attribution always.** Reusing someone's skill is the point — name them clearly. Never imply you
  authored a tool you cloned.
- **Verify, don't assume.** Render and look. Screenshots over vibes.
- **Don't reinvent the inner skill.** Phase 4 runs the installed skill's own process; your job is to
  drive the loop around it, not to rewrite it.

## Common pitfalls

- Treating it as a normal build and forgetting the self-reference — the result falls flat.
- Reimplementing the installed skill instead of invoking it.
- Leaving the installed skill's nested `.git` inside a project-scope copy (it becomes a stray repo).
- Citing raw/private source files the user deliberately set aside, instead of the public source.
- Declaring a rendered artifact "done" without ever opening it.
