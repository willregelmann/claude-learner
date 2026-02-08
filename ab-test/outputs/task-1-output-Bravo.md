Now I'll create a complete Tailwind CSS v4 project with the requested features, following the v4 conventions from the reference guide.

---

**styles/app.css**
```css
@import "tailwindcss";

@custom-variant dark (&:is(.dark *));

@theme {
  /* Custom colors with full shade scale for primary */
  --color-primary-50: #eff6ff;
  --color-primary-100: #dbeafe;
  --color-primary-200: #bfdbfe;
  --color-primary-300: #93c5fd;
  --color-primary-400: #60a5fa;
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;
  --color-primary-700: #1d4ed8;
  --color-primary-800: #1e40af;
  --color-primary-900: #1e3a8a;
  
  --color-secondary: #9333ea;
  --color-accent: #f59e0b;
  
  /* Custom spacing scale */
  --spacing-xs: 0.5rem;
  --spacing-sm: 0.75rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  --spacing-2xl: 3rem;
  --spacing-3xl: 4rem;
  
  /* Custom breakpoints - reset defaults and define custom ones */
  --breakpoint-*: initial;
  --breakpoint-tablet: 48rem;
  --breakpoint-desktop: 64rem;
  --breakpoint-wide: 90rem;
}

/* Dark mode color overrides */
.dark {
  --color-primary-50: #1e3a8a;
  --color-primary-100: #1e40af;
  --color-primary-200: #1d4ed8;
  --color-primary-300: #2563eb;
  --color-primary-400: #3b82f6;
  --color-primary-500: #60a5fa;
  --color-primary-600: #93c5fd;
  --color-primary-700: #bfdbfe;
  --color-primary-800: #dbeafe;
  --color-primary-900: #eff6ff;
  
  --color-secondary: #a78bfa;
  --color-accent: #fbbf24;
}
```

---

**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tailwind CSS v4 Custom Design System</title>
  <link rel="stylesheet" href="styles/app.css">
