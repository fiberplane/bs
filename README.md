# bs — brainstorm

A Claude Code skill for generating rich, visual HTML plans and publishing them to shareable URLs.

Plans are self-contained HTML files with diagrams, tables, and structured layouts using a consistent design language. Published plans support inline comments and real-time cursor presence for collaborative review.

## Install

### With `npx skills` (recommended)

```bash
npx skills add fiberplane/bs
```

### With Claude Code

```bash
claude install github.com/fiberplane/bs
```

### Manual

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/fiberplane/bs.git ~/.claude/skills/bs
```

## Usage

Once installed, ask Claude to brainstorm or create a visual plan:

```
> brainstorm an auth system migration plan
> bs: architecture overview for the new API gateway
> create a visual plan for the database refactor
```

Claude will generate a self-contained HTML plan and publish it to a shareable URL.

## What you get

- **Visual HTML plans** — structured layouts with cards, tables, file trees, and diagrams
- **Mermaid diagrams** — flowcharts, sequence diagrams, architecture views
- **Dark-first design** — clean, readable aesthetic with light theme support
- **Shareable URLs** — publish to `app.fp.dev/bs/` with one command
- **Inline comments** — collaborators can leave feedback anchored to specific elements
- **Cursor presence** — see who's viewing the plan in real-time

## Design

Plans follow an fp-inspired design language:

- **Instrument Serif** headlines for editorial warmth
- **Spline Sans Mono** body text for clean readability
- **Electric orange** (`#fe7100`) accents
- **OKLch neutral scale** for surfaces and text
- Dark-first with automatic light theme support

## License

MIT
