---
name: session-handoff
description: "Manage multi-session projects by splitting big tasks into focused sessions, maintaining a persistent project README, and generating clean handoff context for the next session. Use this skill whenever: the user starts a large multi-step project, the user says 'let's split this up', you notice context is getting long on a complex task, you're approaching the end of a session with ongoing work, or the user asks to wrap up and continue later. Also trigger when you recognize a project will clearly need more than one session to complete — don't wait to be asked."
---

# Session Handoff

You manage multi-session projects so that context is never lost between conversations.

The core problem: when a big project spans multiple Claude sessions, each new session starts cold. Without structure, the user has to re-explain everything, decisions get lost, and work gets repeated. This skill prevents that by creating a persistent project state that any future session can read and immediately pick up where things left off.

## When to activate

Activate this skill in two situations:

1. **Project planning** — When the user brings a task that will clearly take more than one session (multi-phase projects, large-scope work, anything with 3+ distinct phases). Proactively propose splitting into sessions. Don't wait for them to ask.

2. **Session wrap-up** — When you're nearing the end of a session with ongoing work. This could be because: you've completed the current phase, the context window is getting long, the user says they're done for now, or you recognize it's a natural stopping point.

## How it works

### Phase 1: Assess and split

When a big project lands, assess the scope and propose a session plan:

1. **Map the phases** — Break the work into logical chunks that can each be completed in one session
2. **Identify dependencies** — Which phases depend on outputs from earlier ones?
3. **Propose the split** — Present a session plan with what each session will accomplish and what it depends on
4. **Get buy-in** — The user should agree to the session structure before you start executing

A good session boundary is where:
- A discrete deliverable is complete (a list, a draft, a working feature)
- The next phase requires different inputs or a fresh approach
- Context is getting heavy and quality might degrade

### Phase 2: Create the project folder

For any project that spans sessions, create a dedicated folder with a README that serves as the single source of truth:

```
<project-name>/
├── README.md          ← Status tracker, decisions, next steps
├── [phase outputs]    ← Whatever artifacts each phase produces
└── [subfolders]       ← Organize by phase or type as needed
```

The README.md must contain:

```markdown
# [Project Name]

## Overview
[1-2 sentences: what this project is and what the goal is]

## Status Tracker
### Phase 1: [Name] ← COMPLETED
- [x] Task 1
- [x] Task 2

### Phase 2: [Name] ← CURRENT
- [x] Task 3
- [ ] Task 4
- [ ] Task 5

### Phase 3: [Name] ← PENDING
- [ ] Task 6

## Decisions Made
[Numbered list of key decisions with brief rationale]

## Open Questions
[Anything unresolved that the next session needs to address]

## Session Log
### Session 1 — [Date]
**Completed:** [bullet list]
**Next:** [what the next session should do first]
```

### Phase 3: Session wrap-up

At the end of every session (or when transitioning between phases):

1. **Summarize what was completed** — Specific deliverables, not vague descriptions
2. **Summarize what's next** — The first 2-3 concrete steps for the next session
3. **Update the README** — Mark completed tasks, add any new decisions or open questions, append to the session log
4. **Flag blockers** — If the next session needs something from the user (an API key, a decision, access to a tool), call it out explicitly

The README should be self-contained enough that a completely fresh Claude session can read it and know exactly what to do next.

### Phase 4: Session pickup

When starting a new session on an existing project, the first thing to do is read the project README. It tells you:
- What's been done
- What's next
- What decisions were already made (so you don't re-litigate them)
- What tools and approaches were chosen
- Any blockers or open questions

If the user starts a new session and references a previous project, look for the README in the project folder and read it before doing anything else.

## Writing a good handoff

The handoff is the most important part. A bad handoff means the next session wastes time getting oriented. A good handoff means it hits the ground running.

**Good handoff:**
> **Completed:** Built and tested the AI Ark API client (aiark/client.py). People search works with domain + seniority + department filters. Company lookalike search works with seed domains. Apollo raw data collected: 156 EdTech + 166 ConstructionTech companies (need curation).
>
> **Next session should:**
> 1. Read `campaigns/edtech-contech-outbound/README.md` for full context
> 2. Curate EdTech list to 100 companies using AI Ark lookalike (seed: ClassDojo) + filtered Apollo data
> 3. Curate ConstructionTech list to 100 companies — Apollo results are mostly actual construction firms, need to filter to software companies only

**Bad handoff:**
> We made good progress on the campaign. Next time we should keep working on it.

The difference is specificity. The good handoff tells you exactly what files to read, what's done, what's not, and what to do first.

## Key principles

- **The README is the source of truth.** Not memory, not the conversation history, not a mental model. The README.
- **Update before you close.** Never end a session without updating the README. This is non-negotiable.
- **Be specific about what's next.** "Continue the project" is useless. "Run the EdTech company search with AI Ark lookalike seeded from ClassDojo, filter to 51+ employees, deduplicate against Apollo raw list" is useful.
- **Decisions are permanent unless revisited.** If a previous session decided to use Tool X over Tool Y, the next session should respect that unless the user explicitly wants to reconsider.
- **Don't over-split.** Not every project needs 10 sessions. If it can reasonably fit in 2-3, that's fine. The goal is focused sessions, not busywork.
