---
name: vhs-demo-videos
description: Create terminal demo videos as code with VHS .tape scripts, including deterministic waits, off-camera prep, design-token theming, and render hygiene. Use when making or editing a docs video, terminal recording, CLI demo, or any .tape file.
---

# VHS demo videos

How to write `.tape` scripts for [VHS](https://github.com/charmbracelet/vhs)
that render clean, deterministic, re-runnable terminal videos. Each rule
either follows the VHS documentation (linked) or is a field convention from
rendering a full documentation video set, marked as such.

The core idea: the video is an artifact of the script, like a build is an
artifact of source. Commit the tape, render the video, and a future change
is a re-render, not a re-shoot.

## Tape anatomy

A tape has three parts, in this order:

1. **A comment header documenting the prep** (field convention). Six months
   later this comment is the difference between a re-render and
   archaeology. Include: the output path, the exact render command, and
   every environment precondition (caches to warm, files to stage, servers
   to stop, logins required).
2. **`Output` and `Set` commands.** `Require` and all `Set` commands except
   `Set TypingSpeed` must come before any action command, per the
   [command reference](https://github.com/charmbracelet/vhs#vhs-command-reference).
3. **The actions**: `Type`, `Enter`, `Sleep`, `Wait`, `Hide`/`Show`.

```tape
# Install flow for the docs.
# Renders to public/videos/docs/install.mp4. Re-render with:
#   vhs scripts/videos/install.tape
# Prep: stop the local dev server first (the video binds the same port);
# run npm install once beforehand so the cache is warm.

Output public/videos/docs/install.mp4

Set Shell zsh
Set FontFamily "Geist Mono"
Set FontSize 26
Set Width 1920
Set Height 1080
Set Padding 48
Set TypingSpeed 55ms
```

## Appearance: use the project's design tokens

A default VHS render looks like a stranger's terminal. Field conventions
that make videos look designed for the site they live in:

- Build `Set Theme { ... }` from the project's own color tokens (background,
  foreground, accent as cursor color). If the site has a `colors.css` or
  design tokens file, map from it; don't invent a palette.
- Use the site's code font as `FontFamily`.
- Pick one `Width`/`Height`/`FontSize`/`Padding` combination and reuse it
  across every tape in the project, so the whole video set is
  pixel-consistent. 1920×1080 with font size 26 and padding 48 reads well
  in docs layouts.
- Output format: `.mp4` for docs pages, `.gif` for GitHub READMEs. VHS can
  emit [multiple outputs](https://github.com/charmbracelet/vhs#output) from
  one tape.

## Nothing is mocked

The commands genuinely run in a real shell. This is a feature: rendering
the tape is running the flow, so a broken flow fails to render instead of
shipping a stale video. Consequences (field conventions):

- Never fake output with `echo`. If the flow is broken, fix the flow.
- Treat tapes as low-key integration tests: re-render them when the flow
  they demonstrate changes, and the render itself verifies the change.

## Off-camera prep with Hide/Show

Setup that isn't part of the story happens between
[`Hide`](https://github.com/charmbracelet/vhs#hide) and
[`Show`](https://github.com/charmbracelet/vhs#show): environment variables,
staging directories, clearing the screen. The viewer sees only the steps
they'll actually take.

```tape
Hide
Type "export CI=1 && cd /tmp/demo && rm -rf work && clear"
Enter
Sleep 1s
Show
```

Field conventions for what belongs off camera:

- `export CI=1` so CLIs skip update checks and interactive prompts.
- Unset host-specific environment variables that make tools print warnings
  (anything your shell whispers ends up on camera).
- Reset to a pristine state (`rm -rf` the work dir, fresh copy, `git
  checkout -- . && git clean -fd`) so every render starts identical.
- End the hidden block with `clear` and a short `Sleep` before `Show`.

## Determinism: wait on output, not on time

Fixed sleeps either waste seconds or cut off early. Use
[`Wait`](https://github.com/charmbracelet/vhs#wait) with a regex and a
timeout for anything whose duration varies:

```tape
Type "npm run dev"
Enter
Wait+Screen@60s /localhost/
Sleep 5s
```

- `Wait@30s` (no pattern) waits for the shell prompt to return; use it
  after commands like installs and builds.
- `Wait+Screen@60s /pattern/` waits for text anywhere on screen; use it for
  long-running processes that signal readiness (dev servers printing their
  URL).
- Reserve fixed `Sleep` for reading pauses and for interactive TUIs whose
  output VHS can't pattern-match reliably; size those sleeps generously and
  note in a comment that trailing freeze can be trimmed (field convention).

## Pacing

Field conventions that make renders read like a person, not a script:

- `Set TypingSpeed 55ms` is a natural speed; the default 50ms also works.
- `Sleep 500ms`-`800ms` between `Type` and `Enter`, as if reading the
  command before running it.
- `Sleep 1s`-`1.5s` after a command's output settles, before the next one.
- End every tape with a `Sleep` of a few seconds so the final state holds
  on screen instead of cutting instantly.

## Render hygiene

Before rendering (field conventions, learned the annoying way):

- **Free your ports.** If the video starts a dev server, stop your own
  first, or the video ships a "port in use" warning forever.
- **Warm the caches.** Run the install once off camera beforehand; nobody
  needs to watch a cold `npm install`.
- **Check the finished render** for leaked prompts, warnings, wrong
  directory names, or paths that reveal more than intended.
- Use [`Require`](https://github.com/charmbracelet/vhs#require) at the top
  of the tape for any program the flow depends on, so a missing dependency
  fails fast instead of rendering a broken video.

## Interactive TUIs

For flows that drive a TUI (an AI agent CLI, an installer):

- Start from a fresh path every render so first-run dialogs (workspace
  trust, telemetry consent) appear deterministically, and answer them in
  the tape.
- Budget generous fixed sleeps for the TUI's work, and land on a static
  end screen.
- After exiting (`Ctrl+C`), prove the result is real on camera: a
  `git diff --stat` or `ls` of what changed makes the demo credible (field
  convention).

## Before finishing

- [ ] Comment header documents output path, render command, and all prep
- [ ] `Output` and `Set` block before any actions; theme from project tokens
- [ ] Setup and cleanup wrapped in `Hide`/`Show`
- [ ] Variable-duration steps use `Wait`/`Wait+Screen` with timeouts, not sleeps
- [ ] `CI=1` set and noisy env vars unset off camera
- [ ] Tape ends on a held final frame
- [ ] The tape was actually rendered and the output watched once

---

Maintained by [Michael Andreuzza](https://michaelandreuzza.com) at
[Lexington Themes](https://lexingtonthemes.com). Every video in the
Lexington docs is rendered from tapes written this way.
