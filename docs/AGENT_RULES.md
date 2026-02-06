# Agent Rules

## Purpose

This document defines strict boundaries for AI agents working on this project. These rules are **non-negotiable** and designed to prevent security violations, data loss, and workflow disruption.

---

## 🔒 CRITICAL RULES (Never Violate)

### 1. Token & Credential Rules

| Rule | Details |
|------|---------|
| ❌ **NEVER** use exposed tokens | Once a token is exposed, it must be revoked. Never reuse it. |
| ❌ **NEVER** put tokens in URLs | No `https://user:TOKEN@github.com/` patterns |
| ❌ **NEVER** run curl with tokens | No `curl -H "Authorization: Bearer TOKEN"` |
| ❌ **NEVER** modify `.git/config` | Do not add or remove credentials from git config |
| ❌ **NEVER** log tokens | Do not output tokens in any command output or log |

**Sources:**
- GitHub Security: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure
- OWASP A01:2021: https://owasp.org/Top10/A01_2021-Broken_Access_Control/

---

### 2. Git Rules

| Rule | Details |
|------|---------|
| ❌ **NEVER** delete branches without approval | Ask first before any branch deletion |
| ❌ **NEVER** force push to main | Protected branch - never force push |
| ❌ **NEVER** rewrite public history | No `git rebase`, `git filter-branch` on shared branches |
| ❌ **NEVER** push without confirmation | Always show what will be pushed first |
| ❌ **NEVER** modify git remote | Do not change origin URLs |

**Sources:**
- Git Best Practices: https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes
- Vercel Git Integration: https://vercel.com/docs/projects/git

---

### 3. Code Modification Rules

| Rule | Details |
|------|---------|
| ❌ **NEVER** modify security code without Step approval | Middleware, auth, CSP must be approved per step |
| ❌ **NEVER** modify CI/CD without Step approval | Workflow changes require explicit approval |
| ❌ **NEVER** edit .env files | Never create or modify .env.local |
| ❌ **NEVER** commit secrets | No API keys, tokens, or secrets in commits |

**Sources:**
- Next.js Security: https://nextjs.org/docs/app/building-your-application/configuring/environment-variables
- Vercel Security: https://vercel.com/docs/security

---

## 📋 Step-by-Step Requirements

### Before ANY Code Change

1. **STOP** - Read the current step in the staged plan
2. **READ** - Review all related documentation
3. **CHECK** - Verify no conflicts with existing code
4. **ASK** - Get explicit user approval before proceeding

### During Implementation

1. Create feature branch: `git checkout -b step-X-feature-name`
2. Make minimal, focused changes only
3. Test locally: `pnpm run build && pnpm run dev`
4. Show user what changed: `git diff`
5. Wait for approval before committing

### After Implementation

1. Commit with clear message: `git commit -m "step X: description"`
2. Ask before pushing: `git push` (wait for approval)
3. Create PR if required (never merge to main directly)

---

## 🚫 Forbidden Operations

These operations are **NEVER** allowed without explicit user approval:

| Operation | Forbidden Because |
|-----------|------------------|
| `git reset --hard` | Can lose work |
| `git rebase` | Rewrites history |
| `git push --force` | Destructive |
| `rm -rf node_modules` | Breaks dependencies |
| `curl` with tokens | Security violation |
| Editing `.git/config` | Breaks git state |
| Modifying `middleware.ts` | Security-critical |
| Modifying CI workflows | Breaks deployment |

---

## ✅ Required Pre-Flight Checklist

Before any action, agent must verify:

- [ ] I have read the current step requirements
- [ ] I have user approval to proceed
- [ ] I understand what files will change
- [ ] I have a rollback plan
- [ ] I am NOT using any exposed tokens
- [ ] I am NOT modifying security code without approval

---

## 🔄 Recovery From Mistakes

If an agent violates these rules:

1. **STOP** immediately
2. **REPORT** what happened
3. **RESET** to last known good state: `git reset --hard <good-commit>`
4. **ROTATE** any exposed credentials
5. **DOCUMENT** what went wrong

---

## 📖 Source References

These rules are based on:

- **Vercel Security Best Practices**: https://vercel.com/docs/security
- **Next.js Production Checklist**: https://nextjs.org/docs/app/building-your-application/deploying
- **OWASP Top 10**: https://owasp.org/www-project-top-ten/
- **GitHub Security Guidelines**: https://docs.github.com/en/security
- **SLSA Supply Chain Security**: https://slsa.dev

---

*Last Updated: 2026-02-05*
*Version: 1.0*
