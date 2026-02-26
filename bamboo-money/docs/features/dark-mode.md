# Dark Mode

Toggle between light and dark themes with one click. Your preference persists across sessions.

## How It Works

Dark mode uses **Bootstrap 5's native dark theme** via the `data-bs-theme` attribute on `<html>`.

### Toggle Button

A moon/sun icon button in the sidebar footer:

```html
<button onclick="toggleTheme()" class="btn btn-sm">
    <i class="bi bi-moon-fill"></i>
</button>
```

### JavaScript

```javascript
// Applied in <head> before CSS loads — prevents flash of wrong theme
(function() {
    const theme = localStorage.getItem('theme') || 'light';
    document.documentElement.setAttribute('data-bs-theme', theme);
})();

function toggleTheme() {
    const current = document.documentElement.getAttribute('data-bs-theme');
    const next = current === 'dark' ? 'light' : 'dark';
    document.documentElement.setAttribute('data-bs-theme', next);
    localStorage.setItem('theme', next);
    // Update icon
    document.querySelector('.theme-icon').className =
        next === 'dark' ? 'bi bi-sun-fill theme-icon' : 'bi bi-moon-fill theme-icon';
}
```

### Persistence

Theme preference is stored in `localStorage.theme` — no server round-trip needed. The theme is read and applied **before the page renders** (script in `<head>`), preventing the "flash of light theme" problem.

## Custom Overrides

Bootstrap 5's dark mode handles most components automatically. Custom CSS overrides handle edge cases:

```css
[data-bs-theme="dark"] .table-light {
    background: #1a1a2e !important;
}
[data-bs-theme="dark"] .table-light th {
    color: #adb5bd !important;
}
```

### Chart.js Adaptation

Chart.js doesn't automatically respond to Bootstrap's dark mode. The dashboard JavaScript detects the theme and adjusts:

- Grid line colors
- Text/label colors
- Tooltip backgrounds
- Legend text colors

## Use Case

> *It's 11pm and Jose is checking his budget. He clicks the moon icon in the sidebar — everything flips to dark theme instantly. Next time he opens the app, it remembers.*

## Future Enhancements

- **System preference detection** — auto-follow OS dark mode via `prefers-color-scheme`
- **Per-user server storage** — sync preference across devices via the user profile model
