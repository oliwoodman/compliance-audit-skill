# Compliance Audit

A Claude skill that audits your website, app, or product's codebase for **GDPR** and the **EU AI Act**, then hands back a ranked, plain-English list of exactly what to fix.

Works in **Claude Code** and **Claude on the web** (claude.ai).

> **Indicative guidance, not legal advice.** This is an automated read of your code, not a lawyer, and it will not replace one for anything high-stakes. What it does is catch the obvious, expensive gaps fast. Inspired by [Flagged](https://flagged.org.uk/), a compliance scanner for UK and EU websites.

---

## What's a skill?

A skill is a small set of instructions you hand to Claude. Think of it as a job description for one specific task. You install it once, then trigger it with a phrase, and Claude knows exactly how to handle that job from then on.

This skill teaches Claude how to audit a codebase for compliance. Point it at your repo and it asks you the few things the code cannot tell it, reads what your product actually does with personal data, checks both against what the law requires, and tells you where the gaps are.

---

## What it does

Most vibe-coded products are quietly breaking the law. Not on purpose. You pasted in a privacy-policy template, wired up an AI API, dropped in analytics, and shipped. But if anyone in the EU or UK can use it, GDPR applies, and since **2 August 2026** the EU AI Act's transparency rules do too.

If you heard the AI Act was delayed, that is half true. The 2026 amendment pushed the rules for high-risk systems (hiring, credit, medical) to December 2027 and August 2028. The duty to tell people they are talking to an AI was not delayed, and it applies to a side project.

The skill starts with ten questions, asked in one go, about who is behind the product, who uses it, what people give you, which outside services it relies on, whether anyone reads users' data, and how long you keep it. "Don't know" is a fine answer. Then it reads your actual code, forms, API routes, dependencies, env vars, database schema and policy pages, and compares the two. The report is the difference between what you think the product does and what it actually does, ranked, with every finding naming **what is wrong, where it is in your code, the law behind it, and the fix**. Then it offers to make the fixes for you, and it asks for anything it still needs (the legal name, the contact email, retention periods) before it writes a policy, so what it writes is yours rather than a template.

The four things it leads with, the ones nearly every product fails.

- **A privacy policy that matches what you actually collect**, not the template you pasted in.
- **A real way for users to delete their data.** "I'd do it manually" does not count.
- **Telling users when they are dealing with an AI.** Since 2 August 2026, that is the law.
- **Naming every third party that touches their data.** Your AI provider reads every message.

It checks plenty more underneath. Lawful basis and consent, cookie banners, data retention, international transfers, whether any human reads users' data, and the security basics that GDPR cares about.

---

## When to use it

Run it if any of these are true.

- You shipped a product and pasted in a privacy-policy template.
- Anyone in the EU or UK can sign up, so GDPR already applies to you.
- You wired up an AI API and never added a notice that it is AI.
- You have Google Analytics or a Meta Pixel firing with no consent banner.
- You genuinely do not know which third parties end up with your users' data.

---

## How to install it (no terminal needed)

Pick whichever option feels easier. Both work in Claude Code and Claude on the web.

### Option 1, let Claude install it for you

Open a new chat in Claude and paste this in.

> Please install this Claude skill for me. The SKILL.md file lives in this GitHub repo: https://github.com/oliwoodman/compliance-audit-skill
>
> Set it up so I can start using it, then offer to run a compliance audit on my repo straight away.

Claude will grab the file and drop it in the right place. If your setup needs a manual step, it will tell you exactly what to click.

### Option 2, download the file and ask Claude to set it up

1. Click [SKILL.md](./SKILL.md) at the top of this repo.
2. Use the download button on the right of the file view to save it to your computer.
3. Open Claude and paste this in.

> I just downloaded a file called SKILL.md for the Compliance Audit skill. Can you install it for me and walk me through where to put it?

Claude takes it from there.

---

## Running it

Once it is installed, run it in any project by saying

> run compliance-audit

Claude asks you its ten questions, reads the repo, works through the GDPR and EU AI Act checklist, and gives you a ranked list of what to fix. Then it offers to fix it.
