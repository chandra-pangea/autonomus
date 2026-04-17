---
name: tasks-json
description: Output all OpenSpec tasks across changes as structured JSON with status tracking. Use when the user wants a quick JSON snapshot of task progress.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: project-scaffold
  version: "1.0"
---

Output all OpenSpec tasks as structured JSON.

**Input**: Optionally specify a change name to filter (e.g., `/tasks-json add-auth`). If omitted, shows tasks from ALL active changes.

**Steps**

1. **Discover changes**

   ```bash
   openspec list --json
   ```

   - If a change name is provided as argument, filter to only that change
   - If no changes exist, output: `{"changes": [], "summary": {"total": 0, "completed": 0, "in_progress": 0, "todo": 0}}`

2. **For each active change, read its tasks file**

   First get status to find the tasks artifact:
   ```bash
   openspec status --change "<name>" --json
   ```

   Then read the tasks file (typically `openspec/changes/<name>/tasks.md`).

   If no tasks file exists for a change, skip it with a note in the output.

3. **Parse the tasks.md file**

   Extract structured data using these rules:

   - **Group headers**: Lines matching `## N. Group Name` or `## Phase N: Group Name`
     - Extract group number and group name
   - **Tasks**: Lines matching `- [ ] N.N Description` or `- [x] N.N Description`
     - Extract task ID, description, and completion status
   - **Status logic**:
     - `- [x]` or `- [X]` = `"completed"`
     - `- [ ]` where ALL preceding tasks in the same group are `completed` AND this is the FIRST unchecked task in the group = `"in_progress"`
     - All other `- [ ]` = `"todo"`
   - **Sub-bullets**: Lines indented under a task (starting with `  -` or `  *`) are captured as `details` array on the parent task

4. **Build the JSON output**

   Structure:

   ```json
   {
     "generated_at": "2026-04-14T10:30:00Z",
     "changes": [
       {
         "name": "change-name",
         "schema": "spec-driven",
         "groups": [
           {
             "number": 1,
             "name": "Group Name",
             "tasks": [
               {
                 "id": "1.1",
                 "description": "Task description",
                 "status": "completed",
                 "details": []
               },
               {
                 "id": "1.2",
                 "description": "Another task",
                 "status": "in_progress",
                 "details": ["Sub-step detail", "Another detail"]
               }
             ]
           }
         ],
         "summary": {
           "total": 10,
           "completed": 3,
           "in_progress": 1,
           "todo": 6
         }
       }
     ],
     "summary": {
       "total_changes": 1,
       "total_tasks": 10,
       "completed": 3,
       "in_progress": 1,
       "todo": 6
     }
   }
   ```

5. **Output the JSON**

   Print the JSON directly to the conversation. Use proper formatting with 2-space indentation.

**Output Format**

Always output ONLY the JSON — no surrounding explanation, no markdown fences, no commentary before or after.

Exception: if there are no changes or no tasks files, output the empty JSON structure AND a one-line hint:

```
{"changes":[],"summary":{"total_changes":0,"total_tasks":0,"completed":0,"in_progress":0,"todo":0}}

No active changes found. Create one with /opsx-propose
```

**Guardrails**
- Output valid JSON — always
- Do not modify any tasks files — this is read-only
- Include ALL active changes unless filtered by argument
- Skip archived changes unless the user explicitly asks for them with `--archived`
- Use ISO 8601 for `generated_at` timestamp
- If a tasks file has non-standard format (no checkboxes), include the change with `"tasks": []` and add a `"warning"` field
