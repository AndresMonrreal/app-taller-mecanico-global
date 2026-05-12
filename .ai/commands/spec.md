description: "Genera una especificación técnica detallada para el monorepo ULLOA y crea branches sincronizados en todos los paquetes."

argument-hint: "Descripción en lenguaje natural de la feature (e.g. 'Agregar historial de vehículos')"

packages:
  - path: taller-mecanico-api
    type: backend
    tech: "FastAPI, SQLAlchemy, Oracle 23ai, Pydantic, JWT"
  - path: auto-hub-pro
    type: frontend
    tech: "React 19, TanStack Router, TanStack Query, Axios, shadcn/ui, Tailwind CSS"

steps:
  - name: "Step 1 — Pre-flight check"
    description: >
      Abortar inmediatamente si cualquier paquete tiene cambios sin commitear.
      Ejecutar `git status --porcelain` en root, taller-mecanico-api/, y auto-hub-pro/.
      Si cualquier salida no está vacía, imprimir los archivos sucios agrupados por
      paquete y detener con mensaje de error pidiendo stash o commit primero.

  - name: "Step 2 — Leer contexto del proyecto"
    description: >
      Leer en paralelo antes de generar el spec:
      1. graphify-out/GRAPH_REPORT.md — mapa actual del código
      2. OPENCODE.md — convenciones del monorepo
      3. .ai/persistence/lessons-learned.md — errores previos a evitar
      4. _specs/ — specs existentes para evitar duplicados
      No proceder hasta tener todo el contexto.

  - name: "Step 3 — Parsear intención del usuario"
    description: >
      Normalizar la solicitud en lenguaje natural a una estructura técnica.
      Extraer: nombre de la feature (kebab-case), paquetes afectados (backend,
      frontend, ambos), resumen ejecutivo, enfoque técnico Oracle-específico,
      archivos a crear/modificar, y criterios de aceptación medibles.
      Considerar las siguientes dimensiones de la rúbrica del proyecto:
      - ¿Requiere cambios al modelo relacional (3FN, llaves sustitutas)?
      - ¿Involucra tablas auditables (ses_id, triggers)?
      - ¿Afecta roles y permisos (bitmask hex)?
      - ¿Necesita procedimiento almacenado, función o trigger?
      - ¿Requiere vistas nuevas o modificadas?

  - name: "Step 4 — Generar documento de especificación"
    description: >
      Actuar como Arquitecto Principal (razonamiento de alto orden).
      1. Crear directorio `_specs/` si no existe.
      2. Escribir archivo markdown en `_specs/<feature-name>.md`.
      3. Usar información de graphify-out/GRAPH_REPORT.md para precisión técnica.
      El spec DEBE incluir las secciones del template siguiente.
    template: |
      # Spec: {{feature_name}}
      **Fecha:** {{current_date}}
      **Slug:** {{kebab_case_name}}
      **Paquetes afectados:** {{packages}}
      **Branch:** opencode/feature/{{kebab_case_name}}

      ## Resumen ejecutivo
      {{summary}}

      ## Diseño técnico

      ### Base de datos (Oracle 23ai)
      - Tablas nuevas / modificadas: {{db_tables}}
      - Normalización: {{normalization_notes}}
      - Constraints (PK, FK, UNIQUE, CHECK): {{constraints}}
      - Índices necesarios: {{indexes}}
      - Vistas: {{views}}
      - Procedimientos / funciones / triggers: {{plsql_objects}}
      - Auditoría (ses_id, tablas auditables): {{audit_notes}}

      ### Backend (FastAPI + SQLAlchemy)
      - Endpoints nuevos: {{endpoints}}
      - Schemas Pydantic: {{schemas}}
      - CRUD / servicios: {{services}}
      - Permisos requeridos (bitmask): {{permissions}}

      ### Frontend (React + TanStack)
      - Rutas nuevas: {{routes}}
      - Componentes: {{components}}
      - Hooks (useQuery/useMutation): {{hooks}}
      - Módulos API client: {{api_modules}}

      ## Criterios de aceptación
      {{criteria}}

      ## Riesgos y dependencias
      {{risks}}

      ## Estimación de fases
      | Fase | Agente | Descripción | Estimado |
      |------|--------|-------------|----------|
      {{phases_table}}

  - name: "Step 5 — Crear branches sincronizados"
    description: >
      Crear y cambiar a branch `opencode/feature/<kebab-case-name>` en cada repo:
        1. Root: git checkout -b opencode/feature/<name>
        2. taller-mecanico-api/: git checkout -b opencode/feature/<name>
        3. auto-hub-pro/: git checkout -b opencode/feature/<name>
      Si el branch ya existe localmente o en remoto, imprimir error y abortar.

  - name: "Step 6 — Confirmar y sugerir siguiente paso"
    description: >
      Imprimir resumen con:
      - Ruta del spec generado
      - Branches creados por repo
      - Siguiente acción sugerida: `/plan-spec {{kebab_case_name}}`

examples:
  - scenario: "Agregar sistema de permisos con bitmask"
    user: "/spec Sistema de roles con máscara hexadecimal y validación en BD"
    result:
      spec: _specs/roles-permisos.md
      branches:
        - opencode/feature/roles-permisos (root)
        - opencode/feature/roles-permisos (taller-mecanico-api)
        - opencode/feature/roles-permisos (auto-hub-pro)

  - scenario: "Solo cambio de BD"
    user: "/spec Agregar trigger de auditoría en tabla clientes"
    result:
      spec: _specs/trigger-auditoria-clientes.md
      branches:
        - opencode/feature/trigger-auditoria-clientes (root)
        - opencode/feature/trigger-auditoria-clientes (taller-mecanico-api)
      packages: [taller-mecanico-api]