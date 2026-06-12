---
name: compliance-audit
description: "Audit a website, app, or product's codebase for GDPR and EU AI Act compliance, then hand back a prioritized list of plain-English fixes. Reads the actual code — forms, API routes, dependencies, env vars, database schema, policy pages — to work out what personal data the product really collects and who it's shared with, then checks that against what the law requires. The point: catch the gaps a pasted-in policy template hides. MANDATORY TRIGGERS: 'compliance audit', 'run compliance-audit', 'audit my compliance', 'audit this for GDPR', 'privacy audit', 'legal audit'. STRONG TRIGGERS (use when tied to this codebase): 'am I GDPR compliant', 'is my product/site legal', 'is my site breaking the law', 'check the EU AI Act', 'do I need a cookie banner', 'is my privacy policy ok'. Built for vibe-coded products that shipped before anyone read the regulations."
---

# Compliance Audit

Most vibe-coded products are quietly breaking the law. Not on purpose — the founder pasted in a privacy-policy template, wired up an AI API, dropped in Google Analytics, and shipped. But if anyone in the EU or UK can use the product, **GDPR applies**, and from **2 August 2026** the **EU AI Act**'s transparency rules apply too.

This skill audits the codebase the way a careful reviewer would: it reads what the product *actually* does, compares it to what the law requires, and comes back with a short, ranked list of things to fix — each one specific, evidence-backed, and paired with an offer to fix it for them.

It does **not** give legal advice. It gets someone most of the way there and tells them where to get a human if the stakes are high.

---

## The one rule: evidence, not assumptions

Every finding must be grounded in something actually read from the repo — a file, a dependency, a route, a form field, a line of the privacy policy. Cite it (`app/api/signup/route.ts:24`). Never invent a violation, and never wave something through without checking. If you can't find a privacy policy, say "no privacy policy found in the repo" — don't assume one exists elsewhere. If the product clearly handles no personal data at all, say that and score it low. Honesty over alarm; specificity over generic boilerplate.

---

## Step 1 — Scope it (10 seconds)

Glance at the repo and the user's intent, then make a fast call on two things. Only ask the user if you genuinely can't tell:

- **Does it touch people in the EU/UK?** Default to **yes** — the whole premise is that an open signup form means anyone can use it, so GDPR/UK GDPR is in scope. Only treat as out of scope if it's plainly internal/non-public with no real users.
- **What is it?** A web app, a marketing site, a SaaS, an API. This shapes what matters (a static brochure site has lighter obligations than an app with accounts and an AI chatbot).

Don't over-interview. Read the code; it answers most of this.

---

## Step 2 — Gather the evidence

Investigate the repo across these tracks. Use `Glob`/`Grep`/`Read` — read the real files, don't pattern-match from filenames alone.

**a. Who touches the data (third parties / sub-processors).** This is the most under-disclosed thing in vibe-coded products.
- Dependency manifests: `package.json`, `requirements.txt`, `Gemfile`, `go.mod`, `composer.json`. Flag SDKs for AI (`@anthropic-ai/sdk`, `openai`, `@google/generative-ai`), payments (`stripe`, `paddle`), email (`resend`, `@sendgrid/mail`, `nodemailer`, `postmark`), analytics (`@vercel/analytics`, `posthog`, `mixpanel`, `@segment`), error tracking (`@sentry`), auth (`next-auth`, `@clerk/`, `@auth0`, `@supabase`), databases/hosting (`@supabase`, `firebase`, `@planetscale`, `@neondatabase`), SMS (`twilio`), and any `googleapis`.
- Env vars: grep `.env*` and `process.env` / `os.environ` usage for `*_API_KEY`, `*_SECRET`, `*_TOKEN` — each key usually names a service that processes user data.
- Loaded scripts/domains in HTML/JSX: `googletagmanager`, `google-analytics`, `connect.facebook.net` (Meta Pixel), `hotjar`, `clarity.ms`, intercom, etc.

**b. What personal data it collects.** Personal data is anything that identifies a person — name, email, IP, device/cookie IDs, location, user-generated content, uploads, payment details, *and every message a user sends to an AI*.
- Forms and inputs: signup, contact, checkout, newsletter — read the `name`/field list.
- API route handlers (`app/api/**`, `pages/api/**`, server routes, controllers): what's in the request body, what gets persisted, what gets forwarded to a third party.
- Database layer: Prisma schema, SQL migrations, Supabase/Drizzle table defs, ORM models, validation schemas (zod/yup) — the columns are the ground truth of what's stored.
- Logging: grep for `console.log`/`logger` calls that dump request bodies, emails, or tokens.

