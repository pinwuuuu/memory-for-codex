# memory-for-codex

A reusable local `memory` skill for Codex.

It provides one skill entry with four capabilities:
- recall relevant project context before work
- write useful cross-conversation memory after a session
- inspect or search the local memory store
- consolidate stale or duplicate memory entries

## Structure

- `SKILL.md`: single skill entry
- `scripts/`: helper scripts for search, write, and maintenance
- `store/`: local memory directory skeleton
- `meta/`: index metadata directory

## Usage

Copy this folder into a project as `memory/`, then install or link it into your Codex skills directory.

The memory data stays local to that project.
