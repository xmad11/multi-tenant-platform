# Multi-Tenant Platform vs shadi-V2: Configuration Reuse Guide

> **Guardian Reference**: This document compares the Vercel multi-tenant template with shadi-V2 project configurations to identify best practices to leverage.

---

## Executive Summary

| Area | Vercel Template | shadi-V2 | Recommendation |
|------|----------------|----------|----------------|
| **Runtime** | npm/node | Bun | **Use Bun (shadi-V2)** |
| **Linting** | ESLint | Biome | **Use Biome (shadi-V2)** |
| **TypeScript** | Basic | Strict + custom rules | **Use shadi-V2 config** |
| **Tailwind** | 3.x | 4.1.18 + Design Tokens | **Use Tailwind 4 + Design Tokens** |
| **CI/CD** | None | 11-layer audit + security pipeline | **Use shadi-V2 CI/CD** |
| **Git Hooks** | None | Husky + lint-staged | **Use shadi-V2 hooks** |
| **Vercel Config** | Basic | Optimized Bun config | **Use shadi-V2 config** |
| **Security** | Basic | Advanced (CSP, headers, SAST) | **Use shadi-V2 security** |

---

## 1. Package Configuration

### shadi-V2 (SUPERIOR)

```json
{
  "scripts": {
    "dev": "bun scripts/env-guard.ts && bun run scripts/phase1-lock.ts",
    "build": "bun next build",
    "build:with-audit": "bun run audit:ci && bun next build",
    "format": "biome format . --write",
    "lint": "biome lint .",
    "check": "biome check .",
    "check:ts": "tsc --noEmit",
    "audit:ci": "bun run scripts/audit-runner.ts && bun run scripts/track-history.ts",
    "typecheck": "tsc --noEmit --incremental false"
  },
  "lint-staged": {
    "*.{ts,tsx}": ["biome check --write", "biome format --write"]
  }
}
```

**Why superior**:
- Bun runtime everywhere
- Separated audit from build
- Comprehensive scripts for all workflows
- Pre-commit quality gates

### Vercel Template (BASELINE)

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  }
}
```

**Decision**: **Use shadi-V2 scripts** as the foundation, add multi-tenant specific scripts.

---

## 2. TypeScript Configuration

### shadi-V2 (SUPERIOR)

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "strict": true,
    "jsx": "react-jsx",
    "skipLibCheck": true,
    "allowJs": true,
    "incremental": true,
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "lib": ["dom", "dom.iterable", "esnext"],
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

**Why superior**:
- ESNext target for latest features
- Bundler module resolution (faster)
- Incremental compilation
- Path aliases for clean imports

### Vercel Template

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

**Decision**: **Use shadi-V2 config** (ESNext target is better).

---

## 3. Biome Configuration

### shadi-V2 (SUPERIOR)

```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.4/schema.json",
  "files": {
    "ignore": [
      ".next/**", "dist/**", "build/**", "node_modules/**",
      ".turbo/**", "coverage/**", "*.lock", "*.log",
      "next-env.d.ts", "styles/tokens*.css", "styles/themes.css"
    ]
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "suspicious": {
        "noExplicitAny": "off",
        "noConsoleLog": "off",
        "noArrayIndexKey": "warn"
      },
      "style": {
        "useConst": "error",
        "useFragmentSyntax": "error"
      }
    }
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "double",
      "jsxQuoteStyle": "double",
      "trailingCommas": "es5",
      "semicolons": "asNeeded"
    }
  }
}
```

**Why superior**:
- Token file protection (prevents accidental edits)
- Console logs allowed with warnings
- Array index keys warned (not error)
- Organized imports automatically

**Decision**: **Use shadi-V2 Biome config**, add multi-tenant specific ignores.

---

## 4. Vercel Configuration

### shadi-V2 (SUPERIOR)

```json
{
  "buildCommand": "bun run build",
  "installCommand": "bun install",
  "framework": "nextjs",
  "outputDirectory": ".next"
}
```

**Why superior**:
- Bun runtime specified
- Clean build command
- Optimized for Vercel + Bun

**Decision**: **Use shadi-V2 config** exactly.

---

## 5. CI/CD Configuration

### shadi-V2 (ADVANCED - 2 Workflows)

**Workflow 1: Audit & Build**

```yaml
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  audit:
    - Setup Bun
    - Cache dependencies
    - Run 11-layer audit
    - Upload audit reports
    - Comment PR with results
    - Fail on critical violations

  build:
    - Build verification
    - Upload build artifacts
