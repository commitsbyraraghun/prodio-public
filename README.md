# Prodio

Checks whether your AI-assisted product management work actually holds up, before a real stakeholder review exposes a flaw in it.

You paste something real, a PRD section, a roadmap justification, a piece of stakeholder writing. Prodio runs it through a two-call verification pipeline: one call grades the text against a role-and-level rubric for common AI-judgment failures (unsourced claims, confident-but-empty language, over-trust of AI output), a second call independently re-grades the same text and votes on whether it agrees. If the two disagree, the verdict is reported as "uncertain" rather than picking a side.

A verdict alone isn't the point. After each check, you explain why you think the result is right or wrong. An adaptive judge checks whether your answer actually names the real issue, and if it's shallow, asks one follow-up before accepting it. Progress (the "Ladder") only fills in when a check passed *and* you demonstrated real reasoning, never from watching content alone.

## Why this exists

Most people using AI for real work at their job have no reliable way to know, in advance, whether their AI-assisted output will hold up when someone actually checks it. Prodio is a judgment check, not a lesson: it tests whether you can tell good AI-assisted work from bad, using your own real submissions, not a quiz bank.

## How it works

| Step | What happens | LLM involved? |
|---|---|---|
| Level check | 5 questions set a starting level (1-3) and detect a specific gap in judgment | No, deterministic |
| Learn | One primer video, picked by the detected gap, before your first check (or skip it) | No |
| Build | You paste real AI-assisted work | No |
| Checking | Call 1 grades against the role+level rubric; Call 2 independently re-grades and votes agree/disagree | Yes, two LLM calls |
| Result | Pass, fail, or uncertain (on disagreement), plus a fix tip if not a clean pass | No |
| Reasoning | You explain your reasoning; an adaptive judge checks if it actually names the issue | Yes, one LLM call (two on a shallow first answer) |
| Ladder | Fills in only from a passed check with demonstrated reasoning, never from content viewed | No |

Identity is a device cookie, not an account. An email is captured once (no password, no verification) purely so real users can be counted, it does not grant access to anyone else's data.

The marketing landing page (`app/page.tsx`) runs its own design system, [`DESIGN.md`](DESIGN.md), original tokens and components authored for this rebuild, structured around the product's actual mechanism rather than generic SaaS-page conventions: a live example card shows the two grading calls disagree and resolve to "uncertain" in the first viewport. Every other screen shares one app shell (`app/DeviceBootstrap.tsx`), full-width on mobile, a centered card on desktop, with a back button in the shared top bar on every screen but Home.

## Stack

- [Next.js](https://nextjs.org) (App Router, TypeScript)
- [Supabase](https://supabase.com) (Postgres) for persistence
- [Groq API](https://groq.com) (`groq-sdk`), running `openai/gpt-oss-120b` with strict JSON-schema structured outputs, for the two grading calls and the reasoning judge
- [Vitest](https://vitest.dev) for the deterministic logic (placement, gap detection, drills, ladder computation)

Everything that doesn't need a model (level placement, gap detection, video routing, ladder computation) is a pure, fully unit-tested function with zero API dependency. Only grading and the reasoning judge call the LLM.

## Running locally

```bash
npm install
cp .env.local.example .env.local   # fill in your own Supabase and Groq credentials
npm run dev
```

You'll need:
- A Supabase project, with the schema in `supabase/schema.sql` applied via the SQL Editor
- A Groq API key from [console.groq.com](https://console.groq.com) (free tier: 1,000 requests/day, 30 requests/minute)

```bash
npm test         # run the unit test suite
npm run build    # production build
```

## Status

This is a working prototype built for a product management case study, not a finished product. Deployed on Vercel. Known gaps:
- Two of the four Learn video slots (`lib/learn.ts`) are placeholder IDs, pending real content
- A production-only latency issue (identical requests take seconds locally but minutes on Vercel) hasn't been root-caused yet

See `Prodio Build Log.md` for the full build history, including the two real bugs found and fixed during live verification, the findings from a design critique pass, and the landing page rebuild (`DESIGN.md`, an independent finish review, and the fixes it caught).
