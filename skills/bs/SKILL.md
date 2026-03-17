---
name: bs
description: Generate rich visual HTML plans with diagrams and structured layouts, then publish to a shareable URL. Triggers on "brainstorm", "bs", "create a plan", or "visualize architecture".
---

# bs — brainstorm

Generate self-contained HTML plans that are visually rich, structurally clear, and publishable to a shareable URL.

## When to use

- User asks to brainstorm, plan, or think through something visually
- User wants architecture diagrams, implementation plans, or system designs
- User needs a shareable visual document with diagrams and structured layouts
- User says "bs", "/bs", "brainstorm", or "make me a plan"

## Design language

Every plan uses a consistent fp-inspired aesthetic. Follow these rules exactly.

### Typography

Use these Google Fonts imports at the top of every plan:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Instrument+Serif:ital@0;1&family=Spline+Sans+Mono:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
```

- **Headlines (h1, h2, h3):** `"Instrument Serif", Georgia, "Times New Roman", serif` — warm, editorial feel
- **Body text, UI labels, navigation:** `"Spline Sans Mono", "SF Mono", "Cascadia Code", monospace` — clean mono aesthetic
- **Code blocks, inline code:** `"JetBrains Mono", "Fira Code", "Cascadia Code", monospace`
- Always include system font fallbacks for offline resilience

### Color palette

Use CSS custom properties. Define them in `:root` on every plan:

```css
:root {
  /* Neutrals — OKLch scale */
  --neutral-50: oklch(0.985 0 0);
  --neutral-100: oklch(0.97 0.001 106.4);
  --neutral-200: oklch(0.922 0.004 106.4);
  --neutral-300: oklch(0.869 0.005 106.4);
  --neutral-400: oklch(0.708 0.01 106.4);
  --neutral-500: oklch(0.556 0.013 106.4);
  --neutral-600: oklch(0.442 0.014 106.4);
  --neutral-700: oklch(0.355 0.014 106.4);
  --neutral-800: oklch(0.269 0.01 106.4);
  --neutral-900: oklch(0.205 0.008 106.4);
  --neutral-950: oklch(0.145 0.005 106.4);

  /* Primary — electric blue */
  --primary: oklch(0.565 0.239 263.9);
  --primary-light: oklch(0.685 0.169 263.9);
  --primary-subtle: oklch(0.265 0.08 263.9);

  /* Accent — electric orange */
  --accent: #fe7100;
  --accent-light: #ff8c33;
  --accent-subtle: oklch(0.285 0.08 55);

  /* Status */
  --success: oklch(0.723 0.191 142.9);
  --warning: oklch(0.768 0.165 78.5);
  --danger: oklch(0.637 0.237 25.3);

  /* Surfaces */
  --surface-0: var(--neutral-950);
  --surface-1: var(--neutral-900);
  --surface-2: var(--neutral-800);
  --border: rgba(255, 255, 255, 0.08);
  --border-strong: rgba(255, 255, 255, 0.15);

  /* Text */
  --text-primary: var(--neutral-100);
  --text-secondary: var(--neutral-400);
  --text-muted: var(--neutral-500);
}
```

### Spacing and layout

- **Max-width:** 900–960px, centered with `margin: 0 auto`
- **Padding:** generous — at least `2rem` on body, `1.5rem` on cards/sections
- **Section gaps:** `3rem` between major sections, `1.5rem` between subsections
- **Surface depth tiers:**
  - Hero (elevated): slightly lighter background, subtle `box-shadow`
  - Body (flat): base `--surface-0` background
  - Reference (recessed): darker background, inset feel

### Dark-first

- Default theme is dark — `--surface-0` as page background, light text
- Include **both** a `@media (prefers-color-scheme: light)` override and a `[data-theme="light"]` selector. The media query handles system preference; the attribute selector handles the hosting page's manual toggle.
- Light theme example:

```css
@media (prefers-color-scheme: light) {
  :root:not([data-theme]) {
    --surface-0: var(--neutral-50);
    --surface-1: var(--neutral-100);
    --surface-2: var(--neutral-200);
    --border: rgba(0, 0, 0, 0.08);
    --border-strong: rgba(0, 0, 0, 0.15);
    --text-primary: var(--neutral-900);
    --text-secondary: var(--neutral-600);
    --text-muted: var(--neutral-500);
  }
}

[data-theme="light"] {
  --surface-0: var(--neutral-50);
  --surface-1: var(--neutral-100);
  --surface-2: var(--neutral-200);
  --border: rgba(0, 0, 0, 0.08);
  --border-strong: rgba(0, 0, 0, 0.15);
  --text-primary: var(--neutral-900);
  --text-secondary: var(--neutral-600);
  --text-muted: var(--neutral-500);
}
```

### Color discipline

Only use colors from the CSS variable palette. Never hardcode `rgba()`, hex, or `oklch()` values in component styles — always reference a `--token`. When you must use opacity-based values (shadows, overlays, hover states), provide both dark and light variants:

```css
/* shadows — use the --shadow token */
--shadow: 0 1px 3px rgba(0, 0, 0, 0.3);

