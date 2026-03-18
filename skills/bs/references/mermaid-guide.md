# Mermaid Diagram Guide

How to include Mermaid diagrams in bs plans.

## CDN ESM import

Always load Mermaid via CDN ESM. Place this at the end of `<body>`, after all diagram markup:

```html
<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';

  function getThemeVars() {
    const s = getComputedStyle(document.documentElement);
    const v = (name, fb) => s.getPropertyValue(name).trim() || fb;
    const isDark = !document.documentElement.hasAttribute('data-theme')
      ? !window.matchMedia('(prefers-color-scheme: light)').matches
      : document.documentElement.getAttribute('data-theme') !== 'light';
    const border = isDark ? 'rgba(255,255,255,0.15)' : 'rgba(0,0,0,0.15)';
    const text = isDark ? v('--text-primary', '#e8e9e6') : v('--text-primary', '#1a1d1e');
    const surface1 = isDark ? v('--surface-1', '#1a1d1e') : v('--surface-1', '#f5f5f4');
    const surface2 = isDark ? v('--surface-2', '#242626') : v('--surface-2', '#e8e8e6');
    return {
      primaryColor: surface1,
      primaryTextColor: text,
      primaryBorderColor: border,
      lineColor: '#fe7100',
      secondaryColor: surface2,
      tertiaryColor: isDark ? v('--surface-0', '#2a2d2e') : v('--surface-0', '#fafaf9'),
      fontFamily: '"Spline Sans Mono", monospace',
      fontSize: '13px',
      // Sequence diagram
      actorBkg: surface1,
      actorBorder: border,
      actorTextColor: text,
      actorLineColor: border,
      signalColor: '#fe7100',
      signalTextColor: text,
      labelBoxBkgColor: surface2,
      labelBoxBorderColor: border,
      labelTextColor: text,
      loopTextColor: text,
      noteBkgColor: surface2,
      noteTextColor: text,
      noteBorderColor: border,
      activationBkgColor: isDark ? v('--surface-0', '#2a2d2e') : v('--surface-0', '#fafaf9'),
      activationBorderColor: border,
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

  // Re-render on theme toggle (data-theme attribute change)
  new MutationObserver(() => initMermaid()).observe(
    document.documentElement, { attributes: true, attributeFilter: ['data-theme'] }
  );
  // Re-render on system preference change
  window.matchMedia('(prefers-color-scheme: light)').addEventListener('change', () => initMermaid());
</script>
```

### Key config points

- **Always `theme: 'base'`** — this gives `themeVariables` full control over colors. Using `'dark'` or other built-in themes will override your variables.
- **Read CSS variables dynamically** — `getComputedStyle` reads the current theme's surface/text tokens at init time. Never hardcode surface or text hex values.
- **`lineColor: '#fe7100'`** — electric orange for connections/arrows, the signature accent. This stays constant across themes.
- **`fontFamily`** — matches the plan's body font for consistency
- **`startOnLoad: false`** — we call `mermaid.run()` manually so we can re-render on theme changes

### Theme re-rendering

Mermaid's `themeVariables` are set at initialization time — they don't respond to CSS changes. Mermaid also marks rendered elements with `data-processed` and replaces the original `<pre>` text with an `<svg>`. To re-render with new theme colors:

1. **Stash source** — before the first render, save each `<pre class="mermaid">` element's `textContent` as a `data-mermaid-source` attribute
2. **Restore before re-render** — on theme change, restore the original text, remove `data-processed`, then call `mermaid.initialize()` + `mermaid.run()`
3. **Guard against concurrency** — use a `rendering` flag to prevent overlapping `mermaid.run()` calls from MutationObserver firing during an in-flight render
4. **MutationObserver** watches `data-theme` attribute on `<html>` — triggers on manual toggle from the hosting page
5. **matchMedia listener** watches `prefers-color-scheme` — triggers on system preference change

### Avoid inline `style` declarations

Do **not** use Mermaid's `style` syntax to set fill/color on individual nodes:

```
%% BAD — hardcoded colors, won't update on theme change
style NodeA fill:#1a1d1e,stroke:#fe7100,color:#e8e9e6
```

Instead, use `classDef` for accent highlighting (stroke-only, no fill):

```
%% GOOD — fill comes from themeVariables, accent stroke is constant
classDef accent stroke:#fe7100,stroke-width:2px
class NodeA,NodeB accent
```

