# Mermaid Diagram Guide

How to include Mermaid diagrams in bs plans.

## CDN ESM import

Always load Mermaid via CDN ESM. Place this at the end of `<body>`, after all diagram markup:

```html
<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
  mermaid.initialize({
    startOnLoad: true,
    theme: 'base',
    themeVariables: {
      primaryColor: '#1a1d1e',
      primaryTextColor: '#e8e9e6',
      primaryBorderColor: 'rgba(255,255,255,0.15)',
      lineColor: '#fe7100',
      secondaryColor: '#242626',
      tertiaryColor: '#2a2d2e',
      fontFamily: '"Spline Sans Mono", monospace',
      fontSize: '13px'
    }
  });
</script>
```

### Key config points

- **Always `theme: 'base'`** — this gives `themeVariables` full control over colors. Using `'dark'` or other built-in themes will override your variables.
- **`lineColor: '#fe7100'`** — electric orange for connections/arrows, the signature accent
- **`fontFamily`** — matches the plan's body font for consistency

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

Plans are rendered inside a `srcdoc` iframe on the published page. Each iframe is its own document, so Mermaid initializes independently within the iframe — no special handling needed.

The zoom controls work within the iframe context. The parent frame's comment widget accesses the iframe DOM via `contentDocument` (same-origin via `allow-same-origin` sandbox attribute).
