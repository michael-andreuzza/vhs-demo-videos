# VHS Demo Videos

An agent skill for making terminal demo videos with
[VHS](https://github.com/charmbracelet/vhs): docs videos rendered from
`.tape` scripts committed to your repo, instead of screen recordings.

By [Michael Andreuzza](https://michaelandreuzza.com) at
[Lexington Themes](https://lexingtonthemes.com). Every video in the
Lexington docs is rendered this way; this skill is the distilled version of
what makes those renders clean, deterministic, and re-runnable.

Example
[![Watch the demo](thumbnail.png)](https://lexingtonthemes.com/documentation/getting-started#:~:text=2.%20Access%20your%20purchase)


## Why tapes instead of recordings

- A re-shoot is a re-render: edit the script, run one command, done.
- Nothing is mocked. The commands really run, so a broken flow fails to
  render instead of shipping a stale video.
- The videos match your site because the terminal theme comes from your
  design tokens.
- Terminal output compresses absurdly well: a full 1080p install video can
  land around 200 KB.

## Install

With the [skills](https://skills.sh) CLI, for any supported agent:

```bash
npx skills add michael-andreuzza/vhs-demo-videos
```

Or copy `skills/vhs-demo-videos/` into your project's skills directory
(`.cursor/skills/` for Cursor, `.claude/skills/` for Claude Code).

Then ask your agent for a video: "make a docs video showing the install
flow" gets you a `.tape` file that renders it.

## License

MIT