## Diagram markup

Wrap diagrams in a `<pre class="mermaid">` tag:

```html
<div class="diagram-container">
  <pre class="mermaid">
    flowchart TD
      A[Start] --> B{Decision}
      B -->|Yes| C[Action]
      B -->|No| D[Alternative]
  </pre>
</div>
```

### Preferred diagram types

| Type | Use for | Syntax |
|------|---------|--------|
| `flowchart TD` | Architecture, data flows, decision trees | `flowchart TD` |
| `sequenceDiagram` | API calls, interaction flows, protocols | `sequenceDiagram` |
| `stateDiagram-v2` | State machines, lifecycle diagrams | `stateDiagram-v2` |
| `erDiagram` | Database schemas, entity relationships | `erDiagram` |
| `gantt` | Timelines, project phases | `gantt` |

### Prefer `TD` (top-down) over `LR` (left-right)

Top-down diagrams read more naturally and handle long labels better. Use `LR` only when the flow is explicitly horizontal (e.g., a pipeline).

## Zoom controls

Add zoom controls to every diagram container for usability:

```html
<div class="diagram-container" style="position: relative;">
  <div class="diagram-controls">
    <button onclick="zoomDiagram(this, 1.2)" title="Zoom in">+</button>
    <button onclick="zoomDiagram(this, 0.8)" title="Zoom out">-</button>
    <button onclick="resetDiagram(this)" title="Reset">Reset</button>
  </div>
  <div class="diagram-scroll">
    <pre class="mermaid">
      flowchart TD
        A --> B --> C
    </pre>
  </div>
</div>
```

### Zoom control styles

```css
.diagram-container {
  position: relative;
  margin: 1.5rem 0;
}
.diagram-controls {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
  display: flex;
  gap: 0.25rem;
  z-index: 1;
}
.diagram-controls button {
  background: var(--surface-2);
  border: 1px solid var(--border-strong);
  border-radius: 4px;
  color: var(--text-secondary);
  font-family: var(--font-body);
  font-size: 0.75rem;
  padding: 0.25rem 0.5rem;
  cursor: pointer;
}
.diagram-controls button:hover {
  background: var(--surface-1);
  color: var(--text-primary);
}
.diagram-scroll {
  overflow: auto;
  border: 1px solid var(--border);
  border-radius: 6px;
  background: var(--surface-1);
  padding: 1.5rem;
}
```

### Zoom control script

```html
<script>
  function zoomDiagram(btn, factor) {
    const container = btn.closest('.diagram-container');
    const svg = container.querySelector('svg');
    if (!svg) return;
    const current = parseFloat(svg.style.transform?.match(/scale\(([^)]+)\)/)?.[1] || '1');
    const next = Math.min(Math.max(current * factor, 0.3), 3);
    svg.style.transform = `scale(${next})`;
    svg.style.transformOrigin = 'top left';
  }
  function resetDiagram(btn) {
    const container = btn.closest('.diagram-container');
    const svg = container.querySelector('svg');
    if (svg) svg.style.transform = 'scale(1)';
  }
</script>
```

## Best practices

### Node count

Keep diagrams under 20 nodes for clarity. If a diagram has more than 20 nodes, split it into multiple focused diagrams with connecting context.

### Node labels

- Keep labels short (3-5 words max)
- Use `[square brackets]` for processes/actions
- Use `{curly braces}` for decisions
- Use `([stadium shape])` for start/end
- Use `[(cylinder)]` for databases

### Subgraphs

Use subgraphs to group related nodes:

```
flowchart TD
  subgraph API["API Layer"]
    A1[Auth] --> A2[Routes]
  end
  subgraph DB["Data Layer"]
    D1[(Postgres)] --> D2[(Redis)]
  end
  A2 --> D1
```

### Styling nodes

Apply custom styles to highlight important nodes:

```
flowchart TD
  A[Start] --> B[Critical Path]
  B --> C[End]
  style B fill:#fe7100,color:#fff,stroke:none
```

## iframe considerations

Plans are rendered inside a sandboxed `srcdoc` iframe (`sandbox="allow-scripts"`, no `allow-same-origin`). Mermaid initializes independently within the iframe.

The hosting page may set `data-theme` on the iframe's `<html>` via a postMessage bridge. The MutationObserver pattern above detects this and re-renders diagrams with the correct theme colors. The zoom controls also work within the iframe context.
