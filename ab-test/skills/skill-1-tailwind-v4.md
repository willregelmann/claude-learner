---
name: tailwind-v4-configuration
description: Use when configuring Tailwind CSS v4, setting up custom themes, migrating from v3, or working with v4's CSS-based configuration system.
---

# Tailwind CSS v4 Configuration

## When This Skill Applies

- Setting up a new Tailwind CSS v4 project from scratch
- Migrating from Tailwind CSS v3 to v4
- Configuring custom design tokens (colors, spacing, typography, breakpoints)
- Implementing dark mode or multi-theme systems
- Troubleshooting "utility class not found" errors in v4
- Converting JavaScript-based configuration to CSS-based configuration
- Debugging CSS variable conflicts or missing theme values
- Setting up monorepos or multi-package projects with Tailwind v4

## Key Patterns

### Install and Initialize Tailwind v4

Install the package with minimal dependencies:

```bash
npm install tailwindcss@next @tailwindcss/vite
```

For Vite projects, add the plugin to `vite.config.js`:

```javascript
import tailwindcss from '@tailwindcss/vite'

export default {
  plugins: [tailwindcss()]
}
```

Create your main CSS file (e.g., `app.css`) with a single import:

```css
@import "tailwindcss";
```

**Critical**: Use `@import "tailwindcss"`, not the old `@tailwind base; @tailwind components; @tailwind utilities;` directives from v3. The v3 directives will not work in v4.

### Configure Custom Theme Tokens Using @theme

Define design tokens in CSS using the `@theme` directive. Theme variables map directly to utility classes:

```css
@import "tailwindcss";

@theme {
  --color-primary: #1d4ed8;
  --color-secondary: #9333ea;
  --color-accent: #f59e0b;
  
  --font-display: "Inter", sans-serif;
  --font-body: "Roboto", sans-serif;
  
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 2rem;
  --spacing-xl: 4rem;
}
```

This creates utilities like `bg-primary`, `text-secondary`, `font-display`, `p-xs`, `m-lg`, etc. Each namespace (`--color-*`, `--font-*`, `--spacing-*`) corresponds to specific utility APIs.

### Override or Reset Default Theme Values

To completely replace a namespace (e.g., all default colors), use the asterisk wildcard:

```css
@theme {
  /* Remove all default colors */
  --color-*: initial;
  
  /* Define only your custom colors */
  --color-brand-blue: #0066cc;
  --color-brand-gray: #6b7280;
  --color-white: #ffffff;
  --color-black: #000000;
}
```

To extend the default theme while keeping existing values, just add new variables without resetting:

```css
@theme {
  /* Keeps all default colors, adds new ones */
  --color-brand: #1d4ed8;
  --color-success: #10b981;
  --color-warning: #f59e0b;
}
```

### Configure Custom Breakpoints

Define responsive breakpoints using `--breakpoint-*` variables. Always use consistent units (all rem or all px):

```css
@theme {
  /* Replace all default breakpoints */
  --breakpoint-*: initial;
  
  --breakpoint-mobile: 23.4375rem;    /* 375px */
  --breakpoint-tablet: 48rem;         /* 768px */
  --breakpoint-laptop: 64rem;         /* 1024px */
  --breakpoint-desktop: 90rem;        /* 1440px */
}
```

**Warning**: Never mix rem and px units for breakpoints. This breaks the ordering system and causes unpredictable behavior.

Use breakpoints in HTML:

```html
<div class="w-full tablet:w-1/2 desktop:w-1/3">
  Responsive grid item
</div>
```

### Implement Dark Mode with Custom Variants

Define a dark mode variant using `@custom-variant`:

```css
@import "tailwindcss";

/* Class-based dark mode */
@custom-variant dark (&:is(.dark *));

@theme {
  --color-text: #0f172a;
  --color-bg: #ffffff;
}

.dark {
  --color-text: #f1f5f9;
  --color-bg: #0f172a;
}
```

For data-attribute based dark mode:

```css
@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));
```

