# Session Handoff & Compaction — Project Guide

## What This Tool Does

A Claude Code skill that manages context across long projects. It proactively splits big tasks into focused sessions, maintains a persistent README as the source of truth, and intelligently chooses between compaction (compress context in place) and session handoffs (clean break with pickup prompt) based on whether the next step needs conversational context or just the files.

## Architecture

```mermaid
flowchart TD
    A["Big project lands"] --> B{"Scope assessment:\nMore than one session?"}
    B -->|No| C["Execute normally"]
    B -->|Yes| D["Create project folder + README.md"]
    D --> E["Execute current phase"]
    E --> F{"Context getting heavy\nor phase complete?"}
    F -->|No| E
    F -->|Yes| G{"Decision point:\nDoes next step need what's\nin my head, or just the files?"}
    G -->|"Needs conversational context\n(iterating, user feedback,\nmid-workflow)"| H["COMPACT"]
    G -->|"Only needs files\n(new task, different tools,\nREADME is sufficient)"| I["SESSION HANDOFF"]
    H --> J["1. Update README\n2. Capture uncaptured preferences\n3. Compress context\n4. Continue working"]
    I --> K["1. Update README\n2. Summarize completed/next\n3. Generate pickup prompt\n4. End session"]
    J --> E
    K --> L["New session reads README"]
    L --> E

    style D fill:#ffe0b2
    style G fill:#e1f5fe
    style H fill:#c8e6c9
    style I fill:#ffcdd2
```

The skill operates on a loop: assess scope → split into sessions → execute → decide compact vs. handoff → continue or start fresh.

## Installation

The skill is a single file. Copy it to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/session-handoff
cp skill/SKILL.md ~/.claude/skills/session-handoff/SKILL.md
```

## How It Works

Once installed, the skill triggers automatically when Claude Code detects:
- A project too big for one session
- Context getting long on complex work
- You asking to wrap up, compact, or continue later

It creates a project folder with a README.md that tracks:
- **Plan** — What the project is trying to accomplish
- **Status** — What's done, what's in progress, what's pending
- **Decisions** — Key choices made (so they don't get re-litigated)
- **Session log** — What each session accomplished and what the next one should do

## Key Rules

- The README is always the source of truth — not conversation history, not memory
- Never compact or end a session without updating the README first
- **Prefer compaction** when iterating on the same work — it preserves user feedback, preferences, and edge cases that a cold handoff would lose
- **Use handoffs** only at natural phase boundaries where the next task is genuinely different
- Be specific about next steps: file paths, exact commands, concrete first actions
- Decisions from previous sessions are respected unless the user explicitly revisits them
- Don't over-split — if it fits in 2-3 sessions with compaction, that's better than 5 handoffs

## Compact vs. Handoff Decision

**Rule of thumb:** If you could hand the README to a colleague and they'd know exactly what to do without asking you any questions, it's a new session. If they'd need to ask "wait, what did the user say about X?" — compact instead.

## Customizing

The skill works out of the box. If you want to adjust the README template, compact/handoff heuristics, or handoff format, edit `skill/SKILL.md` directly.
