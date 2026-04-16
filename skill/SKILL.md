---
name: session-handoff
description: "Manage multi-session projects by keeping context tight through compaction or session handoffs. Use this skill whenever: the user starts a large multi-step project, the user says 'let's split this up', you notice context is getting long on a complex task, you're approaching the end of a session with ongoing work, or the user asks to wrap up and continue later. Also trigger when you recognize a project will clearly need more than one session to complete — don't wait to be asked. Prefer compaction when continuing the same workflow; use session handoffs when pivoting to a distinct phase."
---

# Session Handoff & Compaction

You manage multi-session projects so that context is never lost — either by compacting within a session or handing off between sessions.

The core problem: when a big project spans multiple Claude sessions, each new session starts cold. Without structure, the user has to re-explain everything, decisions get lost, and work gets repeated. This skill prevents that through two mechanisms:

1. **Compaction** — Compress the current conversation's context in place, preserving tool state and working memory. Best when continuing the same workflow.
2. **Session handoff** — End the session cleanly, write a summary, and generate a pickup prompt. Best when pivoting to a distinct new phase.

## When to activate

Activate this skill in three situations:

1. **Project planning** — When the user brings a task that will clearly take more than one session (multi-phase projects, large-scope work, anything with 3+ distinct phases). Proactively propose splitting into sessions. Don't wait for them to ask.

2. **Context getting heavy** — When the context window is filling up but you're still in the same phase of work. **Prefer compaction** here — it keeps conversational nuance (user feedback, refined preferences, edge cases discussed) that a cold handoff would lose.

3. **Phase transition** — When you've completed a distinct phase and the next phase is genuinely different work. **Use a session handoff** here — the clean break helps the next session start focused.

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

### Phase 3a: Compaction (same workflow continues)

When context is getting heavy but you're continuing the same phase:

1. **Update the README** — Mark completed tasks, add decisions, append to session log
2. **Compact the conversation** — The compressed context + README should be sufficient to continue seamlessly
3. **Continue working** — Pick up where you left off in the same session

Compaction is better than a handoff when:
- You're mid-workflow (not at a natural stopping point)
- The user gave feedback or preferences you'd lose in a cold handoff
- Tool state and working context matter
- The next step is a direct continuation, not a pivot

### Phase 3b: Session handoff (pivoting to new phase)

When you've completed a distinct phase and the next is genuinely different:

1. **Summarize what was completed** — Specific deliverables, not vague descriptions
2. **Summarize what's next** — The first 2-3 concrete steps for the next session
3. **Update the README** — Mark completed tasks, add any new decisions or open questions, append to the session log
4. **Flag blockers** — If the next session needs something from the user (an API key, a decision, access to a tool), call it out explicitly
5. **Generate a ready-to-paste prompt** — A short code block the user can copy into the next session

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
- **Update before you compact or close.** Never compact or end a session without updating the README first. This is non-negotiable.
- **Prefer compaction over handoffs.** Compaction preserves nuance. Use handoffs only at natural phase boundaries.
- **Be specific about what's next.** "Continue the project" is useless. "Run the EdTech company search with AI Ark lookalike seeded from ClassDojo, filter to 51+ employees, deduplicate against Apollo raw list" is useful.
- **Decisions are permanent unless revisited.** If a previous session decided to use Tool X over Tool Y, the next session should respect that unless the user explicitly wants to reconsider.
- **Don't over-split.** Not every project needs 10 sessions. If it can reasonably fit in 2-3 with compaction, that's better. The goal is focused context, not busywork.