Use dark mode utilities in templates:

```html
<div class="bg-bg text-text dark:bg-slate-900 dark:text-slate-100">
  Content adapts to dark mode
</div>
```

Toggle dark mode with JavaScript:

```javascript
// Add/remove class on document element
document.documentElement.classList.toggle('dark')

// Or use data attribute
document.documentElement.setAttribute('data-theme', 'dark')
```

### Use Theme Variables in Custom CSS

Access theme variables as CSS variables anywhere in your styles:

```css
@import "tailwindcss";

@theme {
  --color-primary: #1d4ed8;
  --spacing-section: 4rem;
}

.custom-component {
  background-color: var(--color-primary);
  padding-block: var(--spacing-section);
  border-radius: 0.5rem;
}
```

**Key difference**: Variables defined in `@theme` create utilities AND are available as CSS variables. Regular `:root` variables do not create utilities.

### Control Content Detection with @source

Tailwind v4 auto-detects template files. Override this with `@source`:

```css
@import "tailwindcss";
@source "../components";
@source "../../packages/ui/src";
```

Useful in monorepos where templates live in multiple packages or when you need explicit control over which directories are scanned.

### Structure Files for Maintainability

Organize theme configuration across multiple files using CSS imports:

```css
/* app.css */
@import "tailwindcss";
@import "./theme/colors.css";
@import "./theme/typography.css";
@import "./theme/spacing.css";
@import "./theme/variants.css";
```

```css
/* theme/colors.css */
@theme {
  --color-*: initial;
  --color-primary: #1d4ed8;
  --color-secondary: #9333ea;
  /* ... more colors */
}
```

Tailwind v4's built-in import support handles bundling without additional tools like postcss-import.

### Migrate from v3 Configuration

Use the automated upgrade tool:

```bash
npx @tailwindcss/upgrade
```

This tool:
- Updates dependencies in package.json
- Converts `tailwind.config.js` to CSS-based `@theme` blocks
- Replaces `@tailwind` directives with `@import`
- Updates template syntax where needed

Manual migration checklist:
1. Replace `@tailwind base; @tailwind components; @tailwind utilities;` with `@import "tailwindcss";`
2. Convert `tailwind.config.js` theme extensions to `@theme` blocks
3. Replace `decoration-slice/decoration-clone` with `box-decoration-slice/box-decoration-clone`
4. Update arbitrary values: use underscores for spaces (e.g., `grid-cols-[repeat(3,_1fr)]`)
5. Remove deprecated plugins (they may not have v4 versions yet)

## Common Mistakes to Avoid

**Using v3 @tailwind directives** → Replace `@tailwind base; @tailwind components; @tailwind utilities;` with `@import "tailwindcss";`. The old directives are not supported in v4.

**Defining CSS variables in :root instead of @theme** → Use `@theme { --color-brand: #1d4ed8; }`, not `:root { --color-brand: #1d4ed8; }`. Only `@theme` variables generate utilities.

**Mixing rem and px units for breakpoints** → Use one unit consistently: `--breakpoint-tablet: 48rem;` not mixing with `--breakpoint-laptop: 1024px;`. Mixed units break ordering.

**Expecting all v3 plugins to work** → Many v3 plugins are not v4-compatible yet. Check plugin documentation for v4 support before upgrading.

**Forgetting to reset namespaces when overriding** → Use `--color-*: initial;` before defining custom colors if you want to remove all defaults. Without this, your colors extend the defaults.

**Not updating browser support expectations** → v4 requires Safari 16.4+, Chrome 111+, Firefox 128+. Older browsers are not supported due to modern CSS features like `@property` and `color-mix()`.

**Using commas in arbitrary values** → Replace commas with underscores in utilities like `grid-cols-[repeat(3,_1fr)]`. The v3 comma-to-space conversion no longer exists.

**Assuming tailwind.config.js is required** → v4 uses CSS-first configuration. You only need `tailwind.config.js` for PostCSS-specific or build-tool integrations that can't be done in CSS.

