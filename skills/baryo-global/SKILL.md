---
name: baryo-global
description: Internationalization (i18n) and accessibility (a11y) standards for worldwide reach.
---

# BaryoDev Global Standard

You are the Globalization Expert. **Build for the world, not just your locale.**

## Internationalization (i18n)

### Never Hardcode Text
```typescript
// ❌ Hardcoded
<button>Submit</button>

// ✅ Localized
<button>{t('submit')}</button>
```

### Locale-Aware Formatting
```typescript
// Dates
new Intl.DateTimeFormat(locale).format(date);

// Numbers
new Intl.NumberFormat(locale, { style: 'currency', currency }).format(amount);

// Pluralization
new Intl.PluralRules(locale).select(count);
```

### RTL Support
```css
/* Use logical properties */
margin-inline-start: 1rem; /* Not margin-left */
padding-inline-end: 1rem;  /* Not padding-right */
```

## Accessibility (a11y) - WCAG 2.1

### Semantic HTML
```html
<!-- ❌ Non-semantic -->
<div onclick="submit()">Click me</div>

<!-- ✅ Semantic -->
<button type="submit">Submit form</button>
```

### ARIA Labels
```html
<button aria-label="Close dialog">×</button>
<img src="logo.png" alt="Company Logo">
<input aria-describedby="help-text">
```

### Keyboard Navigation
```typescript
// All interactive elements must be keyboard accessible
element.addEventListener('keydown', (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    handleClick();
  }
});
```

### Color Contrast
- Normal text: 4.5:1 minimum
- Large text: 3:1 minimum
- Never use color alone for information

## Checklist
- [ ] All text externalized to translation files
- [ ] Dates/numbers use Intl formatters
- [ ] RTL layout tested
- [ ] Semantic HTML used
- [ ] All images have alt text
- [ ] Keyboard navigation works
- [ ] Color contrast passes WCAG
- [ ] Screen reader tested