```

**Workflow 2: Security Pipeline**

```yaml
on:
  push:
    branches: [main, develop]
  schedule:
    - cron: '0 0 * * 0'  # Weekly

jobs:
  code-quality:
    - TypeScript type checking
    - Biome lint & format
    - Console statement check
    - 'any' type check
    - Secret detection (Gitleaks)

  security-testing:
    - Environment audit
    - CSP validation
    - Security headers check
    - Dangerous pattern detection
    - Dependency vulnerability scan

  build-verify:
    - Build with secrets
    - Upload artifacts
```

### Vercel Template (NONE)

**Decision**: **Use shadi-V2 CI/CD**, add multi-tenant specific checks:
- Tenant isolation validation
- Cross-tenant access prevention
- Tenant data isolation tests

---

## 6. Git Hooks (Pre-commit)

### shadi-V2 (SUPERIOR)

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky-sh.sh"

# Block token file modifications
# Check for pending rollbacks
# Run lint-staged
# Quick audit on staged files
```

**Features**:
- Token file protection
- Rollback detection
- Fast targeted audit (only changed files)
- PR commenting with results

### Vercel Template (NONE)

**Decision**: **Use shadi-V2 hooks**, add tenant-specific validations.

---

## 7. Security Features

### shadi-V2 (ADVANCED)

| Feature | Implementation |
|---------|----------------|
| **CSP Headers** | Configured in middleware |
| **Security Headers** | X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy |
| **Secret Detection** | Gitleaks in CI/CD |
| **SAST** | Console, 'any' type, dangerous patterns |
| **Dependency Scanning** | bun audit |
| **Environment Validation** | env-guard.ts script |

### Vercel Template (BASIC)

**Decision**: **Use shadi-V2 security**, add:
- Tenant-specific CSP rules
- Multi-tenant data isolation checks
- Per-tenant rate limiting

---

## 8. Design Token System

### shadi-V2 (ADVANCED - OKLCH Color Space)

```css
@theme {
  /* Brand Colors */
  --color-brand-primary: oklch(0.65 0.16 45.0);
  --color-brand-secondary: oklch(0.58 0.14 250.0);

  /* Accent Colors (7 options) */
  --color-accent-rust: oklch(0.58 0.15 50.0);
  --color-accent-sage: oklch(0.62 0.08 150.0);
  --color-accent-teal: oklch(0.55 0.12 220.0);
  --color-accent-berry: oklch(0.52 0.18 350.0);
  --color-accent-honey: oklch(0.7 0.12 85.0);

  /* Spacing Scale (4px base) */
  --spacing-xs: 0.25rem;  /* 4px */
  --spacing-sm: 0.5rem;   /* 8px */
  --spacing-md: 1rem;     /* 16px */
  --spacing-lg: 1.5rem;   /* 24px */
  --spacing-xl: 2rem;     /* 32px */

  /* Typography Scale */
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;
  --font-size-3xl: 1.875rem;
  --font-size-5xl: 3rem;
}
```

**Why superior**:
- OKLCH for perceptual uniformity
- Comprehensive token system
- Dark/Warm mode support
- No hardcoded values

### Vercel Template (BASIC Tailwind)

**Decision**: **Adopt shadi-V2 token system**, add:
- `--tenant-primary-*` namespace for tenant branding
- `--tenant-accent-*` for tenant-specific accents
- Maintain base token structure

---

