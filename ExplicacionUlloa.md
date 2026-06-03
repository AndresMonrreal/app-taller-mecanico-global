riterio 1 — Diseño del modelo relacional (20/20)
Qué decir: "Cumple 3FN completa. Usamos llaves sustitutas (cli_id, veh_id, ord_id, etc.) en lugar de llaves naturales. No hay redundancia ni datos derivados — los totales se calculan con funciones. Cada tabla tiene su PK auto-generada por IDENTITY y todas las FK apuntan correctamente."
Dónde está:
- Modelos: taller-mecanico-api/app/models/cliente.py, vehiculo.py, orden.py, orden_servicio.py, servicio.py, sesion.py, users.py, permiso_rol.py
- Documentación: taller-mecanico-api/db/README.md L22-L99
Criterio 2 — Calidad UI/UX (20/20)
Qué decir: "La interfaz está construida con React 19 + shadcn/ui + Tailwind. Es completamente responsive (funciona en mobile y desktop), tiene tema oscuro profesional, carga de datos con estados de carga, manejo de errores con toasts (sonner), diálogos de confirmación en lugar de alerts nativos, búsqueda en tiempo real, filtros por estado, y un panel de reportes con métricas y gráficos. También validamos los formularios del lado del cliente con feedback visual."
Dónde está:
- Rutas: auto-hub-pro/src/routes/clientes.tsx, vehiculos.tsx, ordenes.tsx, servicios.tsx, reportes.tsx, login.tsx, configuracion.tsx
- Componentes compartidos: auto-hub-pro/src/components/DataTable.tsx, Modal.tsx, ConfirmDialog.tsx, StatusBadge.tsx
Criterio 3 — Tablas auditables (10/10)
Qué decir: "Tenemos 8 tablas (Clients, Vehicles, Orders, ServiceOrders, Service, Sessions, users, permisos_rol). Cada tabla tiene datos de prueba (al menos 3 registros cada una) y todas las FK tienen integridad referencial mediante constraints en BD."
Dónde está:
- Seed data: alembic/versions/c3d4e5f6a7b8_seed_data.py
- FKs: taller-mecanico-api/db/README.md L707-L721
Criterio 4 — Relación auditoría ses_id (15/15)
Qué decir: "Las 6 tablas de negocio tienen una columna ses_id con FK a Sessions. Creamos 6 triggers BEFORE INSERT que asignan automáticamente el SYS_CONTEXT('USERENV', 'SESSIONID') para que el usuario no tenga que preocuparse por llenarlo."
Dónde está:
- Triggers: alembic/versions/b2c3d4e5f6a7_plsql_objects.py L154-L172
- Columnas en modelos: app/models/cliente.py:14, vehiculo.py:20, orden.py:19, etc.
Criterio 5 — Roles y permisos (20/20)
Qué decir: "Los permisos se definen como máscara de bits en hexadecimal (0x01=leer, 0x02=crear, 0x04=editar, 0x08=eliminar). Tenemos 4 roles (admin=255, mecanico=7, recepcion=3, cliente=1). La validación se hace con la función fn_tiene_permiso en Oracle (usa BITAND), llamada desde Python en la dependencia require_permiso. Esto es dinámico: podemos agregar roles sin cambiar código."
Dónde está:
- Tabla permisos_rol: alembic/versions/b2c3d4e5f6a7_plsql_objects.py L37-L48
- Función fn_tiene_permiso: mismo archivo L50-L70
- Validación en Python: app/auth/permissions.py, app/dependencies.py L20-L25
- Hook frontend: auto-hub-pro/src/hooks/usePermisos.ts
Criterio 6 — Restricciones e índices (10/10)
Qué decir: "Tenemos 4 CHECK (año vehículo > 1900, estado orden en ABIERTA/EN_PROCESO/CERRADA, horas positivas, precio positivo), 5 UNIQUE (email, teléfono, placa, nombre servicio, username), 1 UNIQUE compuesto (ord_id + srv_id), 1 índice compuesto por (ord_status, ord_date), y un ÍNDICE FUNCIONAL UPPER(veh_plate) para búsquedas case-insensitive por placa."
Dónde está:
- Migración: alembic/versions/a1b2c3d4e5f6_correcciones_modelo.py L36-L54
- Documentación: taller-mecanico-api/db/README.md L447-L511
Criterio 7 — Vistas (10/10)
Qué decir: "Creamos 3 vistas: vw_ordenes_activas (simple, filtra órdenes activas), vw_resumen_clientes (compleja: subconsulta escalar + JOIN + CASE + GROUP BY), y vm_metricas_taller (VISTA MATERIALIZADA con BUILD IMMEDIATE y REFRESH COMPLETE ON DEMAND). La materializada está pensada para el dashboard de reportes."
Dónde está:
- Código PL/SQL: alembic/versions/b2c3d4e5f6a7_plsql_objects.py L124-L152
- Consumo desde app: app/routers/clientes.py:18-21 (vw_resumen_clientes), app/routers/ordenes.py:24-27 (vw_ordenes_activas)
Criterio 8 — Procedimiento almacenado (15/15)
Qué decir: "Dos procedimientos. sp_cerrar_orden: recibe IN y OUT, usa SAVEPOINT + COMMIT/ROLLBACK, maneja excepciones con WHEN OTHERS. sp_transferir_servicio: simula transacción distribuida llamando a sp_log_transferencia con PRAGMA AUTONOMOUS_TRANSACTION. Ambos se llaman desde endpoints de FastAPI."
Dónde está:
- sp_cerrar_orden: alembic/versions/b2c3d4e5f6a7_plsql_objects.py L86-L122
- sp_transferir_servicio: alembic/versions/d4e5f6a7b8c9_roles_y_transaccion_distribuida.py L45-L87
- Endpoints: app/routers/ordenes.py:56-68 (cerrar), app/routers/ordenes.py:99-114 (transferir)
- Botón cerrar (frontend): auto-hub-pro/src/routes/ordenes.tsx:289-297
- Botón transferir: auto-hub-pro/src/routes/ordenes.tsx:298-304
Criterio 9 — Función definida por usuario (10/10)
Qué decir: "Dos funciones. fn_calcular_total_orden: recibe un ord_id, retorna la suma de ords_total. fn_tiene_permiso: recibe rol y permiso, usa BITAND y retorna 1/0. Ambas se llaman exitosamente dentro de consultas SELECT desde la aplicación."
Dónde está:
- Código: alembic/versions/b2c3d4e5f6a7_plsql_objects.py L50-L84
- Llamada fn_calcular_total_orden: app/routers/ordenes.py:70-76
- Llamada fn_tiene_permiso: app/auth/permissions.py
Criterio 10 — Trigger (15/15)
Qué decir: "6 triggers BEFORE INSERT que asignan ses_id automáticamente. Además, un COMPOUND TRIGGER (TRG_VALIDAR_CIERRE_ORDEN) que valida al cerrar una orden que todos los servicios tengan total calculado. Usamos COMPOUND TRIGGER para evitar el error de tabla mutante, porque el trigger sobre Orders necesita leer ServiceOrders."
Dónde está:
- Compound trigger: alembic/versions/b2c3d4e5f6a7_plsql_objects.py L174-L204
- Triggers ses_id: mismo archivo L154-L172
Criterio 11 — Control transaccional aplicación (5/5)
Qué decir: "La app usa FastAPI + SQLAlchemy con transacciones explícitas (commit/rollback). El endpoint cerrar orden ejecuta sp_cerrar_orden que contiene SAVEPOINT. El endpoint transferir-servicio ejecuta sp_transferir_servicio que a su vez llama a sp_log_transferencia con PRAGMA AUTONOMOUS_TRANSACTION, simulando una transacción distribuida donde una operación local y una remota se ejecutan con atomicidad."
Dónde está:
- Endpoint cerrar: app/routers/ordenes.py:56-68
- Endpoint transferir: app/routers/ordenes.py:99-114
- UI transferir: auto-hub-pro/src/routes/ordenes.tsx:160-192 y L474-L527
- API function: auto-hub-pro/src/api/ordenes.ts:15-16
Criterio 12 — Conexión Oracle 23ai (10/10)
Qué decir: "Conexión a Oracle Autonomous DB 23ai usando Oracle Wallet con autenticación mTLS y driver oracledb en modo thin. Configuramos un pool de conexiones con pool_size=5 y max_overflow=10. Desde la app ejecutamos SELECT, INSERT, UPDATE, DELETE, llamadas a stored procedures (callproc), funciones (desde SELECT), y consultas a vistas normales y materializadas."
Dónde está:
- Configuración conexión: app/core/db.py
- Uso del pool: app/core/db.py:18-25
Criterio 13 — Control de acceso BD (10/10)
Qué decir: "Creamos un usuario taller_app con el principio de mínimo privilegio. Los permisos no se asignan directamente al usuario, sino a través de un ROL DE APLICACIÓN (taller_app_rol). El rol solo tiene SELECT/INSERT/UPDATE/DELETE en las tablas del dueño del esquema, EXECUTE en los objetos PL/SQL, y SELECT en las vistas. Sin acceso a tablas de sistema ni DDL."
Dónde está:
- Script: taller-mecanico-api/db/create_taller_app_user.sql 