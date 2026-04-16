---
name: session-handoff
description: "Manage multi-session projects by splitting big tasks into focused sessions, maintaining a persistent project README, and generating clean handoff context for the next session. PROACTIVELY USE THIS SKILL — do not wait for the user to ask. Trigger it when: (1) the user describes a project with 3+ distinct phases or steps, (2) the task involves building something that will take multiple rounds of work (campaigns, pipelines, migrations, refactors, multi-file features), (3) you estimate the work will fill more than ~60% of a context window, (4) the user mentions anything like 'let's split this up', 'we'll continue later', 'next session', 'wrap up', 'pick this up tomorrow', 'save my progress', (5) you're deep into a session and realize the remaining work should be a fresh session, (6) the user starts a new session and references work from a previous conversation. The user may not know the project is big until they describe it — YOU should recognize it and propose the session structure. Err on the side of using this skill. A project that turns out to be smaller than expected loses nothing from having a README; a project that turns out bigger than expected loses everything without one."
---

# Session Handoff

You manage multi-session projects so that context is never lost between conversations.

The core problem: when a big project spans multiple Claude sessions, each new session starts cold. Without structure, the user has to re-explain everything, decisions get lost, and work gets repeated. This skill prevents that by creating a persistent project state that any future session can read and immediately pick up where things left off.

## When to activate

You should be the one to recognize that a project needs session management — the user won't always know upfront. Watch for these signals:

**During project planning:**
- The user describes something with 3+ phases (e.g., "build a list, then find people, then write emails")
- The task involves multiple tools, data sources, or sequential steps that build on each other
- You estimate the full scope would fill more than half a context window
- The task sounds like it'll take more than ~30 minutes of back-and-forth

**During execution:**
- You've been working for a while and realize there's still a lot left
- You're about to start a new phase that would benefit from a fresh context
- The user says anything about continuing later, wrapping up, or picking up next time

**At session start:**
- The user references previous work ("remember that campaign we were building?")
- Look for a project README in likely directories before asking the user to re-explain

When in doubt, activate. The cost of creating a README for a project that didn't need one is near zero. The cost of NOT having one when you needed it is re-doing hours of work.

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

5. **Generate a ready-to-paste prompt** — Always end the handoff by giving the user a short, copy-pasteable prompt they can drop into the next session. The prompt should:
   - Point to the project README path so the new session reads it first
   - Include 1-2 sentences of context about what to do next
   - Be short enough that the user doesn't need to edit it

   Format it in a code block so it's easy to copy:

   ```
   Continue Phase 2 of [project name]. Read `path/to/README.md` for full context. Next step: [specific action].
   ```

   The user should never have to write the handoff prompt themselves — you generate it, they paste it.

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
