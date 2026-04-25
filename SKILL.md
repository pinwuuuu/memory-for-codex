---
name: memory
description: Local persistent memory for a single repository. Use this single skill to manage cross-conversation memory in four modes: recall relevant history before work, write high-value session outcomes, inspect or search the store directly, and consolidate stale or duplicate entries. Use it whenever work depends on prior project context, when the user asks to remember something, or when session state should persist cleanly outside normal docs.
---

# Memory

This is a single local memory skill for the current repository. It replaces the
split `memory-recall`, `memory-write`, and `memory-consolidate` entry points with
one skill and one store.

The store lives under `{baseDir}/store/` and scripts live under `{baseDir}/scripts/`.

## Store Layout

```text
store/
├── episodic/
│   ├── sessions/
│   │   └── archive/
│   └── events/
├── semantic/
│   ├── projects/
│   ├── people/
│   ├── technologies/
│   └── domain/
├── procedural/
│   ├── workflows/
│   ├── decisions/
│   └── conventions/
└── working/
    └── active.md
```

## Mode 1: Recall

Use before non-trivial work, when resuming prior work, or when the user asks
whether you remember something.

1. Read working memory first:

```bash
cat {baseDir}/store/working/active.md
```

2. Search targeted memories:

```bash
python {baseDir}/scripts/search_memory.py --query "{project or feature}" --type semantic --category projects --limit 8
python {baseDir}/scripts/search_memory.py --query "{technology keywords}" --type semantic --category technologies --limit 6
python {baseDir}/scripts/search_memory.py --query "{task type}" --type procedural --category workflows --limit 4
python {baseDir}/scripts/search_memory.py --query "{project or feature}" --type procedural --category decisions --limit 6
python {baseDir}/scripts/search_memory.py --query "{project or feature}" --type episodic --category sessions --sort recency --limit 3
```

3. Read only clearly relevant full files.
4. Produce a short task-specific brief.

## Mode 2: Write

Use when the user says to remember or save something, when a session ends, after
a non-trivial fix, or after a decision/workflow is established.

1. Search before writing:

```bash
python {baseDir}/scripts/search_memory.py --query "{topic keywords}" --limit 5
```

2. Always write a session memory:

```bash
cat <<'EOF' | python {baseDir}/scripts/write_memory.py \
  --type episodic \
  --category sessions \
  --title "Session: {YYYY-MM-DD} - {short slug}" \
  --content-stdin \
  --tags "{tags}" \
  --project your-project
## Goal
{goal}

## Outcome
{outcome}

## Key Decisions
{decisions}

## Problems Solved
{issues and resolutions}

## Next Steps
{next steps}
EOF
```

3. Write semantic or procedural memory only for facts worth keeping.

Project fact:

```bash
python {baseDir}/scripts/write_memory.py \
  --type semantic \
  --category projects \
  --title "{project fact title}" \
  --content-file /tmp/memory.md \
  --tags "{tags}" \
  --project your-project
```

Decision/workflow:

```bash
python {baseDir}/scripts/write_memory.py \
  --type procedural \
  --category decisions \
  --title "ADR: {decision title}" \
  --content-file /tmp/memory.md \
  --tags "{tags}" \
  --project your-project
```

4. Refresh working handoff:

```bash
cat <<'EOF' | python {baseDir}/scripts/write_memory.py \
  --type working \
  --category active \
  --title "Active Context" \
  --content-stdin \
  --tags "working,active,session" \
  --overwrite
## Current Project
your-project

## Active Task
{task and status}

## In Progress
{partially completed work}

## Next Steps
{ordered next steps}

## Active Files
{important files}

## Open Questions
{unresolved questions}
EOF
```

For multiline content, prefer `--content-stdin` or `--content-file`.

## Mode 3: Direct Inspect/Search

Use when you need to browse the store manually.

```bash
python {baseDir}/scripts/list_memories.py --brief
python {baseDir}/scripts/search_memory.py --stats
python {baseDir}/scripts/search_memory.py --query "gfn2 descriptor"
cat {baseDir}/store/working/active.md
```

## Mode 4: Consolidate

Use periodically or after many writes when recall gets noisy.

```bash
python {baseDir}/scripts/search_memory.py --stats
python {baseDir}/scripts/consolidate.py
python {baseDir}/scripts/consolidate.py --dry-run
python {baseDir}/scripts/search_memory.py --rebuild-index
```

If duplicate titles are reported, read both files, merge them into the better
entry, and rewrite with `write_memory.py`.

## Rules

1. One concept per file.
2. Search before writing to avoid duplicates.
3. Use at least two tags.
4. Set `project` for project-specific memory.
5. Use `confidence: low` for inferred facts.
6. Do not hard-delete episodic memories; archive them instead.
