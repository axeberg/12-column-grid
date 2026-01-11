# Modern CSS Design System

A cutting-edge CSS design system using the latest CSS features. No build step. No preprocessor.

## Features

| Feature | Usage |
|---------|-------|
| **CSS Nesting** | Native `&` selector throughout |
| **@property** | Typed, animatable custom properties |
| **light-dark()** | Single declaration theming |
| **@layer** | Predictable cascade control |
| **Container Queries** | Component-level responsive |
| **Style Queries** | Variants via custom properties |
| **:has()** | Parent selectors, form validation |
| **Scroll Animations** | `animation-timeline: view()` |
| **@starting-style** | Entry animations |
| **Anchor Positioning** | Tooltips without JS |
| **Popover API** | Native modals and dropdowns |
| **View Transitions** | Page transition support |
| **interpolate-size** | Animate to `auto` |

## Installation

```html
<link rel="stylesheet" href="./css/index.css">
```

## Quick Start

```html
<div class="container">
  <div class="row">
    <div class="col-12 md:col-6 lg:col-4">
      <div class="bg-surface border rounded-lg p-4 shadow animate-in">
        Content
      </div>
    </div>
  </div>
</div>
```

## Grid

12-column responsive grid:

```html
<!-- Basic -->
<div class="row">
  <div class="col-6">Half</div>
  <div class="col-6">Half</div>
</div>

<!-- Responsive -->
<div class="col-12 sm:col-6 lg:col-3">
  Full → Half → Quarter
</div>

<!-- Container queries -->
<div class="cq">
  <div class="col-12 @sm:col-6 @md:col-4">
    Responds to container
  </div>
</div>
```

## Theming

Uses `light-dark()` for automatic theme switching:

```css
/* Automatic via prefers-color-scheme */
:root { color-scheme: light dark; }

/* Manual override */
<html data-theme="dark">
```

Change primary color by adjusting hue:

```css
:root { --hue-primary: 250; }  /* Blue */
:root { --hue-primary: 330; }  /* Pink */
:root { --hue-primary: 145; }  /* Green */
```

## Scroll Animations

Animate elements on scroll without JavaScript:

```html
<div class="scroll-fade">Fades in</div>
<div class="scroll-scale">Scales in</div>
<div class="scroll-slide-left">Slides from left</div>
<div class="scroll-slide-right">Slides from right</div>
```

Scroll progress bar:

```html
<div class="scroll-progress"></div>
```

## Entry Animations

Elements animate when they mount:

```html
<div class="animate-in">Fades up</div>
<div class="animate-scale-in">Scales in</div>
<div class="animate-slide-up">Slides up</div>
```

## :has() Utilities

CSS-only form validation:

```html
<div class="form-group">
  <label>Email</label>
  <input type="email" required>
  <!-- Auto-shows * for required, colors for valid/invalid -->
</div>
```

Auto-grid based on children count:

```html
<div class="auto-cols">
  <div>1</div>
  <div>2</div>
  <!-- Automatically creates 2-column grid -->
</div>
```

## Popovers & Dialogs

Native popover with animated entry:

```html
<button popovertarget="menu">Open</button>
<div id="menu" popover>
  Content here
</div>
```

## Anchor Positioning

Position elements relative to anchors:

```html
<button class="anchor" style="anchor-name: --btn;">Click</button>
<div class="anchored-top" style="position-anchor: --btn;">
  Tooltip
</div>
```

## Utilities

### Spacing

```
.p-{0-16}     Padding
.px-{0-8}    Padding inline
.py-{0-16}    Padding block
.m-{0-8}      Margin
.gap-{0-16}   Gap
```

### Typography

```
.text-{xs-6xl}  Font size
.font-{thin-black}  Weight
.text-{left,center,right}
.truncate  Ellipsis
.line-clamp-{2,3,4}
```

### Colors

```
.text-{default,muted,subtle,primary,error,success}
.bg-{default,surface,muted,primary,error,success}
.bg-{primary,success,warning,error}-subtle
```

### Layout

```
.flex .flex-col .flex-wrap
.items-{start,center,end}
.justify-{start,center,between}
.grow .shrink-0
```

## Animation Timing

```css
--ease-linear
--ease-in
--ease-out
--ease-in-out
--ease-bounce
--ease-elastic
--spring  /* Spring physics via linear() */
```

## Browser Support

| Browser | Version |
|---------|---------|
| Chrome | 125+ |
| Edge | 125+ |
| Firefox | 128+ |
| Safari | 17.4+ |

## File Structure

```
css/
├── index.css      # Entry point with @layer order
├── tokens.css     # @property, colors, spacing
├── base.css       # Reset with nesting
├── grid.css       # Grid + container queries
└── utilities.css  # All utilities
```

## License

MIT
