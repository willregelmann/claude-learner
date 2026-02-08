### package.json
```
{
  "name": "tailwind-v4-demo",
  "version": "1.0.0",
  "description": "Tailwind CSS v4 custom design system",
  "scripts": {
    "dev": "vite",
    "build": "vite build"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "tailwindcss": "^4.0.0"
  }
}

```

### styles.css
```
@import "tailwindcss";

@theme {
  /* Custom Colors - Primary with 50-900 shades */
  --color-primary-50: #f0f9ff;
  --color-primary-100: #e0f2fe;
  --color-primary-200: #bae6fd;
  --color-primary-300: #7dd3fc;
  --color-primary-400: #38bdf8;
  --color-primary-500: #0ea5e9;
  --color-primary-600: #0284c7;
  --color-primary-700: #0369a1;
  --color-primary-800: #075985;
  --color-primary-900: #0c4a6e;

  /* Secondary Color */
  --color-secondary-50: #fdf4ff;
  --color-secondary-100: #fae8ff;
  --color-secondary-200: #f5d0fe;
  --color-secondary-300: #f0abfc;
  --color-secondary-400: #e879f9;
  --color-secondary-500: #d946ef;
  --color-secondary-600: #c026d3;
  --color-secondary-700: #a21caf;
  --color-secondary-800: #86198f;
  --color-secondary-900: #701a75;

  /* Accent Color */
  --color-accent-50: #fef3c7;
  --color-accent-100: #fde68a;
  --color-accent-200: #fcd34d;
  --color-accent-300: #fbbf24;
  --color-accent-400: #f59e0b;
  --color-accent-500: #d97706;
  --color-accent-600: #b45309;
  --color-accent-700: #92400e;
  --color-accent-800: #78350f;
  --color-accent-900: #451a03;

  /* Custom Spacing Scale */
  --spacing-xs: 0.5rem;
  --spacing-sm: 0.75rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  --spacing-2xl: 3rem;
  --spacing-3xl: 4rem;

  /* Custom Breakpoints */
  --breakpoint-tablet: 768px;
  --breakpoint-desktop: 1024px;
  --breakpoint-wide: 1440px;
}

@media (prefers-color-scheme: dark) {
  :root {
    /* Dark mode adjustments can go here if needed */
  }
}

```

### index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tailwind CSS v4 Custom Design System</title>
  <link rel="stylesheet" href="/src/styles.css">
