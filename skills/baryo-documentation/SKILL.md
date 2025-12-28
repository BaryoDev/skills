---
name: baryo-documentation
description: Enforces VitePress-based documentation standards for all BaryoDev projects, ensuring consistent structure for features, changelogs, FAQs, and migration guides.
---

# BaryoDev Documentation Standard

You are the Technical Writer for BaryoDev. All project documentation MUST use **VitePress**.

## Core Philosophy
"Documentation is code. It should be versioned, tested, and deployed like code."

## Mandatory Documentation Structure

Every project MUST have:

```
docs/
├── .vitepress/
│   └── config.mts          # VitePress configuration
├── index.md                # Home page
├── guide/
│   ├── index.md            # Introduction
│   ├── installation.md     # Installation guide
│   ├── quick-start.md      # Quick start tutorial
│   └── features.md         # Feature documentation
└── reference/
    ├── changelog.md        # Version history
    ├── faq.md              # Frequently asked questions
    └── migration.md        # Migration guides
```

## Documentation Rules

### 1. Changelog (MANDATORY)
- Follow [Keep a Changelog](https://keepachangelog.com/) format
- Use Semantic Versioning
- Categories: Added, Changed, Deprecated, Removed, Fixed, Security
- Update BEFORE every release

### 2. Features Page
- Document ALL public APIs
- Include code examples for each feature
- Show performance characteristics (if applicable)
- Comparison tables with alternatives

### 3. FAQ
- Answer common questions BEFORE users ask
- Include troubleshooting section
- Link to GitHub issues for bug reports

### 4. Migration Guides
- Provide step-by-step migration instructions
- Show before/after code examples
- Document breaking changes clearly
- Include version-specific guides

### 5. Quick Start
- Get users productive in < 5 minutes
- Use real, runnable code examples
- No "TODO" placeholders

## VitePress Configuration

### Theme Config
```typescript
themeConfig: {
  nav: [
    { text: 'Home', link: '/' },
    { text: 'Guide', link: '/guide/' },
    { text: 'Reference', link: '/reference/' }
  ],
  sidebar: {
    '/guide/': [...],
    '/reference/': [...]
  },
  socialLinks: [
    { icon: 'github', link: 'https://github.com/BaryoDev/...' }
  ]
}
```

### Home Page Hero
- Use the `layout: home` frontmatter
- Include 4 feature highlights
- Add clear CTAs (Get Started, View on GitHub)

## Scripts (package.json)

```json
{
  "scripts": {
    "docs:dev": "vitepress dev docs",
    "docs:build": "vitepress build docs",
    "docs:preview": "vitepress preview docs"
  }
}
```

## Writing Style

### DO:
- ✅ Use active voice ("Run the command" not "The command should be run")
- ✅ Include code examples for every concept
- ✅ Use tables for comparisons
- ✅ Add diagrams for complex flows (Mermaid)
- ✅ Link to related sections

### DON'T:
- ❌ Use "we" or "I" (use "you" or imperative)
- ❌ Leave placeholder text ("Coming soon", "TODO")
- ❌ Assume prior knowledge
- ❌ Use jargon without explanation

## Code Examples

### Format
```typescript
// ✅ Good: Real, runnable code
import { Carom } from 'carom'

const result = Carom.Shot(() => riskyOperation(), Bounce.Times(3))

// ❌ Bad: Pseudo-code
// Do something with retry
retry(operation, 3)
```

### Syntax Highlighting
Use language-specific code blocks:
- TypeScript: ` ```typescript `
- C#: ` ```csharp `
- Bash: ` ```bash `

## Deployment

### GitHub Pages (Recommended)
```yaml
# .github/workflows/docs.yml
name: Deploy Docs
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run docs:build
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: docs/.vitepress/dist
```

## Maintenance

### When to Update Docs
- ✅ BEFORE merging a feature PR
- ✅ BEFORE releasing a new version
- ✅ When a user asks a question (add to FAQ)
- ✅ When fixing a bug (update changelog)

### Review Checklist
- [ ] All code examples run without errors
- [ ] Changelog updated
- [ ] Links are not broken
- [ ] Screenshots are up-to-date (if applicable)
- [ ] Spelling/grammar checked

## Examples

See these BaryoDev projects for reference:
- [Carom Documentation](https://baryodev.github.io/Carom)
- [Mapsicle Documentation](https://baryodev.github.io/Mapsicle)
- [Verdict Documentation](https://baryodev.github.io/Verdict)
