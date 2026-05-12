# Lessons Learned & Known Bugs Log
*Context for agents: You must read this file before executing commands to avoid repeating past mistakes and to be aware of the current technical debt.*

## Active Bugs to Fix (WARNING!)
- **CRUD Naming Error:** In `app/crud/vehiculo.py`, the class is incorrectly named `CRUDOrden` instead of `CRUDVehicle` and operates on the wrong model.
- **Security:** The `SECRET_KEY` is hardcoded in `auth/jwt.py`. It must be read from `core/config.py`.
- **Dead Code:** There are two definitions of `require_permiso` in `dependencies.py`. The first one must be removed.
- **Unprotected Endpoints:** Current routers are not enforcing `get_current_user`.

## Previous Lessons
- *No lessons registered yet. Log complex compilation errors and their solutions here as you resolve them.*