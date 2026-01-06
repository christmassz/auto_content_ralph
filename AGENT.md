# Agent Instructions

This file contains instructions for AI agents working on the Ralpha project.

## Operating Rules

Claude must follow the Ralph loop:

1. `bd ready` → pick **exactly one** ready bead (one task).
2. `bd show <id>` → treat as the only scope.
3. Search the codebase before implementing (no duplicate implementations).
4. Implement the smallest change that satisfies the bead's DoD.
5. Run the verification commands defined in the bead.
6. Update the bead notes with:
   * what changed
   * what command(s) ran + result
   * any new sub-beads discovered
7. Commit with a message referencing the bead id.

## Hard constraints

* **Do not broaden scope** beyond the selected bead.
* **No placeholders / TODO implementations** unless explicitly allowed by the bead.
* If unrelated failures appear, **create a new bead**; don't ignore.