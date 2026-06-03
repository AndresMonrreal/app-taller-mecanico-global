# Documentación Completa — Sistema Integral de Gestión para Taller Mecánico

> **Plataforma empresarial para la administración de servicios automotrices**  
> Backend: FastAPI + Oracle 23ai | Frontend: React 19 + TanStack Router | Auth: JWT + Bitmask Permissions  
> Desarrollado por: **Jose Andres Monrreal Muñoz** — Instituto Tecnológico de Linares (ITL) — 2026

---

## Índice General

1. [Visión General del Sistema](#1-visión-general-del-sistema)
2. [Arquitectura General](#2-arquitectura-general)
3. [Backend — taller-mecanico-api](#3-backend--taller-mecanico-api)
   - 3.1 [Estructura del Backend](#31-estructura-del-backend)
   - 3.2 [Arquitectura en Capas](#32-arquitectura-en-capas)
   - 3.3 [Endpoints de la API](#33-endpoints-de-la-api)
   - 3.4 [Control Transaccional](#34-control-transaccional)
   - 3.5 [Manejo de Errores](#35-manejo-de-errores)
   - 3.6 [Autenticación y Permisos](#36-autenticación-y-permisos)
4. [Base de Datos — Oracle 23ai](#4-base-de-datos--oracle-23ai)
   - 4.1 [Modelo Relacional](#41-modelo-relacional)
   - 4.2 [Objetos PL/SQL](#42-objetos-plsql)
   - 4.3 [Vistas](#43-vistas)
   - 4.4 [Triggers](#44-triggers)
   - 4.5 [Restricciones e Índices](#45-restricciones-e-índices)
   - 4.6 [Roles y Permisos (Bitmask)](#46-roles-y-permisos-bitmask)
   - 4.7 [Usuario de Base de Datos](#47-usuario-de-base-de-datos)
   - 4.8 [Seed Data](#48-seed-data)
   - 4.9 [Migraciones](#49-migraciones)
5. [Frontend — auto-hub-pro](#5-frontend--auto-hub-pro)
   - 5.1 [Estructura del Frontend](#51-estructura-del-frontend)
   - 5.2 [Arquitectura de la UI](#52-arquitectura-de-la-ui)
   - 5.3 [Rutas y Páginas](#53-rutas-y-páginas)
   - 5.4 [Consumo de API](#54-consumo-de-api)
   - 5.5 [Aislamiento de Datos (Multitenancy)](#55-aislamiento-de-datos-multitenancy)
   - 5.6 [Manejo de Errores Visual](#56-manejo-de-errores-visual)
6. [Guía de Inicio Rápido](#6-guía-de-inicio-rápido)
7. [Rúbrica de Evaluación](#7-rúbrica-de-evaluación)
8. [Infraestructura AI/OpenCode](#8-infraestructura-aiopencode)
9. [Dependencias del Proyecto](#9-dependencias-del-proyecto)

---

## 1. Visión General del Sistema

Sistema modular para la **gestión integral de talleres mecánicos** con dos paquetes independientes:

| Package | Descripción |
|---------|-------------|
| `taller-mecanico-api/` | FastAPI + SQLAlchemy + Oracle Autonomous DB 23ai — REST API |
| `auto-hub-pro/` | React 19 + TanStack Router + TanStack Query + shadcn/ui — Frontend SPA |

### Capacidades del Sistema

- **Gestión de Clientes**: CRUD completo con validación de unicidad (email, teléfono)
- **Administración de Vehículos**: Catálogo por cliente con búsqueda por placa (UPPER index)
- **Órdenes de Servicio**: Ciclo de vida ABIERTA → EN_PROCESO → CERRADA con validación por compound trigger
- **Catálogo de Servicios**: Precios por hora + cálculo automático de totales
- **Seguridad por Capas**: JWT + bcrypt + bitmask permissions + Oracle wallet mTLS
- **Asistencia IA**: Estimación de horas de trabajo vía OpenAI GPT-4o-mini
- **Reportes**: Vistas materializadas con métricas diarias del taller
- **Multitenancy**: Aislamiento de datos por sesión (`ses_id`) con triggers automáticos

---

## 2. Arquitectura General

```
┌──────────────────────────────────────────────────────────────────┐
│                        CLIENTE (Browser)                         │
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐   │
│   │              auto-hub-pro (React 19 + TS)                 │   │
│   │                                                          │   │
│   │  ┌──────────┐  ┌──────────────┐  ┌───────────────────┐   │   │
│   │  │  Lucide  │  │  TanStack    │  │  Axios Client     │   │   │
│   │  │  Icons   │  │  Router      │  │  (JWT en Header)  │   │   │
│   │  └──────────┘  └──────────────┘  └─────────┬─────────┘   │   │
│   └─────────────────────────────────────────────┼─────────────┘   │
└─────────────────────────────────────────────────┼─────────────────┘
                                                   │
                                            ┌──────┴──────┐
                                            │   Internet   │
                                            │   (HTTPS)    │
                                            └──────┬──────┘
                                                   │
┌─────────────────────────────────────────────────┼─────────────────┐
│              taller-mecanico-api (FastAPI)       │                 │
│                                                  │                 │
│  ┌───────────────────────────────────────────────┴─────────────┐  │
│  │                    Middleware Layer                           │  │
│  │  ┌──────────────┐  ┌────────────────┐  ┌─────────────────┐  │  │
│  │  │  CORS        │  │  Logging       │  │  Error Handler  │  │  │
│  │  │  (multi-     │  │  Middleware    │  │  (val+db+gen)   │  │  │
│  │  │   origin)    │  │                │  │                  │  │  │
│  │  └──────────────┘  └────────────────┘  └─────────────────┘  │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    Auth Layer                                 │  │
│  │  ┌──────────────────┐  ┌────────────────────────────────┐   │  │
│  │  │  JWT (HS256)     │  │  Permisos (Bitmask 0xFF/0x07/  │   │  │
│  │  │  bcrypt hashing  │  │  0x03/0x01 via fn_tiene_permiso│   │  │
│  │  └──────────────────┘  └────────────────────────────────┘   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    API Routers (/api/v1)                      │  │
│  │  ┌────────┐ ┌───────────┐ ┌──────────┐ ┌────────┐ ┌──────┐  │  │
│  │  │ Auth   │ │ Clientes  │ │ Vehículos│ │ Órdenes│ │ IA   │  │  │
│  │  └────────┘ └───────────┘ └──────────┘ └────────┘ └──────┘  │  │
│  └──────────────────────┬───────────────────────────────────────┘  │
│                         │                                         │
│  ┌──────────────────────┴───────────────────────────────────────┐  │
│  │                    CRUD Layer (SQLAlchemy 2.x)                │  │
│  │    CRUDBase<T> → CRUDCliente|CRUDVehicle|CRUDOrden|...       │  │
│  └──────────────────────┬───────────────────────────────────────┘  │
│                         │                                         │
│  ┌──────────────────────┴───────────────────────────────────────┐  │
│  │           Oracle Database (Oracle Autonomous DB 23ai)         │  │
│  │                                                              │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────────┐  │  │
│  │  │ Tables   │ │ Views    │ │  SP/     │ │ Constraints/   │  │  │
│  │  │ (8)      │ │ (2+1 MV) │ │  Func.   │ │ Indexes        │  │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └────────────────┘  │  │
│  │                                                              │  │
│  │  Conexión: oracledb thin mode + Wallet (mTLS)                │  │
│  └──────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

### Flujo de Datos

```
[React UI] --Axios (JWT Bearer)--> [FastAPI Router] --SQLAlchemy--> [Oracle Cloud]
     ^                                      │                           │
     │                                      │                           │
     └──── JSON Response <──────────────────┴── Raw Result <────────────┘
```

---

## 3. Backend — taller-mecanico-api

### 3.1 Estructura del Backend

```
taller-mecanico-api/
│
├── app/
│   ├── main.py                  # FastAPI entry point, CORS, error handlers, routers
│   ├── dependencies.py          # DI: get_current_user(), require_permiso(bitmask)
│   │
│   ├── core/
│   │   ├── config.py            # Pydantic settings desde .env
│   │   ├── db.py                # Engine + Session Oracle (Wallet mTLS, pool 5-10)
│   │   └── errors.py            # Global exception handlers (val, db, gen)
│   │
│   ├── auth/
│   │   ├── jwt.py               # HS256 token creation/decode + bcrypt hashing
│   │   └── permissions.py       # Bitmask constants (0x01..0xFF)
│   │
│   ├── models/                  # SQLAlchemy 2.x ORM (8 modelos)
│   │   ├── cliente.py           # Client
│   │   ├── vehiculo.py          # Vehicle
│   │   ├── orden.py             # Order (ck_orden_estado + idx compuesto)
│   │   ├── orden_servicio.py    # ServiceOrder (UNIQUE compuesto)
│   │   ├── servicio.py          # Service
│   │   ├── sesion.py            # Session (ses_id)
│   │   ├── users.py             # User (JWT auth)
│   │   └── permiso_rol.py       # Rol+Bitmask (modelo BD)
│   │
│   ├── schemas/                 # Pydantic v2 (Base/Create/Out)
│   │   ├── cliente.py
│   │   ├── vehiculo.py
│   │   ├── orden.py
│   │   ├── servicio.py
│   │   ├── users.py
│   │   └── pagination.py        # Pagination<T> genérico
│   │
│   ├── crud/                    # Capa de acceso a datos
│   │   ├── base.py              # CRUDBase<T> genérico
│   │   ├── cliente.py           # CRUDCliente
│   │   ├── vehiculo.py          # CRUDVehicle
│   │   ├── orden.py             # CRUDOrden (con SAVEPOINT)
│   │   └── servicio.py          # CRUDService
│   │
│   ├── routers/                 # Endpoints REST
│   │   ├── auth.py              # /api/v1/auth/*
│   │   ├── clientes.py          # /api/v1/clients/*
│   │   ├── vehiculos.py         # /api/v1/vehiculos/*
│   │   ├── ordenes.py           # /api/v1/ordenes/*
│   │   ├── servicios.py         # /api/v1/servicios/*
│   │   └── ia.py                # /api/v1/ia/* (OpenAI)
│   │
│   ├── middleware/
│   │   └── logging_middleware.py # Request logging (method, path, status, ms)
│   │
│   └── services/
│       └── claude_service.py    # Integración OpenAI GPT-4o-mini
│
├── db/
│   ├── create_taller_app_user.sql  # Usuario BD con mínimos privilegios
│   └── README.md                   # Documentación de base de datos
│
├── alembic/                     # Migraciones
│   ├── versions/
│   │   ├── 0a18a24c8531_tablas_iniciales.py
│   │   ├── a1b2c3d4e5f6_correcciones_modelo.py
│   │   ├── b2c3d4e5f6a7_plsql_objects.py
│   │   ├── c3d4e5f6a7b8_seed_data.py
│   │   └── d4e5f6a7b8c9_roles_y_transaccion_distribuida.py
│   ├── env.py
│   └── script.py.mako
│
├── seeds/
├── wallet/                      # Oracle Wallet (mTLS)
├── test_oracle.py               # Test de conectividad Oracle
├── requirements.txt
├── alembic.ini
└── .env
```

### 3.2 Arquitectura en Capas

```
                    ┌─────────────────────────────┐
                    │      Cliente HTTP            │
                    │   (Axios + JWT Bearer)       │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │       main.py (FastAPI)      │
                    │  ┌─────────────────────────┐ │
                    │  │  CORS · Logging · Error  │ │
                    │  │   Handlers (val/db/gen)  │ │
                    │  └─────────────────────────┘ │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │      dependencies.py         │
                    │  get_current_user()          │
                    │  require_permiso(bitmask)    │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │         Routers              │
                    │  Validación → Auth → Lógica  │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │     CRUD Layer (SQLAlchemy)  │
                    │  CRUDBase<T> → específicos   │
                    │  SAVEPOINT (begin_nested)    │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │   Oracle 23ai (oracledb)     │
                    │  Tablas · Vistas · SP · Trig │
                    │  Wallet mTLS · Pool (5-10)   │
                    └─────────────────────────────┘
```

### 3.3 Endpoints de la API

Todas las rutas bajo `/api/v1/...`

| Método | Ruta | Descripción | Auth |
|--------|------|-------------|------|
| **Auth** | | | |
| POST | `/auth/register` | Registrar nuevo usuario | ❌ |
| POST | `/auth/login` | Iniciar sesión → JWT | ❌ |
| GET | `/auth/roles` | Listar roles con bitmask | ❌ |
| GET | `/auth/permisos/{rol}` | Obtener permisos de un rol | ❌ |
| **Clientes** | | | |
| GET | `/clients` | Listar clientes | ✅ |
| GET | `/clients/vista/resumen` | Vista `vw_resumen_clientes` | ✅ |
| GET | `/clients/{cli_id}` | Cliente por ID | ✅ |
| POST | `/clients` | Crear cliente | ✅ |
| PATCH | `/clients/{cli_id}` | Actualizar cliente | ✅ |
| DELETE | `/clients/{cli_id}` | Eliminar cliente | ✅ (permiso) |
| **Vehículos** | | | |
| GET | `/vehiculos` | Listar vehículos (paginado) | ✅ |
| GET | `/vehiculos/{veh_id}` | Vehículo por ID | ✅ |
| GET | `/vehiculos/cliente/{cli_id}` | Vehículos de un cliente | ✅ |
| POST | `/vehiculos` | Crear vehículo | ✅ |
| PATCH | `/vehiculos/{veh_id}` | Actualizar vehículo | ✅ |
| DELETE | `/vehiculos/{veh_id}` | Eliminar vehículo | ✅ (permiso) |
| **Órdenes** | | | |
| GET | `/ordenes` | Listar órdenes | ✅ |
| GET | `/ordenes/vista/activas` | Vista `vw_ordenes_activas` | ✅ |
| GET | `/ordenes/{ord_id}` | Orden por ID | ✅ |
| POST | `/ordenes` | Crear orden con servicios (SAVEPOINT) | ✅ |
| PATCH | `/ordenes/{ord_id}` | Actualizar orden | ✅ |
| DELETE | `/ordenes/{ord_id}` | Eliminar orden + servicios en cascada | ✅ (permiso) |
| PATCH | `/ordenes/{ord_id}/cerrar` | Ejecutar `sp_cerrar_orden` | ✅ (permiso) |
| GET | `/ordenes/{ord_id}/total` | Ejecutar `fn_calcular_total_orden` | ✅ |
| **Servicios** | | | |
| GET | `/servicios` | Listar servicios | ✅ |
| GET | `/servicios/{srv_id}` | Servicio por ID | ✅ |
| POST | `/servicios` | Crear servicio | ✅ |
| PATCH | `/servicios/{srv_id}` | Actualizar servicio | ✅ |
| DELETE | `/servicios/{srv_id}` | Eliminar servicio | ✅ (permiso) |
| **IA** | | | |
| POST | `/ia/estimar` | Estimar horas vía OpenAI | ✅ |

### 3.4 Control Transaccional

El sistema implementa **tres niveles de control transaccional**:

| Estrategia | Lugar | Uso | Efecto |
|-----------|-------|-----|--------|
| `db.commit()` | CRUD (Python) | Operaciones simples (crear/editar/eliminar un registro) | Persiste en BD |
| `db.begin_nested()` | CRUD (Python) | `create_with_services()` — múltiples inserts relacionados | Rollback parcial sin afectar transacción padre |
| `SAVEPOINT ... ROLLBACK TO` | Oracle PL/SQL | `sp_cerrar_orden` — validaciones + UPDATE crítico | Reversión atómica dentro del SP |
| `EXCEPTION ... ROLLBACK TO ... RAISE` | Oracle PL/SQL | `sp_cerrar_orden` | Manejo robusto de errores inesperados |

**SAVEPOINT en CRUD (Python):**
```python
def create_with_services(self, db: Session, order_data: dict, services: list[dict]):
    try:
        with db.begin_nested():  # ← SAVEPOINT
            db_order = Order(**order_data)
            db.add(db_order)
            db.flush()
            for srv in services:
                service = db.query(Service).get(srv["srv_id"])
                total = srv["ords_hours"] * service.srv_price_hour
                so = ServiceOrder(ord_id=db_order.ord_id, srv_id=srv["srv_id"],
                                  ords_hours=srv["ords_hours"], ords_total=total)
                db.add(so)
        db.commit()
    except Exception:
        db.rollback()
        raise
```

**SAVEPOINT en Oracle (sp_cerrar_orden):**
```sql
PROCEDURE sp_cerrar_orden(p_ord_id IN NUMBER, p_mensaje OUT VARCHAR2) IS
BEGIN
    SAVEPOINT sp_save;
    -- validaciones...
    UPDATE "Orders" SET ord_status = 'CERRADA' WHERE ord_id = p_ord_id;
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK TO sp_save;
        RAISE;
END;
```

**Transacción Distribuida Simulada:** `sp_transferir_servicio` usa `PRAGMA AUTONOMOUS_TRANSACTION` en `sp_log_transferencia` para simular un sistema remoto cuyo commit es independiente de la transacción principal.

### 3.5 Manejo de Errores

**Mapa de Errores Oracle → HTTP:**

| Error Oracle | Causa Típica | Código HTTP | Mensaje |
|-------------|-------------|-------------|---------|
| ORA-01722 | String en campo numérico | 400 | "Error de conversión numérica. Verifica los valores ingresados." |
| ORA-02290 | CHECK constraint violada | 400 | "Restricción violada: ck_orden_estado. Estado no válido." |
| ORA-00001 | UNIQUE violada | 409 | "Ya existe un registro con ese valor único." |
| ORA-02291 | FK no encontrada | 404 | "El registro relacionado no existe." |
| ORA-20001 | Compound trigger | 400 | "No se puede cerrar la orden: tiene servicios sin total." |

**Validación adicional en Schemas (Pydantic):**
```python
class VehicleCreate(VehicleBase):
    veh_year: int = Field(gt=1900, lt=2100)  # Rechazar antes de llegar a BD
```

### 3.6 Autenticación y Permisos

**JWT (HS256 + bcrypt):**
```python
def create_access_token(data: dict, expires_delta: timedelta = timedelta(hours=1)):
    expire = datetime.now(timezone.utc) + expires_delta
    to_encode = data.copy()
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, settings.SECRET_KEY, algorithm="HS256")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)
```

**Bitmask Permissions:**
```python
PERM_VER      = 0x01  # 0001
PERM_CREAR    = 0x02  # 0010
PERM_EDITAR   = 0x04  # 0100
PERM_ELIMINAR = 0x08  # 1000
PERM_ADMIN    = 0xFF  # 11111111
```

**Roles y Máscaras:**

| Rol | Máscara Hex | Binario | Permisos |
|-----|-------------|---------|----------|
| `admin` | 0xFF (255) | 11111111 | VER + CREAR + EDITAR + ELIMINAR |
| `mecanico` | 0x07 (7) | 00000111 | VER + CREAR + EDITAR |
| `recepcion` | 0x03 (3) | 00000011 | VER + CREAR |
| `cliente` | 0x01 (1) | 00000001 | VER |

**Validación como dependencia FastAPI:**
```python
def require_permiso(permiso: int):
    def checker(current_user=Depends(get_current_user), db=Depends(get_db)):
        if not tiene_permiso_bd(db, current_user.usr_rol, permiso):
            raise HTTPException(status_code=403, detail="Sin permisos suficientes")
        return current_user
    return checker
```

---

## 4. Base de Datos — Oracle 23ai

### 4.1 Modelo Relacional

**8 tablas en esquema:**

```
Sessions ── ses_id ──→ Clients ── cli_id ──→ Vehicles ── veh_id ──→ Orders
  │                     │                      │                      │
  │                     │ 1:N                  │ 1:N                  │ 1:N
  │                     │                      │                      │
  ├── ses_id ──→ users  │                      │                      ▼
  │                     ▼                      ▼              ServiceOrders
  │               (ses_id FK)            (ses_id FK)        ──────────────
  │              en todas las tablas                        srv_ord_id (PK)
  │              de negocio                                ord_id (FK) ──┐
  │                                                         srv_id (FK) ──┤
  │              permisos_rol                                ords_hours    │
  │              ────────────                                ords_total    │
  │              rol_id (PK)                                 ses_id (FK)   │
  │              rol_nombre (UQ)                              UQ(ord_id,   │
  │              permiso_bitmask                                srv_id)     │
  │                                                                        │
  │              Service ←── srv_id ────────────────────────────────────────┘
  │              ────────
  │              srv_id (PK)
  │              srv_name (UQ)
  │              srv_price_hour
  │              ses_id (FK)
  │
  └── (ses_id FK en todas las tablas de negocio)
```

**Convenciones de Nomenclatura:**

| Elemento | Convención | Ejemplo |
|----------|------------|---------|
| Tablas | PascalCase con comillas dobles | `"Clients"`, `"Orders"` |
| Columnas | snake_case con prefijo de tabla | `cli_name`, `veh_plate` |
| PK | `{tabla}_id` | `cli_id`, `ord_id` |
| FK | mismo nombre que PK referenciada | `cli_id`, `ses_id` |
| Timestamp mod | `{tabla}_date_mod` | `cli_date_mod` |
| CHECK | `ck_{tabla}_{prop}` | `ck_orden_estado` |
| Índice | `idx_{tabla}_{cols}` | `idx_ordenes_status_fecha` |
| Trigger | `trg_{tabla}_{accion}` | `trg_clients_ses_id` |
| Vista | `vw_{desc}` | `vw_resumen_clientes` |
| MV | `vm_{desc}` | `vm_metricas_taller` |
| Función | `fn_{desc}` | `fn_tiene_permiso` |
| Procedimiento | `sp_{accion}_{obj}` | `sp_cerrar_orden` |

### 4.2 Objetos PL/SQL

| Objeto | Tipo | Propósito |
|--------|------|-----------|
| `sp_cerrar_orden` | PROCEDURE | Cierre transaccional con SAVEPOINT, COMMIT/ROLLBACK, OUT param |
| `sp_transferir_servicio` | PROCEDURE | Transacción distribuida simulada con PRAGMA AUTONOMOUS_TRANSACTION |
| `sp_log_transferencia` | PROCEDURE | Log autónomo (simula sistema remoto) |
| `fn_calcular_total_orden` | FUNCTION | Suma de totales por orden |
| `fn_tiene_permiso` | FUNCTION | Validación de bitmask permissions vía BITAND |

**`sp_cerrar_orden`** — Procedimiento estrella:
- Parámetro IN `p_ord_id`, OUT `p_mensaje`
- SAVEPOINT al inicio, COMMIT en éxito, ROLLBACK TO SAVEPOINT en error
- Validaciones: orden no cerrada, servicios sin total
- `EXCEPTION WHEN OTHERS` con rollback y propagación

**`sp_transferir_servicio`** — Demostración de transacción distribuida:
- SAVEPOINT + validación de orden destino no cerrada
- Llama a `sp_log_transferencia` con `PRAGMA AUTONOMOUS_TRANSACTION`
- Demuestra el riesgo distribuido: la TX autónoma COMMITea independientemente

### 4.3 Vistas

**`vw_resumen_clientes`** (Vista Compleja):
- Subconsulta escalar: `(SELECT COUNT(*) FROM Vehicles ...)`
- LEFT JOIN + subconsulta en JOIN
- CASE + SUM para conteo condicional
- GROUP BY para métricas consolidadas

**`vw_ordenes_activas`** (Vista Simple):
```sql
CREATE VIEW vw_ordenes_activas AS
SELECT ord_id, veh_id, ord_status, ord_date, ord_urgency
FROM "Orders"
WHERE ord_status IN ('ABIERTA', 'EN_PROCESO');
```

**`vm_metricas_taller`** (Vista Materializada):
```sql
CREATE MATERIALIZED VIEW vm_metricas_taller
BUILD IMMEDIATE REFRESH COMPLETE ON DEMAND AS
SELECT TRUNC(ord_date) AS dia, COUNT(*) AS total_ordenes,
       SUM(CASE WHEN ord_status = 'CERRADA' THEN 1 ELSE 0 END) AS cerradas
FROM "Orders" GROUP BY TRUNC(ord_date);
```

### 4.4 Triggers

**6 Triggers de Auditoría (`ses_id`):**
```sql
CREATE OR REPLACE TRIGGER trg_clients_ses_id
BEFORE INSERT ON "Clients" FOR EACH ROW
BEGIN
    IF :NEW.ses_id IS NULL THEN
        :NEW.ses_id := SYS_CONTEXT('USERENV', 'SESSIONID');
    END IF;
END;
```
Cubren: `Clients`, `Vehicles`, `Orders`, `ServiceOrders`, `Service`, `users`

**Compound Trigger `TRG_VALIDAR_CIERRE_ORDEN`:**
```sql
FOR UPDATE OF ord_status ON "Orders" COMPOUND TRIGGER
```
Recolecta `ord_id` en `BEFORE EACH ROW`, valida servicios sin total en `AFTER STATEMENT`. Resuelve el problema de **tabla mutante** (Oracle no permite leer `ServiceOrders` en un `BEFORE ROW` sobre `Orders`).

### 4.5 Restricciones e Índices

**CHECK Constraints:**

| Nombre | Tabla | Expresión |
|--------|-------|-----------|
| `ck_orden_estado` | Orders | `ord_status IN ('ABIERTA','EN_PROCESO','CERRADA')` |
| `ck_vehiculo_anio` | Vehicles | `veh_year > 1900` |
| `ck_ords_horas_positivas` | ServiceOrders | `ords_hours > 0` |
| `ck_servicio_precio` | Service | `srv_price_hour > 0` |

**UNIQUE Constraints:** email, teléfono, placa, nombre servicio, username, **UNIQUE compuesto** `(ord_id, srv_id)`

**Índices:**
- **Compuesto:** `idx_ordenes_status_fecha` on `(ord_status, ord_date)` — optimiza dashboard de órdenes activas
- **Funcional:** `idx_vehiculo_upper_plate` on `UPPER(veh_plate)` — búsqueda case-insensitive por placa

### 4.6 Roles y Permisos (Bitmask)

```
PERM_VER      = 0x01  (0001)  →  Consultar
PERM_CREAR    = 0x02  (0010)  →  Crear registros
PERM_EDITAR   = 0x04  (0100)  →  Editar registros
PERM_ELIMINAR = 0x08  (1000)  →  Eliminar registros
PERM_ADMIN    = 0xFF  (11111111) →  Todos los permisos
```

**Función `fn_tiene_permiso`:**
```sql
FUNCTION fn_tiene_permiso(p_rol VARCHAR2, p_permiso NUMBER) RETURN NUMBER IS
    v_mask NUMBER(3);
BEGIN
    SELECT permiso_bitmask INTO v_mask FROM permisos_rol WHERE rol_nombre = p_rol;
    IF BITAND(v_mask, p_permiso) = p_permiso THEN RETURN 1; ELSE RETURN 0; END IF;
EXCEPTION WHEN NO_DATA_FOUND THEN RETURN 0;
END;
```

### 4.7 Usuario de Base de Datos

Principio de **mínimos privilegios** mediante **rol de aplicación** (no grants directos al usuario):

```sql
CREATE ROLE taller_app_rol IDENTIFIED BY "TallerApp2026!";
GRANT CONNECT, CREATE SESSION TO taller_app_rol;
-- DML en 8 tablas de negocio
GRANT SELECT, INSERT, UPDATE, DELETE ON "Clients" TO taller_app_rol;
-- ... (vehículos, órdenes, servicios, etc.)
-- Ejecución de PL/SQL
GRANT EXECUTE ON fn_tiene_permiso TO taller_app_rol;
GRANT EXECUTE ON fn_calcular_total_orden TO taller_app_rol;
GRANT EXECUTE ON sp_cerrar_orden TO taller_app_rol;
GRANT EXECUTE ON sp_transferir_servicio TO taller_app_rol;
GRANT EXECUTE ON sp_log_transferencia TO taller_app_rol;
-- Consulta de vistas
GRANT SELECT ON vw_resumen_clientes TO taller_app_rol;
GRANT SELECT ON vw_ordenes_activas TO taller_app_rol;
GRANT SELECT ON vm_metricas_taller TO taller_app_rol;
-- Usuario recibe el rol (no grants directos)
CREATE USER taller_app IDENTIFIED BY "TallerApp2026!";
GRANT taller_app_rol TO taller_app;
```

### 4.8 Seed Data

| Tabla | Filas | Detalle |
|-------|-------|---------|
| `Sessions` | 3 | admin, mecanico, recepcion |
| `Clients` | 3 | Juan Pérez, María García, Carlos López |
| `Vehicles` | 5 | 2 de Juan, 2 de María, 1 de Carlos |
| `Service` | 4 | Cambio de Aceite, Frenos, Alineación, Diagnóstico |
| `Orders` | 3 | 1 ABIERTA, 1 EN_PROCESO, 1 CERRADA |
| `ServiceOrders` | 8 | 2-3 servicios por orden (1 con `ords_total = NULL`) |
| `users` | 3 | admin/admin123, mecanico/mec123, recepcion/rec123 (bcrypt) |
| `permisos_rol` | 4 | admin(0xFF), mecanico(0x07), recepcion(0x03), cliente(0x01) |

### 4.9 Migraciones (Alembic)

| Archivo | Contenido |
|---------|-----------|
| `0a18a24c8531_tablas_iniciales.py` | Creación de 7 tablas con SQLAlchemy |
| `a1b2c3d4e5f6_correcciones_modelo.py` | Renombrar columnas, CHECK, índices, UNIQUE compuesto |
| `b2c3d4e5f6a7_plsql_objects.py` | Todos los objetos PL/SQL (SP, funciones, vistas, triggers) |
| `c3d4e5f6a7b8_seed_data.py` | Seed data de prueba |
| `d4e5f6a7b8c9_roles_y_transaccion_distribuida.py` | Transacción distribuida simulada + roles |

---

## 5. Frontend — auto-hub-pro

### 5.1 Estructura del Frontend

```
auto-hub-pro/
├── src/
│   ├── api/                       # Módulos API (Axios)
│   │   ├── axios.ts               # Axios instance + interceptores JWT
│   │   ├── clientes.ts            # Cliente CRUD
│   │   ├── ordenes.ts             # Órdenes CRUD
│   │   ├── servicios.ts           # Servicios CRUD
│   │   ├── vehiculos.ts           # Vehículos CRUD
│   │   └── ia.ts                  # Diagnóstico IA
│   ├── components/
│   │   ├── ui/                    # 46 componentes shadcn/ui
│   │   │   ├── button.tsx, input.tsx, select.tsx, table.tsx
│   │   │   ├── dialog.tsx, card.tsx, badge.tsx, skeleton.tsx
│   │   │   └── ... (38 más)
│   │   ├── DataTable.tsx          # Tabla genérica con paginación
│   │   ├── Layout.tsx             # Layout (Sidebar + Topbar)
│   │   ├── Modal.tsx              # Modal reutilizable
│   │   ├── Sidebar.tsx            # Navegación lateral
│   │   ├── StatusBadge.tsx        # Badge de estado
│   │   └── Topbar.tsx             # Barra superior
│   ├── routes/
│   │   ├── __root.tsx             # Root layout + Auth guard
│   │   ├── index.tsx              # Dashboard
│   │   ├── login.tsx              # Inicio de sesión
│   │   ├── clientes.tsx           # Gestión de clientes
│   │   ├── vehiculos.tsx          # Gestión de vehículos
│   │   ├── ordenes.tsx            # Órdenes de servicio
│   │   ├── servicios.tsx          # Catálogo de servicios
│   │   ├── reportes.tsx           # Reportes y métricas
│   │   └── configuracion.tsx      # Configuración
│   ├── types/index.ts             # Interfaces TypeScript
│   ├── hooks/use-mobile.tsx       # Mobile detection
│   ├── lib/
│   │   ├── utils.ts               # cn() utility
│   │   ├── error-capture.ts       # SSR error capture
│   │   ├── error-page.ts          # HTML error page
│   │   └── mock-data.ts           # Mock data
│   ├── styles.css                 # Tailwind v4 + design tokens + dark mode
│   ├── router.tsx                 # Router factory
│   ├── routeTree.gen.ts           # Route tree (auto-generado)
│   ├── server.ts                  # SSR entry (Cloudflare)
│   └── start.ts                   # TanStack Start
├── package.json
├── vite.config.ts
├── tsconfig.json
└── .env
```

### 5.2 Arquitectura de la UI

```
┌──────────────────────────────────────────────────────────────┐
│                     __root.tsx (Auth Guard)                    │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              QueryClientProvider (TanStack Query)        │  │
│  │              ┌──────────────────────────────────────┐    │  │
│  │              │         Router (TanStack Router)      │    │  │
│  │              │    ┌──────────────────────────────┐  │    │  │
│  │              │    │        Layout.tsx             │  │    │  │
│  │              │    │  ┌──────┐ ┌──────┐ ┌──────┐  │  │    │  │
│  │              │    │  │Sidebar│ │Topbar│ │Main  │  │  │    │  │
│  │              │    │  │(nav) │ │(user)│ │content│  │  │    │  │
│  │              │    │  └──────┘ └──────┘ └──────┘  │  │    │  │
│  │              │    └──────────────────────────────┘  │    │  │
│  │              └──────────────────────────────────────┘    │  │
│  └─────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

**Diseño Visual:**
- **46 componentes shadcn/ui** sobre Radix UI con accesibilidad WAI-ARIA
- **Glassmorphism**: Paneles semitransparentes con `backdrop-filter: blur(12px)`
- **Tema oscuro**: Clase `.dark` con paleta completa de colores invertida
- **Responsive**: Layout adaptable a tablets y móviles (390×844 a 1280×800)
- **TanStack Query**: Cache configurado con `staleTime: 30_000`, `retry: 1`

### 5.3 Rutas y Páginas

| Ruta | Componente | Descripción |
|------|-----------|-------------|
| `/login` | `LoginPage` | Autenticación con JWT |
| `/` | `Index` | Dashboard con métricas |
| `/clientes` | `ClientesPage` | CRUD de clientes |
| `/vehiculos` | `VehiculosPage` | CRUD de vehículos |
| `/ordenes` | `OrdenesPage` | Órdenes + IA (estimación horas) |
| `/servicios` | `ServiciosPage` | Catálogo de servicios |
| `/reportes` | `ReportesPage` | Reportes y métricas |
| `/configuracion` | (inline) | Configuración |

### 5.4 Consumo de API

**Interceptor Axios (JWT automático):**
```typescript
const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,  // http://localhost:8000/api/v1
  headers: { "Content-Type": "application/json" },
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401 && localStorage.getItem("token")) {
      localStorage.removeItem("token");
      window.location.href = "/login";
    }
    return Promise.reject(error);
  }
);
```

**Módulos API:** `clientes.ts`, `vehiculos.ts`, `ordenes.ts`, `servicios.ts`, `ia.ts`

### 5.5 Aislamiento de Datos (Multitenancy)

**Arquitectura de Aislamiento por Sesión:**

```
[Usuario Login] → POST /auth/login → JWT { sub, rol, ses_id }
                                                         ↓
                                  localStorage → Axios interceptor → Authorization: Bearer <token>
                                                         ↓
                                  FastAPI → get_current_user() → filtra CRUD queries por ses_id
                                                         ↓
                                  Oracle → trg_*_ses_id asigna SYS_CONTEXT('USERENV','SESSIONID')
```

- Cada request HTTP lleva el JWT con `ses_id`
- Backend filtra toda query por `Vehicle.ses_id == current_user.ses_id`
- Triggers en BD asignan `ses_id` automáticamente en INSERT
- El usuario SOLO ve datos de su propio taller

### 5.6 Manejo de Errores Visual

| Escenario | HTTP | Feedback | Componente |
|-----------|------|----------|------------|
| Datos inválidos | 422 | Mensaje en formulario | Texto `text-destructive` |
| Registro duplicado | 409 | Alerta en modal | Modal + mensaje |
| Violación CHECK | 500 | Mensaje descriptivo | Badge + mensaje |
| No autorizado | 401 | Redirect a `/login` | Interceptor global |
| Sin permisos | 403 | "No tienes permisos" | Texto `text-destructive` |
| Error de red | 0 | "Error de conexión" | Texto `text-destructive` |
| No encontrado | 404 | "No existe" | Texto en página |

**Patrón de manejo en páginas:**
```tsx
const [error, setError] = useState<string | null>(null);
const [saving, setSaving] = useState(false);

const handleSave = async () => {
  try {
    setSaving(true); setError(null);
    await createCliente(data);
    setModalOpen(false); fetchData();
  } catch (e: any) {
    const msg = e?.response?.data?.detail ?? "Error al guardar. Intenta de nuevo.";
    setError(msg);
  } finally { setSaving(false); }
};
```

---

## 6. Guía de Inicio Rápido

### Backend

```bash
cd taller-mecanico-api
python -m venv venv
.\venv\Scripts\Activate.ps1   # Windows
pip install -r requirements.txt
```

Configurar `taller-mecanico-api/.env`:
```env
DB_USER=taller_app
DB_PASS=TuPasswordSeguro
DB_HOST=adb.us-ashburn-1.oraclecloud.com
DB_PORT=1522
DB_SERVICE=tu_servicio_low.adb.oraclecloud.com
DB_DSN=...
WALLET_LOCATION=wallet
WALLET_PASSWORD=WalletPassword
SECRET_KEY=your-super-secret-key-change-me
OPENAI_API_KEY=sk-...
```

```bash
# Verificar conexión Oracle
python test_oracle.py

# Migraciones
alembic upgrade head

# Iniciar servidor
uvicorn app.main:app --reload --port 8000
```

**URLs:** API: `http://localhost:8000/api/v1` | Swagger: `http://localhost:8000/docs`

### Frontend

```bash
cd auto-hub-pro
npm install
```

Configurar `auto-hub-pro/.env`:
```env
VITE_API_URL=http://localhost:8000/api/v1
```

```bash
npm run dev
# UI: http://localhost:5173
```

### Credenciales de Prueba

| Usuario | Contraseña | Rol | Permisos |
|---------|-----------|-----|----------|
| `admin` | `admin123` | admin | Todos (0xFF) |
| `mecanico` | `mec123` | mecanico | Ver + Crear + Editar (0x07) |
| `recepcion` | `rec123` | recepcion | Ver + Crear (0x03) |

---

## 7. Rúbrica de Evaluación

| # | Criterio | Estado | Implementación |
|---|----------|--------|----------------|
| 1 | **Diseño modelo relacional (3FN)** | ✅ | Llaves sustitutas, sin redundancia, FK correctas, `GENERATED BY DEFAULT AS IDENTITY` |
| 2 | **Calidad UI/UX** | ✅ | shadcn/ui, responsive, dark mode, carga/error/empty states, glass design |
| 3 | **Tablas auditables** | ✅ | 8 tablas con ≥3 registros cada una, FKs con integridad referencial |
| 4 | **Relación auditoría ses_id** | ✅ | 6 triggers BEFORE INSERT auto-asignan `SYS_CONTEXT('USERENV','SESSIONID')` |
| 5 | **Roles y permisos** | ✅ | Bitmask (0x01/0x02/0x04/0x08), `fn_tiene_permiso` con BITAND, 4 roles |
| 6 | **Restricciones e índices** | ✅ | 4 CHECK, 5 UNIQUE, 1 UNIQUE compuesto, índice compuesto + funcional |
| 7 | **Vistas** | ✅ | `vw_ordenes_activas` (simple), `vw_resumen_clientes` (compleja), `vm_metricas_taller` (materializada) |
| 8 | **Procedimiento almacenado** | ✅ | `sp_cerrar_orden` (SAVEPOINT+COMMIT/ROLLBACK+OUT), `sp_transferir_servicio` (TX distribuida) |
| 9 | **Función definida por usuario** | ✅ | `fn_calcular_total_orden` (suma), `fn_tiene_permiso` (BITAND), llamadas desde SELECT |
| 10 | **Trigger** | ✅ | 6 ses_id triggers + `TRG_VALIDAR_CIERRE_ORDEN` (compound trigger antimutante) |
| 11 | **Control transaccional app** | ✅ | `db.begin_nested()` + SAVEPOINT en SP + PRAGMA AUTONOMOUS TRANSACTION |
| 12 | **Conexión Oracle 23ai** | ✅ | Wallet mTLS, oracledb thin mode, pool 5-10, SELECT/INSERT/UPDATE/DELETE/callproc |
| 13 | **Control de acceso BD** | ✅ | Usuario `taller_app` con rol de aplicación, mínimo privilegio, sin DDL |

---

## 8. Infraestructura AI/OpenCode

### Protocolo "No Excuses" (OPENCODE.md)

1. **Graph First**: Consultar `graphify-out/GRAPH_REPORT.md` antes de escribir código
2. **Mandatory Delegation**: Cargar agente especializado según el paquete
3. **Skill Utilization**: Verificar `.ai/skills/` antes de implementar lógica custom
4. **Memory Check**: Leer `.ai/persistence/lessons-learned.md` antes de ejecutar
5. **Auto-Adaptation**: Ejecutar `graphify update .` tras modificaciones

### Agentes Especializados

| Agente | Rol | Modelo | Cuándo usar |
|--------|-----|--------|-------------|
| `backend-developer` | Constructor | DeepSeek V4 Flash | FastAPI, schemas, CRUD, auth |
| `tanstack-wizard` | Constructor | DeepSeek V4 Flash | React, TanStack, shadcn/ui |
| `db-architect` | Arquitecto | Qwen 3.5 Plus | Oracle schema, migraciones, PL/SQL |
| `security-code-reviewer` | Auditor | MiMo V2.5 Pro | ses_id, auth, permisos — pre-merge |
| `project-explorer` | Navegador | Kimi K2.6 | Conocimiento del código, graph |
| `test-engineer` | QA Lead | MiniMax M2.7 | pytest, Vitest, Playwright |

### Skills Disponibles

- `oracle-sqlalchemy.md` — Patrones Oracle (VARCHAR2, NUMBER, TIMESTAMP, Identity)
- `permissions-bitmask.md` — Implementación de bitmask permissions
- `monorepo-conventions.md` — Convenciones del monorepositorio
- `explore-codebase.md` — Protocolo de exploración
- `workflow-execute-phase.md` — Protocolo de ejecución con dispatch de agentes

### Flujo de Features

| Comando | Propósito |
|---------|-----------|
| `/spec <idea>` | Crea spec en `_specs/` + feature branches |
| `/plan-spec <slug>` | Lee spec → genera plan en `_plans/` |
| `/execute-plan <slug>` | Ejecuta plan con agentes especializados |

### Oracle Persistence Rules

- Usar `VARCHAR2(n)` en lugar de `VARCHAR`
- Usar `NUMBER(p,s)` para decimales, nunca `FLOAT` o `REAL`
- Usar `TIMESTAMP` en lugar de `DATETIME`
- Identity columns: `GENERATED BY DEFAULT AS IDENTITY`
- Bind variables (`:param`) en raw SQL, nunca f-string interpolation

---

## 9. Dependencias del Proyecto

### Backend (Python)

| Paquete | Propósito |
|---------|-----------|
| `fastapi` | Framework web asíncrono |
| `uvicorn` | Servidor ASGI |
| `sqlalchemy` ^2.0 | ORM para Oracle |
| `oracledb` | Driver Oracle (thin mode, sin cliente Oracle) |
| `alembic` | Migraciones de base de datos |
| `pydantic` ^2.0 | Validación de datos (schemas) |
| `python-jose` | JWT (HS256) |
| `passlib[bcrypt]` | Hashing de contraseñas |
| `python-multipart` | Soporte para form-data |
| `openai` | Integración GPT-4o-mini |
| `pydantic-settings` | Configuración desde .env |

### Frontend (Node.js)

| Paquete | Versión | Propósito |
|---------|---------|-----------|
| `react` | ^19.2.0 | UI Framework |
| `@tanstack/react-router` | latest | Enrutamiento SPA + SSR |
| `@tanstack/react-query` | latest | Cache y estados de API |
| `axios` | latest | Cliente HTTP con interceptores |
| `lucide-react` | latest | Iconos profesionales |
| `tailwindcss` | ^4.x | CSS utility-first |
| `shadcn/ui` | latest | 46 componentes sobre Radix UI |
| `react-hook-form` | ^7.71.2 | Manejo de formularios |
| `sonner` | latest | Toast notifications |
| `date-fns` | latest | Manipulación de fechas |
| `recharts` | latest | Gráficas del dashboard |

---

## Documentación Relacionada

| Archivo | Contenido |
|---------|-----------|
| `README.md` | Visión general, arquitectura, rúbrica, quick start |
| `taller-mecanico-api/README.md` | Documentación detallada del backend |
| `auto-hub-pro/README.md` | Documentación detallada del frontend |
| `taller-mecanico-api/db/README.md` | Documentación completa de base de datos |
| `ExplicacionUlloa.md` | Mapeo rubrica → ubicaciones exactas en código |
| `_specs/maximizar-calificacion-rubrica.md` | Especificación técnica para maximizar rúbrica |
| `_plans/maximizar-calificacion-rubrica.md` | Plan de implementación detallado |
| `.ai/persistence/lessons-learned.md` | Lecciones aprendidas y bugs resueltos |

---

<div align="center">
  <sub>Documentación completa del Sistema Integral de Gestión para Taller Mecánico</sub>
  <br/>
  <sub>FastAPI · Oracle 23ai · React 19 · TypeScript · Tailwind v4 · shadcn/ui · TanStack Router</sub>
  <br/>
  <sub>Desarrollado por Jose Andres Monrreal Muñoz — ITL — 2026</sub>
</div>
