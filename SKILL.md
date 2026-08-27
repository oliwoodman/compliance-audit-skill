---
name: compliance-audit
description: "Audit a website, app, or product's codebase for GDPR and EU AI Act compliance, then hand back a ranked list of plain-English fixes. Asks the owner the handful of questions the code cannot answer, reads the actual code (forms, API routes, dependencies, env vars, database schema, policy pages) to work out what personal data the product really collects and who it is shared with, then checks that against what the law requires. It exists to catch the gaps a pasted-in policy template hides. MANDATORY TRIGGERS: 'compliance audit', 'run compliance-audit', 'audit my compliance', 'audit this for GDPR', 'privacy audit', 'legal audit'. STRONG TRIGGERS (use when tied to this codebase): 'am I GDPR compliant', 'is my product/site legal', 'is my site breaking the law', 'check the EU AI Act', 'do I need a cookie banner', 'is my privacy policy ok'. Built for vibe-coded products that shipped before anyone read the regulations."
---

# Compliance Audit

Most vibe-coded products are quietly breaking the law. Not on purpose. The founder pasted in a privacy-policy template, wired up an AI API, dropped in Google Analytics, and shipped. But if anyone in the EU or UK can use the product, **GDPR applies**, and since **2 August 2026** the **EU AI Act**'s transparency rules apply too.

A note on the AI Act delay, because the person you are helping may have heard about it. The 2026 amendment moved the **high-risk** obligations (hiring tools, credit scoring, medical, education, essential services) to 2 December 2027 and 2 August 2028. It did not move the duty to tell people they are talking to an AI. That has applied since 2 August 2026 and it applies to a side project.

This skill works the way a careful reviewer would. It asks the owner the few things the code cannot tell it, reads what the product *actually* does, compares both to what the law requires, and comes back with a short, ranked list of things to fix. Each one is specific, evidence-backed, and paired with an offer to fix it.

It does **not** give legal advice. It gets someone most of the way there and tells them where to get a human if the stakes are high.

---

## The one rule, evidence over assumptions

Every finding must be grounded in something actually read from the repo or actually said by the owner. Cite it (`app/api/signup/route.ts:24`, or "you said in the interview that..."). Never invent a violation, and never wave something through without checking. If you cannot find a privacy policy, say "no privacy policy found in the repo" rather than assuming one exists elsewhere. If the product clearly handles no personal data at all, say that and score it low. Honesty over alarm, specificity over boilerplate.

---

## Step 1, the interview (before you read anything)

The code tells you what the product does. It cannot tell you who is behind it, who it is for, or what the owner intends, and a privacy policy written without those is a template with the blanks filled in by guesswork. So before reading a file, ask the questions below **in one message**, numbered, and say plainly that "don't know" and "not yet" are fine answers. Do not drip them out one at a time, and do not skip this step because the repo looks simple. The answers are half the evidence.

1. **What is it, in a sentence, and who uses it?** A side project for a few mates, a paying product, an internal tool.
2. **Is it live, and can anyone sign up?** Roughly how many users, and do you know whether any are in the EU or the UK?
3. **Who is legally behind it?** A person or a company, the name, the country, and the email address people should use for privacy questions. This goes on the policy, and a policy with no contact route fails on its own.
4. **What do people give you or make inside it?** Accounts, messages to an AI, uploads, payments, location, and anything sensitive such as health, money or children.
5. **Which outside services does it use?** The AI provider, hosting, database, payments, email, analytics. Name what you know and the code will confirm the rest.
6. **Does any person, you included, ever read users' data or their AI conversations?** For support, out of curiosity, to improve the prompts.
7. **How long do you keep data, and what happens when someone leaves?** "Forever, I never thought about it" is a useful answer.
8. **Could anyone under 16 be using it?**
9. **Does it make or influence decisions about people?** Hiring, lending, grading, eligibility, medical. Or is it a chatbot, a helper, a content tool?
10. **What do you already have?** A privacy policy, terms, a cookie banner, a delete-account button. Did you write them or paste a template?

