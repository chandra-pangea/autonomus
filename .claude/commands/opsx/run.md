---
name: "OPSX: Run"
description: "Run a single task by ID. Auto-archives when all tasks complete."
category: Workflow
tags: [workflow, tasks, run, experimental]
---

Run a single task by its ID from an OpenSpec change. When the last task completes, auto-archives the change.

**Input**: Specify a task ID and optionally a change name.

```
/opsx:run 1.1                    # Run task 1.1 (auto-selects change if only one)
/opsx:run 2.3 add-auth           # Run task 2.3 from change "add-auth"
/opsx:run next                   # Run the next pending task
/opsx:run next add-auth          # Run the next pending task from "add-auth"
```

**Steps**

1. **Select the change**

   If a change name is provided, use it. Otherwise:
   - Auto-select if only one active change exists
   - If multiple exist, run `openspec list --json` and use the **AskUserQuestion tool** to let the user pick
   - If no changes exist: "No active changes. Create one with `/opsx:propose`" — stop

   Announce: "Change: **<name>**"

2. **Load context**

   ```bash
   openspec status --change "<name>" --json
   ```

   Then:
   ```bash
   openspec instructions apply --change "<name>" --json
   ```

   Read all files listed in `contextFiles` (proposal, specs, design, tasks).

   If `state: "blocked"`: show message, stop.
   If `state: "all_done"`: skip to step 5 (archive).

3. **Find the target task**

   Read the tasks file (typically `openspec/changes/<name>/tasks.md`).

   - If input is a **task ID** (e.g., `1.1`, `2.3`): find the matching task line `- [ ] X.Y ...`
     - If the task is already `- [x]`: tell the user "Task X.Y is already completed" — stop
     - If the task ID doesn't exist: list available tasks and stop
   - If input is **`next`**: find the first unchecked task (`- [ ]`) in the file

   Show:
   ```
   Task <ID>: <description>
   ```

4. **Implement the single task**

   - Make the code changes required for THIS task only
   - Keep changes minimal and focused
   - Run relevant tests if they exist
   - Mark task complete in the tasks file: `- [ ]` → `- [x]`

   On success show:
   ```
   Task <ID> complete

   Progress: <completed>/<total> tasks
   Remaining: <list of remaining task IDs>
   ```

   If blocked or unclear, pause and ask the user. Do not guess.

5. **Auto-archive if all tasks are done**

   After marking the task complete, check if ANY `- [ ]` remain in the tasks file.

   **If tasks remain**: show the next pending task as a suggestion:
   ```
   Next up: <next-task-ID> — <description>
   Run: /opsx:run next
   ```

   **If ALL tasks are complete** (`- [x]`):

   a. Show:
      ```
      All tasks complete! Auto-archiving...
      ```

   b. Archive the change:
      ```bash
      mkdir -p openspec/changes/archive
      mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
      ```

   c. Show final status:
      ```
      ## Done

      **Change:** <name>
      **Tasks:** <total>/<total> complete
      **Archived:** openspec/changes/archive/YYYY-MM-DD-<name>/
      ```

**Guardrails**
- Only implement ONE task per invocation — never batch
- Always read context files before implementing
- Mark the task complete immediately after implementation
- Pause on blockers — do not skip or guess
- Auto-archive only when zero `- [ ]` remain — no confirmation needed
- Never modify tasks other than the target task's checkbox
