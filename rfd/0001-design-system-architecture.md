# RFD 0001: CSS Design System Architecture for Web Components

- **Authors:** [Author]
- **State:** Prediscussion
- **Labels:** architecture, css, web-components

## What

This document defines the structure of a CSS design system for Web Components using shadow DOM. It specifies:

- The boundary between document-scope and shadow-scope styles
- Token naming conventions (primitive vs. semantic)
- Directory structure and file organization
- The JavaScript module interface for `adoptedStyleSheets`

## Why

Shadow DOM encapsulation creates constraints that existing CSS frameworks don't account for:

1. Styles in the document don't apply inside shadow roots
2. Utility classes (`.flex`, `.p-4`) are useless inside components unless explicitly adopted
3. CSS custom properties *do* inherit into shadow roots (per CSS spec)
4. `@property` registrations must occur at document scope—they cannot be registered inside shadow DOM

These constraints dictate a specific architecture. This document captures it.

## Details

### Constraint: The Shadow Boundary

From the DOM spec, shadow DOM is an encapsulation boundary. Selectors from outer stylesheets don't match elements inside shadow roots.

Exception: CSS custom properties inherit through shadow boundaries. This is intentional—the CSS Working Group designed custom properties as the theming mechanism for web components.

Implication:

| Concern | Scope | Delivery Mechanism |
|---------|-------|-------------------|
| `@property` registrations | Document | `<link>` |
| Token values (custom properties) | Document | `<link>`, cascades into shadow |
| Reset / normalize | Shadow root | `adoptedStyleSheets` |
| Layout utilities | Shadow root | `adoptedStyleSheets` |
| Component-specific styles | Shadow root | `adoptedStyleSheets` |

### Decision: Utility Class Strategy

**Option A: Utilities adopted into shadow roots**

```html
<!-- Inside shadow DOM template -->
<div class="flex gap-4 p-4">
  <slot></slot>
</div>
```

Tradeoffs:
- (+) Familiar pattern for developers used to Tailwind/utility-first
- (+) Fast iteration—change class, see result
- (−) Couples component templates to design system vocabulary
- (−) Stylesheet must be adopted into every shadow root
- (−) Renaming a utility class is a breaking change across all components

**Option B: Custom properties only**

```css
:host {
  display: flex;
  gap: var(--ds-space-4);
  padding: var(--ds-space-4);
}
```

Tradeoffs:
- (+) Component owns its styles entirely
- (+) No adoption required—tokens cascade automatically
- (+) Smaller API surface (property names only)
- (−) More verbose
- (−) Every component writes its own CSS for common patterns

**Decision: Ship both**

Tokens are the primary interface. They cascade without adoption.

Utilities are optional. Components that benefit from them adopt the stylesheet. Components that don't, don't.

Rationale: Different components have different complexity. Forcing one approach creates friction in the other direction.

### Decision: Token Naming

Two layers: primitives and semantics.

**Primitives** — raw values, no implied usage:
```css
--ds-blue-500: oklch(55% 0.2 250);
--ds-gray-100: oklch(95% 0 0);
--ds-space-4: 1rem;
```

**Semantics** — purpose-driven aliases:
```css
--ds-color-primary: var(--ds-blue-500);
--ds-color-text: var(--ds-gray-900);
--ds-color-text-muted: var(--ds-gray-600);
```

**Rationale for both layers:**

Primitives only:
- Component authors must choose which value fits each use case
- Different authors make different choices for the same concept
- No single point of change when design evolves

Semantics only:
- Can't access values outside defined semantics
- Leads to hardcoded values when the semantic doesn't exist
- Over-proliferation of semantics to cover edge cases

Both:
- Semantics are the default reach
- Primitives are available when semantics don't fit
- Brand change (blue → purple) requires one line change in semantics
- Components using primitives directly are explicitly opting out of that indirection

### Constraint: `@property` Registration

CSS Houdini's `@property` rule:

```css
@property --ds-hue-primary {
  syntax: "<number>";
  inherits: true;
  initial-value: 250;
}
```

Capabilities:
- Type validation (invalid values fall back to initial)
- Enables animation of custom properties
- Provides initial values without `var()` fallbacks

Constraint: `@property` must be registered at document scope. The browser needs type information before any stylesheet references the property. Registration inside shadow DOM is not supported.