Treat the answers as the owner's **intent**. The code is the **reality**. The report is the difference between the two. If they answer only some, carry on with the rest and state in the report which defaults you used (the main one being that an open signup form means EU and UK users are in scope).

---

## Step 2, gather the evidence

Investigate the repo across these tracks. Use `Glob`, `Grep` and `Read`. Read the real files rather than pattern-matching from filenames.

**a. Who touches the data (third parties and sub-processors).** The most under-disclosed thing in vibe-coded products.
- Dependency manifests, `package.json`, `requirements.txt`, `Gemfile`, `go.mod`, `composer.json`. Flag SDKs for AI (`@anthropic-ai/sdk`, `openai`, `@google/generative-ai`), payments (`stripe`, `paddle`), email (`resend`, `@sendgrid/mail`, `nodemailer`, `postmark`), analytics (`@vercel/analytics`, `posthog`, `mixpanel`, `@segment`), error tracking (`@sentry`), auth (`next-auth`, `@clerk/`, `@auth0`, `@supabase`), databases and hosting (`@supabase`, `firebase`, `@planetscale`, `@neondatabase`), SMS (`twilio`), and any `googleapis`.
- Env vars. Grep `.env*` and `process.env` or `os.environ` usage for `*_API_KEY`, `*_SECRET`, `*_TOKEN`. Each key usually names a service that processes user data.
- Loaded scripts and domains in HTML or JSX, `googletagmanager`, `google-analytics`, `connect.facebook.net` (Meta Pixel), `hotjar`, `clarity.ms`, intercom, and so on.

**b. What personal data it collects.** Personal data is anything that identifies a person. Name, email, IP, device and cookie IDs, location, user-generated content, uploads, payment details, *and every message a user sends to an AI*.
- Forms and inputs. Signup, contact, checkout, newsletter. Read the field list.
- API route handlers (`app/api/**`, `pages/api/**`, server routes, controllers). What is in the request body, what gets persisted, what gets forwarded to a third party.
- Database layer. Prisma schema, SQL migrations, Supabase or Drizzle table definitions, ORM models, validation schemas (zod, yup). The columns are the ground truth of what is stored.
- Logging. Grep for `console.log` and `logger` calls that dump request bodies, emails, or tokens.

**c. The privacy and legal surface.** Find pages or content named `privacy`, `terms`, `cookie`, `legal`, `gdpr`, `dpa` (routes, `.tsx`, `.md`, `.mdx`, CMS content). **Read the actual text.** Note what it claims to collect and which third parties it names, because you will diff that against (a) and (b).

**d. AI features.** Any LLM or ML call. Imports of the SDKs above, calls to `/chat/completions` or `messages.create`, streaming chat UIs, "assistant" or "agent" components, AI-generated images, audio, video or text shown to users. Then check the UI. Is the user told they are talking to, or looking at the output of, an AI?

**e. Consent and cookies.** Look for a cookie or consent banner component and consent state. The key question is whether analytics or marketing scripts load **before** the user opts in.

**f. Deletion and rights.** Search for an account-deletion path. A `DELETE` route, a `deleteUser` or `deleteAccount` handler, a "delete my account" option in settings. Absence is itself a finding.

**g. The second round of questions.** Once you have read the code, compare it with the interview. Ask about contradictions and gaps only, in one short message. "The code sends every message to OpenAI and you did not list them, is that right?" "There is a `phone` column in the users table you did not mention, is it still used?" Do not re-ask anything already answered.

---

## Step 3, run the checks

Work through these. The first four are the essentials most products fail on; the rest are the standard GDPR and EU AI Act surface. For each, decide **Pass**, **Partial** or **Fail**, and capture the evidence.

### The four essentials

