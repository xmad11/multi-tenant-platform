# Multi-Tenant Platform - Complete Staged Plan
## Guardian-Managed Implementation with Worker Agents

> **My Role**: Guardian/Manager - I audit, review, and coordinate. I do NOT implement directly.
> **Worker Agents**: DeepSeek V3 (complex tasks), Groq (fast tasks), GLM API (debugging)
> **Goal**: 10/10 clean production-ready multi-tenant platform

---

## Project Overview

**Stack**: Next.js 16 (App Router) + Bun + Biome + Strict TypeScript + Tailwind CSS 4.x
**Architecture**: Multi-tenant (start) → Multi-project (future scale)
**Base Template**: Vercel SaaS Platform Starter Kit

**Documentation Sources**:
- https://vercel.com/docs/multi-tenant-platforms/concepts
- https://vercel.com/docs/platform-elements/blocks/report-abuse
- https://vercel.com/docs/platform-elements/blocks/custom-domain
- https://vercel.com/docs/platform-elements/actions/deploy-files
- https://vercel.com/templates/saas/platforms-starter-kit
- https://github.com/vercel/nextjs-platform/tree/main/examples/multi-tenant

---

## Stage 1: Foundation & Setup (2-3 hours)

**Objective**: Clone template, configure stack, verify everything runs

### Tasks

#### 1.1 Clone & Initial Setup
```bash
# Clone official multi-tenant template
git clone https://github.com/vercel/nextjs-platform.git multi-tenant-platform
cd multi-tenant-platform/examples/multi-tenant
```

**Worker Agent**: Groq (fast, straightforward task)
**Deliverables**:
- Project cloned successfully
- Structure verified
- README reviewed

#### 1.2 Runtime Configuration (Bun)
```bash
# Replace npm with Bun
bun install
# Update package.json scripts to use bun
```

**Worker Agent**: Groq
**Deliverables**:
- All dependencies installed with Bun
- package.json scripts updated (dev, build, test, lint)
- Bun version locked in package.json

#### 1.3 Linter & Formatter (Biome)
```bash
bun add -D @biomejs/cli
npx biome init
```

**Worker Agent**: Groq
**Deliverables**:
- biome.toml configured with "recommended" + "typescript" extends
- .editorconfig for consistency
- Biome replaces ESLint/Prettier completely
- package.json scripts: "biome:check", "biome:fix"

