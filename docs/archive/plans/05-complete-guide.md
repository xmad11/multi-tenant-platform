# Complete Multi-Tenant & Multi-Project Platform Guide
## From Vercel Agent - Official Best Practices

> **Source**: Vercel Agent Conversation
> **Date**: 2025
> **Purpose**: Complete reference for building secure multi-tenant and multi-project platforms on Vercel

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Official Templates & Resources](#2-official-templates--resources)
3. [Security Framework - 3 Stages](#3-security-framework---3-stages)
4. [Modular Security SSOT Approach](#4-modular-security-ssot-approach)
5. [Implementation Code Modules](#5-implementation-code-modules)
6. [Setup Instructions](#6-setup-instructions)
7. [Migration Path: Multi-Tenant to Multi-Project](#7-migration-path-multi-tenant-to-multi-project)
8. [Recommended UI Templates Structure](#8-recommended-ui-templates-structure)

---

## 1. Architecture Overview

### Multi-Tenant Architecture
- **Definition**: Single codebase serving multiple tenants from one application
- **Routing**: Tenant identification via subdomain, path, or custom domain
- **Data Isolation**: Tenant-aware data separation with tenant IDs
- **Use Case**: Best when tenants share application structure with minor customizations

### Multi-Project Architecture
- **Definition**: Each tenant has their own isolated Vercel project
- **Isolation**: Separate Git repository, environment variables, deployments per project
- **Use Case**: Best when tenants require unique applications, heavy customization, or isolated deployments

### Recommended Approach
```
Start Simple → Scale When Ready
Multi-Tenant (single project with subdomains) → Multi-Project (isolated per tenant)
```

**Key Insight**: You can start with multi-project architecture using only one project initially, then scale by adding more projects per tenant.

---

## 2. Official Templates & Resources

### Official Vercel Templates

| Resource | URL | Description |
|----------|-----|-------------|
| **Multi-Tenant Starter Kit** | https://vercel.com/templates/saas/platforms-starter-kit | Official SaaS platform with tenant routing, auth, domain handling |
| **GitHub Repo** | https://github.com/vercel/nextjs-platform/tree/main/examples/multi-tenant | Source code for multi-tenant template |
| **OSS AI Vibe Coding** | https://vercel.com/docs/examples/oss-coding-agent | AI-powered coding environment with sandboxes |

### Platform Elements (Security Blocks)

| Block | URL | Purpose |
|-------|-----|---------|
| **Custom Domain** | https://vercel.com/docs/platform-elements/blocks/custom-domain | DNS verification, SSL provisioning, domain validation |
| **Report Abuse** | https://vercel.com/docs/platform-elements/blocks/report-abuse | Content moderation UI with privacy-focused design |
| **Deploy Files** | https://vercel.com/docs/platform-elements/actions/deploy-files | Programmatic deployment with SSO protection |

### Documentation URLs

| Topic | URL |
|-------|-----|
| Multi-Tenant Concepts | https://vercel.com/docs/multi-tenant-platforms/concepts |
| Multi-Project Concepts | https://vercel.com/docs/multi-project-platforms/concepts |
| Middleware & Routing | https://vercel.com/docs/multi-tenant-platforms/middleware-and-routing |
| Configuring Domains | https://vercel.com/docs/multi-tenant-platforms/configuring-domains |
| Vercel SDK Reference | https://vercel.com/docs/multi-project-platforms/reference#create-project |

---

## 3. Security Framework - 3 Stages

### Stage 1: Default Minimum (Non-Negotiable)
**Required for ALL Next.js projects**

- [x] HTTPS (default on Vercel)
- [x] Secure authentication (JWT/OAuth with refresh tokens, secure cookie flags)
- [x] Input validation and sanitization (XSS, SQL injection prevention)
- [x] Secure environment variables (Vercel secrets management)
- [x] Content Security Policy (CSP) headers
- [x] CORS policies (limit allowed origins)
- [x] HTTP security headers:
  - HSTS (Strict-Transport-Security)
  - X-Content-Type-Options
  - X-Frame-Options
  - Referrer-Policy
- [x] Strong password policies
- [x] Regular dependency updates and security audits
- [x] Serverless functions with minimal privilege access
- [x] Rate limiting on endpoints
- [x] Helmet middleware for common HTTP security headers

### Stage 2: Advanced Best Practices (10/10 Security)
**Enhanced protections for production applications**

- [x] Multi-factor authentication (MFA)
- [x] Encryption at rest, in transit, and in backups
- [x] Audit logging for security-relevant events
- [x] Role-based access control (RBAC) or Attribute-based (ABAC)
- [x] Strict tenant data isolation per request
- [x] Secure token storage and rotation mechanisms
- [x] Hardened middleware for tenant validation
- [x] Automated security tests in CI/CD pipelines
- [x] Security monitoring and alerting integration
- [x] Comprehensive error handling (no sensitive info exposure)
- [x] Serverless function cold start protections
- [x] Abuse Reporting UI components
- [x] Regular security training and threat modeling

### Stage 3: Beyond Best Practices
**Advanced tools for multi-tenant & multi-project platforms**

- [x] Vercel isolated builds and deployments per tenant
- [x] Automated tenant project creation/teardown with Vercel SDK
- [x] Edge Middleware for high-performance tenant routing
- [x] Programmatic domain verification via Vercel API
- [x] Secure sandboxing environments (Vercel Sandboxes)
- [x] AI-based threat detection for user activity monitoring
- [x] Blockchain/cryptographic attestations for critical data
- [x] Per-tenant rate limits and anomaly detection
- [x] Third-party security suites (vulnerability scanning, pen testing)
- [x] Zero-trust networking principles
- [x] Per-tenant secrets management with rotation and audit logs
- [x] AI-powered continuous compliance verification
- [x] Advanced content moderation models
- [x] Routine penetration testing and red teaming

---

## 4. Modular Security SSOT Approach

### Single Source of Truth (SSOT) Strategy

Create centralized, reusable security modules that can be shared across all projects:

```
security-ssot/
├── packages/
│   ├── middleware/          # Shared middleware
│   │   ├── tenantResolver.ts
│   │   ├── auth.ts
│   │   └── securityHeaders.ts
│   ├── components/          # Shared UI components
│   │   ├── AbuseReport.tsx
│   │   └── CustomDomain.tsx
│   └── utils/              # Shared utilities
│       ├── validation.ts
│       └── encryption.ts
├── ci-cd/                  # Shared CI/CD pipelines
└── docs/                   # Security playbook
```

### Best Practices for Modularity

1. **Create centralized security middleware and utility libraries**
   - Develop shared npm packages or private repos
   - Publish packages for reuse across all projects

2. **Use Vercel Edge Middleware as shared enforcement layer**
   - Enforce security policies at the edge
   - Reusable across apps/projects

3. **Centralize secrets and config management**
   - Use Vercel's environment secrets
   - Inject securely across projects

4. **Use Infrastructure as Code (IaC) and GitOps**
   - Configure security rules as code
   - Version control in central repository

5. **Build standard CI/CD pipelines**
   - Reusable pipeline scripts
   - Security checks, tests, deployment validations

6. **Document security components**
   - Maintain security playbook
   - Clear integration instructions

7. **Scheduled updates and audits**
   - Automate dependency updates
   - Central vulnerability scans
   - Propagate fixes to downstream projects

---

## 5. Implementation Code Modules

### Folder Structure

```
/middleware
  tenantResolver.ts    # Tenant identification from hostname
  auth.ts              # Authentication middleware
  securityHeaders.ts   # HTTP security headers
/lib
  abuseReport.tsx      # Abuse reporting component
/pages
  api/
    deploy.ts          # Vercel SDK deployment automation
  admin/
    dashboard.tsx      # Admin dashboard
  _middleware.ts       # Edge middleware entry point
  index.tsx            # Default AI Coder UI
  ecommerce.tsx        # E-commerce placeholder
  restaurant.tsx       # Restaurant placeholder
  booking.tsx          # Booking/search placeholder
  saas.tsx             # SaaS AI marketing placeholder
/vercel.json
/tsconfig.json
/package.json
/tailwind.config.js
/biome.toml
```

### Module 1: Tenant Resolver Middleware

**File**: `middleware/tenantResolver.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server';

export async function tenantResolver(req: NextRequest) {
  const hostname = req.headers.get('host') || '';
  const tenant = extractTenantFromHostname(hostname);

  // Validate tenant in your DB or config here
  if (!tenant || !(await tenantExists(tenant))) {
    return NextResponse.redirect('/404');
  }

  // Add tenant info to request headers for downstream usage
  const res = NextResponse.next();
  res.headers.set('x-tenant-id', tenant);
  return res;
}

function extractTenantFromHostname(hostname: string): string | null {
  // Simple regex to get subdomain as tenant name
  const match = hostname.match(/^([a-z0-9-]+)\./);
  return match ? match[1] : null;
}

async function tenantExists(tenant: string): Promise<boolean> {
  // TODO: Check the tenant in your DB or config
  const allowedTenants = ['tenant1', 'tenant2'];
  return allowedTenants.includes(tenant);
}
```

### Module 2: Authentication Middleware

**File**: `middleware/auth.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server';

export function authMiddleware(req: NextRequest) {
  const token = req.cookies.get('token');
  if (!token || !validateToken(token)) {
    return NextResponse.redirect('/login');
  }
  return NextResponse.next();
}

function validateToken(token: string): boolean {
  // TODO: implement JWT or session validation
  return Boolean(token);
}
```

### Module 3: Security Headers Middleware

**File**: `middleware/securityHeaders.ts`

```typescript
import { NextResponse } from 'next/server';

export function securityHeadersMiddleware() {
  const res = NextResponse.next();
  res.headers.set('Strict-Transport-Security', 'max-age=63072000; includeSubDomains; preload');
  res.headers.set('X-Content-Type-Options', 'nosniff');
  res.headers.set('X-Frame-Options', 'DENY');
  res.headers.set('Referrer-Policy', 'no-referrer');
  res.headers.set('Content-Security-Policy', "default-src 'self'; script-src 'none'; object-src 'none';");
  return res;
}
```

### Module 4: Abuse Reporting Component

**File**: `lib/abuseReport.tsx`

```typescript
'use client';

import { useState } from 'react';

interface AbuseReportProps {
  onSubmit: (data: { category: string; description: string; email: string }) => void;
}

export function AbuseReport({ onSubmit }: AbuseReportProps) {
  const [category, setCategory] = useState('');
  const [description, setDescription] = useState('');
  const [email, setEmail] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (category && description && email) {
      onSubmit({ category, description, email });
    }
  };

  return (
    <form onSubmit={handleSubmit} className="p-4 border rounded shadow-md">
      <label className="block mb-2">
        Category
        <select
          value={category}
          onChange={e => setCategory(e.target.value)}
          required
          className="w-full mt-1 p-2 border rounded"
        >
          <option value="">Select</option>
          <option value="spam">Spam</option>
          <option value="abuse">Abuse</option>
          <option value="other">Other</option>
        </select>
      </label>
      <label className="block mb-2">
        Description
        <textarea
          value={description}
          onChange={e => setDescription(e.target.value)}
          required
          className="w-full mt-1 p-2 border rounded"
        />
      </label>
      <label className="block mb-2">
        Email (optional for follow-up)
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
          className="w-full mt-1 p-2 border rounded"
        />
      </label>
      <button
        type="submit"
        className="mt-2 px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600"
      >
        Report Abuse
      </button>
    </form>
  );
}
```

### Module 5: Combined Edge Middleware

**File**: `pages/_middleware.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { tenantResolver } from '../middleware/tenantResolver';
import { authMiddleware } from '../middleware/auth';
import { securityHeadersMiddleware } from '../middleware/securityHeaders';

export async function middleware(req: NextRequest) {
  let res;

  // 1. Tenant resolution (first priority)
  res = await tenantResolver(req);
  if (res.headers.get('location')) return res; // Redirect or error from tenant resolver

  // 2. Authentication (skip for public routes)
  const isPublicRoute = req.nextUrl.pathname.startsWith('/login') ||
                        req.nextUrl.pathname.startsWith('/api/public');
  if (!isPublicRoute) {
    res = authMiddleware(req);
    if (res.headers.get('location')) return res;
  }

  // 3. Security headers (always apply)
  res = securityHeadersMiddleware();
  return res;
}
```

### Module 6: Admin Dashboard

**File**: `pages/admin/dashboard.tsx`

```typescript
import React from 'react';

// Sample tenant data - replace with real data from your database
const sampleTenants = [
  { id: 'tenant1', name: 'Tenant 1', activeUsers: 120, projects: 5, status: 'active' },
  { id: 'tenant2', name: 'Tenant 2', activeUsers: 80, projects: 3, status: 'active' },
  { id: 'tenant3', name: 'Tenant 3', activeUsers: 45, projects: 2, status: 'inactive' },
];

export default function AdminDashboard() {
  return (
    <div className="p-6 bg-gray-100 min-h-screen">
      <h1 className="text-3xl font-bold mb-6">Admin Dashboard</h1>

      {/* Summary Cards */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
        <div className="bg-white p-4 rounded shadow">
          <h3 className="text-gray-500 text-sm">Total Tenants</h3>
          <p className="text-2xl font-bold">{sampleTenants.length}</p>
        </div>
        <div className="bg-white p-4 rounded shadow">
          <h3 className="text-gray-500 text-sm">Active Users</h3>
          <p className="text-2xl font-bold">
            {sampleTenants.reduce((sum, t) => sum + t.activeUsers, 0)}
          </p>
        </div>
        <div className="bg-white p-4 rounded shadow">
          <h3 className="text-gray-500 text-sm">Total Projects</h3>
          <p className="text-2xl font-bold">
            {sampleTenants.reduce((sum, t) => sum + t.projects, 0)}
          </p>
        </div>
      </div>

      {/* Tenants Table */}
      <div className="bg-white shadow rounded overflow-hidden">
        <table className="w-full">
          <thead className="bg-gray-200">
            <tr>
              <th className="p-4 text-left">Tenant ID</th>
              <th className="p-4 text-left">Name</th>
              <th className="p-4 text-left">Active Users</th>
              <th className="p-4 text-left">Projects</th>
              <th className="p-4 text-left">Status</th>
              <th className="p-4 text-left">Actions</th>
            </tr>
          </thead>
          <tbody>
            {sampleTenants.map((tenant) => (
              <tr key={tenant.id} className="border-t hover:bg-gray-50">
                <td className="p-4">{tenant.id}</td>
                <td className="p-4">{tenant.name}</td>
                <td className="p-4">{tenant.activeUsers}</td>
                <td className="p-4">{tenant.projects}</td>
                <td className="p-4">
                  <span className={`px-2 py-1 rounded text-xs ${
                    tenant.status === 'active' ? 'bg-green-100 text-green-800' : 'bg-gray-100 text-gray-800'
                  }`}>
                    {tenant.status}
                  </span>
                </td>
                <td className="p-4">
                  <button className="text-blue-600 hover:underline mr-2">Edit</button>
                  <button className="text-red-600 hover:underline">Delete</button>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

### Module 7: Default AI Coder Template

**File**: `pages/index.tsx`

```typescript
export default function AICoder() {
  return (
    <main className="min-h-screen bg-gradient-to-br from-gray-900 to-gray-800 text-white">
      <div className="container mx-auto px-4 py-12">
        <h1 className="text-5xl font-extrabold mb-6 bg-clip-text text-transparent bg-gradient-to-r from-blue-400 to-purple-500">
          AI Coder
        </h1>
        <p className="text-xl text-gray-300 mb-8 max-w-2xl">
          Welcome to your AI coding environment! Describe the vibe of what you want to build,
          and we'll generate the code for you.
        </p>

        {/* Input Section */}
        <div className="bg-gray-800 rounded-lg p-6 shadow-xl">
          <textarea
            placeholder="Describe what you want to build..."
            className="w-full h-32 bg-gray-700 text-white rounded p-4 border border-gray-600 focus:border-blue-500 focus:outline-none resize-none"
          />

          {/* Action Buttons */}
          <div className="flex gap-4 mt-4">
            <button className="px-6 py-3 bg-blue-600 hover:bg-blue-700 rounded-lg font-semibold transition">
              Generate Code
            </button>
            <button className="px-6 py-3 bg-gray-600 hover:bg-gray-700 rounded-lg font-semibold transition flex items-center gap-2">
              <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4m-4-8a3 3 0 01-3-3V5a3 3 0 116 0v6a3 3 0 01-3 3z" />
              </svg>
              Voice Input
            </button>
            <button className="px-6 py-3 bg-gray-600 hover:bg-gray-700 rounded-lg font-semibold transition flex items-center gap-2">
              <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12" />
              </svg>
              Upload Files
            </button>
          </div>
        </div>
      </div>
    </main>
  );
}
```

### Configuration Files

**biome.toml**
```toml
[settings]
extends = ["recommended", "typescript"]
lint = true
format = true

[lint.rules]
style.useTemplate = "error"
style.noParameterAssign = "warn"
```

**tsconfig.json**
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "jsx": "preserve",
    "module": "esnext",
    "target": "es2020",
    "moduleResolution": "node",
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

**tailwind.config.js**
```javascript
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
    './lib/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          200: '#bae6fd',
          300: '#7dd3fc',
          400: '#38bdf8',
          500: '#0ea5e9',
          600: '#0284c7',
          700: '#0369a1',
          800: '#075985',
          900: '#0c4a6e',
        },
      },
    },
  },
  plugins: [],
};
```

---

## 6. Setup Instructions

### Step 1: Documentation Study (Agent Prerequisite)

Before starting, read these Vercel documentation pages:

```
1. https://vercel.com/docs/multi-tenant-platforms/concepts
2. https://vercel.com/docs/multi-project-platforms/concepts
3. https://vercel.com/docs/multi-tenant-platforms/middleware-and-routing
4. https://vercel.com/docs/platform-elements/blocks/custom-domain
5. https://vercel.com/docs/platform-elements/blocks/report-abuse
6. https://vercel.com/docs/platform-elements/actions/deploy-files
7. https://vercel.com/docs/multi-project-platforms/reference#create-project
```

### Step 2: Clone and Setup

```bash
# Clone the official multi-tenant template
git clone https://github.com/vercel/nextjs-platform.git my-multitenant-app
cd my-multitenant-app
cd examples/multi-tenant

# Install dependencies using Bun (not npm/yarn)
bun install

# Add Biome for linting and formatting
bun add -D @biomejs/cli
npx biome init

# Upgrade Tailwind CSS to latest version
bun add tailwindcss@latest postcss@latest autoprefixer@latest

# Run development server
bun dev
```

### Step 3: Configure TypeScript Strict Mode

Edit `tsconfig.json` to add:
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

### Step 4: Implement Security Modules

1. Create the modular middleware files as shown in Section 5
2. Add them to your project's `/middleware` directory
3. Update `pages/_middleware.ts` to combine all middleware
4. Test locally with `bun dev`

### Step 5: Implement Security Phases

**Phase 1 - Stage 1 (Minimum):**
- Enable all security headers
- Implement basic auth
- Add input validation
- Configure CSP and CORS

**Phase 2 - Stage 2 (Advanced):**
- Add MFA support
- Implement RBAC
- Add audit logging
- Configure rate limiting

**Phase 3 - Stage 3 (Beyond):**
- Integrate Vercel SDK for project automation
- Add per-tenant secrets management
- Implement advanced threat detection
- Set up automated security scanning

### Step 6: Deploy to Vercel

```bash
# Install Vercel CLI (if not installed)
bun i -g vercel

# Deploy
vercel
```

### Step 7: Configure Environment Variables

Set these in Vercel dashboard:
```
DATABASE_URL=your_database_url
AUTH_SECRET=your_auth_secret
VERCEL_TEAM_ID=your_team_id
```

---

## 7. Migration Path: Multi-Tenant to Multi-Project

### Preparation Steps (Do These Early)

1. **Design for Isolation**
   - Keep tenant data isolated from the start
   - Use tenant IDs in all database queries
   - Avoid hardcoded tenant references

2. **Modular Architecture**
   - Build reusable middleware and utilities
   - Create npm packages for shared logic
   - Document all tenant-specific configurations

3. **Standardize Interfaces**
   - Use universal interfaces for tenant config
   - Create consistent routing patterns
   - Standardize authentication flows

### Migration Process

When ready to move from multi-tenant to multi-project:

```
1. Identify tenant to migrate
2. Create new isolated Vercel project via Vercel SDK
3. Copy tenant-specific code and configurations
4. Migrate tenant data to isolated database
5. Update DNS/routing to point to new project
6. Verify all functionality
7. Decommission old tenant routing
```

### Vercel SDK Usage

```typescript
import { Vercel } from '@vercel/sdk';

const vercel = new Vercel({
  token: process.env.VERCEL_TOKEN,
  teamId: process.env.VERCEL_TEAM_ID,
});

// Create new project for tenant
const project = await vercel.projects.create({
  name: `tenant-${tenantId}`,
  framework: 'nextjs',
});

// Add custom domain
await vercel.projects.addDomain({
  projectId: project.id,
  domain: `${tenantId}.yourdomain.com`,
});

// Deploy with environment variables
await vercel.deployments.create({
  projectId: project.id,
  env: {
    TENANT_ID: tenantId,
    DATABASE_URL: tenantDbUrl,
  },
});
```

---

## 8. Recommended UI Templates Structure

### Tenant UI Templates (Subdomain Routes)

| Subdomain | Route | Purpose |
|-----------|-------|---------|
| `*.yourdomain.com` | `/pages/index.tsx` | Default: AI Coder Template |
| `shop.*.yourdomain.com` | `/pages/ecommerce.tsx` | E-commerce UI |
| `food.*.yourdomain.com` | `/pages/restaurant.tsx` | Restaurant/Supermarket UI |
| `search.*.yourdomain.com` | `/pages/booking.tsx` | Search + Booking (cars, yachts, tours) |
| `business.*.yourdomain.com` | `/pages/saas.tsx` | SaaS AI Marketing & Automation |

### Dashboard UI (Admin)

| Route | Purpose |
|-------|---------|
| `/pages/admin/dashboard.tsx` | Main admin dashboard with tenant/project management |
| `/pages/admin/tenants.tsx` | Tenant CRUD operations |
| `/pages/admin/analytics.tsx` | Platform analytics and monitoring |

### Template Implementation Pattern

Each template should include:
1. **Beautiful futuristic UI** using Tailwind CSS
2. **Placeholder components** for future functionality
3. **Responsive design** (mobile-first)
4. **Dark mode support**
5. **Accessibility features** (ARIA, keyboard navigation)

### Example: E-commerce Template Placeholder

**File**: `pages/ecommerce.tsx`

```typescript
export default function EcommerceTemplate() {
  return (
    <main className="min-h-screen bg-gray-50">
      {/* Header */}
      <header className="bg-white shadow-sm">
        <div className="container mx-auto px-4 py-4 flex justify-between items-center">
          <h1 className="text-2xl font-bold text-gray-900">Store</h1>
          <nav className="flex gap-6">
            <a href="#" className="text-gray-600 hover:text-gray-900">Products</a>
            <a href="#" className="text-gray-600 hover:text-gray-900">Categories</a>
            <a href="#" className="text-gray-600 hover:text-gray-900">Cart</a>
          </nav>
        </div>
      </header>

      {/* Hero Section */}
      <section className="bg-gradient-to-r from-blue-600 to-purple-600 text-white py-20">
        <div className="container mx-auto px-4 text-center">
          <h2 className="text-5xl font-bold mb-4">Shop the Future</h2>
          <p className="text-xl mb-8">Discover amazing products</p>
          <button className="bg-white text-blue-600 px-8 py-3 rounded-full font-semibold hover:bg-gray-100">
            Browse Products
          </button>
        </div>
      </section>

      {/* Product Grid Placeholder */}
      <section className="container mx-auto px-4 py-12">
        <h3 className="text-2xl font-bold mb-6">Featured Products</h3>
        <div className="grid grid-cols-1 md:grid-cols-3 lg:grid-cols-4 gap-6">
          {[1, 2, 3, 4, 5, 6, 7, 8].map((i) => (
            <div key={i} className="bg-white rounded-lg shadow-md overflow-hidden">
              <div className="h-48 bg-gray-200"></div>
              <div className="p-4">
                <h4 className="font-semibold">Product {i}</h4>
                <p className="text-gray-600">$99.99</p>
              </div>
            </div>
          ))}
        </div>
      </section>
    </main>
  );
}
```

---

## Summary Checklist

### Pre-Development
- [ ] Read all Vercel documentation links
- [ ] Clone multi-tenant starter template
- [ ] Setup Bun, Biome, strict TypeScript, Tailwind
- [ ] Create modular folder structure

### Stage 1 Security Implementation
- [ ] Implement all middleware modules
- [ ] Configure security headers
- [ ] Set up authentication
- [ ] Add input validation
- [ ] Configure CSP and CORS

### Stage 2 Security Implementation
- [ ] Add MFA support
- [ ] Implement RBAC
- [ ] Add audit logging
- [ ] Configure rate limiting
- [ ] Add abuse reporting UI

### Stage 3 Security Implementation
- [ ] Integrate Vercel SDK
- [ ] Set up per-tenant secrets
- [ ] Add advanced threat detection
- [ ] Configure automated security scanning
- [ ] Implement zero-trust networking

### UI Templates
- [ ] Build admin dashboard
- [ ] Create AI Coder default template
- [ ] Add e-commerce placeholder
- [ ] Add restaurant placeholder
- [ ] Add booking/search placeholder
- [ ] Add SaaS AI marketing placeholder

### Deployment
- [ ] Configure environment variables
- [ ] Deploy to Vercel
- [ ] Set up custom domains
- [ ] Configure monitoring
- [ ] Document all processes

---

## Additional Resources

### Learning Materials
- [Vercel Learn](https://vercel.com/learn)
- [Next.js Documentation](https://nextjs.org/docs)
- [Vercel SDK GitHub](https://github.com/vercel/vercel)

### Community
- [Vercel Discord](https://vercel.com/discord)
- [Vercel GitHub Discussions](https://github.com/vercel/vercel/discussions)

### Security Resources
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Web Security Guidelines](https://web.dev/secure/)
- [Next.js Security](https://nextjs.org/docs/app/building-your-application/configuring/security)

---

*This document compiles official Vercel best practices and recommendations for building secure, scalable multi-tenant and multi-project platforms.*