1. **Does the privacy policy match what you actually collect?**
   Diff the policy text (Step 2c) against the real data and processors (2a, 2b) and against the interview. A template that says "we may collect your name and email" while the app stores phone numbers, uploads, IP logs and chat history, or never mentions the AI provider, is a transparency failure. The law is GDPR Article 13, the information that must be provided. No policy at all is a critical fail.

2. **Is there a real way for users to delete their data?**
   A user-triggered deletion path that actually removes their data. "I'd do it manually if someone emailed" does not satisfy the right to erasure. The law is GDPR Article 17.

3. **Are you telling users when they are dealing with AI?**
   If there is a chatbot, an AI agent, or AI-generated content, the user must be told, unless it is obvious from context. The law is EU AI Act Article 50. The duty to disclose an AI interaction (50(1)) has applied since 2 August 2026. The duty to mark AI-generated content (50(2)) also applies, with one allowance, a system that was already on the market before 2 August 2026 has until 2 December 2026 to comply with the marking rule.

4. **Is every third party that touches user data named?**
   Each processor from Step 2a should be disclosed in the policy, above all the AI provider, which processes every message a user sends. The law is GDPR Article 13(1)(e), recipients, and Article 28, processors.

### The rest of the GDPR surface

5. **Lawful basis and marketing consent.** Is there a basis for what is collected? Marketing emails and newsletter signups need opt-in consent, no pre-ticked boxes, no bundling. Articles 6 and 7.
6. **Cookie and tracking consent.** Non-essential cookies (analytics, ad pixels, session replay) need **prior** opt-in consent through a banner; essential cookies do not. If GA or the Meta Pixel fire on page load, that is a fail. PECR and ePrivacy.
7. **Data subject rights and contact.** Beyond deletion, can users access, export and correct their data, and is there a real contact route (an email or a DPO) for requests? Articles 15 to 22.
8. **Data retention.** Does the policy state how long data is kept, and does the code, or the interview, suggest anything is kept forever for no reason? Article 5(1)(e).
9. **International transfers.** Most processors (AI, analytics, email, hosting) are US-based. Transfers outside the EU and UK need a safeguard (adequacy or SCCs) and a mention in the policy. Articles 44 to 49.
10. **Security basics, the GDPR-visible ones.** Secrets committed to the repo (is `.env` gitignored, are keys hardcoded in source), plaintext password storage, personal data in logs, data endpoints with no auth. Not a full pentest, surface the obvious. Article 32.
11. **Children.** If the audience could include under-16s (interview question 8), is there age handling or parental consent? Article 8.
12. **Humans reading data.** If the owner said in the interview that people read users' data or AI conversations, that use has to be described in the policy and covered by a basis. It usually is not.

### The EU AI Act tier (quick read)

Most LLM-API products are **limited risk**, so the duty is mainly the transparency of check 3. Flag louder only if you spot a **high-risk** use (Annex III, hiring and CV screening, credit scoring, biometric ID, education grading, essential-services eligibility) or a **prohibited** one (social scoring, manipulative or emotion-recognition systems). High-risk obligations now apply from 2 December 2027 (Annex III) and 2 August 2028 (Annex I), which is time to prepare rather than a reason to ignore them, and both tiers warrant "get a specialist".

---

## Step 4, rank by severity

- **Critical.** Likely unlawful *and* high-exposure right now. No privacy policy; tracking with no consent; processors, above all the AI provider, undisclosed; no deletion path; secrets or personal data leaking.
- **Medium.** Real gaps to close. Policy out of date or missing required items; no retention or transfer language; no AI disclosure; weak or absent marketing consent.
- **Low.** Best-practice polish. Clearer wording, data minimisation, DPO contact, granular cookie controls.

Rank by legal risk multiplied by the likelihood it bites, not by how easy it is to fix.

---

## Step 5, deliver the report

Output directly in chat as markdown, clean, scannable, ranked. Do not write files unless asked. Use this shape.