Implication: Property registrations must be in a file loaded at document level, separate from shadow-adopted styles.

### Directory Structure

```
design-system/
├── tokens/
│   ├── properties.css     # @property registrations
│   ├── primitives.css     # Raw palette, spacing scale, type scale
│   └── semantics.css      # Purpose-driven aliases
│
├── foundation/
│   ├── reset.css          # Box-sizing, margin reset, etc.
│   └── layout.css         # Grid system, container queries
│
├── utilities/
│   ├── spacing.css        # Margin, padding, gap utilities
│   ├── typography.css     # Font size, weight, line-height
│   ├── color.css          # Text color, background, border
│   ├── display.css        # Flex, grid, visibility
│   └── index.css          # Imports all utility files
│
├── document.css           # Imports tokens/* (for document <link>)
│
└── index.js               # Exports CSSStyleSheet objects
```

**`tokens/`**

Three files, three concerns:

- `properties.css`: Schema. Defines what typed properties exist. Rarely changes. Breaking change if property removed.
- `primitives.css`: Values. The full palette and scales. Changes when design tokens update.
- `semantics.css`: Mappings. Connects primitives to purposes. Changes when design language evolves.

Separation allows independent versioning. Adding a new primitive doesn't touch schema. Changing a semantic mapping doesn't touch primitives.

**`foundation/`**

Prerequisites for components. Reset ensures consistent box model. Layout provides grid primitives.

Named "foundation" because nearly all components depend on these. They're adopted as a baseline, not selectively.

**`utilities/`**

Split by concern for selective adoption.

A component needing only flexbox utilities adopts `display.css` (≈2KB). It doesn't pay for `typography.css` (≈4KB) it never uses.

`index.css` exists for convenience when granularity doesn't matter.

**`document.css`**

Single entry point for document-level styles:

```css
@import "./tokens/properties.css";
@import "./tokens/primitives.css";
@import "./tokens/semantics.css";
```

One `<link>` tag. Tokens cascade to all shadow roots automatically.

**`index.js`**

Exports pre-parsed `CSSStyleSheet` objects for `adoptedStyleSheets`:

```js
async function loadSheet(path) {
  const sheet = new CSSStyleSheet();
  const css = await fetch(new URL(path, import.meta.url)).then(r => r.text());
  sheet.replaceSync(css);
  return sheet;
}

export const reset = await loadSheet('./foundation/reset.css');
export const layout = await loadSheet('./foundation/layout.css');
export const spacing = await loadSheet('./utilities/spacing.css');
export const typography = await loadSheet('./utilities/typography.css');
export const color = await loadSheet('./utilities/color.css');
export const display = await loadSheet('./utilities/display.css');
export const utilities = await loadSheet('./utilities/index.css');
```

Top-level await ensures sheets are parsed before export. Multiple components adopting the same sheet share one `CSSStyleSheet` instance in memory.

When CSS module scripts are available:

```js
export { default as reset } from './foundation/reset.css' with { type: 'css' };
export { default as layout } from './foundation/layout.css' with { type: 'css' };
// ...
```

### Usage

**Document:**
```html
<link rel="stylesheet" href="design-system/document.css">
```

**Component:**
```js
import { reset, layout, spacing } from 'design-system';
import styles from './button.css' with { type: 'css' };

class DSButton extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.adoptedStyleSheets = [reset, layout, spacing, styles];
  }
}
customElements.define('ds-button', DSButton);
```

### Out of Scope

- Light DOM fallback path
- Server-side rendering / declarative shadow DOM
- Legacy browser support
- Component implementations (this is infrastructure only)

### Open Questions

1. **Token prefix**: `--ds-*` is placeholder. Final prefix TBD.

2. **Dark mode**: `light-dark()` requires `color-scheme` property. Should `document.css` set this, or leave to consumer?

## References

- [CSS Cascading and Inheritance Level 5](https://www.w3.org/TR/css-cascade-5/)
- [CSS Properties and Values API Level 1](https://www.w3.org/TR/css-properties-values-api-1/)
- [DOM Standard: Shadow Trees](https://dom.spec.whatwg.org/#shadow-trees)
- [Constructable Stylesheets](https://web.dev/constructable-stylesheets/)
