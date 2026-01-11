# Design System

A modern, vanilla CSS design system foundation. No build step. No dependencies.

## Features

- **CSS Grid** - 12-column grid with responsive breakpoints
- **Container Queries** - Component-level responsive design
- **CSS Layers** - Predictable cascade with `@layer`
- **Custom Properties** - Full design token system
- **Dark Mode** - Automatic `prefers-color-scheme` support + manual toggle
- **Accessibility** - Screen reader utilities, skip links, focus states
- **Logical Properties** - RTL-ready with `margin-inline`, `padding-block`, etc.

## Installation

Copy the `css/` directory to your project:

```
css/
├── index.css      # Entry point (import this)
├── tokens.css     # Design tokens
├── base.css       # Reset & defaults
├── grid.css       # Grid system
└── utilities.css  # Utility classes
```

Link in your HTML:

```html
<link rel="stylesheet" href="./css/index.css">
```

## Usage

### Grid

12-column grid using CSS Grid:

```html
<div class="row">
  <div class="col-4">1/3</div>
  <div class="col-4">1/3</div>
  <div class="col-4">1/3</div>
</div>
```

### Responsive

Mobile-first breakpoints: `sm` (640px), `md` (768px), `lg` (1024px), `xl` (1280px), `2xl` (1536px)

```html
<div class="col-12 sm:col-6 lg:col-3">
  Full width → Half → Quarter
</div>
```

### Container Queries

Component-level responsive design:

```html
<div class="cq-container">
  <div class="row">
    <div class="col-12 @sm:col-6 @md:col-4">
      Responds to container width
    </div>
  </div>
</div>
```

Container breakpoints: `@xs` (320px), `@sm` (480px), `@md` (640px), `@lg` (800px)

### Column Positioning

```html
<!-- Offset -->
<div class="col-6 col-start-4">Centered</div>

<!-- Explicit start/end -->
<div class="col-start-2 col-end-8">Columns 2-7</div>
```

### Dark Mode

Automatic via `prefers-color-scheme`, or manual:

```html
<html data-theme="dark">
```

```html
<html data-theme="light">
```

## Design Tokens

All values are CSS custom properties. Override in your own CSS:

```css
:root {
  --color-primary: oklch(55% 0.2 280);
  --grid-gap: 2rem;
  --font-sans: "Inter", system-ui, sans-serif;
}
```

### Spacing Scale

```
--space-1:  0.25rem (4px)
--space-2:  0.5rem  (8px)
--space-3:  0.75rem (12px)
--space-4:  1rem    (16px)
--space-6:  1.5rem  (24px)
--space-8:  2rem    (32px)
--space-12: 3rem    (48px)
--space-16: 4rem    (64px)
```

### Typography Scale

```
--text-xs:  0.75rem
--text-sm:  0.875rem
--text-base: 1rem
--text-lg:  1.125rem
--text-xl:  1.25rem
--text-2xl: 1.5rem
--text-3xl: 1.875rem
--text-4xl: 2.25rem
```

### Colors

Semantic color tokens that adapt to light/dark mode:

```
--color-bg
--color-surface
--color-text
--color-text-muted
--color-primary
--color-error
--color-success
--color-warning
```

## Utilities Reference

### Layout

| Class | Description |
|-------|-------------|
| `.container` | Centered max-width container |
| `.row` | Grid container (12 columns) |
| `.col-{1-12}` | Column span |
| `.col-start-{1-13}` | Column start position |
| `.col-full` | Full width (all 12 columns) |

### Flexbox

| Class | Description |
|-------|-------------|
| `.flex` | `display: flex` |
| `.flex-col` | Column direction |
| `.items-center` | Align items center |
| `.justify-between` | Space between |
| `.gap-{0-16}` | Gap using spacing scale |
| `.grow` / `.shrink-0` | Flex grow/shrink |

### Spacing

| Class | Description |
|-------|-------------|
| `.p-{0-16}` | Padding (all sides) |
| `.px-{0-12}` | Padding inline |
| `.py-{0-12}` | Padding block |
| `.m-{0-8}` | Margin (all sides) |
| `.mx-auto` | Center horizontally |
| `.my-{0-8}` | Margin block |

### Typography

| Class | Description |
|-------|-------------|
| `.text-{xs-6xl}` | Font size |
| `.font-{normal,medium,semibold,bold}` | Font weight |
| `.text-{left,center,right}` | Alignment |
| `.truncate` | Ellipsis overflow |

### Colors

| Class | Description |
|-------|-------------|
| `.text-{default,muted,subtle}` | Text colors |
| `.text-{primary,error,success}` | Semantic text |
| `.bg-{default,surface,muted}` | Backgrounds |
| `.bg-{primary,error,success}` | Semantic backgrounds |

### Borders

| Class | Description |
|-------|-------------|
| `.border` | 1px border |
| `.border-{t,b,s,e}` | Single side |
| `.rounded` | Default radius |
| `.rounded-{sm,md,lg,xl,full}` | Radius sizes |

### Effects

| Class | Description |
|-------|-------------|
| `.shadow-{sm,md,lg,xl}` | Box shadows |
| `.opacity-{0-100}` | Opacity |
| `.transition` | Smooth transitions |

### Accessibility

| Class | Description |
|-------|-------------|
| `.sr-only` | Screen reader only |
| `.skip-link` | Skip navigation link |

## Browser Support

- Chrome/Edge 105+
- Firefox 121+
- Safari 16.4+

Requires support for: CSS Grid, Container Queries, CSS Layers, `oklch()`.

## Architecture

CSS is organized in layers for predictable cascade:

1. `tokens` - Custom properties
2. `base` - Reset and defaults
3. `layout` - Grid system
4. `utilities` - Utility classes

Later layers override earlier ones. Utilities always win.

## License

MIT