**c. The privacy / legal surface.** Find pages or content named `privacy`, `terms`, `cookie`, `legal`, `gdpr`, `dpa` (routes, `.tsx`, `.md`, `.mdx`, CMS content). **Read the actual text.** Note what it claims to collect and which third parties it names — you'll diff this against (a) and (b).

**d. AI features.** Any LLM/ML call: imports of the SDKs above, calls to `/chat/completions`, `messages.create`, streaming chat UIs, "assistant"/"agent" components, AI-generated images/audio/video/text shown to users. Then check the UI: is the user told they're talking to / seeing output from an AI?

**e. Consent & cookies.** Look for a cookie/consent banner component and consent state. The key question: do analytics or marketing scripts load **before** the user opts in?

**f. Deletion & rights.** Search for an account-deletion path: a `DELETE` route, `deleteUser`/`deleteAccount` handler, a "delete my account / data" option in settings. Absence is itself a finding.

---

## Step 3 — Run the checks

Work through these. The first four are the essentials from which most products fail; the rest are the standard GDPR / EU AI Act surface. For each, decide ✅ pass / ⚠️ partial / ❌ fail and capture the evidence.

### The four essentials

1. **Does the privacy policy match what you actually collect?**
   Diff the policy text (Step 2c) against the real data and processors (2a, 2b). A template that says "we may collect your name and email" while the app stores phone numbers, uploads, IP logs and chat history — or never mentions the AI provider — is a transparency failure. *Law: GDPR Art. 13 (information to be provided). No policy at all = ❌ critical.*

2. **Is there a real way for users to delete their data?**
   A user-triggered deletion path that actually removes their data. "I'd do it manually if someone emailed" does not satisfy the right to erasure. *Law: GDPR Art. 17 (right to erasure).*

3. **Are you telling users when they're dealing with AI?**
   If there's a chatbot, AI agent, or AI-generated content, the user must be told (unless it's obvious). *Law: EU AI Act Art. 50 transparency obligations — apply from 2 August 2026. AI-generated media (deepfakes/synthetic content) must also be labelled as artificially generated.*

4. **Is every third party that touches user data named?**
   Each processor from Step 2a should be disclosed in the policy — especially the AI provider, which processes every message a user sends. *Law: GDPR Art. 13(1)(e) (recipients) & Art. 28 (processors).*

### The rest of the GDPR surface

5. **Lawful basis & marketing consent** — Is there a basis for what's collected? Marketing emails / newsletter signups need opt-in consent (no pre-ticked boxes, no bundling). *Art. 6, Art. 7.*
6. **Cookie/tracking consent** — Non-essential cookies (analytics, ad pixels, session replay) need **prior** opt-in consent via a banner; essential cookies don't. If GA/Meta Pixel fire on page load, that's a ❌. *PECR / ePrivacy.*
7. **Data subject rights & contact** — Beyond deletion: can users access/export/correct their data, and is there a real contact route (email/DPO) for requests? *Art. 15–22.*
8. **Data retention** — Does the policy state how long data is kept, and does the code suggest anything is kept forever with no reason? *Art. 5(1)(e).*
9. **International transfers** — Most processors (AI, analytics, email, hosting) are US-based. Transfers outside the EU/UK need a safeguard (adequacy / SCCs) and a mention in the policy. *Art. 44–49.*
10. **Security basics (the GDPR-visible ones)** — Secrets committed to the repo (is `.env` gitignored? hardcoded keys in source?), plaintext password storage, PII in logs, data endpoints with no auth. Not a full pentest — surface the obvious. *Art. 32.*
11. **Children** — If the audience could include under-16s, is there age handling / parental consent? *Art. 8.*

### The EU AI Act tier (quick read)

Most LLM-API products are **limited risk** → the duty is mainly the **transparency** of check 3. Flag louder only if you spot a **high-risk** use (Annex III: hiring/CV screening, credit scoring, biometric ID, education grading, essential-services eligibility) or a **prohibited** one (social scoring, manipulative or emotion-recognition systems) — those carry far heavier obligations and warrant "get a specialist."