#### 1.4 TypeScript Strict Mode
```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

**Worker Agent**: DeepSeek V3 (needs careful analysis of existing types)
**Deliverables**:
- tsconfig.json updated with strict settings
- All type errors fixed
- No `any` types in codebase

#### 1.5 Tailwind CSS Latest
```bash
bun add tailwindcss@latest postcss@latest autoprefixer@latest
```

**Worker Agent**: Groq
**Deliverables**:
- Tailwind upgraded to latest (4.x)
- tailwind.config.js updated with content paths
- PostCSS configured
- Design tokens defined in Tailwind config

#### 1.6 Verify & Run
```bash
bun run dev
```

**Worker Agent**: Groq
**Deliverables**:
- Development server starts successfully
- No console errors
- Basic routing works
- Homepage loads

**Stage 1 Exit Criteria**:
- [ ] Project runs locally with `bun dev`
- [ ] Biome linting passes
- [ ] TypeScript strict mode enabled with zero errors
- [ ] Tailwind styling works
- [ ] All package.json scripts use Bun

**Guardian Audit Checkpoints**:
1. Review package.json for correct Bun configuration
2. Verify tsconfig.json strict mode settings
3. Check biome.toml configuration
4. Confirm dev server starts cleanly
5. Validate no broken imports or missing dependencies

---

## Stage 2: Security - Stage 1 Foundation (2-3 hours)

**Objective**: Implement minimum must-have security (non-negotiable)

### Tasks

#### 2.1 Security Headers Middleware
**File**: `middleware/security-headers.ts`

**Worker Agent**: DeepSeek V3 (security-critical code)
**Requirements**:
- HSTS (max-age=63072000; includeSubDomains; preload)
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY
- Referrer-Policy: no-referrer
- Content-Security-Policy (strict, default-src 'self')
- Permissions-Policy

**Deliverables**:
- Modular security headers middleware
- CSP configured for Next.js
- No inline scripts allowed
- Report-only CSP for development

#### 2.2 Authentication Foundation
**Files**: `lib/auth.ts`, `middleware/auth.ts`

**Worker Agent**: DeepSeek V3 (security-critical)
**Requirements**:
- JWT-based authentication
- Secure cookie flags (httpOnly, secure, sameSite=strict)
- Token refresh mechanism
- Password hashing with bcrypt
- Session management

**Deliverables**:
- Auth utility functions
- Auth middleware for protected routes
- Login/logout API routes
- Session handling

#### 2.3 Input Validation & Sanitization
**File**: `lib/validation.ts`

**Worker Agent**: DeepSeek V3 (security-critical)
**Requirements**:
- Zod schemas for all inputs
- XSS prevention
- SQL injection prevention
- File upload validation
- URL validation

**Deliverables**:
- Zod validation schemas
- Sanitization utilities
- Validator middleware
- Type-safe validation

#### 2.4 CORS & API Security
**File**: `lib/api-security.ts`

**Worker Agent**: DeepSeek V3 (security-critical)
**Requirements**:
- CORS configuration
- API rate limiting
- Request size limits
- API key validation
- IP-based rate limiting

**Deliverables**:
- CORS middleware
- Rate limiting middleware
- API security utilities
- Abuse detection basics

#### 2.5 Environment Variables Security
**File**: `.env.example`, `lib/env.ts`

**Worker Agent**: Groq
**Requirements**:
- .env.example with all required variables
- Env validation with Zod
- No hardcoded secrets
- Vercel secrets documentation

**Deliverables**:
- Complete .env.example
- Runtime env validation
- Typed env variables
- Documentation for each env var

#### 2.6 HTTPS & Secure Cookies
**File**: `lib/cookies.ts`

**Worker Agent**: DeepSeek V3
**Requirements**:
- Secure cookie configuration
- httpOnly flag
- sameSite=strict
- Proper cookie domains
- Cookie encryption

**Deliverables**:
- Secure cookie utilities
- Cookie configuration
- Session cookie handling
- Cookie consent banner (GDPR)

**Stage 2 Exit Criteria**:
- [ ] All security headers present and tested
- [ ] Authentication flow working (login/logout/session)
- [ ] Input validation on all endpoints
- [ ] CORS properly configured
- [ ] Environment variables validated at runtime
- [ ] Cookies are secure (httpOnly, secure, sameSite)

**Guardian Audit Checkpoints**:
1. Review all security middleware for correctness
2. Verify CSP doesn't break the app
3. Test authentication flow
4. Validate input sanitization
5. Check CORS configuration
6. Verify no secrets in code

---

## Stage 3: Platform Blocks Integration (3-4 hours)

**Objective**: Integrate official Vercel platform blocks

### Tasks

#### 3.1 Report Abuse Block
**Installation**: `npx @vercel/platforms@latest add report-abuse`

**Worker Agent**: DeepSeek V3 (complex integration)
**Requirements**:
- Install and configure block
- Create server actions for abuse handling
- Add platform-specific categories
- Implement moderation workflow
- Add rate limiting

**Deliverables**:
- ReportAbuse component integrated
- `actions/report-abuse.ts` server action
- `lib/moderation.ts` moderation workflow
- Categories: spam, inappropriate, harassment, copyright, misinformation
- Rate limiting: 3 reports per minute per email
- Email notifications for high-priority reports

#### 3.2 Custom Domain Block
**Installation**: `npx @vercel/platforms@latest add custom-domain`

**Worker Agent**: DeepSeek V3 (complex integration with Vercel SDK)
**Requirements**:
- Install and configure block
- Implement `addDomain` server action
- Implement `getDomainStatus` server action
- Add domain validation
- Implement DNS verification flow
- Add rate limiting

**Deliverables**:
- CustomDomain component integrated
- `actions/domains.ts` with addDomain and getDomainStatus
- Domain validation (format, ownership)
- DNS status tracking
- Real-time verification (20s intervals)
- Rate limiting: 3 domains per hour per tenant

#### 3.3 Deploy Files Action
**Installation**: `npx @vercel/platforms@latest add deploy-files`

**Worker Agent**: DeepSeek V3 (complex SDK integration)
**Requirements**:
- Install and configure action
- Create deployment server actions
- Add deployment configuration
- Implement deployment queue
- Add deployment status tracking

**Deliverables**:
- DeployFiles action integrated
- `actions/deploy.ts` with deployment functions
- Deployment configuration templates
- Deployment status tracking
- Webhook handling for deployment events
- Error handling and rollback

#### 3.4 Block Styling & Theming
**File**: `styles/blocks.css`

**Worker Agent**: Groq
**Requirements**:
- Style blocks to match platform design
- Dark mode support
- Responsive design
- Accessibility (ARIA)

**Deliverables**:
- Consistent block styling
- Dark/light mode toggle
- Mobile-responsive blocks
- ARIA labels and roles

**Stage 3 Exit Criteria**:
- [ ] Report Abuse block working with server actions
- [ ] Custom Domain block with DNS verification
- [ ] Deploy Files action with Vercel SDK integration
- [ ] All blocks styled consistently
- [ ] Dark mode working
- [ ] Mobile responsive

**Guardian Audit Checkpoints**:
1. Verify all blocks use official Vercel components
2. Test server actions for security
3. Validate rate limiting implementation
4. Check DNS verification flow
5. Review deployment action security
6. Test block styling in both light/dark modes

---

## Stage 4: Multi-Tenant Core (4-5 hours)

**Objective**: Implement tenant resolution, routing, and data isolation

### Tasks

#### 4.1 Tenant Resolver Middleware
**File**: `middleware/tenant-resolver.ts`

**Worker Agent**: DeepSeek V3 (core architecture)
**Requirements**:
- Subdomain-based tenant resolution
- Custom domain support
- Path-based fallback
- Edge Config integration for performance
- Tenant validation

**Deliverables**:
- Tenant extraction from hostname
- Tenant lookup (Edge Config + DB fallback)
- Tenant validation
- Request headers injection (x-tenant-id, x-tenant-name)
- Invalid tenant handling

#### 4.2 Edge Config Setup
**File**: `lib/edge-config.ts`

**Worker Agent**: DeepSeek V3
**Requirements**:
- Edge Config client setup
- Tenant configuration schema
- Cache invalidation strategy
- Fallback to database

**Deliverables**:
- Edge Config integration
- Tenant config schema
- Sub-10ms lookup performance
- Cache invalidation
- Database fallback

#### 4.3 Tenant Context Propagation
**Files**: `lib/tenant-context.ts`, hooks for tenant access

**Worker Agent**: DeepSeek V3
**Requirements**:
- Server component tenant context
- Client component tenant hooks
- API route tenant helpers
- Type-safe tenant access

**Deliverables**:
- `useTenant()` hook for client components
- `getTenant()` helper for server components
- Tenant context type definitions
- Tenant-aware data fetching

#### 4.4 Tenant Data Isolation
**Files**: Database utilities, middleware

**Worker Agent**: DeepSeek V3 (security-critical)
**Requirements**:
- Tenant ID in all queries
- Row-level security
- Query interception
- Data isolation validation

**Deliverables**:
- Database query utilities with tenant filtering
- Row-level security implementation
- Query middleware
- Isolation testing utilities

#### 4.5 Tenant Routing
**Files**: Route handlers, middleware configuration

**Worker Agent**: DeepSeek V3
**Requirements**:
- Wildcard domain routing
- Custom domain routing
- Path-based routing fallback
- Route protection
- Tenant-specific redirects

**Deliverables**:
- Middleware routing configuration
- Route handlers for tenant patterns
- Redirect utilities
- Route protection

#### 4.6 Tenant Models & Types
**File**: `types/tenant.ts`, database schema

**Worker Agent**: Groq (straightforward type definitions)
**Requirements**:
- Tenant type definitions
- Database schema
- Tenant configuration types
- Tenant status enum

**Deliverables**:
- Complete tenant TypeScript types
- Database migration for tenants
- Tenant configuration schema
- Tenant status types (active, suspended, pending)

**Stage 4 Exit Criteria**:
- [ ] Tenant resolution working for subdomains
- [ ] Custom domain routing working
- [ ] Edge Config integration (<10ms lookups)
- [ ] Tenant context accessible in all components
- [ ] Data isolation enforced
- [ ] Tenant types and schema complete

**Guardian Audit Checkpoints**:
1. Verify tenant extraction logic
2. Test Edge Config performance
3. Validate data isolation (cross-tenant access prevention)
4. Review routing security
5. Test tenant context propagation
6. Check database query isolation

---

## Stage 5: Admin Dashboard (3-4 hours)

**Objective**: Build production-ready admin dashboard

### Tasks

#### 5.1 Dashboard Layout & Navigation
**File**: `app/admin/layout.tsx`, navigation components

**Worker Agent**: Groq (UI work)
**Requirements**:
- Admin layout with sidebar
- Navigation menu
- Breadcrumb navigation
- Mobile-responsive
- Dark mode toggle

**Deliverables**:
- Admin layout component
- Navigation menu
- Breadcrumb component
- Responsive sidebar
- Theme toggle

#### 5.2 Tenant Management Page
**File**: `app/admin/tenants/page.tsx`

**Worker Agent**: DeepSeek V3 (complex data table)
**Requirements**:
- Tenant list table
- Search and filter
- Pagination
- Tenant status badges
- Quick actions

**Deliverables**:
- Tenants list page
- Search functionality
- Status filtering
- Pagination (10/25/50 per page)
- Tenant status badges
- Quick action buttons

#### 5.3 Tenant Detail Page
**File**: `app/admin/tenants/[id]/page.tsx`

**Worker Agent**: DeepSeek V3
**Requirements**:
- Tenant information display
- Domain configuration
- Usage statistics
- Activity log
- Action buttons

**Deliverables**:
- Tenant detail view
- Domain configuration display
- Usage stats (users, storage, API calls)
- Activity log
- Edit/delete actions

#### 5.4 Analytics & Monitoring
**File**: `app/admin/analytics/page.tsx`

**Worker Agent**: DeepSeek V3 (data visualization)
**Requirements**:
- Platform statistics
- Tenant growth chart
- Usage metrics
- Performance metrics
- Security alerts

**Deliverables**:
- Analytics dashboard
- Tenant growth chart
- Usage metrics cards
- Performance monitoring
- Security alerts panel

#### 5.5 Settings Page
**File**: `app/admin/settings/page.tsx`

**Worker Agent**: Groq (forms)
**Requirements**:
- Platform settings form
- Security settings
- Email configuration
- API key management
- Save/Cancel actions

**Deliverables**:
- Settings form
- Security settings panel
- Email configuration
- API key management
- Form validation

#### 5.6 Real-time Updates
**File**: hooks for real-time data

**Worker Agent**: DeepSeek V3
**Requirements**:
- WebSocket or polling for updates
- Live tenant count
- Live system status
- Notification system

**Deliverables**:
- Real-time updates hook
- Live tenant count
- System status indicator
- Notification toast component

**Stage 5 Exit Criteria**:
- [ ] Dashboard layout complete with navigation
- [ ] Tenant list with search, filter, pagination
- [ ] Tenant detail page with all information
- [ ] Analytics with charts
- [ ] Settings page with forms
- [ ] Real-time updates working

**Guardian Audit Checkpoints**:
1. Verify admin authentication
2. Test data table performance
3. Validate search/filter logic
4. Check form validation
5. Review real-time update security
6. Test responsive design

---

## Stage 6: Testing & Quality Assurance (2-3 hours)

**Objective**: Comprehensive testing and quality checks

### Tasks

#### 6.1 Unit Tests Setup
**Files**: Test configuration, example tests

**Worker Agent**: Groq
**Requirements**:
- Test framework setup (Vitest)
- Test utilities
- Example tests
- Coverage configuration

**Deliverables**:
- Vitest configuration
- Test utilities
- Example unit tests
- Coverage reporting

#### 6.2 Integration Tests
**Files**: Integration test suites

**Worker Agent**: DeepSeek V3
**Requirements**:
- API route tests
- Middleware tests
- Database tests
- Authentication flow tests

**Deliverables**:
- API route test suite
- Middleware test suite
- Database integration tests
- Auth flow tests

#### 6.3 Security Tests
**Files**: Security test suite

**Worker Agent**: DeepSeek V3 (security-critical)
**Requirements**:
- XSS prevention tests
- SQL injection tests
- CSRF tests
- Rate limiting tests
- Tenant isolation tests

**Deliverables**:
- Security test suite
- Penetration test scenarios
- Vulnerability scans
- Security audit report

#### 6.4 Performance Tests
**Files**: Performance benchmarks

**Worker Agent**: Groq
**Requirements**:
- Load testing
- Response time benchmarks
- Database query performance
- Edge Config performance

**Deliverables**:
- Load test suite
- Performance benchmarks
- Database query analysis
- Edge Config timing

#### 6.5 Linting & Type Checking
**Files**: Quality checks

**Worker Agent**: Groq
**Requirements**:
- Biome linting (no errors)
- TypeScript strict mode (no errors)
- Code formatting check
- Import organization

**Deliverables**:
- Zero Biome errors
- Zero TypeScript errors
- Consistent code formatting
- Organized imports

#### 6.6 Accessibility Audit
**Files**: Accessibility tests

**Worker Agent**: Groq
**Requirements**:
- ARIA labels check
- Keyboard navigation
- Screen reader tests
- Color contrast checks

**Deliverables**:
- Accessibility audit report
- ARIA label fixes
- Keyboard navigation
- Color contrast compliance

**Stage 6 Exit Criteria**:
- [ ] All tests passing
- [ ] 80%+ code coverage
- [ ] Zero security vulnerabilities
- [ ] Performance benchmarks met
- [ ] Zero linting/type errors
- [ ] WCAG AA accessibility compliance

**Guardian Audit Checkpoints**:
1. Review all test suites
2. Validate security tests
3. Check performance benchmarks
4. Verify accessibility compliance
5. Review code coverage
6. Final quality gate

---

## Stage 7: Deployment & Documentation (2-3 hours)

**Objective**: Deploy to production and complete documentation

### Tasks

#### 7.1 Vercel Deployment Configuration
**Files**: vercel.json, environment setup

**Worker Agent**: Groq
**Requirements**:
- Vercel project configuration
- Environment variables setup
- Build configuration
- Deployment hooks

**Deliverables**:
- vercel.json configuration
- Environment variables documented
- Build settings optimized
- Preview deployments configured

#### 7.2 Production Build
**Files**: Build configuration

**Worker Agent**: Groq
**Requirements**:
- Optimized production build
- Bundle analysis
- Asset optimization
- Source maps configuration

**Deliverables**:
- Production build
- Bundle analysis report
- Optimized assets
- Source maps for production

#### 7.3 Monitoring & Error Tracking
**Files**: Monitoring setup

**Worker Agent**: DeepSeek V3
**Requirements**:
- Error tracking (Sentry)
- Analytics (Vercel Analytics)
- Uptime monitoring
- Performance monitoring

**Deliverables**:
- Sentry integration
- Vercel Analytics setup
- Uptime monitoring
- Performance dashboards

#### 7.4 Documentation
**Files**: README, API docs, deployment guide

**Worker Agent**: Groq (documentation)
**Requirements**:
- Complete README
- API documentation
- Deployment guide
- Security guide
- Contributing guidelines

**Deliverables**:
- Comprehensive README
- API documentation
- Deployment guide
- Security best practices
- Contributing guide

#### 7.5 Final Security Review
**Files**: Security checklist

**Worker Agent**: DeepSeek V3 (security audit)
**Requirements**:
- Final security audit
- Penetration testing
- Dependency vulnerability scan
- Configuration review

**Deliverables**:
- Security audit report
- Vulnerability scan results
- Configuration review
- Security checklist completed

#### 7.6 Launch Checklist
**Files**: Launch checklist

**Worker Agent**: Groq
**Requirements**:
- Pre-launch checklist
- Smoke tests
- Rollback plan
- Monitoring verification

**Deliverables**:
- Launch checklist
- Smoke test suite
- Rollback procedures
- Monitoring verification

**Stage 7 Exit Criteria**:
- [ ] Deployed to Vercel production
- [ ] Environment variables configured
- [ ] Monitoring and error tracking active
- [ ] Documentation complete
- [ ] Security audit passed
- [ ] Launch checklist complete

**Guardian Audit Checkpoints**:
1. Review deployment configuration
2. Verify environment variables
3. Test production build
4. Validate monitoring setup
5. Review documentation completeness
6. Final security gate

---

## Stage 8: Optional - Advanced Integrations (Future)

**Objective**: AI, TTS, STT, and advanced features (CAN BE SKIPPED)

> This stage is completely optional and can be implemented later if needed.

### Tasks (If Implemented)

#### 8.1 AI SDK Integration
**Resources**:
- https://www.dplooy.com/blog/vercel-ai-sdk-agents-complete-2026-implementation-guide
- https://ai-sdk.dev/docs/reference/ai-sdk-core/generate-text

**Worker Agent**: DeepSeek V3 (AI integration)

#### 8.2 Speech-to-Text (STT)
**Resources**:
- https://elevenlabs.io/docs/developers/guides/cookbooks/speech-to-text/batch/vercel-ai-sdk
- https://soniox.com/docs/stt/integrations/vercel-ai-sdk

**Worker Agent**: DeepSeek V3 (STT integration)

#### 8.3 Text-to-Speech (TTS)
**Resources**:
- https://dev.hume.ai/docs/integrations/vercel-ai-sdk
- https://fullstack-components.vercel.app/docs/audio

**Worker Agent**: DeepSeek V3 (TTS integration)

#### 8.4 AI Coder Template
**Resources**:
- https://vercel.com/docs/examples/oss-coding-agent

**Worker Agent**: DeepSeek V3 (AI code generation)

**Stage 8 is OPTIONAL and can be deferred indefinitely.**

---

## Guardian Management Strategy

### My Role as Guardian

I will NOT implement code directly. Instead, I will:

1. **Audit Plans**: Review all work before delegation
2. **Select Agents**: Choose appropriate worker (DeepSeek/Groq/GLM API)
3. **Monitor Execution**: Watch progress and intervene if needed
4. **Review Code**: Quality check all deliverables
5. **Gate Stages**: Ensure exit criteria are met before proceeding
6. **Coordinate**: Manage dependencies between stages

### Worker Agent Selection

| Task Type | Agent | Rationale |
|-----------|-------|-----------|
| Simple setup, files, UI | Groq | Fast, efficient for straightforward tasks |
| Security, architecture, complex logic | DeepSeek V3 | Best for complex, critical code |
| Debugging, troubleshooting | GLM API | Best debugging capabilities |
| Code review | DeepSeek V3 | Best analysis capabilities |

### Guardian Audit Process

For each stage:
1. **Pre-delegation**: Review task requirements and complexity
2. **Agent selection**: Choose best worker for the job
3. **Monitoring**: Track progress and watch for issues
4. **Code review**: Audit deliverables against requirements
5. **Exit gate**: Verify all criteria met before next stage

### Rollback Strategy

If any stage fails:
1. Git revert to previous stage completion
2. Document what went wrong
3. Adjust plan and retry
4. Guardian must approve retry

---

## Stage Dependencies

```
Stage 1 (Foundation)
    ↓