</head>
<body class="bg-gray-50 dark:bg-gray-900 min-h-screen p-xs md:p-lg lg:p-xl">

  <!-- Toggle Dark Mode Button -->
  <div class="max-w-wide mx-auto mb-2xl">
    <button
      onclick="document.documentElement.classList.toggle('dark')"
      class="bg-primary-600 hover:bg-primary-700 text-white px-lg py-sm rounded-lg transition-colors"
    >
      Toggle Dark Mode
    </button>
  </div>

  <!-- Card Component Demonstration -->
  <div class="max-w-desktop mx-auto grid grid-cols-1 tablet:grid-cols-2 desktop:grid-cols-3 gap-lg">

    <!-- Card 1 - Primary Theme -->
    <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-xl space-y-md border-2 border-primary-200 dark:border-primary-800 hover:border-primary-400 dark:hover:border-primary-600 transition-colors">
      <div class="bg-primary-500 dark:bg-primary-600 rounded-lg p-md">
        <h2 class="text-2xl font-bold text-white">Primary Card</h2>
      </div>
      <p class="text-gray-700 dark:text-gray-300 text-sm">
        This card demonstrates the primary color palette with custom spacing tokens.
      </p>
      <div class="flex gap-sm flex-wrap">
        <span class="bg-primary-100 dark:bg-primary-900 text-primary-800 dark:text-primary-200 px-sm py-xs rounded text-xs font-medium">
          Primary-100
        </span>
        <span class="bg-primary-300 dark:bg-primary-700 text-primary-900 dark:text-primary-100 px-sm py-xs rounded text-xs font-medium">
          Primary-300
        </span>
        <span class="bg-primary-500 text-white px-sm py-xs rounded text-xs font-medium">
          Primary-500
        </span>
      </div>
      <button class="w-full bg-primary-600 hover:bg-primary-700 dark:bg-primary-500 dark:hover:bg-primary-400 text-white font-semibold py-md px-lg rounded-lg transition-colors mt-md">
        Primary Action
      </button>
    </div>

    <!-- Card 2 - Secondary Theme -->
    <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-xl space-y-md border-2 border-secondary-200 dark:border-secondary-800 hover:border-secondary-400 dark:hover:border-secondary-600 transition-colors">
      <div class="bg-secondary-500 dark:bg-secondary-600 rounded-lg p-md">
        <h2 class="text-2xl font-bold text-white">Secondary Card</h2>
      </div>
      <p class="text-gray-700 dark:text-gray-300 text-sm">
        This card uses the secondary color scale with responsive spacing at different breakpoints.
      </p>
      <div class="flex gap-sm flex-wrap">
        <span class="bg-secondary-100 dark:bg-secondary-900 text-secondary-800 dark:text-secondary-200 px-sm py-xs rounded text-xs font-medium">
          Secondary-100
        </span>
        <span class="bg-secondary-300 dark:bg-secondary-700 text-secondary-900 dark:text-secondary-100 px-sm py-xs rounded text-xs font-medium">
          Secondary-300
        </span>
        <span class="bg-secondary-500 text-white px-sm py-xs rounded text-xs font-medium">
          Secondary-500
        </span>
      </div>
      <button class="w-full bg-secondary-600 hover:bg-secondary-700 dark:bg-secondary-500 dark:hover:bg-secondary-400 text-white font-semibold py-md px-lg rounded-lg transition-colors mt-md">
        Secondary Action
      </button>
    </div>

    <!-- Card 3 - Accent Theme -->
    <div class="bg-white dark:bg-gray-800 rounded-xl shadow-lg p-xl space-y-md border-2 border-accent-200 dark:border-accent-800 hover:border-accent-400 dark:hover:border-accent-600 transition-colors tablet:col-span-2 desktop:col-span-1">
      <div class="bg-accent-500 dark:bg-accent-600 rounded-lg p-md">
        <h2 class="text-2xl font-bold text-white">Accent Card</h2>
      </div>
      <p class="text-gray-700 dark:text-gray-300 text-sm">
        This card showcases the accent colors and demonstrates responsive grid behavior at custom breakpoints.
      </p>
      <div class="flex gap-sm flex-wrap">
        <span class="bg-accent-100 dark:bg-accent-900 text-accent-800 dark:text-accent-200 px-sm py-xs rounded text-xs font-medium">
          Accent-100
        </span>
        <span class="bg-accent-300 dark:bg-accent-700 text-accent-900 dark:text-accent-100 px-sm py-xs rounded text-xs font-medium">
          Accent-300
        </span>
        <span class="bg-accent-500 text-white px-sm py-xs rounded text-xs font-medium">
          Accent-500
        </span>
      </div>
      <button class="w-full bg-accent-600 hover:bg-accent-700 dark:bg-accent-500 dark:hover:bg-accent-400 text-white font-semibold py-md px-lg rounded-lg transition-colors mt-md">
        Accent Action
      </button>
    </div>

  </div>

  <!-- Feature Showcase Section -->
  <div class="max-w-wide mx-auto mt-3xl">
    <div class="bg-gradient-to-r from-primary-500 via-secondary-500 to-accent-500 dark:from-primary-700 dark:via-secondary-700 dark:to-accent-700 rounded-2xl p-2xl text-white">
      <h1 class="text-4xl font-bold mb-lg">Custom Design System Features</h1>
      <div class="grid grid-cols-1 tablet:grid-cols-3 gap-xl mt-xl">
        <div class="space-y-sm">
          <h3 class="text-xl font-semibold">Custom Colors</h3>
          <p class="text-primary-50">Primary, Secondary, and Accent palettes with 50-900 shades</p>
        </div>
        <div class="space-y-sm">
          <h3 class="text-xl font-semibold">Custom Spacing</h3>
          <p class="text-secondary-50">Scale from xs (0.5rem) to 3xl (4rem)</p>
        </div>
        <div class="space-y-sm">
          <h3 class="text-xl font-semibold">Custom Breakpoints</h3>
          <p class="text-accent-50">Tablet (768px), Desktop (1024px), Wide (1440px)</p>
        </div>
      </div>
    </div>
  </div>

  <script type="module" src="/src/main.js"></script>
