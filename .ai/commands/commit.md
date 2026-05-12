description: "Create a commit message by analyzing git diffs, grouped by package and change type, with emoji and why-focused explanation."

allowed-tools: "Bash(git status), Bash(git diff --staged), Bash(git commit), Read, Edit, Write"

steps:
  - name: "Context gathering"
    description: >
      Run `git status` and `git diff --staged` in the current working directory.
      This command is multi-repo aware — it works from ULLOA root,
      taller-mecanico-api/, or auto-hub-pro/. Do not assume any fixed structure.

  - name: "Package detection"
    description: >
      Detect which package(s) the changes belong to based on the current directory:
      - If cwd is ULLOA root, group by subdirectory (taller-mecanico-api/, auto-hub-pro/, .ai/)
      - If cwd is taller-mecanico-api/, label changes as Backend (API)
      - If cwd is auto-hub-pro/, label changes as Frontend (UI)
      - Changes in .ai/ are labeled as Workflow/Config

  - name: "Change categorization"
    description: >
      Within each package, further categorize:
      - Schema/Migration: Alembic revisions, model changes, Oracle DDL
      - Logic/Service: FastAPI routes, CRUD, services, middleware, auth
      - UI/Component: React components, TanStack routes, hooks, styles
      - Config: Dependencies, environment, tooling
      - Workflow: Agent files, commands, specs, plans

  - name: "OPENCODE.md assessment"
    description: >
      After analyzing the staged diff but before proposing the commit message:
      1. Check if OPENCODE.md exists in the current directory. If not, skip silently.
      2. If it exists, read it and assess whether staged changes warrant an update.
         Signals include: new .ai/agents/, new .ai/comandos/, new .ai/skills/,
         new conventions, new dependencies, new environment variables.
      3. Prioritize documenting new agents, skills, commands added to .ai/.
      4. If update is warranted, propose specific changes and ask for approval.
      5. Never update OPENCODE.md without explicit user approval.

  - name: "Commit message generation"
    description: >
      Format: <emoji> <type>: <concise_description>
      Only include an optional body when the reason for the change is non-obvious.
      Group summary by package and change type.
    emoji_map:
      "New feature": "✨ feat:"
      "Bug fix": "🐛 fix:"
      "Refactoring": "🔨 refactor:"
      "Documentation": "📝 docs:"
      "Styling/formatting": "🎨 style:"
      "Tests": "✅ test:"
      "Performance": "⚡ perf:"
      "Maintenance/config": "🗑️ chore:"
      "Security hardening": "🔒 security:"
      "CI/CD": "🚀 ci:"
      "Revert": "♻️ revert:"

  - name: "Summary grouping rules"
    description: >
      Group staged changes in the summary as:
      1. Backend (taller-mecanico-api) — sub-group by Schema/Migration vs Logic/Service
      2. Frontend (auto-hub-pro) — sub-group by UI/Component vs Config
      3. Workflow (.ai/) — sub-group by Agent, Command, Spec, Plan

  - name: "Confirmation"
    description: >
      1. Display the smart-diff summary (grouped by package and type)
      2. Display OPENCODE.md assessment (if applicable)
      3. Display the proposed commit message
      4. Ask for user approval before committing
    guard: >
      STRICTLY FORBIDDEN to auto-commit. Always wait for explicit user approval.

examples:
  - scenario: "Backend schema + frontend UI changes in root"
    user: "/commit"
    summary: |
      Backend (taller-mecanico-api):
        Schema/Migration: new Orders.ord_urgency column (Alembic revision)
        Logic/Service: updated ordenes CRUD with urgency filter
      Frontend (auto-hub-pro):
        UI/Component: added urgency dropdown to OrdenForm
    proposed: |
      ✨ feat: add urgency field to service orders

      Backend: add ord_urgency column via Alembic migration and update CRUD filter
      Frontend: add urgency dropdown selector to the orden creation form

  - scenario: "New agent or command added"
    user: "/commit"
    summary: |
      Workflow (.ai/):
        Agent: new security-code-reviewer.md agent definition
        Command: new /plan-spec command
    proposed: |
      ✨ feat: add security-reviewer agent and plan-spec command

      Document the security-code-reviewer agent for proactive audit of auth and
      workshop isolation boundaries. Add /plan-spec to execute plans from .ai/plans/.