---

## Step 4 — Rank by severity

- 🔴 **Critical** — likely unlawful *and* high-exposure right now: no privacy policy; tracking with no consent; processors (esp. the AI provider) undisclosed; no deletion path; secrets/PII leaking.
- 🟠 **Medium** — real gaps to close: policy out of date or missing required items; no retention or transfer language; missing AI disclosure ahead of the August deadline; weak/no marketing consent.
- 🟡 **Low** — best-practice polish: clearer wording, data minimisation, DPO contact, granular cookie controls.

Rank by *legal risk × likelihood it bites*, not by how easy it is to fix.

---

## Step 5 — Deliver the report

Output directly in chat as markdown — clean, scannable, ranked. Don't write files unless asked. Use this shape:

```
# Compliance Audit — {product}

> ⚠️ **Indicative guidance, not legal advice.** This is an automated read of your
> code, not a lawyer. For anything high-stakes, get a professional. Built to get
> you most of the way there.

## Risk snapshot
**Overall: 🔴 High risk** · 3 critical · 2 medium · 1 low
One sentence in plain English on the headline problem.

## The four essentials
| # | Check | Status |
|---|-------|--------|
| 1 | Privacy policy matches what you collect | ❌ |
| 2 | Real way to delete user data           | ❌ |
| 3 | Users told they're dealing with AI      | ⚠️ |
| 4 | Every third party named                 | ❌ |

## Findings

### 🔴 Critical
**No privacy policy in the repo** — anyone in the EU can sign up, so GDPR Art. 13
requires you to tell them what you collect and why. Nothing found under any
`privacy`/`legal` route.
→ Fix: publish a policy that matches your actual data (I can generate one).

**The AI provider isn't disclosed** — every message users type in the chat at
`app/api/chat/route.ts:18` is sent to Anthropic, but no policy names them.
→ Fix: add Anthropic (and your other processors) to a "who we share data with" list.

### 🟠 Medium
... (same shape: what · where · law · fix)

### 🟡 Low
...

## Do these first
1. ...
2. ...
3. ...

## Want me to fix them?
I can, right now in this repo:
- Generate a **privacy policy** that matches what your code actually collects
- Scaffold a real **delete-my-account** flow (route + UI hook)
- Add an **"you're chatting with AI"** disclosure to the chat UI
- Add a **consent-gated cookie banner** so analytics only fire after opt-in
- Produce a **sub-processor list** of every third party touching user data
Tell me which and I'll do it.
```

Rules for the report:
- Lead with the four essentials — they're what the audience came for.
- Every finding names **what's wrong, where (file ref), the law, and the fix**. No vague "you should review your data practices."
- Keep the AI-provider point sharp: *your AI provider processes every message — your users should know.*
- End by offering to implement the fixes. The value is in the doing, not just the list.

---

## Step 6 — Fix on request

If they say yes, implement against what the code actually does — never a generic template:
- **Privacy policy** — generate from the audited reality: the exact data collected, each named processor, transfers, retention, and how to exercise rights. Match the site's stack/format (a `/privacy` page or a markdown doc).
- **Deletion flow** — a real endpoint + a settings hook that deletes the user's rows and tells downstream processors to delete where it can.
- **AI disclosure** — a clear, unobtrusive notice in the AI surface ("Responses are AI-generated").
- **Cookie consent** — a banner that blocks non-essential scripts until opt-in, wired to the analytics it found.
- **Sub-processor list** — a maintained list/table of every third party and what it processes.

Then re-state which findings are now closed.

---

## Important notes

- **Not legal advice.** Say it once, clearly, at the top of the report. This gets someone most of the way; it doesn't replace a lawyer for high-stakes calls.
- **GDPR ≈ UK GDPR.** The audit covers both; the obligations are near-identical.
- **The August date is real and close.** EU AI Act Art. 50 transparency applies from 2 August 2026 — lead the AI-disclosure finding with that if it's still upcoming.
- **Tailor to the stack.** A Next.js app, a Rails API, and a static site each hide their data flows in different places — adapt Step 2 to what you actually find.
- **Don't pad the report.** A product with three real problems should get three findings, not a checklist of twenty maybes. Signal over noise.