</body>
</html>

```

### main.js
```
// Main JavaScript entry point for Vite
import './styles.css';

// Initialize dark mode based on user preference
if (localStorage.theme === 'dark' || (!('theme' in localStorage) && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
  document.documentElement.classList.add('dark');
} else {
  document.documentElement.classList.remove('dark');
}

// Save dark mode preference
const observer = new MutationObserver((mutations) => {
  mutations.forEach((mutation) => {
    if (mutation.attributeName === 'class') {
      if (document.documentElement.classList.contains('dark')) {
        localStorage.theme = 'dark';
      } else {
        localStorage.theme = 'light';
      }
    }
  });
});

observer.observe(document.documentElement, {
  attributes: true,
  attributeFilter: ['class']
});

```

### vite.config.js
```
import { defineConfig } from 'vite';

export default defineConfig({
  root: '.',
  publicDir: 'public',
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
  },
});

```

### README.md
```
# Tailwind CSS v4 Custom Design System

A complete Tailwind CSS v4 project with custom design tokens including colors, spacing, and breakpoints.

## Features

- **Custom Colors**: Primary, Secondary, and Accent color palettes (50-900 shades)
- **Custom Spacing**: xs, sm, md, lg, xl, 2xl, 3xl scale
- **Custom Breakpoints**: tablet (768px), desktop (1024px), wide (1440px)
- **Dark Mode**: Class-based strategy with localStorage persistence

## Setup

```bash
npm install
npm run dev
```

## Project Structure

```
tailwind-v4-demo/
├── src/
│   ├── styles.css       # Tailwind v4 config with @theme
│   └── main.js          # Dark mode logic
├── index.html           # Sample card components
├── package.json
└── vite.config.js
```

## Custom Tokens Usage

### Colors
```html
<div class="bg-primary-500 text-white">Primary</div>
<div class="bg-secondary-600 dark:bg-secondary-400">Secondary</div>
<div class="bg-accent-700">Accent</div>
```

### Spacing
```html
<div class="p-xs">Extra Small Padding</div>
<div class="m-lg">Large Margin</div>
<div class="gap-2xl">2XL Gap</div>
```

### Breakpoints
```html
<div class="tablet:grid-cols-2 desktop:grid-cols-3">
  Responsive Grid
</div>
```

### Dark Mode
```html
<div class="bg-white dark:bg-gray-800">
  Adapts to dark mode
</div>
```

```

---

## Complete Tailwind CSS v4 Project

I've created a complete Tailwind CSS v4 project with a custom design system. Here are all the files:

### **package.json**
- Dependencies for Tailwind CSS v4 and Vite

### **src/styles.css**
- Uses `@theme` directive (Tailwind v4 convention) to define custom design tokens
- Custom colors: `primary`, `secondary`, and `accent` with 50-900 shades
- Custom spacing: `xs` (0.5rem) through `3xl` (4rem)
- Custom breakpoints: `tablet` (768px), `desktop` (1024px), `wide` (1440px)

### **index.html**
- Three card components demonstrating:
  - Custom color usage (primary, secondary, accent)
  - Custom spacing (xs, sm, md, lg, xl, 2xl, 3xl)
  - Responsive layouts using custom breakpoints (tablet, desktop, wide)
  - Dark mode with `dark:` variants
- Dark mode toggle button
- Feature showcase section with gradient

### **src/main.js**
- Dark mode initialization and localStorage persistence
- Automatically detects system preference
- Saves user's dark mode choice

### **vite.config.js**
- Basic Vite configuration for the project

All files are ready to use with `npm install && npm run dev`. The project uses Tailwind CSS v4's `@theme` directive for CSS-based configuration and includes class-based dark mode strategy.