**Not configuring dark mode variants** → Dark mode requires explicit variant setup in v4 using `@custom-variant dark (&:is(.dark *))`. It's not automatic.

**Forgetting content detection in monorepos** → Use `@source` directives to explicitly include template directories when auto-detection misses files in multi-package projects.

## Examples

### Complete Multi-Theme Setup with Dark Mode

```css
/* app.css */
@import "tailwindcss";

@custom-variant dark (&:is(.dark *));

@theme {
  /* Base colors */
  --color-text: #1e293b;
  --color-background: #ffffff;
  --color-primary: #3b82f6;
  --color-secondary: #8b5cf6;
  
  /* Semantic spacing */
  --spacing-component: 1.5rem;
  --spacing-section: 4rem;
  
  /* Custom breakpoints */
  --breakpoint-*: initial;
  --breakpoint-sm: 40rem;
  --breakpoint-md: 48rem;
  --breakpoint-lg: 64rem;
  --breakpoint-xl: 80rem;
}

/* Dark theme overrides */
.dark {
  --color-text: #f1f5f9;
  --color-background: #0f172a;
  --color-primary: #60a5fa;
  --color-secondary: #a78bfa;
}
```

```html
<!-- Usage in templates -->
<body class="bg-background text-text">
  <header class="p-component lg:p-section">
    <h1 class="text-primary">Welcome</h1>
  </header>
  
  <button class="bg-primary text-white dark:bg-primary dark:text-slate-900">
    Click me
  </button>
</body>

<script>
  // Toggle dark mode
  const toggle = document.getElementById('theme-toggle')
  toggle.addEventListener('click', () => {
    document.documentElement.classList.toggle('dark')
    localStorage.setItem('theme', 
      document.documentElement.classList.contains('dark') ? 'dark' : 'light'
    )
  })
  
  // Persist preference
  if (localStorage.getItem('theme') === 'dark') {
    document.documentElement.classList.add('dark')
  }
</script>
```

### Vite + Monorepo Configuration

```javascript
// apps/web/vite.config.js
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [tailwindcss()]
})
```

```css
/* apps/web/src/styles/app.css */
@import "tailwindcss";

/* Include UI package components */
@source "../../packages/ui/src/components";

/* Shared theme */
@import "../../../packages/ui/theme/tokens.css";
```

```css
/* packages/ui/theme/tokens.css */
@theme {
  --color-*: initial;
  
  --color-brand-primary: #1d4ed8;
  --color-brand-secondary: #9333ea;
  --color-neutral-50: #f9fafb;
  --color-neutral-900: #111827;
  
  --font-sans: "Inter", system-ui, sans-serif;
  --font-mono: "Fira Code", monospace;
  
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 1rem;
}
```

Sources:
- [Tailwind CSS v4.0 Official Release](https://tailwindcss.com/blog/tailwindcss-v4)
- [Theme Variables Documentation](https://tailwindcss.com/docs/theme)
- [Upgrade Guide v3 to v4](https://tailwindcss.com/docs/upgrade-guide)
- [Dark Mode in Tailwind v4](https://tailwindcss.com/docs/dark-mode)
- [Tailwind CSS v4 Deep Dive](https://dev.to/dataformathub/tailwind-css-v4-deep-dive-why-the-oxide-engine-changes-everything-in-2026-2595)
- [Configuring Tailwind CSS v4](https://bryananthonio.com/blog/configuring-tailwind-css-v4/)
- [Migration Guide for v4](https://typescript.tv/hands-on/upgrading-to-tailwind-css-v4-a-migration-guide/)
- [Dark Mode Implementation Tutorial](https://dev.to/tene/dark-mode-using-tailwindcss-v40-2lc6)
- [Theming in Tailwind v4](https://medium.com/@sir.raminyavari/theming-in-tailwind-css-v4-support-multiple-color-schemes-and-dark-mode-ba97aead5c14)
- [Custom Breakpoints Guide](https://bordermedia.org/blog/tailwind-css-4-breakpoint-override)
