description: "Execute a feature implementation plan from _plans/ by reading the plan, creating tracked tasks, and routing each phase to the appropriate specialized agent."

argument-hint: "Plan slug or filename (e.g. vehicle-history or vehicle-history.md)"

packages:
  taller-mecanico-api:
    type: backend
    conventions: ".ai/agents/backend-developer.md"
  auto-hub-pro:
    type: frontend
    conventions: ".ai/agents/tanstack-wizard.md"

steps:
  - name: "Step 1 — Resolve the plan file"
    description: >
      From the argument, extract the plan slug (strip .md extension).
      The plan file is at _plans/<slug>.md.
      If no argument given, list _plans/*.md and ask the user to choose.
      If the file does not exist, print an error and stop.

  - name: "Step 2 — Read all necessary context"
    description: >
      Read in parallel before starting implementation:
      1. _plans/<slug>.md — the implementation plan
      2. .ai/specs/<slug>.md — the matching spec (if it exists), for acceptance criteria
      3. OPENCODE.md — monorepo-level conventions
      4. .ai/agents/backend-developer.md — backend agent conventions
      5. .ai/agents/tanstack-wizard.md — frontend agent conventions
      6. .ai/agents/db-architect.md — schema agent conventions
      7. .ai/agents/security-code-reviewer.md — security review rules
      8. .ai/agents/test-engineer.md — testing conventions
      Do not start implementation until all context is fully read.

  - name: "Step 3 — Create a task list"
    description: >
      Break the plan into tasks matching its phases and steps. Track them so the
      user can follow progress. Only one task should be in_progress at a time.
      Verbally acknowledge each phase transition (e.g. "Starting Phase 1: Database
      Migration").

  - name: "Step 4 — Execute each phase in order"
    description: >
      Work through each phase sequentially. Within a phase, steps modifying
      independent files can proceed in parallel via multiple agent calls in one
      message. Steps with dependencies must remain sequential.
    routing:
      - work: "New SQLAlchemy model"
        agent: "db-architect"
        briefing: "Copy the exact model spec from the plan, specify the file path in app/models/, existing related models, and naming conventions (prefix columns, PascalCase tablename)."
      - work: "New Alembic migration (table, column, index, constraint)"
        agent: "db-architect"
        guard: "MUST use db-architect — never route schema changes to backend-developer"
        briefing: "Provide the exact column specs, Oracle types (VARCHAR2, NUMBER, TIMESTAMP), foreign keys, and whether Identity columns or Sequences are needed."
      - work: "New FastAPI route file or endpoint"
        agent: "backend-developer"
        briefing: "Specify the router prefix, HTTP method, Pydantic schema to use, CRUD operations needed, and any auth dependencies (get_current_user, require_permiso)."
      - work: "New Pydantic schema"
        agent: "backend-developer"
        briefing: "Specify the model fields, whether it is Create/Update/Out variant, Oracle Decimal handling, and Config with from_attributes."
      - work: "New CRUD class or service"
        agent: "backend-developer"
        briefing: "Specify the model class, base CRUD to extend (CRUDBase), custom query methods needed, and Oracle-specific logic (stored procs, functions, views)."
      - work: "Auth middleware or dependency (JWT, permissions)"
        agent: "backend-developer"
        briefing: "Specify the permission bitmask (PERM_VER, PERM_CREAR, etc.), role requirements, and whether ses_id workshop isolation is needed."
      - work: "Any other backend change"
        agent: "backend-developer"
      - work: "New TanStack route (createFileRoute)"
        agent: "tanstack-wizard"
        briefing: "Specify the path segment, parent route, component to render, head meta, and any data fetching with useQuery."
      - work: "New React component or page"
        agent: "tanstack-wizard"
        briefing: "Specify the file path in src/components/ or src/routes/, existing UI components to reuse (DataTable, Modal, Layout, StatusBadge), and shadcn/ui primitives."
      - work: "New custom hook (useQuery/useMutation)"
        agent: "tanstack-wizard"
        briefing: "Specify the API endpoints to call, query key structure, and invalidation rules."
      - work: "New API client module in src/api/"
        agent: "tanstack-wizard"
        briefing: "Specify the base URL path, export function signatures, and TypeScript response types from src/types/."
      - work: "Any other frontend change"
        agent: "tanstack-wizard"
      - work: "Writing backend tests"
        agent: "test-engineer"
        briefing: "Specify pytest + httpx, which endpoints to test, mock Oracle DB session, and edge cases (404, 422, 500)."
      - work: "Writing frontend tests"
        agent: "test-engineer"
        briefing: "Specify Vitest, which component to test, states to cover (loading, empty, error, populated), and Playwright E2E flows if needed."
      - work: "Small, targeted edits to a file already read"
        tool: "Edit"

  - name: "Mandatory security review"
    description: >
      After completing any step that touches authentication, middleware,
      authorization logic, ses_id workshop isolation, or database queries,
      launch the security-code-reviewer agent on the changed files
      before continuing to the next phase. This is not optional.

  - name: "Step 5 — Visual validation (frontend changes only)"
    description: >
      After all phases touching auto-hub-pro are complete:
      1. Verify the dev server can start (cd auto-hub-pro && bun run dev)
      2. Navigate to every route that was added or modified
      3. Check for layout breaks, overflow, console errors, or loading failures
      4. Capture visual state at desktop viewport (1280x800)
      5. Fix any issues found, then re-validate

  - name: "Step 6 — Verification checklist"
    description: >
      Work through every item in the plan's Verification section:
      - For lint checks: run the appropriate command per package
        taller-mecanico-api: ruff check .
        auto-hub-pro: npm run lint
      - For type checks: run tsc or npm run typecheck
      - For backend: verify endpoints respond correctly via curl or httpx
      - For frontend: navigate to modified routes and confirm behavior
      If any check fails, diagnose the root cause and fix before marking
      the task complete.

  - name: "Step 7 — Final report"
    description: >
      After all tasks are complete, respond with a concise summary:
      Plan executed: _plans/<slug>.md
      Phases completed: <list>
      Files created: <count>
      Files modified: <count>
      Lint/type checks: <pass/fail summary>
      Visual validation: <pass/fail or N/A>
      Do not produce a long narrative. A short table or bullet list is enough.

examples:
  - scenario: "Execute a multi-package plan"
    user: "/execute-plan vehicle-history"
    flow: |
      Step 1: Resolve _plans/vehicle-history.md
      Step 2: Read context (spec, OPENCODE.md, all agents)
      Step 3: Create tasks: [Phase 1 Schema, Phase 2 Backend, Phase 3 Frontend]
      Step 4:
        Phase 1 → db-architect: new VehicleHistory model + Alembic migration
        Phase 2 → backend-developer: new GET /vehiculos/{id}/history endpoint
        Security review → security-code-reviewer: audit ses_id filter
        Phase 3 → tanstack-wizard: new /vehiculos/[id]/historial route
        Tests → test-engineer: pytest + vitest
      Step 5: bun run dev → navigate to /vehiculos/1/historial
      Step 6: ruff check . + npm run lint + tsc
      Step 7: Report summary
