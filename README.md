# Compliance Audit

A Claude skill that audits your website, app, or product's codebase for **GDPR** and the **EU AI Act** — then hands back a ranked, plain-English list of exactly what to fix.

Works in **Claude Code** and **Claude on the web** (claude.ai).

> ⚠️ **Indicative guidance, not legal advice.** This is an automated read of your code, not a lawyer — it won't replace one for anything high-stakes. What it does is catch the obvious, expensive gaps fast. Inspired by [Flagged](https://flagged.org.uk/), a compliance scanner for UK and EU websites.

---

## What's a skill?

A skill is a small set of instructions you hand to Claude. Think of it as a job description for one specific task. You install it once, then trigger it with a phrase, and Claude knows exactly how to handle that job from then on.

This skill teaches Claude how to audit a codebase for compliance. Point it at your repo and it reads what your product actually does with personal data, checks that against what the law requires, and tells you where the gaps are.

---

## What it does

Most vibe-coded products are quietly breaking the law. Not on purpose — you pasted in a privacy-policy template, wired up an AI API, dropped in analytics, and shipped. But if anyone in the EU or UK can use it, GDPR applies, and from **2 August 2026** the EU AI Act's transparency rules do too.

This skill reads your actual code — forms, API routes, dependencies, env vars, database schema, policy pages — and checks it against what the law requires. Then it returns a ranked report where every finding names **what's wrong, where it is in your code, the law behind it, and the fix** — and offers to make the fixes for you.

The four things it leads with — the ones nearly every product fails:

- **A privacy policy that matches what you actually collect** — not the template you pasted in.
- **A real way for users to delete their data** — "I'd do it manually" doesn't count.
- **Telling users when they're dealing with an AI** — from 2 August 2026, that's the law.
- **Naming every third party that touches their data** — your AI provider reads every message.

It checks plenty more underneath: lawful basis and consent, cookie banners, data retention, international transfers, and the security basics that GDPR cares about.

---

## When to use it

Run it if:

- You shipped a product and pasted in a privacy-policy template.
- Anyone in the EU or UK can sign up — so GDPR already applies to you.
- You wired up an AI API and never added a notice that it's AI.
- You've got Google Analytics or a Meta Pixel firing with no consent banner.
- You genuinely don't know which third parties end up with your users' data.

---

## How to install it (no terminal needed)

Pick whichever option feels easier. Both work in Claude Code and Claude on the web.

### Option 1: Let Claude install it for you

Open a new chat in Claude and paste this in:

> Please install this Claude skill for me. The SKILL.md file lives in this GitHub repo: https://github.com/oliwoodman/compliance-audit-skill
>
> Set it up so I can start using it, then offer to run a compliance audit on my repo straight away.

Claude will grab the file and drop it in the right place. If your setup needs a manual step, it'll tell you exactly what to click.

### Option 2: Download the file and ask Claude to set it up

1. Click [SKILL.md](./SKILL.md) at the top of this repo.
2. Use the download button on the right of the file view to save it to your computer.
3. Open Claude and paste this in:

> I just downloaded a file called SKILL.md for the Compliance Audit skill. Can you install it for me and walk me through where to put it?

Claude takes it from there.

---

## Running it

Once it's installed, run it in any project by saying:

> run compliance-audit

Claude reads the repo, works through the GDPR and EU AI Act checklist, and gives you a ranked list of what to fix — then offers to fix it.