## 9. Middleware Patterns

### shadi-V2 (ADVANCED)

```typescript
// Route protection
const protectedRoutes = ["/dashboard", "/admin", "/owner", "/user"]
const authRoutes = ["/login", "/register", "/forgot-password"]
const publicRoutes = ["/", "/restaurants", "/blog"]

// Supabase auth integration
// Smart redirects
// Cookie handling
```

### Vercel Template (MULTI-TENANT SPECIFIC)

```typescript
// Tenant resolution from subdomain
// Tenant context injection
// Custom domain support
```

**Decision**: **Merge both patterns**:
- Keep shadi-V2 auth/route protection
- Add Vercel template tenant resolution
- Combine for multi-tenant + auth middleware

---

## 10. Environment Variables

### shadi-V2 (SUPERIOR)

```bash
# .env.local.example
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_APP_NAME="Shadi Restaurant Platform"

# Supabase
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=your_supabase_publishable_key
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anonymous_key

# Maps (for "Near Me" and distance features)
NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN=your_mapbox_token_here
```

**Features**:
- Clear naming
- Comprehensive
- Well-documented

### Vercel Template

**Decision**: **Use shadi-V2 template**, add:
- Vercel SDK keys
- Edge Config URL
- Multi-tenant specific variables

---

## 11. Advanced Features (shadi-V2 Only)

| Feature | Description | Reuse for Multi-Tenant? |
|---------|-------------|------------------------|
| **State Machine Architecture** | Coordinator system for task management | ✅ Adapt for tenant provisioning |
| **11-Layer Audit System** | Comprehensive code quality checks | ✅ Add tenant-specific layers |
| **Rollback System** | Failed fix recovery protocol | ✅ Use for deployment rollbacks |
| **Zero-Trust Governance** | Layer enforcement, phase locking | ✅ Adapt for tenant isolation |
| **Design Token Discipline** | No hardcoded values | ✅ Add tenant-specific tokens |
| **Mobile-First Responsive** | Breakpoint system | ✅ Use directly |

---

## Final Configuration Strategy

### 100% shadi-V2 (Adopt As-Is)

```bash
✅ Bun runtime
✅ Biome linting/formatting
✅ TypeScript strict configuration
✅ Vercel configuration
✅ Pre-commit hooks (Husky)
✅ CI/CD workflows
✅ Security headers
✅ Design token system
✅ Package.json scripts
✅ Environment variable template
```

### Merged (Combine Both)

```bash
🔄 Middleware: shadi-V2 auth + Vercel tenant resolution
🔄 Design tokens: shadi-V2 base + tenant namespace
🔄 Scripts: shadi-V2 base + multi-tenant commands
```

### New Multi-Tenant Additions

```bash
🆕 Vercel SDK integration
🆕 Edge Config setup
🆕 Tenant isolation validation
🆕 Custom domain management
🆕 Report Abuse block
🆕 Deploy Files action
```

---

## Implementation Order for Stage 1

1. Clone Vercel multi-tenant template
2. Replace package.json with shadi-V2 structure
3. Copy biome.json from shadi-V2
4. Copy tsconfig.json from shadi-V2
5. Copy vercel.json from shadi-V2
6. Copy .github/workflows from shadi-V2
7. Copy .husky from shadi-V2
8. Copy scripts/env-guard.ts from shadi-V2
9. Create .env.local from shadi-V2 template
10. Adapt design tokens for multi-tenant

---

## Guardian Audit Checklist

When workers complete Stage 1, verify:

- [ ] Bun is runtime (not npm/node)
- [ ] Biome configured and working
- [ ] TypeScript strict mode enabled
- [ ] Tailwind 4.x installed
- [ ] Vercel config uses Bun
- [ ] CI/CD workflows copied
- [ ] Git hooks installed
- [ ] Environment template created
- [ ] Project runs with `bun dev`
- [ ] Zero lint/type errors

---

*This document is the authoritative source for configuration reuse decisions.*