/* in light theme override: */
--shadow: 0 1px 3px rgba(0, 0, 0, 0.1);

/* table row hover — provide both */
tr:hover td { background: rgba(255, 255, 255, 0.02); }

@media (prefers-color-scheme: light) {
  tr:hover td { background: rgba(0, 0, 0, 0.02); }
}
[data-theme="light"] tr:hover td { background: rgba(0, 0, 0, 0.02); }
```

The only exceptions are the `--accent` and `--primary` hex values, which are defined once in `:root` and don't change between themes.

### Animations

- Staggered `fadeUp` entrance animations on sections
- Respect `prefers-reduced-motion`:

```css
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}

@media (prefers-reduced-motion: no-preference) {
  .section { animation: fadeUp 0.4s ease-out both; }
  .section:nth-child(2) { animation-delay: 0.08s; }
  .section:nth-child(3) { animation-delay: 0.16s; }
  .section:nth-child(4) { animation-delay: 0.24s; }
}
```

### Anti-slop rules (IMPORTANT — read carefully)

Do NOT use any of the following patterns. They are hallmarks of generic AI output:

- No neon glows, gradient text, or glowing `box-shadow`
- No emoji in headings (h1, h2, h3) — emoji in body text is fine sparingly
- No Inter, Roboto, or system sans-serif as the primary display font
- No cyan-magenta-pink color combos
- No "glassmorphism" or frosted glass effects
- No excessive border-radius (keep it 4–8px for cards, 2–4px for small elements; pill badges use `9999px`)
- No generic motivational copy ("Let's build something amazing!")
- No decorative gradients that serve no informational purpose
- Every visual element must serve a structural or informational purpose

## HTML generation patterns

### Self-contained

Every plan is a single, self-contained HTML file. All CSS is inline in a `<style>` tag. All JS is inline in a `<script>` tag (except CDN imports like Mermaid). No external CSS files, no build step.

### Document structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Plan Title — bs</title>
  <!-- Font imports -->
  <!-- CSS custom properties + styles -->
</head>
<body>
  <main>
    <header class="hero">
      <h1>Plan Title</h1>
      <p class="subtitle">Brief description</p>
    </header>
    <!-- Sections with semantic HTML -->
  </main>
</body>
</html>
```

### Common patterns

- **Cards:** `border: 1px solid var(--border); border-radius: 6px; background: var(--surface-1); padding: 1.5rem;`
- **Section headers:** Instrument Serif, with a thin `border-bottom: 1px solid var(--border)` below
- **Code blocks:** JetBrains Mono, `background: var(--surface-1); padding: 1rem; border-radius: 4px; overflow-x: auto;`
- **Tables:** full-width, `border-collapse: collapse`, alternating row tones, header row with `--surface-2` background
- **Status badges:** small pill shapes with status colors, Spline Sans Mono at 0.75rem
- **File trees:** use `<pre>` or a `<div>` with `white-space: pre` and tree characters (├── └── │), styled in Spline Sans Mono
- **Grid layouts:** CSS Grid for card grids, `grid-template-columns: repeat(auto-fit, minmax(280px, 1fr))`, `gap: 1rem`

## Mermaid diagrams

Use Mermaid for flowcharts, sequence diagrams, and other structured diagrams. Load via CDN ESM import:

```html
<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';

  function getThemeVars() {
    const s = getComputedStyle(document.documentElement);
    const v = (name, fb) => s.getPropertyValue(name).trim() || fb;
    const isDark = !document.documentElement.hasAttribute('data-theme')
      ? !window.matchMedia('(prefers-color-scheme: light)').matches
      : document.documentElement.getAttribute('data-theme') !== 'light';
    return {
      primaryColor: isDark ? v('--surface-1', '#1a1d1e') : v('--surface-1', '#f5f5f4'),
      primaryTextColor: isDark ? v('--text-primary', '#e8e9e6') : v('--text-primary', '#1a1d1e'),
      primaryBorderColor: isDark ? 'rgba(255,255,255,0.15)' : 'rgba(0,0,0,0.15)',
      lineColor: '#fe7100',
      secondaryColor: isDark ? v('--surface-2', '#242626') : v('--surface-2', '#e8e8e6'),
      tertiaryColor: isDark ? v('--surface-0', '#2a2d2e') : v('--surface-0', '#fafaf9'),
      fontFamily: '"Spline Sans Mono", monospace',
      fontSize: '13px',
    };
  }

  // Stash original source before first render (mermaid replaces <pre> content with SVG)
  document.querySelectorAll('.mermaid').forEach(el => {
    el.setAttribute('data-mermaid-source', el.textContent);
  });

  let rendering = false;
  async function initMermaid() {
    if (rendering) return;
    rendering = true;
    try {
      // Restore original source and clear processed state for re-renders
      document.querySelectorAll('.mermaid').forEach(el => {
        const src = el.getAttribute('data-mermaid-source');
        if (src) { el.removeAttribute('data-processed'); el.innerHTML = src; }
      });
      mermaid.initialize({ startOnLoad: false, theme: 'base', themeVariables: getThemeVars() });
      await mermaid.run();
    } finally { rendering = false; }
  }

  await initMermaid();

  // Re-render on theme toggle or system preference change
  new MutationObserver(() => initMermaid()).observe(
    document.documentElement, { attributes: true, attributeFilter: ['data-theme'] }
  );
  window.matchMedia('(prefers-color-scheme: light)').addEventListener('change', () => initMermaid());
</script>
```