Stage 2 (Security Stage 1)
    ↓
Stage 3 (Platform Blocks)
    ↓
Stage 4 (Multi-Tenant Core)
    ↓
Stage 5 (Admin Dashboard)
    ↓
Stage 6 (Testing & QA)
    ↓
Stage 7 (Deployment & Docs)
    ↓
Stage 8 (Optional - AI/STT/TTS)
```

Each stage MUST complete successfully before proceeding to the next.

---

## Timeline Estimate

| Stage | Duration | Worker Agent Hours |
|-------|----------|-------------------|
| Stage 1 | 2-3 hours | ~4 agent hours |
| Stage 2 | 2-3 hours | ~5 agent hours |
| Stage 3 | 3-4 hours | ~6 agent hours |
| Stage 4 | 4-5 hours | ~8 agent hours |
| Stage 5 | 3-4 hours | ~6 agent hours |
| Stage 6 | 2-3 hours | ~4 agent hours |
| Stage 7 | 2-3 hours | ~4 agent hours |
| **Total (Stages 1-7)** | **18-25 hours** | **~37 agent hours** |
| Stage 8 (Optional) | TBD | TBD |

---

## Success Criteria

The project will be considered complete when:

1. ✅ All Stages 1-7 exit criteria met
2. ✅ Zero security vulnerabilities
3. ✅ Zero TypeScript/Biome errors
4. ✅ 80%+ test coverage
5. ✅ Deployed to Vercel production
6. ✅ Documentation complete
7. ✅ Guardian audit passed for all stages

---

## Quick Reference: Guardian Commands

### To Start Each Stage

```
GUARDIAN: "Begin Stage X with task analysis"
→ Analyzes stage requirements
→ Selects appropriate worker agents
→ Creates execution plan
→ Delegates to workers
```

### To Audit Progress

```
GUARDIAN: "Audit Stage X progress"
→ Reviews completed tasks
→ Checks deliverables
→ Validates against requirements
→ Approves or requests changes
```

### To Gate Stage Completion

```
GUARDIAN: "Verify Stage X exit criteria"
→ Checks all exit criteria
→ Runs quality checks
→ Reviews code changes
→ Approves stage completion
```

---

## Next Steps

When you're ready to begin, simply say:

**"Begin Stage 1"**

I will:
1. Analyze Stage 1 requirements
2. Select appropriate worker agents
3. Create detailed execution plan
4. Delegate to workers
5. Monitor progress
6. Audit deliverables
7. Gate completion

---

*This plan is designed to be executed with Guardian supervision using worker agents. I do NOT implement code directly - I audit, review, and coordinate.*