```
# Compliance Audit, {product}

> **Indicative guidance, not legal advice.** This is an automated read of your
> code and your answers, not a lawyer. For anything high-stakes, get a
> professional. Built to get you most of the way there.

## Risk snapshot
**Overall, high risk.** 3 critical, 2 medium, 1 low.
One sentence in plain English on the headline problem.

## What you told me, and what the code says
Two or three lines where the interview and the code disagree, because those
are the findings that matter most.

## The four essentials
| # | Check | Status |
|---|-------|--------|
| 1 | Privacy policy matches what you collect | Fail |
| 2 | Real way to delete user data | Fail |
| 3 | Users told they are dealing with AI | Partial |
| 4 | Every third party named | Fail |

## Findings

### Critical
**No privacy policy in the repo.** Anyone in the EU can sign up, so GDPR
Article 13 requires you to tell them what you collect and why. Nothing found
under any `privacy` or `legal` route.
The fix is to publish a policy that matches your actual data, and I can
generate one.

**The AI provider is not disclosed.** Every message users type in the chat at
`app/api/chat/route.ts:18` is sent to Anthropic, but no policy names them.
The fix is to add Anthropic, and your other processors, to a "who we share
data with" list.

### Medium
(same shape, what, where, the law, the fix)

### Low
...

## Do these first
1. ...
2. ...
3. ...

## Want me to fix them?
I can, right now in this repo.
- Generate a **privacy policy** that matches what your code actually collects
- Scaffold a real **delete-my-account** flow (route plus UI hook)
- Add a **"you are chatting with AI"** disclosure to the chat UI
- Add a **consent-gated cookie banner** so analytics only fire after opt-in
- Produce a **sub-processor list** of every third party touching user data
Tell me which and I will do it.
```

Rules for the report.
- Lead with the four essentials. They are what the audience came for.
- Every finding names **what is wrong, where (a file reference or an interview answer), the law, and the fix**. No vague "you should review your data practices".
- Keep the AI-provider point sharp. *Your AI provider processes every message, and your users should know.*
- End by offering to implement the fixes. The value is in the doing, not the list.

---

## Step 6, fix on request

If they say yes, implement against what the code actually does and what they told you. Never a generic template.

**Before writing a privacy policy, check you have everything it needs.** Most of it came from the interview. Anything still missing, ask for in one message before you write a word. The list is the legal name and address of whoever is responsible, the contact email for privacy requests, each category of data and why it is collected, every processor by name and what it does, where the data is hosted and whether anything leaves the EU and UK, how long each kind of data is kept, whether any person reads it, and whether under-16s are handled. A policy that guesses at any of these is the template problem all over again.

- **Privacy policy.** Generate from the audited reality, the exact data collected, each named processor, transfers, retention, and how to exercise rights. Match the site's stack and format (a `/privacy` page or a markdown doc).
- **Deletion flow.** A real endpoint plus a settings hook that deletes the user's rows and tells downstream processors to delete where it can.
- **AI disclosure.** A clear, unobtrusive notice in the AI surface ("Responses are AI-generated").
- **Cookie consent.** A banner that blocks non-essential scripts until opt-in, wired to the analytics it found.
- **Sub-processor list.** A maintained list or table of every third party and what it processes.

Then re-state which findings are now closed.

---

## Important notes

- **Not legal advice.** Say it once, clearly, at the top of the report. This gets someone most of the way and does not replace a lawyer for high-stakes calls.
- **GDPR and UK GDPR.** The audit covers both, and the obligations are near-identical.
- **The AI Act dates.** Article 50 transparency has applied since 2 August 2026. The 2026 amendment delayed the high-risk tier only, to 2 December 2027 and 2 August 2028, and gave products already on the market until 2 December 2026 for content marking. If the owner says "the AI Act was delayed", explain the split rather than agreeing.
- **Tailor to the stack.** A Next.js app, a Rails API, and a static site each hide their data flows in different places. Adapt Step 2 to what you find.
- **Do not pad the report.** A product with three real problems should get three findings, not a checklist of twenty maybes. Signal over noise.