</head>
<body class="bg-primary-50 dark:bg-slate-900 p-lg transition-colors">
  
  <div class="max-w-screen-tablet mx-auto">
    <header class="mb-2xl">
      <h1 class="text-primary-700 dark:text-primary-300 text-4xl font-bold mb-md">
        Custom Design System Demo
      </h1>
      <button 
        id="theme-toggle"
        class="bg-accent text-white px-lg py-sm rounded-lg hover:opacity-90 transition-opacity"
      >
        Toggle Dark Mode
      </button>
    </header>

    <!-- Sample Card Component -->
    <div class="grid grid-cols-1 tablet:grid-cols-2 desktop:grid-cols-3 gap-lg">
      
      <!-- Card 1 -->
      <article class="bg-white dark:bg-slate-800 rounded-xl shadow-lg p-xl transition-colors">
        <div class="bg-primary-500 dark:bg-primary-600 w-full h-48 rounded-lg mb-md"></div>
        <h2 class="text-primary-800 dark:text-primary-200 text-2xl font-bold mb-sm">
          Custom Card
        </h2>
        <p class="text-slate-600 dark:text-slate-300 mb-md">
          This card demonstrates custom spacing (p-xl, mb-md), custom colors (primary shades), 
          and custom breakpoints (tablet, desktop).
        </p>
        <div class="flex gap-sm">
          <button class="bg-primary-600 dark:bg-primary-500 text-white px-md py-xs rounded-lg hover:bg-primary-700 dark:hover:bg-primary-400 transition-colors">
            Primary
          </button>
          <button class="bg-secondary text-white px-md py-xs rounded-lg hover:opacity-90 transition-opacity">
            Secondary
          </button>
        </div>
      </article>

      <!-- Card 2 -->
      <article class="bg-white dark:bg-slate-800 rounded-xl shadow-lg p-xl transition-colors">
        <div class="bg-secondary w-full h-48 rounded-lg mb-md"></div>
        <h2 class="text-primary-800 dark:text-primary-200 text-2xl font-bold mb-sm">
          Responsive Grid
        </h2>
        <p class="text-slate-600 dark:text-slate-300 mb-md">
          On mobile: 1 column<br>
          On tablet (48rem): 2 columns<br>
          On desktop (64rem): 3 columns
        </p>
        <div class="flex gap-sm">
          <button class="bg-accent text-white px-md py-xs rounded-lg hover:opacity-90 transition-opacity">
            Accent
          </button>
        </div>
      </article>

      <!-- Card 3 -->
      <article class="bg-white dark:bg-slate-800 rounded-xl shadow-lg p-xl transition-colors">
        <div class="bg-accent w-full h-48 rounded-lg mb-md"></div>
        <h2 class="text-primary-800 dark:text-primary-200 text-2xl font-bold mb-sm">
          Dark Mode
        </h2>
        <p class="text-slate-600 dark:text-slate-300 mb-md">
          Click the toggle button above to switch between light and dark themes. 
          All colors adapt automatically using the dark: variant.
        </p>
        <div class="space-y-xs">
          <div class="flex items-center gap-xs">
            <div class="w-xs h-xs bg-primary-500 rounded-full"></div>
            <span class="text-sm text-slate-700 dark:text-slate-300">Custom spacing: xs</span>
          </div>
          <div class="flex items-center gap-sm">
            <div class="w-sm h-sm bg-primary-600 rounded-full"></div>
            <span class="text-sm text-slate-700 dark:text-slate-300">Custom spacing: sm</span>
          </div>
          <div class="flex items-center gap-md">
            <div class="w-md h-md bg-primary-700 rounded-full"></div>
            <span class="text-sm text-slate-700 dark:text-slate-300">Custom spacing: md</span>
          </div>
        </div>
      </article>

    </div>

    <!-- Spacing Scale Demo -->
    <section class="mt-3xl bg-white dark:bg-slate-800 rounded-xl shadow-lg p-2xl transition-colors">
      <h2 class="text-primary-800 dark:text-primary-200 text-3xl font-bold mb-xl">
        Custom Spacing Scale
      </h2>
      <div class="space-y-lg">
        <div class="flex items-center gap-md">
          <code class="bg-primary-100 dark:bg-primary-900 text-primary-800 dark:text-primary-200 px-sm py-xs rounded">xs</code>
          <div class="bg-accent h-xs"></div>
          <span class="text-slate-600 dark:text-slate-300">0.5rem</span>
        </div>
        <div class="flex items-center gap-md">
          <code class="bg-primary-100 dark:bg-primary-900 text-primary-800 dark:text-primary-200 px-sm py-xs rounded">sm</code>
          <div class="bg-accent h-sm"></div>
          <span class="text-slate-600 dark:text-slate-300">0.75rem</span>
        </div>
        <div class="flex items-center gap-md">
          <code class="bg-primary-100 dark:bg-primary-900 text-primary-800 dark:text-primary-200 px-sm py-xs rounded">md</code>
          <div class="bg-accent h-md"></div>
          <span class="text-slate-600 dark:text-slate-300">1rem</span>
        </div>
        <div class="flex items-center gap-md">
          <code class="bg-primary-100 dark:bg-primary-900 text-primary-800 dark:text-primary-200 px-sm py-xs rounded">lg</code>
          <div class="bg-accent h-lg"></div>
          <span class="text-slate-600 dark:text-slate-300">1.5rem</span>
        </div>
        <div class="flex items-center gap-md">
          <code class="bg-primary-100 dark:bg-primary-900 text-primary-800 dark:text-primary-200 px-sm py-xs rounded">xl</code>
          <div class="bg-accent h-xl"></div>
          <span class="text-slate-600 dark:text-slate-300">2rem</span>
        </div>
        <div class="flex items-center gap-md">
          <code class="bg-primary-100 dark:bg-primary-900 text-primary-800 dark:text-primary-200 px-sm py-xs rounded">2xl</code>
          <div class="bg-accent h-2xl"></div>
          <span class="text-slate-600 dark:text-slate-300">3rem</span>
        </div>
        <div class="flex items-center gap-md">
          <code class="bg-primary-100 dark:bg-primary-900 text-primary-800 dark:text-primary-200 px-sm py-xs rounded">3xl</code>
          <div class="bg-accent h-3xl"></div>
          <span class="text-slate-600 dark:text-slate-300">4rem</span>
        </div>
      </div>
    </section>

  </div>

  <script>
    const toggle = document.getElementById('theme-toggle')
    const html = document.documentElement
    
    // Initialize from localStorage
    if (localStorage.getItem('theme') === 'dark') {
      html.classList.add('dark')
    }
    
    // Toggle dark mode
    toggle.addEventListener('click', () => {
      html.classList.toggle('dark')
      const isDark = html.classList.contains('dark')
      localStorage.setItem('theme', isDark ? 'dark' : 'light')
      toggle.textContent = isDark ? 'Toggle Light Mode' : 'Toggle Dark Mode'
    })
    
    // Set initial button text
    toggle.textContent = html.classList.contains('dark') ? 'Toggle Light Mode' : 'Toggle Dark Mode'
  </script>

</body>
</html>
```

---

**package.json**
```json
{
  "name": "tailwind-v4-custom-design",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "devDependencies": {
    "@tailwindcss/vite": "^4.0.0",
    "tailwindcss": "^4.0.0",
    "vite": "^6.0.0"
  }
}
```

---

**vite.config.js**
```javascript
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [tailwindcss()]
})
```

---

This complete Tailwind CSS v4 project includes:

1. **CSS-based configuration** in `styles/app.css` with:
   - Custom primary color with 50-900 shades
   - Secondary and accent colors
   - Custom spacing scale (xs through 3xl)
   - Custom breakpoints (tablet: 48rem, desktop: 64rem, wide: 90rem)

2. **Dark mode support** using the class strategy with `@custom-variant dark (&:is(.dark *))` and inverted primary color shades for dark mode

3. **Sample card component** in `index.html` demonstrating:
   - Custom color usage (primary shades, secondary, accent)
   - Custom spacing tokens (xs, sm, md, lg, xl, 2xl, 3xl)
   - Responsive breakpoints (tablet, desktop)
   - Dark mode variants with `dark:` prefix
   - JavaScript toggle for dark mode with localStorage persistence

All files follow Tailwind CSS v4 conventions using `@import "tailwindcss"`, `@theme` directive for configuration, and proper CSS variable definitions.