# RFD 0001: CSS Design System Architecture for Web Components

- **Authors:** [Author]
- **State:** Prediscussion
- **Labels:** architecture, css, web-components

## What

This document defines how we structure a CSS design system intended for consumption by Web Components. It covers the boundary between document-level styles and shadow DOM, the role of utility classes, and token naming conventions.

## Why

We're building a vanilla CSS design system. No Sass, no build step, no framework. The distribution target is Web Components using shadow DOM.

This creates constraints that traditional CSS frameworks ignore. Tailwind assumes global scope. CSS-in-JS assumes a JavaScript runtime. Neither fits.

Shadow DOM changes the game. Styles don't leak in. Global utility classes are useless inside a shadow root unless explicitly adopted. But CSS custom properties *do* cascade through—that's by design, per spec.

We need to make intentional decisions about:
- What lives in the document vs. what gets adopted into shadow roots
- Whether utilities belong inside components at all
- How to name tokens so they're useful without being limiting

## Details

### The Shadow Boundary

The DOM spec defines shadow DOM as an encapsulation boundary. Styles from the outer document don't apply inside a shadow root. This is the whole point.

But custom properties are different. They inherit. A `--color-primary` set on `:root` is available inside every shadow root in the document. No adoption required. This was an intentional design decision by the CSS working group—custom properties are the theming API for web components.

This gives us a natural split:

| Concern | Scope | Mechanism |
|---------|-------|-----------|
| Property registration (`@property`) | Document | Loaded once via `<link>` |
| Token values | Document | Cascade via inheritance |
| Reset/normalize | Per shadow root | `adoptedStyleSheets` |
| Layout utilities | Per shadow root | `adoptedStyleSheets` |
| Component styles | Per shadow root | `adoptedStyleSheets` |

### Utility Classes: The Tradeoff

There are two schools of thought here.

**Option A: Adopt utilities into shadow roots**

Components get access to `.flex`, `.p-4`, `.text-sm`, etc. Write markup like:

```html
<div class="flex gap-4 p-4">
  <slot></slot>
</div>
```

This is familiar. It's fast to write. It's also coupling your component internals to the design system's class vocabulary. If we rename `.p-4` to `.padding-4`, every component template breaks.

The stylesheet has to be adopted into every shadow root. That's not expensive—`adoptedStyleSheets` shares the parsed CSSOM object—but it's still a thing you have to do. Forget it and your component renders without styles.

**Option B: Custom properties only**

Components write their own CSS using token values:

```css
:host {
  display: flex;
  gap: var(--ds-space-4);
  padding: var(--ds-space-4);
}
```

More verbose. But the component owns its styles completely. The design system provides *values*, not *classes*. The contract is smaller—just the property names.

No adoption needed for tokens. They just cascade.

**Option C: Both, with clear boundaries**

Tokens cascade. Always available.

Utilities exist as an adoptable sheet for components that want them. Some components adopt it. Some don't. Their choice.

This is where I land. Forcing one approach is limiting. Some components are simple enough that utilities make sense. Others have complex internal styling where classes would be noise.

The key is making the utilities *optional*, not required.

**Recommendation:** Ship both. Document that tokens are the primary API. Utilities are a convenience layer, adopted per-component.

### Token Naming: Primitive vs. Semantic

This is less controversial but still worth documenting.

**Primitives** are raw values:
```css
--ds-blue-500: oklch(55% 0.2 250);
--ds-gray-100: oklch(95% 0 0);
--ds-space-4: 1rem;
```

**Semantics** map meaning to primitives:
```css
--ds-color-primary: var(--ds-blue-500);
--ds-color-text: var(--ds-gray-900);
--ds-color-text-muted: var(--ds-gray-600);
```

You need both.

Without primitives, you can't build a one-off component that needs "a blue" that isn't the primary color. You end up hardcoding hex values, defeating the system.

Without semantics, every component author has to decide which gray is the right gray for muted text. They'll pick differently. The UI becomes inconsistent.

The pattern:
1. Define the full primitive palette
2. Define semantic tokens that reference primitives
3. Components use semantics by default, primitives when they have a reason

**File structure:**
```
tokens/
├── primitives.css    # Raw palette, spacing scale, type scale
└── semantics.css     # Meaningful aliases
```

