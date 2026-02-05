# Demo Pages & Implementation Examples
## Multi-Tenant Platform - Recommended Tools Integration

**Date**: 2026-02-05
**Purpose**: Complete demo pages and code examples for all recommended tools

---

## Table of Contents

1. [Authentication Demo - Clerk](#1-authentication-demo---clerk)
2. [AI Integration Demo - Vercel AI SDK v5](#2-ai-integration-demo---vercel-ai-sdk-v5)
3. [Rate Limiting Demo - Upstash](#3-rate-limiting-demo---upstash)
4. [Admin Dashboard Demo - TailAdmin V2](#4-admin-dashboard-demo---tailadmin-v2)
5. [Billing Demo - Flexprice](#5-billing-demo---flexprice)
6. [Monitoring Demo - Grafana](#6-monitoring-demo---grafana)
7. [Workflow Demo - n8n](#7-workflow-demo---n8n)
8. [Notifications Demo - Novu](#8-notifications-demo---novu)
9. [Analytics Demo - PostHog](#9-analytics-demo---posthog)
10. [Low-Code Builder Demo - Puck](#10-low-code-builder-demo---puck)

---

## 1. Authentication Demo - Clerk

### Installation

```bash
pnpm add @clerk/nextjs
```

### Middleware Setup

**File**: `middleware.ts`

```typescript
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server';

const isPublicRoute = createRouteMatcher([
  '/',
  '/sign-in(.*)',
  '/sign-up(.*)',
  '/api/webhooks(.*)',
  '/s/:subdomain',
]);

export default clerkMiddleware(async (auth, req) => {
  const { userId, tenantId } = await auth();

  // Multi-tenant organization check
  if (userId && !tenantId && !isPublicRoute(req)) {
    // Redirect to org selection
    return Response.redirect(new URL('/select-org', req.url));
  }
});

export const config = {
  matcher: ['/((?!.*\\..*|_next).*)', '/', '/(api|trpc)(.*)'],
};
```

### Organization Selector Page

**File**: `app/select-org/page.tsx`

```typescript
'use client';

import { useAuth, useUser } from '@clerk/nextjs';
import { useState } from 'react';

export default function SelectOrganizationPage() {
  const { user } = useUser();
  const { signOut } = useAuth();
  const [loading, setLoading] = useState(false);

  const organizations = user?.organizationMemberships?.map(
    (membership) => membership.organization
  ) || [];

  return (
    <div className="min-h-screen bg-gradient-to-br from-gray-900 to-gray-800 flex items-center justify-center p-4">
      <div className="bg-white rounded-2xl shadow-2xl p-8 max-w-md w-full">
        <h1 className="text-2xl font-bold mb-2 text-gray-900">
          Select Your Workspace
        </h1>
        <p className="text-gray-600 mb-6">
          Choose an organization to continue
        </p>

        <div className="space-y-3 mb-6">
          {organizations.map((org: any) => (
            <a
              key={org.id}
              href={`/${org.slug}`}
              className="block p-4 border border-gray-200 rounded-xl hover:border-blue-500 hover:bg-blue-50 transition-all"
            >
              <div className="font-medium text-gray-900">{org.name}</div>
              <div className="text-sm text-gray-500 capitalize">
                {org.pending ? 'Pending' : 'Active'}
              </div>
            </a>
          ))}
        </div>

        <div className="border-t pt-6">
          <button
            onClick={() => signOut()}
            className="w-full py-3 px-4 border border-gray-300 rounded-xl hover:bg-gray-50 transition-all font-medium"
          >
            Sign Out
          </button>
        </div>
      </div>
    </div>
  );
}
```

### Protected API Route

**File**: `app/api/tenants/route.ts`

```typescript
import { auth } from '@clerk/nextjs/server';
import { NextResponse } from 'next/server';

export async function GET() {
  const { userId, orgId } = await auth();

  if (!userId || !orgId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // Fetch tenant data based on orgId
  const tenants = await getTenantsForOrg(orgId);

  return NextResponse.json({ tenants });
}
```

---

## 2. AI Integration Demo - Vercel AI SDK v5

### Installation

```bash
pnpm add ai @ai-sdk/openai
```

### AI Chat Component

**File**: `app/components/ai-chat.tsx`

```typescript
'use client';

import { useChat } from 'ai/react';
import { useState } from 'react';

export function AIChat() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat({
    api: '/api/chat',
    initialMessages: [
      {
        id: 'welcome',
        role: 'assistant',
        content: 'Hello! I\'m your AI assistant. How can I help you build something today?',
      },
    ],
  });

  const [agentMode, setAgentMode] = useState<'creative' | 'precise'>('creative');

  return (
    <div className="flex flex-col h-screen bg-gray-900 text-white">
      {/* Header */}
      <div className="border-b border-gray-800 p-4">
        <div className="flex items-center justify-between max-w-4xl mx-auto">
          <h1 className="text-xl font-bold">AI Coder Assistant</h1>
          <div className="flex gap-2">
            <button
              onClick={() => setAgentMode('creative')}
              className={`px-4 py-2 rounded-lg transition-all ${
                agentMode === 'creative'
                  ? 'bg-purple-600 text-white'
                  : 'bg-gray-800 text-gray-400'
              }`}
            >
              Creative
            </button>
            <button
              onClick={() => setAgentMode('precise')}
              className={`px-4 py-2 rounded-lg transition-all ${
                agentMode === 'precise'
                  ? 'bg-blue-600 text-white'
                  : 'bg-gray-800 text-gray-400'
              }`}
            >
              Precise
            </button>
          </div>
        </div>
      </div>

      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-4">
        <div className="max-w-4xl mx-auto space-y-4">
          {messages.map((message) => (
            <div
              key={message.id}
              className={`flex ${message.role === 'user' ? 'justify-end' : 'justify-start'}`}
            >
              <div
                className={`max-w-2xl rounded-2xl px-6 py-4 ${
                  message.role === 'user'
                    ? 'bg-purple-600 text-white'
                    : 'bg-gray-800 text-gray-100'
                }`}
              >
                <div className="prose prose-invert max-w-none">
                  {message.content}
                </div>
              </div>
            </div>
          ))}
          {isLoading && (
            <div className="flex justify-start">
              <div className="bg-gray-800 rounded-2xl px-6 py-4">
                <div className="flex space-x-2">
                  <div className="w-2 h-2 bg-gray-500 rounded-full animate-bounce" />
                  <div className="w-2 h-2 bg-gray-500 rounded-full animate-bounce delay-100" />
                  <div className="w-2 h-2 bg-gray-500 rounded-full animate-bounce delay-200" />
                </div>
              </div>
            </div>
          )}
        </div>
      </div>

      {/* Input */}
      <div className="border-t border-gray-800 p-4">
        <form onSubmit={handleSubmit} className="max-w-4xl mx-auto">
          <div className="flex gap-3">
            <textarea
              value={input}
              onChange={handleInputChange}
              placeholder="Describe what you want to build... e.g., 'Create a dashboard with dark mode and charts'"
              className="flex-1 bg-gray-800 border border-gray-700 rounded-xl px-4 py-3 resize-none focus:outline-none focus:border-purple-500 transition-all"
              rows={3}
            />
            <button
              type="submit"
              disabled={isLoading || !input.trim()}
              className="px-6 py-3 bg-purple-600 hover:bg-purple-700 disabled:bg-gray-700 disabled:cursor-not-allowed rounded-xl font-medium transition-all"
            >
              Generate
            </button>
          </div>
        </form>
      </div>
    </div>
  );
}
```

### AI API Route with Tool Calling

**File**: `app/api/chat/route.ts`

```typescript
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

export const runtime = 'edge';

// Allow streaming responses up to 30 seconds
export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: openai('gpt-4-turbo'),
    messages,
    temperature: 0.7,
    maxTokens: 2000,
    tools: {
      generate_code: {
        description: 'Generate code for a specific component or feature',
        parameters: {
          type: 'object',
          properties: {
            component: {
              type: 'string',
              description: 'The component to generate',
            },
            framework: {
              type: 'string',
              enum: ['react', 'nextjs', 'vue'],
              description: 'The framework to use',
            },
            styling: {
              type: 'string',
              enum: ['tailwind', 'css', 'styled-components'],
              description: 'The styling approach',
            },
          },
          required: ['component', 'framework'],
        },
      },
      analyze_requirements: {
        description: 'Analyze project requirements and suggest architecture',
        parameters: {
          type: 'object',
          properties: {
            description: {
              type: 'string',
              description: 'Project description',
            },
            constraints: {
              type: 'array',
              items: { type: 'string' },
              description: 'Technical constraints',
            },
          },
          required: ['description'],
        },
      },
    },
  });

  return result.toDataStreamResponse();
}
```

---

## 3. Rate Limiting Demo - Upstash

### Installation

```bash
pnpm add @upstash/redis @upstash/ratelimit
```

### Rate Limiter Configuration

**File**: `lib/ratelimit.ts`

```typescript
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

// Create a new ratelimiter for each use case
export const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10 s'),
  analytics: true,
  prefix: '@upstash/ratelimit',
});

export const apiRatelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(100, '1 m'),
  analytics: true,
  prefix: '@upstash/ratelimit:api',
});

export const tenantCreateRatelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.tokenBucket(5, '1 h', 20),
  analytics: true,
  prefix: '@upstash/ratelimit:tenant-create',
});
```

### API Route with Rate Limiting

**File**: `app/api/tenants/create/route.ts`

```typescript
import { auth } from '@clerk/nextjs/server';
import { NextResponse } from 'next/server';
import { tenantCreateRatelimit } from '@/lib/ratelimit';

export async function POST(req: Request) {
  const { userId } = await auth();

  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // Rate limit check
  const { success, limit, reset, remaining } = await tenantCreateRatelimit.limit(userId);

  if (!success) {
    return NextResponse.json(
      {
        error: 'Too many requests',
        limit,
        reset,
        remaining,
      },
      {
        status: 429,
        headers: {
          'X-RateLimit-Limit': limit.toString(),
          'X-RateLimit-Remaining': remaining.toString(),
          'X-RateLimit-Reset': reset.toString(),
        },
      }
    );
  }

  // Create tenant logic
  const tenant = await createTenant(await req.json());

  return NextResponse.json(
    { tenant },
    {
      headers: {
        'X-RateLimit-Limit': limit.toString(),
        'X-RateLimit-Remaining': remaining.toString(),
        'X-RateLimit-Reset': reset.toString(),
      },
    }
  );
}
```

---

## 4. Admin Dashboard Demo - TailAdmin V2

### Dashboard Layout

**File**: `app/admin/layout.tsx`

```typescript
import Link from 'next/link';
import {
  LayoutDashboard,
  Users,
  Building2,
  Settings,
  BarChart3,
  Bell,
  FileText,
  CreditCard
} from 'lucide-react';

export default function AdminLayout({ children }: { children: React.ReactNode }) {
  const navigation = [
    { name: 'Overview', href: '/admin', icon: LayoutDashboard },
    { name: 'Tenants', href: '/admin/tenants', icon: Building2 },
    { name: 'Users', href: '/admin/users', icon: Users },
    { name: 'Billing', href: '/admin/billing', icon: CreditCard },
    { name: 'Analytics', href: '/admin/analytics', icon: BarChart3 },
    { name: 'Notifications', href: '/admin/notifications', icon: Bell },
    { name: 'Logs', href: '/admin/logs', icon: FileText },
    { name: 'Settings', href: '/admin/settings', icon: Settings },
  ];

  return (
    <div className="min-h-screen bg-gray-50">
      {/* Sidebar */}
      <aside className="fixed inset-y-0 left-0 w-64 bg-white border-r border-gray-200">
        <div className="flex flex-col h-full">
          {/* Logo */}
          <div className="p-6 border-b border-gray-200">
            <h1 className="text-xl font-bold text-gray-900">Platform Admin</h1>
            <p className="text-sm text-gray-500 mt-1">Multi-Tenant Management</p>
          </div>

          {/* Navigation */}
          <nav className="flex-1 p-4 space-y-1 overflow-y-auto">
            {navigation.map((item) => (
              <Link
                key={item.name}
                href={item.href}
                className="flex items-center gap-3 px-4 py-3 text-gray-700 rounded-lg hover:bg-gray-100 transition-all group"
              >
                <item.icon className="w-5 h-5 text-gray-400 group-hover:text-gray-600" />
                <span className="font-medium">{item.name}</span>
              </Link>
            ))}
          </nav>

          {/* User Section */}
          <div className="p-4 border-t border-gray-200">
            <div className="flex items-center gap-3 px-4 py-3">
              <div className="w-10 h-10 bg-gradient-to-br from-purple-600 to-blue-600 rounded-full flex items-center justify-center text-white font-medium">
                A
              </div>
              <div className="flex-1 min-w-0">
                <p className="font-medium text-gray-900 truncate">Admin User</p>
                <p className="text-sm text-gray-500 truncate">admin@platform.com</p>
              </div>
            </div>
          </div>
        </div>
      </aside>

      {/* Main Content */}
      <main className="ml-64">
        {children}
      </main>
    </div>
  );
}
```

### Dashboard Overview Page

**File**: `app/admin/page.tsx`

```typescript
import {
  Building2,
  Users,
  TrendingUp,
  DollarSign,
  Activity,
  AlertCircle
} from 'lucide-react';

export default async function AdminDashboard() {
  // Fetch real data
  const stats = await getDashboardStats();
  const recentActivity = await getRecentActivity();
  const alerts = await getAlerts();

  return (
    <div className="p-8">
      {/* Header */}
      <div className="mb-8">
        <h1 className="text-3xl font-bold text-gray-900">Dashboard Overview</h1>
        <p className="text-gray-600 mt-1">Platform performance and metrics</p>
      </div>

      {/* Stats Grid */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
        <StatCard
          title="Total Tenants"
          value={stats.totalTenants}
          change={stats.tenantChange}
          icon={Building2}
          trend="up"
        />
        <StatCard
          title="Active Users"
          value={stats.activeUsers}
          change={stats.userChange}
          icon={Users}
          trend="up"
        />
        <StatCard
          title="Revenue (MTD)"
          value={`$${stats.revenue.toLocaleString()}`}
          change={stats.revenueChange}
          icon={DollarSign}
          trend="up"
        />
        <StatCard
          title="API Requests"
          value={stats.apiRequests.toLocaleString()}
          change={stats.apiChange}
          icon={Activity}
          trend="down"
        />
      </div>

      {/* Two Column Layout */}
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* Recent Activity */}
        <div className="lg:col-span-2 bg-white rounded-xl shadow-sm border border-gray-200">
          <div className="p-6 border-b border-gray-200">
            <h2 className="text-lg font-semibold text-gray-900">Recent Activity</h2>
          </div>
          <div className="p-6">
            <div className="space-y-4">
              {recentActivity.map((activity) => (
                <ActivityItem key={activity.id} activity={activity} />
              ))}
            </div>
          </div>
        </div>

        {/* Alerts */}
        <div className="bg-white rounded-xl shadow-sm border border-gray-200">
          <div className="p-6 border-b border-gray-200">
            <h2 className="text-lg font-semibold text-gray-900">Alerts</h2>
          </div>
          <div className="p-6">
            <div className="space-y-4">
              {alerts.map((alert) => (
                <AlertItem key={alert.id} alert={alert} />
              ))}
            </div>
          </div>
        </div>
      </div>

      {/* Charts Section */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 mt-6">
        <ChartCard title="Tenant Growth" type="line" />
        <ChartCard title="Revenue Distribution" type="bar" />
      </div>
    </div>
  );
}

// Components
function StatCard({
  title,
  value,
  change,
  icon: Icon,
  trend
}: {
  title: string;
  value: string | number;
  change: string;
  icon: any;
  trend: 'up' | 'down';
}) {
  return (
    <div className="bg-white rounded-xl shadow-sm border border-gray-200 p-6">
      <div className="flex items-start justify-between">
        <div>
          <p className="text-sm font-medium text-gray-600">{title}</p>
          <p className="text-3xl font-bold text-gray-900 mt-2">{value}</p>
          <p className={`text-sm mt-2 ${trend === 'up' ? 'text-green-600' : 'text-red-600'}`}>
            {trend === 'up' ? <TrendingUp className="inline w-4 h-4 mr-1" /> : null}
            {change} from last month
          </p>
        </div>
        <div className="p-3 bg-gray-100 rounded-lg">
          <Icon className="w-6 h-6 text-gray-600" />
        </div>
      </div>
    </div>
  );
}

function ActivityItem({ activity }: { activity: any }) {
  return (
    <div className="flex items-start gap-4 p-4 bg-gray-50 rounded-lg">
      <div className={`w-10 h-10 rounded-full flex items-center justify-center ${
        activity.type === 'create' ? 'bg-green-100 text-green-600' :
        activity.type === 'update' ? 'bg-blue-100 text-blue-600' :
        activity.type === 'delete' ? 'bg-red-100 text-red-600' :
        'bg-gray-100 text-gray-600'
      }`}>
        {activity.type === 'create' && <Users className="w-5 h-5" />}
        {activity.type === 'update' && <Activity className="w-5 h-5" />}
        {activity.type === 'delete' && <AlertCircle className="w-5 h-5" />}
      </div>
      <div className="flex-1 min-w-0">
        <p className="font-medium text-gray-900">{activity.title}</p>
        <p className="text-sm text-gray-600">{activity.description}</p>
      </div>
      <p className="text-sm text-gray-500">{activity.time}</p>
    </div>
  );
}

function AlertItem({ alert }: { alert: any }) {
  return (
    <div className={`p-4 rounded-lg border ${
      alert.severity === 'high' ? 'bg-red-50 border-red-200' :
      alert.severity === 'medium' ? 'bg-yellow-50 border-yellow-200' :
      'bg-blue-50 border-blue-200'
    }`}>
      <div className="flex items-start gap-3">
        <AlertCircle className={`w-5 h-5 ${
          alert.severity === 'high' ? 'text-red-600' :
          alert.severity === 'medium' ? 'text-yellow-600' :
          'text-blue-600'
        }`} />
        <div className="flex-1">
          <p className="font-medium text-gray-900">{alert.title}</p>
          <p className="text-sm text-gray-600 mt-1">{alert.message}</p>
        </div>
      </div>
    </div>
  );
}

function ChartCard({ title, type }: { title: string; type: 'line' | 'bar' }) {
  return (
    <div className="bg-white rounded-xl shadow-sm border border-gray-200 p-6">
      <h3 className="text-lg font-semibold text-gray-900 mb-4">{title}</h3>
      <div className="h-64 flex items-center justify-center bg-gray-50 rounded-lg">
        <p className="text-gray-500">Chart placeholder - Integrate Recharts or Chart.js</p>
      </div>
    </div>
  );
}
```

---

## 5. Billing Demo - Flexprice

### Installation

```bash
pnpm add @flexprice/node
```

### Usage Tracking Component

**File**: `app/components/usage-tracker.tsx`

```typescript
'use client';

import { useEffect, useState } from 'react';

export function UsageTracker({ tenantId }: { tenantId: string }) {
  const [usage, setUsage] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUsage();
  }, [tenantId]);

  async function fetchUsage() {
    const response = await fetch(`/api/billing/usage?tenantId=${tenantId}`);
    const data = await response.json();
    setUsage(data);
    setLoading(false);
  }

  if (loading) return <div>Loading usage data...</div>;

  return (
    <div className="bg-white rounded-xl shadow-sm border border-gray-200 p-6">
      <h3 className="text-lg font-semibold text-gray-900 mb-4">Usage Overview</h3>

      <div className="space-y-4">
        <UsageMetric
          name="API Requests"
          used={usage?.apiRequests.used}
          limit={usage?.apiRequests.limit}
          unit="requests"
        />
        <UsageMetric
          name="Storage"
          used={usage?.storage.used}
          limit={usage?.storage.limit}
          unit="GB"
        />
        <UsageMetric
          name="Bandwidth"
          used={usage?.bandwidth.used}
          limit={usage?.bandwidth.limit}
          unit="GB"
        />
      </div>

      <div className="mt-6 p-4 bg-gray-50 rounded-lg">
        <div className="flex justify-between items-center">
          <span className="font-medium text-gray-900">Current Bill</span>
          <span className="text-2xl font-bold text-gray-900">
            ${usage?.currentBill.toFixed(2)}
          </span>
        </div>
      </div>
    </div>
  );
}

function UsageMetric({
  name,
  used,
  limit,
  unit
}: {
  name: string;
  used: number;
  limit: number;
  unit: string;
}) {
  const percentage = (used / limit) * 100;

  return (
    <div>
      <div className="flex justify-between mb-2">
        <span className="text-sm font-medium text-gray-700">{name}</span>
        <span className="text-sm text-gray-600">
          {used.toLocaleString()} / {limit.toLocaleString()} {unit}
        </span>
      </div>
      <div className="w-full bg-gray-200 rounded-full h-2">
        <div
          className={`h-2 rounded-full transition-all ${
            percentage > 90 ? 'bg-red-500' :
            percentage > 70 ? 'bg-yellow-500' :
            'bg-green-500'
          }`}
          style={{ width: `${Math.min(percentage, 100)}%` }}
        />
      </div>
    </div>
  );
}
```

---

## 6. Monitoring Demo - Grafana

### Metrics Exporter

**File**: `lib/metrics.ts`

```typescript
import { Counter, Histogram, Registry } from 'prom-client';

// Create registry
export const register = new Registry();

// Define metrics
export const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register],
});

export const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register],
});

export const tenantActiveUsers = new Gauge({
  name: 'tenant_active_users',
  help: 'Number of active users per tenant',
  labelNames: ['tenant_id'],
  registers: [register],
});

// Middleware to track requests
export function metricsMiddleware(req: Request, res: Response) {
  const start = Date.now();

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const route = req.nextUrl.pathname;
    const method = req.method;
    const statusCode = res.status;

    httpRequestDuration.labels(method, route, statusCode.toString()).observe(duration);
    httpRequestsTotal.labels(method, route, statusCode.toString()).inc();
  });
}
```

### Metrics Endpoint

**File**: `app/api/metrics/route.ts`

```python
from prometheus_client import generate_latest, CONTENT_TYPE_LATEST
from fastapi import Response

@app.get("/api/metrics")
async def metrics():
    return Response(generate_latest(), media_type=CONTENT_TYPE_LATEST)
```

---

## 7. Workflow Demo - n8n

### Self-Hosted n8n Setup

**docker-compose.yml**

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=your_password
      - WEBHOOK_URL=https://your-domain.com
    volumes:
      - n8n_data:/home/node/.n8n
    restart: unless-stopped

volumes:
  n8n_data:
```

### Custom Workflow Node

**File**: `n8n-nodes/tenant-creator/tenant-creator.node.ts`

```typescript
import {
  IExecuteFunctions,
  INodeExecutionData,
  INodeProperties,
} from 'n8n-workflow';

export const tenantCreatorDescription: INodeProperties = {
  displayName: 'Tenant Creator',
  name: 'tenantCreator',
  icon: 'fa:building',
  group: ['transform'],
  version: 1,
  description: 'Create a new tenant in the multi-tenant platform',
  defaults: {
    name: 'Tenant Creator',
  },
  inputs: ['main'],
  outputs: ['main'],
  properties: [
    {
      displayName: 'Tenant Name',
      name: 'tenantName',
      type: 'string',
      required: true,
      default: '',
    },
    {
      displayName: 'Subdomain',
      name: 'subdomain',
      type: 'string',
      required: true,
      default: '',
    },
    {
      displayName: 'Plan',
      name: 'plan',
      type: 'options',
      options: [
        { name: 'Free', value: 'free' },
        { name: 'Pro', value: 'pro' },
        { name: 'Enterprise', value: 'enterprise' },
      ],
      default: 'free',
    },
  ],
};

export async function execute(
  this: IExecuteFunctions
): Promise<INodeExecutionData[][]> {
  const items = this.getInputData();
  const returnData: INodeExecutionData[] = [];

  for (let i = 0; i < items.length; i++) {
    const tenantName = this.getNodeParameter('tenantName', i) as string;
    const subdomain = this.getNodeParameter('subdomain', i) as string;
    const plan = this.getNodeParameter('plan', i) as string;

    // Create tenant via API
    const response = await fetch('https://api.your-platform.com/tenants', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.getCredentials('api').apiKey}`,
      },
      body: JSON.stringify({
        name: tenantName,
        subdomain,
        plan,
      }),
    });

    const tenant = await response.json();

    returnData.push({
      json: tenant,
    });
  }

  return [returnData];
}
```

---

## 8. Notifications Demo - Novu

### Installation

```bash
pnpm add @novu/node
```

### Notification Service

**File**: `lib/notifications.ts`

```typescript
import { Novu } from '@novu/node';

const novu = new Novu(process.env.NOVU_API_KEY!);

export async function sendNotification({
  tenantId,
  userId,
  type,
  data,
}: {
  tenantId: string;
  userId: string;
  type: 'welcome' | 'invoice' | 'alert' | 'update';
  data: Record<string, any>;
}) {
  try {
    await novu.trigger(type, {
      to: {
        subscriberId: `${tenantId}:${userId}`,
      },
      payload: {
        ...data,
        tenantName: await getTenantName(tenantId),
      },
    });
  } catch (error) {
    console.error('Failed to send notification:', error);
    throw error;
  }
}

// Example usage
export async function sendWelcomeEmail(userId: string, tenantId: string) {
  await sendNotification({
    tenantId,
    userId,
    type: 'welcome',
    data: {
      userName: await getUserName(userId),
      dashboardUrl: `https://${tenantId}.yourplatform.com`,
    },
  });
}

export async function sendInvoiceAlert(
  userId: string,
  tenantId: string,
  amount: number,
  dueDate: Date
) {
  await sendNotification({
    tenantId,
    userId,
    type: 'invoice',
    data: {
      amount,
      dueDate: dueDate.toISOString(),
      invoiceUrl: `https://${tenantId}.yourplatform.com/billing`,
    },
  });
}
```

### Notification Preferences Component

**File**: `app/components/notification-preferences.tsx`

```typescript
'use client';

import { useState } from 'react';

const channels = [
  { id: 'email', name: 'Email', icon: '✉️' },
  { id: 'sms', name: 'SMS', icon: '📱' },
  { id: 'push', name: 'Push', icon: '🔔' },
  { id: 'in_app', name: 'In-App', icon: '💬' },
];

const notificationTypes = [
  { id: 'welcome', name: 'Welcome Email' },
  { id: 'invoice', name: 'Invoice Reminders' },
  { id: 'alert', name: 'System Alerts' },
  { id: 'update', name: 'Product Updates' },
];

export function NotificationPreferences() {
  const [preferences, setPreferences] = useState({
    welcome: { email: true, sms: false, push: false, in_app: true },
    invoice: { email: true, sms: true, push: true, in_app: true },
    alert: { email: true, sms: true, push: true, in_app: true },
    update: { email: false, sms: false, push: false, in_app: true },
  });

  const toggleChannel = (type: string, channel: string) => {
    setPreferences({
      ...preferences,
      [type]: {
        ...preferences[type],
        [channel]: !preferences[type][channel],
      },
    });
  };

  return (
    <div className="bg-white rounded-xl shadow-sm border border-gray-200 p-6">
      <h3 className="text-lg font-semibold text-gray-900 mb-6">
        Notification Preferences
      </h3>

      <div className="overflow-x-auto">
        <table className="w-full">
          <thead>
            <tr className="border-b border-gray-200">
              <th className="text-left py-3 px-4 font-medium text-gray-700">
                Notification Type
              </th>
              {channels.map((channel) => (
                <th key={channel.id} className="text-center py-3 px-4 font-medium text-gray-700">
                  <span className="text-xl">{channel.icon}</span>
                  <span className="ml-2 text-sm">{channel.name}</span>
                </th>
              ))}
            </tr>
          </thead>
          <tbody>
            {notificationTypes.map((type) => (
              <tr key={type.id} className="border-b border-gray-100">
                <td className="py-3 px-4 font-medium text-gray-900">
                  {type.name}
                </td>
                {channels.map((channel) => (
                  <td key={channel.id} className="text-center py-3 px-4">
                    <label className="inline-flex items-center">
                      <input
                        type="checkbox"
                        checked={preferences[type.id][channel.id]}
                        onChange={() => toggleChannel(type.id, channel.id)}
                        className="w-5 h-5 text-purple-600 rounded focus:ring-purple-500"
                      />
                    </label>
                  </td>
                ))}
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

---

## 9. Analytics Demo - PostHog

### Installation

```bash
pnpm add posthog-js
```

### PostHog Provider

**File**: 'app/providers/posthog-provider.tsx'

```typescript
'use client';

import { PostHogProvider as PHProvider } from 'posthog-js/react';
import { usePathname, useSearchParams } from 'next/navigation';
import { useEffect } from 'react';
import posthog from 'posthog-js';

export function PostHogProvider({ children }: { children: React.ReactNode }) {
  const pathname = usePathname();
  const searchParams = useSearchParams();

  useEffect(() => {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      api_host: process.env.NEXT_PUBLIC_POSTHOG_HOST,
      capture_pageview: false,
      capture_pageleave: true,
    });
  }, []);

  useEffect(() => {
    if (pathname) {
      let url = pathname;
      if (searchParams.toString()) {
        url = `${url}?${searchParams.toString()}`;
      }
      posthog.capture('$pageview', { $current_url: url });
    }
  }, [pathname, searchParams]);

  return <PHProvider client={posthog}>{children}</PHProvider>;
}
```

### Custom Event Tracking

**File**: `lib/analytics.ts`

```typescript
import posthog from 'posthog-js';

export function trackTenantEvent(event: string, properties: Record<string, any>) {
  posthog.capture(event, {
    ...properties,
    timestamp: new Date().toISOString(),
  });
}

// Example events
export function trackTenantCreated(tenantId: string, plan: string) {
  trackTenantEvent('tenant_created', {
    tenantId,
    plan,
    source: 'admin_dashboard',
  });
}

export function trackTenantUpgraded(tenantId: string, fromPlan: string, toPlan: string) {
  trackTenantEvent('tenant_upgraded', {
    tenantId,
    fromPlan,
    toPlan,
    revenueImpact: calculateRevenueImpact(fromPlan, toPlan),
  });
}

export function trackFeatureUsed(tenantId: string, feature: string) {
  trackTenantEvent('feature_used', {
    tenantId,
    feature,
    timestamp: Date.now(),
  });
}

// Usage in components
export function useAnalytics() {
  return {
    trackEvent: trackTenantEvent,
    trackPageView: (page: string) => posthog.capture('$pageview', { page }),
    identify: (userId: string, properties: Record<string, any>) => {
      posthog.identify(userId, properties);
    },
    reset: () => posthog.reset(),
  };
}
```

---

## 10. Low-Code Builder Demo - Puck

### Installation

```bash
pnpm add @measured/puck
```

### Visual Editor Component

**File**: 'app/components/visual-builder.tsx`

```typescript
'use client';

import { Puck, Render } from '@measured/puck';
import '@measured/puck/puck.css';

const config = {
  components: {
    Heading: ({ heading }) => (
      <h1 className="text-3xl font-bold">{heading}</h1>
    ),
    Button: ({ label, variant }) => (
      <button className={`px-6 py-3 rounded-lg ${
        variant === 'primary'
          ? 'bg-purple-600 text-white'
          : 'bg-gray-200 text-gray-800'
      }`}>
        {label}
      </button>
    ),
    Card: ({ title, content }) => (
      <div className="bg-white rounded-lg shadow-md p-6 border border-gray-200">
        <h3 className="text-xl font-semibold mb-2">{title}</h3>
        <p className="text-gray-600">{content}</p>
      </div>
    ),
  },
};

const initialData = {
  content: [
    {
      type: 'Heading',
      props: {
        heading: 'Welcome to Your Dashboard',
      },
    },
    {
      type: 'Card',
      props: {
        title: 'Quick Stats',
        content: 'Your platform metrics at a glance',
      },
    },
  ],
  root: { props: { title: 'Dashboard' } },
};

export function VisualBuilder({ onSave }: { onSave: (data: any) => void }) {
  const [data, setData] = useState(initialData);
  const [isEditing, setIsEditing] = useState(true);

  return (
    <div className="h-screen flex">
      {/* Builder */}
      <div className="flex-1">
        {isEditing ? (
          <Puck
            config={config}
            data={data}
            onChange={setData}
            onPublish={onSave}
          />
        ) : (
          <div className="p-8">
            <Render config={config} data={data} />
          </div>
        )}
      </div>

      {/* Toggle Edit Mode */}
      <div className="fixed bottom-6 right-6">
        <button
          onClick={() => setIsEditing(!isEditing)}
          className="px-6 py-3 bg-purple-600 text-white rounded-lg shadow-lg hover:bg-purple-700 transition-all"
        >
          {isEditing ? 'Preview' : 'Edit'}
        </button>
      </div>
    </div>
  );
}
```

---

## Installation Quick Reference

```bash
# Core dependencies
pnpm add @clerk/nextjs ai @ai-sdk/openai @upstash/redis @upstash/ratelimit

# Analytics & monitoring
pnpm add posthog-js @opentelemetry/api @opentelemetry/sdk-node

# Billing (choose one)
pnpm add @flexprice/node
# OR
pnpm add @lago/node

# Notifications
pnpm add @novu/node

# UI components
pnpm add @measured/puck lucide-react
```

---

**Demo Guide compiled by Guardian Agent**
*All examples use TypeScript and modern React patterns*
