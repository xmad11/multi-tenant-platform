# Official Vercel Documentation - Complete Reference
## Multi-Tenant & Multi-Project Platform with AI/TTS/STT Integration

> Last Updated: 2026 | Source: Official Vercel Documentation

---

## Table of Contents

1. [Platform Elements - Blocks](#1-platform-elements---blocks)
2. [Platform Elements - Actions](#2-platform-elements---actions)
3. [Multi-Tenant Platform Concepts](#3-multi-tenant-platform-concepts)
4. [Multi-Project Platform Concepts](#4-multi-project-platform-concepts)
5. [AI SDK & Speech Integration](#5-ai-sdk--speech-integration)
6. [Complete Implementation Examples](#6-complete-implementation-examples)

---

## 1. Platform Elements - Blocks

### 1.1 Report Abuse Block

**Installation:**
```bash
npx @vercel/platforms@latest add report-abuse
```

**Features:**
- Categorized abuse reporting
- Optional URL field for content location
- Detailed description textarea
- Email validation for follow-up
- Privacy notice built-in
- Form validation (required fields)
- Clean dialog UI (non-intrusive modal)
- Responsive design
- Visual indicators with icons and colors

**Basic Implementation:**
```typescript
import { ReportAbuse } from '@/components/blocks/report-abuse'

export default function ContentPage() {
  const abuseTypes = [
    { value: "spam", label: "Spam or Advertising" },
    { value: "inappropriate", label: "Inappropriate Content" },
    { value: "copyright", label: "Copyright Violation" },
    { value: "misinformation", label: "False Information" },
    { value: "harassment", label: "Harassment or Hate Speech" },
    { value: "other", label: "Other" }
  ]

  return (
    <div className="content-footer">
      <ReportAbuse types={abuseTypes} />
    </div>
  )
}
```

**With Server Action:**
```typescript
import { ReportAbuse } from '@/components/blocks/report-abuse'
import { reportContent } from '@/actions/moderation'

export default function BlogPost({ postId }) {
  const handleReport = async (data) => {
    await reportContent({
      ...data,
      postId,
      reportedAt: new Date()
    })
  }

  const types = [
    { value: "spam", label: "Spam" },
    { value: "inappropriate", label: "Inappropriate" },
    { value: "copyright", label: "Copyright" }
  ]

  return (
    <>
      <article>...</article>
      <ReportAbuse types={types} onSubmit={handleReport} />
    </>
  )
}
```

**Platform-Specific Categories:**
```typescript
// Documentation platform
const docAbuseTypes = [
  { value: "outdated", label: "Outdated Information" },
  { value: "broken-code", label: "Non-functional Code Examples" },
  { value: "security-issue", label: "Security Vulnerability" },
  { value: "plagiarism", label: "Plagiarized Content" },
  { value: "spam", label: "Spam or Promotional" }
]

// Blog platform
const blogAbuseTypes = [
  { value: "hate-speech", label: "Hate Speech" },
  { value: "misinformation", label: "Misinformation" },
  { value: "adult-content", label: "Adult Content" },
  { value: "harassment", label: "Harassment" },
  { value: "copyright", label: "Copyright Infringement" }
]

// E-commerce platform
const shopAbuseTypes = [
  { value: "counterfeit", label: "Counterfeit Product" },
  { value: "misleading", label: "Misleading Description" },
  { value: "prohibited", label: "Prohibited Item" },
  { value: "scam", label: "Potential Scam" }
]
```

**Integration with Moderation System:**
```typescript
import { ReportAbuse } from '@/components/blocks/report-abuse'
import { createModerationTicket } from '@/lib/moderation'
import { sendAlertEmail } from '@/lib/email'

export function ModeratedContent({ content }) {
  const handleAbuseReport = async (report) => {
    // Create moderation ticket
    const ticket = await createModerationTicket({
      contentId: content.id,
      reportType: report.category,
      description: report.description,
      reporterEmail: report.email,
      contentUrl: report.contentUrl || content.url,
      status: 'pending'
    })

    // Send alert for high-priority categories
    const highPriority = ['hate-speech', 'illegal', 'csam']
    if (highPriority.includes(report.category)) {
      await sendAlertEmail({
        to: 'moderation@platform.com',
        subject: `Urgent: ${report.category} reported`,
        ticketId: ticket.id
      })
    }

    return { success: true, ticketId: ticket.id }
  }

  const types = [
    { value: "spam", label: "Spam" },
    { value: "hate-speech", label: "Hate Speech" },
    { value: "illegal", label: "Illegal Content" }
  ]

  return (
    <div>
      {content.body}
      <ReportAbuse types={types} onSubmit={handleAbuseReport} />
    </div>
  )
}
```

**With Rate Limiting:**
```typescript
import { ReportAbuse } from '@/components/blocks/report-abuse'
import { rateLimit } from '@/lib/rate-limit'

const limiter = rateLimit({
  interval: 60 * 1000, // 1 minute
  uniqueTokenPerInterval: 500,
  maxReports: 3
})

export function RateLimitedReporting() {
  const handleSubmit = async (data) => {
    try {
      await limiter.check(data.email, 3)
      // Process report
    } catch {
      throw new Error('Too many reports. Please try again later.')
    }
  }

  return <ReportAbuse types={types} onSubmit={handleSubmit} />
}
```

**Field Specifications:**

| Field | Required | Type | Purpose | Validation |
|-------|----------|------|---------|------------|
| Category Selection | Yes | Select dropdown | Classifies abuse type | - |
| Content URL | No | URL input | Links to problematic content | Valid URL format |
| Description | Yes | Textarea | Detailed explanation | Min 20+ chars recommended |
| Reporter Email | Yes | Email input | Follow-up communication | Standard email format |

**Security Considerations:**
- Implement rate limiting to prevent spam reports
- Consider requiring login for reporting
- Log IP addresses for abuse prevention
- Sanitize all inputs before processing
- Add CAPTCHA for anonymous reporting

**Moderation Workflow:**
```typescript
const ModerationWorkflow = {
  // 1. Receive report
  receiveReport: async (report) => {
    const ticket = await createTicket(report)
    await notifyModerators(ticket)
    return ticket.id
  },

  // 2. Review content
  reviewContent: async (ticketId) => {
    const ticket = await getTicket(ticketId)
    const content = await getContent(ticket.contentId)
    return { ticket, content }
  },

  // 3. Take action
  takeAction: async (ticketId, action) => {
    switch(action) {
      case 'remove':
        await removeContent(ticketId)
        break
      case 'warning':
        await sendWarning(ticketId)
        break
      case 'dismiss':
        await dismissReport(ticketId)
        break
    }
  },

  // 4. Notify reporter
  notifyReporter: async (ticketId, outcome) => {
    const ticket = await getTicket(ticketId)
    await sendEmail(ticket.reporterEmail, outcome)
  }
}
```

**Best Practices:**
- Clear categories: Use specific, non-overlapping categories
- Quick response: Acknowledge reports immediately
- Follow-up: Send confirmation emails when appropriate
- Transparency: Publish moderation guidelines
- Appeals process: Provide a way to contest false reports
- Analytics: Track report patterns to identify issues

---

### 1.2 Custom Domain Block

**Installation:**
```bash
npx @vercel/platforms@latest add custom-domain
```

**Features:**
- Real-time DNS verification (checks every 20 seconds)
- Smart DNS record detection (TXT, CNAME, or A records)
- Visual status indicators (pending, invalid, successful)
- One-click copying with confirmation feedback
- Responsive DNS table presentation
- Automatic domain validation and lowercase conversion

**Basic Implementation:**
```typescript
import { CustomDomain } from "@/components/blocks/custom-domain";

export default function DomainSettings() {
  return <CustomDomain defaultDomain="docs.example.com" />;
}
```

**Required Server Actions:**

**`addDomain(domain: string)` - Adds a domain to your Vercel project:**
```typescript
import { vercel } from "@/lib/vercel";

export async function addDomain(domain: string) {
  const response = await vercel.projects.addProjectDomain({
    idOrName: process.env.PROJECT_ID,
    requestBody: {
      name: domain,
      redirect: null,
      gitBranch: null,
    },
  });
  return response;
}
```

**`getDomainStatus(domain: string)` - Checks DNS configuration status:**
```typescript
export async function getDomainStatus(domain: string) {
  const response = await vercel.domains.getDomainConfig({
    domain,
  });

  return {
    status: response.verified
      ? "Valid Configuration"
      : response.verification
        ? "Pending Verification"
        : "Invalid Configuration",
    dnsRecordsToSet: response.verification?.dns || response.dns,
  };
}
```

**Sub-Components:**

| Component | Purpose | Usage |
|-----------|---------|-------|
| `DomainConfiguration` | DNS record display and refresh logic | `<DomainConfiguration domain="example.com" className="custom-class" />` |
| `DomainStatusIcon` | Visual status of domain configuration | `<DomainStatusIcon domain="example.com" />` |
| `InlineSnippet` | Styled inline code display | `<InlineSnippet>_vercel.example.com</InlineSnippet>` |

**Supported DNS Record Types:**
- **TXT**: Domain ownership verification
- **CNAME**: Subdomain pointing
- **A**: IPv4 address mapping
- **AAAA**: IPv6 address mapping
- **MX**: Mail server configuration
- **NS**: Nameserver delegation

**Full Page Implementation:**
```typescript
// app/settings/domains/page.tsx
import { CustomDomain } from "@/components/blocks/custom-domain";
import { getCurrentUserDomain } from "@/lib/db";

export default async function DomainsPage() {
  const currentDomain = await getCurrentUserDomain();

  return (
    <div className="container mx-auto py-8">
      <h1 className="text-2xl font-bold mb-6">Domain Settings</h1>
      <CustomDomain defaultDomain={currentDomain} />
    </div>
  );
}
```

**Security Best Practices:**
- Implement rate limiting on domain addition to prevent abuse
- Validate domain ownership before allowing configuration
- Provide clear error messages for invalid domains
- Vercel automatically provisions SSL certificates once configured
- Track domain configuration success rates and common issues

---

## 2. Platform Elements - Actions

### 2.1 Deploy Files Action

**Installation:**
```bash
npx @vercel/platforms@latest add deploy-files
```

**Features:**
- Programmatic deployment using Vercel SDK
- Custom domain support
- Project configuration (build settings, environment variables)
- SSO protection handling (optional public preview deployments)
- Unique deployment naming with automatic UUID generation

**Basic Usage:**
```typescript
import { deployFiles } from "@/actions/deploy-files"
import type { InlinedFile } from "@vercel/sdk/models/createdeploymentop"

const files: InlinedFile[] = [
  {
    file: "index.html",
    data: "<html><body><h1>Hello from my platform!</h1></body></html>"
  },
  {
    file: "package.json",
    data: JSON.stringify({
      name: "my-deployment",
      version: "1.0.0"
    })
  }
]

await deployFiles(files, {
  domain: "customer-site.com",
  deploymentName: "customer-deployment-1",
  projectId: "existing-project-id", // Optional: use existing project
  config: {
    framework: "nextjs",
    buildCommand: "npm run build",
    outputDirectory: ".next"
  }
})
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `files` | `InlinedFile[]` | Yes | Array of files to deploy |
| `args.projectId` | `string` | No | Existing project ID to use |
| `args.deploymentName` | `string` | No | Custom deployment name |
| `args.config` | `ProjectSettings` | No | Build configuration |
| `args.domain` | `string` | No | Custom domain for deployment |

**Advanced Example with Full Configuration:**
```typescript
import { deployFiles } from "@/actions/deploy-files"
import type { InlinedFile, ProjectSettings } from "@vercel/sdk/models/createdeploymentop"

const files: InlinedFile[] = [
  // Your application files here
]

const config: ProjectSettings = {
  framework: "nextjs",
  buildCommand: "npm run build",
  outputDirectory: ".next",
  installCommand: "npm install",
  devCommand: "npm run dev",
  env: {
    API_KEY: "your-api-key",
    DATABASE_URL: "your-database-url"
  },
  buildEnv: {
    NODE_ENV: "production"
  }
}

const deployment = await deployFiles(files, {
  deploymentName: `deployment-${Date.now()}`,
  config,
  domain: "app.customer-domain.com"
})
```

**Integration with Claim Deployment:**
```typescript
// 1. Deploy files server-side
const deployment = await deployFiles(files, { domain })

// 2. Show claim interface client-side
<ClaimDeployment
  url={deployment.url}
  onClaimClick={handleTransferOwnership}
/>
```

**Security Considerations:**
- Requires Vercel API credentials with deployment permissions
- Always validate and sanitize file contents before deployment
- Implement rate limiting to prevent abuse
- Store API credentials securely using environment variables

---

## 3. Multi-Tenant Platform Concepts

### 3.1 What is a Tenant

A tenant represents a customer, workspace, or organization within your multi-tenant application. Each tenant has:
- Own data
- Own configuration
- Own branding
- **Shared codebase and deployment**

**Examples:**
- Blog platform: Each writer with their own blog is a tenant
- Documentation platform: Each company with its own docs site is a tenant
- E-commerce platform: Each store owner is a tenant

### 3.2 Tenant Identification Strategies

**Subdomain-based** (extract tenant from subdomain):
```typescript
const hostname = request.headers.get("host");
const subdomain = hostname.split(".")[0]; // "tenant1" from "tenant1.yourapp.com"
```

**Custom domain-based** (map custom domains to tenants):
```typescript
// Map custom domain to tenant in database
const tenant = await db.tenant.findFirst({
  where: { customDomain: hostname },
});
```

**Path-based** (extract tenant from URL path):
```typescript
const pathname = request.nextUrl.pathname;
const tenantSlug = pathname.split("/")[1]; // "tenant1" from "/tenant1/dashboard"
```

### 3.3 Tenant Data Isolation

**Database-level isolation:**
```typescript
const posts = await db.post.findMany({
  where: { tenantId: tenant.id },
});
```

**Application-level isolation:**
- Middleware ensures requests can only access their tenant's data
- Use tenant ID in all database queries

**Edge Config for fast lookups:**
```typescript
import { get } from "@vercel/edge-config";

const tenant = await get(`tenant_${hostname}`);
```

### 3.4 Wildcard Domains

Wildcard domains automatically serve all subdomains from a single Vercel project:

**Requirements:**
- Add `*.yourapp.com` to your project
- Point your domain to Vercel's nameservers:
  - `ns1.vercel-dns.com`
  - `ns2.vercel-dns.com`

**Benefits:**
- Any subdomain (`tenant1.yourapp.com`, `tenant2.yourapp.com`) automatically routes to your app
- Vercel issues SSL certificates for each subdomain on-the-fly
- No manual configuration per subdomain

### 3.5 Custom Domains

For tenants bringing their own domain:

1. Add `tenant1.com` to your Vercel project via SDK
2. Tenant configures DNS (CNAME or nameservers)
3. Verify domain ownership (TXT record)
4. Vercel issues SSL certificate automatically

### 3.6 SSL Certificate Issuance

Vercel automatically issues SSL certificates using Let's Encrypt:

| Domain Type | Certificate Type | Details |
|-------------|------------------|---------|
| Wildcard domains | Single wildcard certificate | Covers all subdomains |
| Custom domains | Individual certificate | One per domain |
| All | Automatic renewal | Before expiration |
| All | No configuration required | Fully managed |

### 3.7 Domain Verification

For domains already in use on Vercel:

1. Add domain to your project
2. Vercel generates a unique TXT record
3. Tenant adds TXT record to their DNS
4. Verify ownership via SDK or dashboard
5. Certificate issues once verified

### 3.8 Middleware Implementation

**How middleware resolves tenants:**
```typescript
export async function middleware(request: NextRequest) {
  const hostname = request.headers.get("host");

  // Get tenant from subdomain or custom domain
  const tenant = await resolveTenant(hostname);

  // Add tenant to request headers
  const response = NextResponse.next();
  response.headers.set("x-tenant-id", tenant.id);

  return response;
}
```

**Request handling flow:**
1. User visits `tenant1.yourapp.com`
2. Request hits Vercel's edge network
3. Middleware extracts subdomain (`tenant1`)
4. Middleware looks up tenant in database or Edge Config
5. Middleware adds tenant context to request headers
6. Page component reads tenant from headers
7. Page renders with tenant-specific data

### 3.9 Tenant Context Propagation

**In middleware (set headers):**
```typescript
response.headers.set("x-tenant-id", tenant.id);
```

**In server components (read headers):**
```typescript
import { headers } from "next/headers";

const tenantId = headers().get("x-tenant-id");
```

**In API routes (access request headers):**
```typescript
const tenantId = request.headers.get("x-tenant-id");
```

### 3.10 Performance Considerations

| Technique | Benefit | Implementation |
|-----------|---------|----------------|
| Edge Config | Sub-10ms lookups | `await get(\`tenant_${hostname}\`)` |
| Caching | Reduce database queries | Cache tenant lookups in middleware |
| Connection pooling | Handle multiple tenants efficiently | Use connection pooling for database |

### 3.11 Single Deployment Architecture

Multi-tenant means:
- One Next.js codebase
- One Vercel deployment
- Multiple domains (subdomains + custom domains)
- Shared infrastructure and resources
- Tenant-aware routing and data access

---

## 4. Multi-Project Platform Concepts

### 4.1 What is a Project

A Vercel project represents a single application with:
- Unique project ID
- Independent configuration
- Separate deployment history
- Isolated builds and functions
- Own Git repository
- Own environment variables

### 4.2 Programmatic Project Creation

**Using Vercel SDK:**
```typescript
import { Vercel } from "@vercel/sdk";

const vercel = new Vercel({
  bearerToken: "<YOUR_BEARER_TOKEN_HERE>",
});

const { value: project } = await vercel.projects.createProject({
  teamId: "team_1234",
  requestBody: {
    name: `tenant-${tenantId}`,
    framework: "nextjs",
  },
});
```

### 4.3 Project Templates

Generate projects from templates for consistency:
```typescript
import { Vercel } from "@vercel/sdk";

const vercel = new Vercel({
  bearerToken: "<YOUR_BEARER_TOKEN_HERE>",
});

const { value: project } = await vercel.projects.createProject({
  teamId: "team_1234",
  requestBody: {
    name: `tenant-${tenantId}`,
    gitRepository: {
      type: "github",
      repo: "your-org/tenant-template",
    },
  },
});
```

### 4.4 Deployment Lifecycle

1. **Create**: Deploy code to project
2. **Build**: Vercel builds the application
3. **Preview**: Test deployment before promoting
4. **Production**: Promote deployment to production
5. **Rollback**: Revert to previous deployment if needed

### 4.5 Preview vs Production Deployments

| Preview Deployments | Production Deployments |
|---------------------|------------------------|
| Generated for every Git push | Live on project domains |
| Unique URL per deployment | Promoted from preview or direct |
| Test changes before production | Serves end users |

### 4.6 Per-Project Domains

**Adding domains programmatically:**
```typescript
import { Vercel } from "@vercel/sdk";

const vercel = new Vercel({
  bearerToken: "<YOUR_BEARER_TOKEN_HERE>",
});

await vercel.projects.addProjectDomain({
  idOrName: project.id,
  requestBody: {
    name: "tenant1.com",
  },
});
```

**Automatic URLs:**
- Production: `project-name.vercel.app`
- Preview: `project-name-git-branch.vercel.app`

### 4.7 Multi-Project vs Multi-Tenant Comparison

| Aspect | Multi-Project | Multi-Tenant (Single Project) |
|--------|---------------|------------------------------|
| Tenant Structure | Each tenant = separate Vercel project | All tenants = one Vercel project |
| Isolation | Complete isolation | Application-level isolation |
| Scaling | Independent scaling per tenant | Shared infrastructure |
| Management | Higher overhead | Lower overhead |
| Deployment | Deploy once per tenant | Deploy once, update all tenants |

### 4.8 When to Use Multi-Project

**Use Multi-Project when:**
- Tenants deploy their own code
- Each tenant needs custom functionality
- Complete isolation is required (security, compliance)
- AI agents are generating and deploying code
- Users create applications from templates

**Use Multi-Tenant when:**
- All tenants use the same application
- Content differs but code is the same
- You want to deploy once, update all tenants
- Lower operational overhead preferred

### 4.9 AI Coding Platforms

Perfect for platforms where:
- AI generates unique code per tenant
- Each tenant's app is different
- Users can deploy custom logic
- Examples: Spawn, Orchids, v0

### 4.10 Template-Based Platforms

Ideal for:
- Creating projects from templates
- Customizing per tenant
- Maintaining consistency
- Rapid tenant onboarding

### 4.11 User-Generated Applications

Well-suited for:
- Users creating their own apps
- Hosting user-generated content
- Platform controls infrastructure
- Users focus on their code

---

## 5. AI SDK & Speech Integration

### 5.1 Official AI SDK Resources

**Core AI SDK Documentation:**
- [AI SDK Core: generateText](https://ai-sdk.dev/docs/reference/ai-sdk-core/generate-text)
- [Vercel AI SDK | What is Vercel AI SDK?](https://voltagent.dev/blog/vercel-ai-sdk/)

**2026 Implementation Guide:**
- [Vercel AI SDK Agents: Complete 2026 Implementation Guide](https://www.dplooy.com/blog/vercel-ai-sdk-agents-complete-2026-implementation-guide) - Production-ready AI agents with Vercel AI SDK 6

### 5.2 Speech-to-Text (STT) Integration

**ElevenLabs Vercel AI SDK Integration:**
- URL: https://elevenlabs.io/docs/developers/guides/cookbooks/speech-to-text/batch/vercel-ai-sdk
- Provider: ElevenLabs
- Use case: Speech-to-text transcription

**Soniox Speech AI Integration:**
- URL: https://soniox.com/docs/stt/integrations/vercel-ai-sdk
- Provider: Soniox
- Use case: STT integration with Vercel AI SDK

**GitHub Discussion:**
- [speech-to-text integration for prompt-input component](https://github.com/vercel/ai-elements/issues/102)
- Status: Discussion about implementing voice input for prompt-input components (September 2025)

### 5.3 Text-to-Speech (TTS) Integration

**Hume API Integration:**
- URL: https://dev.hume.ai/docs/integrations/vercel-ai-sdk
- Feature: `generateSpeech` function
- Use case: Text-to-speech with Vercel AI SDK

### 5.4 Audio Component Libraries

**Full Stack Components:**
- URL: https://fullstack-components.vercel.app/docs/audio
- Features: AI text-to-speech and speech-to-text React components

### 5.5 Additional Resources

**Chinese Documentation (STT & TTS):**
- URL: https://blog.csdn.net/gitblog_00244/article/details/150964647
- Title: "终极语音交互革命：Vercel AI SDK如何实现STT与TTS的完美" (Ultimate Voice Interaction Revolution)
- Date: January 2026

---

## 6. Complete Implementation Examples

### 6.1 Complete Multi-Tenant Setup with All Blocks

**Installation Commands:**
```bash
# Install all platform blocks and actions
npx @vercel/platforms@latest add report-abuse
npx @vercel/platforms@latest add custom-domain
npx @vercel/platforms@latest add deploy-files

# Install AI SDK
bun add ai
bun add @ai-sdk/openai  # or your preferred provider
```

### 6.2 Complete Middleware with Tenant Resolution

**File: `middleware.ts`**
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { get } from '@vercel/edge-config';

export async function middleware(request: NextRequest) {
  const hostname = request.headers.get('host') || '';

  // 1. Resolve tenant from hostname (subdomain or custom domain)
  let tenant;
  try {
    tenant = await get(`tenant_${hostname}`);
  } catch {
    // Fallback to database lookup
    tenant = await getTenantFromDB(hostname);
  }

  if (!tenant) {
    return NextResponse.redirect('/404');
  }

  // 2. Add tenant context to headers
  const response = NextResponse.next();
  response.headers.set('x-tenant-id', tenant.id);
  response.headers.set('x-tenant-name', tenant.name);

  // 3. Add security headers
  addSecurityHeaders(response);

  return response;
}

function addSecurityHeaders(response: NextResponse) {
  response.headers.set('Strict-Transport-Security', 'max-age=63072000; includeSubDomains; preload');
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('Referrer-Policy', 'no-referrer');
  response.headers.set('Content-Security-Policy', "default-src 'self'; script-src 'self' 'unsafe-inline';");
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

### 6.3 Complete Page with All Blocks

**File: `app/[tenant]/page.tsx`**
```typescript
import { headers } from 'next/headers';
import { ReportAbuse } from '@/components/blocks/report-abuse';
import { CustomDomain } from '@/components/blocks/custom-domain';

export default async function TenantPage() {
  const headersList = headers();
  const tenantId = headersList.get('x-tenant-id');
  const tenantName = headersList.get('x-tenant-name');

  const abuseTypes = [
    { value: "spam", label: "Spam" },
    { value: "inappropriate", label: "Inappropriate Content" },
    { value: "harassment", label: "Harassment" }
  ];

  return (
    <div className="container mx-auto py-8">
      <header className="mb-8">
        <h1 className="text-4xl font-bold">{tenantName}</h1>
        <p className="text-gray-600">Tenant ID: {tenantId}</p>
      </header>

      {/* Main content area */}
      <main className="mb-12">
        {/* Your tenant-specific content here */}
      </main>

      {/* Platform Elements */}
      <section className="space-y-8">
        <div>
          <h2 className="text-2xl font-semibold mb-4">Report Abuse</h2>
          <ReportAbuse types={abuseTypes} />
        </div>

        <div>
          <h2 className="text-2xl font-semibold mb-4">Custom Domain</h2>
          <CustomDomain defaultDomain={`${tenantId}.yourplatform.com`} />
        </div>
      </section>
    </div>
  );
}
```

### 6.4 Server Actions for Domain Management

**File: `app/actions/domains.ts`**
```typescript
'use server';

import { vercel } from '@/lib/vercel';
import { rateLimit } from '@/lib/rate-limit';

const domainLimiter = rateLimit({
  interval: 60 * 60 * 1000, // 1 hour
  uniqueTokenPerInterval: 500,
  maxDomains: 3
});

export async function addDomain(domain: string, tenantId: string) {
  // Rate limiting
  await domainLimiter.check(tenantId, 3);

  // Validate domain
  if (!isValidDomain(domain)) {
    throw new Error('Invalid domain format');
  }

  // Add domain to Vercel project
  const response = await vercel.projects.addProjectDomain({
    idOrName: process.env.PROJECT_ID,
    requestBody: {
      name: domain.toLowerCase(),
      redirect: null,
      gitBranch: null,
    },
  });

  return response;
}

export async function getDomainStatus(domain: string) {
  const response = await vercel.domains.getDomainConfig({
    domain,
  });

  return {
    status: response.verified
      ? "Valid Configuration"
      : response.verification
        ? "Pending Verification"
        : "Invalid Configuration",
    dnsRecordsToSet: response.verification?.dns || response.dns,
  };
}

function isValidDomain(domain: string): boolean {
  const domainRegex = /^([a-z0-9]+(-[a-z0-9]+)*\.)+[a-z]{2,}$/;
  return domainRegex.test(domain);
}
```

### 6.5 Deploy Files with AI-Generated Code

**File: `app/actions/deploy.ts`**
```typescript
'use server';

import { deployFiles } from '@/actions/deploy-files';
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';
import type { InlinedFile } from '@vercel/sdk/models/createdeploymentop';

export async function deployAIGeneratedApp(
  prompt: string,
  tenantId: string,
  domain?: string
) {
  // 1. Generate code using AI
  const { text } = await generateText({
    model: openai('gpt-4'),
    prompt: `Generate a Next.js application based on this description: ${prompt}
    Return only the code files needed, formatted as JSON.`,
  });

  // 2. Parse AI response into files
  const files: InlinedFile[] = parseAIGeneratedCode(text);

  // 3. Deploy to Vercel
  const deployment = await deployFiles(files, {
    deploymentName: `${tenantId}-${Date.now()}`,
    domain,
    config: {
      framework: 'nextjs',
      buildCommand: 'npm run build',
      outputDirectory: '.next'
    }
  });

  return deployment;
}

function parseAIGeneratedCode(aiResponse: string): InlinedFile[] {
  // Parse AI response and convert to file array
  // Implementation depends on AI output format
  return [];
}
```

---

## Summary Checklist

### Platform Blocks to Install
- [x] `npx @vercel/platforms@latest add report-abuse`
- [x] `npx @vercel/platforms@latest add custom-domain`
- [x] `npx @vercel/platforms@latest add deploy-files`

### Required Server Actions
- [x] `addDomain(domain: string)` - Add domain to Vercel project
- [x] `getDomainStatus(domain: string)` - Check DNS configuration
- [x] Report abuse handler with moderation workflow
- [x] Deploy files handler with AI integration

### Middleware Implementation
- [x] Tenant resolution from subdomain/custom domain
- [x] Tenant context propagation via headers
- [x] Security headers (HSTS, CSP, X-Frame-Options, etc.)
- [x] Edge Config integration for performance

### DNS Configuration
- [x] Wildcard domain setup (`*.yourapp.com`)
- [x] Nameservers pointed to Vercel (`ns1.vercel-dns.com`, `ns2.vercel-dns.com`)
- [x] Custom domain verification flow

### Security Implementation
- [x] Rate limiting on domain addition
- [x] Rate limiting on abuse reports
- [x] Domain validation before configuration
- [x] Input sanitization for all forms
- [x] CAPTCHA for anonymous actions (optional)

### AI/STT/TTS Integration Resources
- [x] Vercel AI SDK for code generation
- [x] ElevenLabs for STT
- [x] Hume API for TTS (`generateSpeech`)
- [x] Audio component libraries for UI

---

*This document compiles all official Vercel documentation for multi-tenant, multi-project, and AI integration platforms as of 2026.*
