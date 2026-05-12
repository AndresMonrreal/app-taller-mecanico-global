# Skill: workflow-execute-phase

## Propósito

Ejecutar una o más fases de un plan de manera eficiente, despachando cada paso al agente correcto con contexto completo pre-cargado, minimizando tokens y evitando que los agentes arranquen en frío.

## Cuándo usar este skill

Úsalo desde `/execute-plan` en el **Paso 4** cuando vayas a ejecutar una fase. En lugar de lanzar agentes uno por uno sin contexto, este skill agrupa, pre-carga y despacha.

---

## Protocolo de ejecución

### Paso A — Pre-cargar contexto UNA sola vez

Antes de despachar cualquier agente, lee en paralelo todos los archivos críticos de la fase:

```
Lee en paralelo:
- .ai/plans/<slug>.md                 (solo la fase que vas a ejecutar)
- OPENCODE.md                          (convenciones del monorepo)
- .ai/agents/backend-developer.md      (si la fase toca el backend)
- .ai/agents/tanstack-wizard.md        (si la fase toca el frontend)
- .ai/agents/db-architect.md           (si la fase toca esquema/base de datos)
- Cada archivo que el plan marque como "Modify" en esa fase
```

**No leas archivos de fases que aún no vas a ejecutar.** Solo la fase actual.

### Paso B — Agrupar pasos por agente

Antes de ejecutar, clasifica cada paso de la fase usando esta tabla:

| Tipo de trabajo | Herramienta |
|---|---|
| Nuevo modelo SQLAlchemy | `Agent: db-architect` |
| Nueva migración Alembic | `Agent: db-architect` |
| Asociaciones, scopes, índices entre modelos | `Agent: db-architect` |
| Nueva ruta FastAPI / endpoint | `Agent: backend-developer` |
| Nuevo schema Pydantic (Create/Update/Out) | `Agent: backend-developer` |
| Nueva clase CRUD o servicio | `Agent: backend-developer` |
| Auth / JWT / middleware / dependencias | `Agent: backend-developer` |
| Cualquier otro cambio backend | `Agent: backend-developer` |
| Nueva ruta TanStack (createFileRoute) | `Agent: tanstack-wizard` |
| Nuevo componente o página React | `Agent: tanstack-wizard` |
| Nuevo hook personalizado (useQuery/useMutation) | `Agent: tanstack-wizard` |
| Nuevo módulo API client (src/api/) | `Agent: tanstack-wizard` |
| Cualquier otro cambio frontend | `Agent: tanstack-wizard` |
| Tests backend (pytest + httpx) | `Agent: test-engineer` |
| Tests frontend (Vitest) | `Agent: test-engineer` |
| Edición pequeña en archivo ya leído | `Edit` directo |

### Paso C — Identificar qué puede ir en paralelo

Dos pasos pueden ejecutarse en paralelo si:
- Tocan archivos **distintos** y sin dependencia entre sí
- No comparten modelos que el otro necesite ya modificados

Dos pasos deben ir **secuenciales** si:
- El segundo importa o extiende algo que crea el primero
- Ejemplo: crear un modelo SQLAlchemy **debe** terminar antes de que otro modelo lo asocie vía `relationship()`

### Paso D — Brief completo para cada agente

Cada agente que lances **debe recibir** en su prompt de invocación:

```
## Tu tarea
<copia exacta del texto del paso en el plan>

## Archivos a crear/modificar
<lista de paths exactos del plan>

## Contexto del proyecto
<sección relevante de OPENCODE.md>
<sección relevante del archivo .ai/agents/<nombre>.md>

## Contenido actual del archivo (si es modificación)
<contenido del archivo que ya leíste en el Paso A>

## Dependencias ya resueltas
<si este agente depende de algo que ya se creó antes, pégalo aquí>

## Instrucción
Implementa directamente. No investigues, no explores — ya tienes todo el contexto.
```

**Nunca lances un agente sin el brief completo.** Un agente sin contexto consume 3-5x más tokens explorando.

### Paso E — Security review obligatorio

Después de cualquier paso que toque:
- Auth / JWT / middleware de autenticación
- Workshop isolation (`ses_id` scoping)
- Queries a la base de datos (SQLAlchemy o raw SQL)
- Routers o endpoints con acceso a datos sensibles

