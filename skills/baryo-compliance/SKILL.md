---
name: baryo-compliance
description: Manages BaryoDev's legal sanitization, licensing (MPL-2.0/MIT), and git history scrubbing policies.
---

# BaryoDev Compliance & Legal

You are the Legal Guardian of the codebase. Your job is to ensure clean IP rights and consistent licensing.

## Licensing Strategy

### 1. Libraries & Frameworks (The Product)
-   **License**: **MPL-2.0** (Mozilla Public License 2.0).
-   **Reason**: Allows use in proprietary software (file-level copyleft), but modifications to *our* files must remain open. Stronger than MIT, looser than GPL.

### 2. Demos, Samples, Examples, Websites
-   **License**: **MIT**.
-   **Reason**: We want users to copy-paste this code freely without worry.

## File Headers
All source files (`.cs`, `.ts`, `.js`, etc.) MUST start with the standard header.

**Format**:
```csharp
// Copyright (c) BaryoDev. All rights reserved.
// Licensed under the [License Name] license. See LICENSE file in the project root for full license information.
```

## Git History Cleanliness (Legal Sanitization)

When converting or initializing a repo, specific steps must be taken to remove "ghosts" of the past.

### Sanitization via `git-filter-repo`
If sensitive data or old licenses exist in history, use this workflow:

1.  **Analyze**: `git filter-repo --analyze`
2.  **Scrub Author**:
    ```bash
    git filter-repo --email-callback 'return email.replace(b"old-email@example.com", b"opensource@baryodev.com")'
    ```
3.  **Remvove Paths**:
    ```bash
    git filter-repo --path-glob '*.secret' --invert-paths
    ```

## Checklist Before Commit
1.  [ ] Are all secrets (`appsettings.development.json`, `.env`) in `.gitignore`?
2.  [ ] Do all new files have the Header?
3.  [ ] Is `package.json` / `.csproj` Author set to `BaryoDev`?
