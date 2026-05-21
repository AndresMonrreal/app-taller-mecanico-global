# Skill: Codebase & Architecture Exploration

Before proposing a refactor, modifying a database model, or adding a complex new feature, you are REQUIRED to follow these steps:

1. **Review Dependencies:** Navigate to the `/graphify-out` folder and read the generated dependency maps.
2. **Impact Analysis:** Determine which other modules (e.g., files in `app/crud/` or `app/routers/`) depend on the file you are about to modify.
3. If you modify a model in `app/models/`, you must warn the user that a new Alembic migration needs to be generated.