→ Lanza `Agent: security-code-reviewer` sobre esos archivos **antes de continuar a la siguiente fase**.

Brief para security-code-reviewer:

```
## Archivos a revisar
<lista de archivos modificados en este paso>

## Contexto
<sección de OPENCODE.md sobre workshop isolation y auth>
<sección de .ai/agents/security-code-reviewer.md>

## Qué buscar
- Queries sin filtro de ses_id (cross-workshop data leakage)
- Oracle bind variables (`:param`) vs interpolación f-string
- JWT manejado correctamente (firma, expiración, secret key)
- Validación de inputs en boundaries de seguridad
- HTTP codes semánticos correctos (401 vs 403 vs 404)
- Permisos bitmask correctos (PERM_VER, PERM_CREAR, PERM_EDITAR, PERM_ELIMINAR)
```

### Paso F — Verificación rápida post-fase

Al terminar cada fase, antes de marcar el task como `completed`:

1. Si se creó una migración → corre `cd taller-mecanico-api && alembic upgrade head`
2. Si se modificó un modelo → verifica que las asociaciones sean espejo de las FK de la migración
3. Si se modificó un endpoint → verifica que `ses_id` esté en todas las queries SQLAlchemy
4. Si se modificó el frontend → verifica que las rutas TanStack existan y el componente compile
5. Si se modificaron tipos → verifica que los TypeScript types en `auto-hub-pro/src/types/` reflejen los schemas Pydantic

---

## Ejemplo de ejecución — Plan Vehicle History

### Fase 1 (Migration + Model)

**Paralelo posible:** Sí. El modelo y la migración son archivos independientes.

```
Lanzar en paralelo:
  - Agent: db-architect → Phase 1.1 del plan (nuevo modelo VehicleHistory + Alembic migration)
  - Agent: db-architect → Phase 1.2 del plan (asociaciones en Vehicle y Order)

Después de ambos:
  - Bash: cd taller-mecanico-api && alembic upgrade head
```

### Fase 2 (Endpoint + CRUD)

**Secuencial:** El schema Pydantic debe existir antes que el router y el CRUD.

```
Secuencial:
  1. Agent: backend-developer → Phase 2.1 (schema Pydantic VehicleHistoryOut)
  2. Agent: backend-developer → Phase 2.2 (CRUD class con get_by_vehicle + ses_id filter)
  3. Agent: backend-developer → Phase 2.3 (nuevo GET /api/v1/vehiculos/{id}/history)

Después: security-code-reviewer sobre app/routers/vehiculos.py y app/crud/vehiculo.py
```

### Fase 3 (Frontend)

**Paralelo posible:** Sí. Tipos, API client y componente son independientes.

```
Lanzar en paralelo:
  - Agent: tanstack-wizard → Phase 3.1 (types VehicleHistory en src/types/)
  - Agent: tanstack-wizard → Phase 3.2 (API client getVehicleHistory en src/api/)
  - Agent: tanstack-wizard → Phase 3.3 (nuevo componente VehicleHistoryTable)

Después de los tres:
  - Agent: tanstack-wizard → Phase 3.4 (nueva ruta /vehiculos/$vehiculoId/historial con createFileRoute)
```

### Fase 4 (Tests)

```
Lanzar en paralelo:
  - Agent: test-engineer → Phase 4.1 (pytest: test GET /vehiculos/{id}/history)
  - Agent: test-engineer → Phase 4.2 (Vitest: test VehicleHistoryTable component)

Después: Bash:
  - cd taller-mecanico-api && pytest tests/ -k "vehicle_history"
  - cd auto-hub-pro && npx vitest run --reporter=verbose
```

---

## Reglas de oro

1. **Un solo read por archivo por fase** — pre-carga en el Paso A, no vuelvas a leer el mismo archivo
2. **Brief completo o no lances el agente** — sin contexto = tokens desperdiciados
3. **Paralelo solo cuando no hay dependencia** — duda → secuencial
4. **Security review no es opcional** — si toca auth/queries/ses_id, siempre
5. **Usa el grafo primero** — antes de cualquier Grep/Glob, consulta `code-review-graph` (semantic_search_nodes / query_graph) para encontrar archivos relacionados