Both get loaded at document level. Both cascade into shadow roots.

### `@property` Registration

CSS Houdini's `@property` rule lets us register typed custom properties:

```css
@property --ds-hue-primary {
  syntax: "<number>";
  inherits: true;
  initial-value: 250;
}
```

This enables:
- Type checking (invalid values fall back gracefully)
- Animation (you can't animate an unregistered custom property)
- Default values without `var()` fallbacks everywhere

The catch: `@property` must be registered at document scope. You can't register properties inside shadow DOM.

This means we need a dedicated file for registrations, loaded once in the document.

**File structure (revised):**
```
tokens/
├── properties.css    # @property registrations only
├── primitives.css    # Palette, scales
└── semantics.css     # Meaningful aliases
```

### Proposed Directory Structure

```
design-system/
├── tokens/
│   ├── properties.css     # @property registrations
│   ├── primitives.css     # Raw values
│   └── semantics.css      # Semantic mappings
│
├── sheets/
│   ├── reset.css          # Normalize, box-sizing, etc.
│   ├── layout.css         # Grid, container queries
│   └── utilities.css      # Optional utility classes
│
├── document.css           # Imports tokens/* for <link> in document
│
└── index.js               # Exports CSSStyleSheet objects for adoption
```

**Document loads:**
```html
<link rel="stylesheet" href="design-system/document.css">
```

**Components adopt what they need:**
```js
import { reset, layout, utilities } from 'design-system';
import styles from './button.css' with { type: 'css' };

class Button extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    // reset + layout always, utilities optional
    this.shadowRoot.adoptedStyleSheets = [reset, layout, styles];
  }
}
```

### JavaScript Entry Point

The JS module pre-parses stylesheets into shareable `CSSStyleSheet` objects:

```js
// design-system/index.js

const sheets = {
  reset: new CSSStyleSheet(),
  layout: new CSSStyleSheet(),
  utilities: new CSSStyleSheet(),
};

// Fetch and parse once
await Promise.all([
  fetch(new URL('./sheets/reset.css', import.meta.url))
    .then(r => r.text())
    .then(css => sheets.reset.replaceSync(css)),
  fetch(new URL('./sheets/layout.css', import.meta.url))
    .then(r => r.text())
    .then(css => sheets.layout.replaceSync(css)),
  fetch(new URL('./sheets/utilities.css', import.meta.url))
    .then(r => r.text())
    .then(css => sheets.utilities.replaceSync(css)),
]);

export const { reset, layout, utilities } = sheets;
```

When browsers ship CSS module scripts more broadly, this simplifies to:

```js
export { default as reset } from './sheets/reset.css' with { type: 'css' };
export { default as layout } from './sheets/layout.css' with { type: 'css' };
export { default as utilities } from './sheets/utilities.css' with { type: 'css' };
```

### What's Out of Scope

- **Light DOM fallback**: Real use case, not addressing now.
- **Server-side rendering**: Declarative shadow DOM exists but has its own constraints.
- **Legacy browser support**: We're targeting modern browsers only.
- **Component library**: This system provides tokens and base styles, not components.

### Open Questions

1. **Utility granularity**: Do we ship one `utilities.css` or split into `spacing.css`, `typography.css`, etc.? More files = more granular adoption, but also more import boilerplate.

2. **Naming prefix**: `--ds-*` is generic. Should we pick something more distinctive? Or let consumers alias at their document root?

3. **Dark mode strategy**: `light-dark()` is clean but requires `color-scheme` to be set. Document this requirement or handle it in the system?

## References

- [CSS Cascading and Inheritance Level 5](https://www.w3.org/TR/css-cascade-5/) — `@layer` specification
- [CSS Properties and Values API](https://www.w3.org/TR/css-properties-values-api-1/) — `@property` specification
- [DOM Living Standard: Shadow DOM](https://dom.spec.whatwg.org/#shadow-trees) — encapsulation rules
- [Constructable Stylesheets](https://web.dev/constructable-stylesheets/) — `adoptedStyleSheets` explainer
- [Oxide Design System](https://github.com/oxidecomputer/design-system) — prior art for token structure