- Always use `theme: 'base'` so `themeVariables` has full control
- **Read CSS variables dynamically** via `getComputedStyle` — never hardcode surface/text colors in Mermaid config
- **Re-render on theme change** — MutationObserver on `data-theme` + matchMedia listener calls `mermaid.run()` again
- **Avoid inline `style` declarations** in diagram markup (e.g., `style NodeA fill:#1a1d1e`) — these cannot be updated on theme change. Use `classDef` with accent strokes instead
- Prefer `flowchart TD` (top-down) over `LR` (left-right) for better readability
- Add zoom controls on every diagram container (see `references/mermaid-guide.md`)
- Keep diagrams under 20 nodes for clarity — split large diagrams into multiple smaller ones
- Wrap each diagram in a container with `overflow: auto` for horizontal scroll on mobile

## Title convention

Always include a descriptive title. The title should:
- Describe what the plan covers, not what it is ("Auth System Migration" not "Plan")
- Be concise — under 60 characters
- Not include "Plan:" prefix — the context makes it clear

## Publish flow

After generating the HTML plan, publish it to get a shareable URL.

### Publishing (anonymous — no credentials needed)

```bash
curl -s -X POST https://app.fp.dev/bs/api/plans \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg title "PLAN_TITLE" --arg html "$(cat PLAN_FILE)" \
    '{title: $title, html_content: $html}')"
```

The response includes:
- `url` — shareable link (e.g., `https://app.fp.dev/bs/a8k2m1x`)
- `claimToken` — save this to claim the plan later (anonymous plans expire in 3 days)
- `slug` — the plan's unique identifier

### Publishing (authenticated — plan persists)

If fp credentials exist at `~/.fiberplane/credentials.toml`, include the Bearer token:

```bash
# Read token from credentials
FP_TOKEN=$(grep -m1 'api_token' ~/.fiberplane/credentials.toml 2>/dev/null | sed 's/.*= *"\(.*\)"/\1/')

curl -s -X POST https://app.fp.dev/bs/api/plans \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $FP_TOKEN" \
  -d "$(jq -n --arg title "PLAN_TITLE" --arg html "$(cat PLAN_FILE)" \
    '{title: $title, html_content: $html}')"
```

Authenticated plans don't expire and can be updated with new versions.

### Updating an existing plan

```bash
curl -s -X PUT https://app.fp.dev/bs/api/plans/SLUG \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $FP_TOKEN" \
  -d "$(jq -n --arg html "$(cat PLAN_FILE)" \
    '{html_content: $html}')"
```

### After publishing

Always present the URL to the user. Example output:

```
Published: https://app.fp.dev/bs/a8k2m1x
```

### Error handling

If the curl command fails or returns a non-2xx status:
- Check the response body for a JSON `message` field explaining the error
- Common errors: 413 (HTML exceeds 2 MB limit), 401 (invalid token), 400 (missing title or html_content)
- If `jq` is not available, use `grep` or `python3 -c` to extract the `url` field from the JSON response
- If publishing fails entirely, still present the local HTML file path to the user so the plan is not lost

### Workflow

1. Generate the self-contained HTML plan following the design language
2. Write it to a temp file
3. Publish via curl POST, extracting `url` from the JSON response
4. Present the shareable URL to the user (or the local file path if publish failed)
5. Clean up the temp file

## References

For detailed specifications beyond this prompt:

- `references/design-tokens.md` — full CSS variable palette, font stacks, spacing scale, surface depth tiers with code examples
- `references/mermaid-guide.md` — CDN ESM import snippet, theme config, zoom controls pattern, iframe considerations
- `references/publish-api.md` — curl examples for create/update/delete, auth header format, full response schemas, claim flow, claim token deletion
