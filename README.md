# Task Board — AI Product Planning

A kanban with an opinion, grown into a planning platform. Task Board helps a
product manager decide **what to work on, what fits in a sprint, and what to
say no to** — with an AI that recommends, explains itself, and never moves a
card on its own.

**Live demo:** https://darsh-pm.github.io/Task-Board/?demo=1 (lands directly
in demo mode — try `?demo=1&view=matrix` for the Priority Matrix)
**Case study:** see the Task Board case study on my portfolio for the product
narrative, key decisions, and tradeoffs.

## What it does

| Capability | How |
|---|---|
| **Prioritize** | Every task carries an impact (1–5) and effort (1–5) score — set by hand or suggested by AI with a one-line rationale. The Priority Matrix plots all active work into Quick Wins / Big Bets / Fill-ins / Time Sinks. |
| **Rank** | AI Rank reads the full signal set — status, priority, deadlines, subtask progress, impact/effort — and returns a do-first order with a reason per task, plus an explicit "can wait" list. |
| **Plan** | AI Sprint Plan takes an effort-point capacity budget and returns a committed set, a sprint goal, an explicitly deferred set, and the plan's biggest risk. |
| **Track** | The familiar three-column kanban underneath: drag-and-drop, subtasks with completion nudges, overdue tracking, and a backlog health strip (active / overdue / unscored / quick wins / WIP warning). |

## AI philosophy

Same contract as my other product, Pulse:

- **AI suggests, the human decides** — the model never moves a card, sets a
  score without review, or closes anything.
- **Every recommendation is explainable** and links back to the task it came
  from. You can disagree with the model intelligently.
- **Analysis persists** per user, and **flags itself stale** the moment the
  board changes underneath it — a one-click re-run keeps it honest.

## Architecture

```
board state → serialize full signals → rank / plan with rationale → render → human moves the cards
```

- **React** (via CDN, deliberately no build step) — board, matrix, modal,
  and analysis panels.
- **Supabase** — PostgreSQL + Auth with Row Level Security for per-user
  boards.
- **Groq (Llama 3.3 70B)** — through a Supabase Edge Function proxy, so the
  API key never reaches the client. Powers scoring suggestions, ranked
  analysis, and sprint planning with strict JSON outputs.
- **Priority Matrix** — rendered with plain HTML/CSS positioning; no chart
  libraries.

## Database migration (existing deployments)

Impact/effort scoring needs two columns on the `tasks` table:

```sql
alter table tasks
  add column if not exists impact int2,
  add column if not exists effort int2;
```

The app degrades gracefully without them (demo mode needs nothing), but
score saves for signed-in users will fail until the migration runs.

## Try it

Open the live demo — it loads a realistic product backlog with every matrix
quadrant populated and two tasks left unscored so you can watch the AI score
them. Or sign up for a private board.

## Roadmap

- Plan history — compare this sprint's commitments against last sprint's
- Score-aware board sorting (value score within columns)
- Team boards once the single-player loop is fully proven
