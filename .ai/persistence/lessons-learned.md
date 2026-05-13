# Lessons Learned & Known Bugs Log
*Context for agents: You must read this file before executing commands to avoid repeating past mistakes and to be aware of the current technical debt.*

## Active Bugs to Fix
*(All known bugs resolved as of 2026-05-12)*

## Resolved Bugs
| Bug | Fix |
|-----|-----|
| CRUD Naming Error | Renamed class to `CRUDVehicle` in `crud/vehiculo.py` |
| Hardcoded SECRET_KEY | Now read from `settings.SECRET_KEY` in `jwt.py` |
| Dead Code in dependencies.py | Removed first `require_permiso` definition |
| Unprotected Endpoints | All routers now have `Depends(get_current_user)` |
| `cli_date_mod` column | Renamed to `veh_date_mod` |
| Missing CHECK/UNIQUE constraints | Added via `__table_args__` on models |
| Frontend missing JWT interceptor | Added to `axios.ts` |
| Wrong type `ords_id` | Corrected to `srv_ord_id` in `types/index.ts` |
| Missing leading `/` in delete path | Fixed in `vehiculos.tsx` |

## Security Observations
- `app/auth/permissions.py` still has hardcoded `ROLES` dict — redundant with `permisos_rol` BD table, kept as fallback for local testing without DB connection. **Not critical.**
- `require_permiso` now uses `fn_tiene_permiso` via BD call (`tiene_permiso_bd`).
- Compound trigger `trg_validar_cierre_orden` prevents invalid order closure at DB level.
- SAVEPOINT (`db.begin_nested()`) added to `crud/orden.py:create_with_services`.
- Frontend auto-redirects to `/login` on 401 via axios interceptor.