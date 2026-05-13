# Sistema Integral de Gestión — Taller Mecánico

> **Plataforma empresarial para la administración de servicios automotrices**  
> Backend: FastAPI + Oracle 23ai | Frontend: React 19 + TanStack Router | Auth: JWT + Bitmask Permissions

---

## Índice

- [Arquitectura General](#arquitectura-general)
- [Propósito del Sistema](#propósito-del-sistema)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Guía de Inicio Rápido](#guía-de-inicio-rápido)
- [Rúbrica de Evaluación](#rúbrica-de-evaluación)
- [Créditos](#créditos)

---

## Arquitectura General

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
│  │  │ (7)      │ │ (2+1 MV) │ │  Func.   │ │ Indexes        │  │  │
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

## Propósito del Sistema

Sistema modular para la **gestión integral de talleres mecánicos**, diseñado con:

| Dimensión | Detalle |
|-----------|---------|
|  **Gestión de Clientes** | CRUD completo con validación de unicidad (email, teléfono) |
|  **Administración de Vehículos** | Catálogo por cliente con búsqueda por placa (UPPER index) |
|  **Órdenes de Servicio** | Ciclo de vida ABIERTA → EN_PROCESO → CERRADA con validación por compound trigger |
|  **Catálogo de Servicios** | Precios por hora + cálculo automático de totales |
|  **Seguridad por Capas** | JWT + bcrypt + bitmask permissions + Oracle wallet mTLS |
|  **Asistencia IA** | Estimación de horas de trabajo vía OpenAI GPT-4o-mini |
|  **Reportes** | Vistas materializadas con métricas diarias del taller |

---

## Estructura del Proyecto

```
Ulloa/
│
├── README.md                          # ← Este archivo (Global)
│
├── taller-mecanico-api/               # Backend (FastAPI + Oracle)
│   ├── app/
│   │   ├── main.py                    # Punto de entrada FastAPI
│   │   ├── dependencies.py            # Inyección de dependencias (auth)
│   │   ├── core/
│   │   │   ├── config.py              # Settings desde .env
│   │   │   ├── db.py                  # Engine + Session Oracle (Wallet)
│   │   │   └── errors.py              # Manejadores globales de excepción
│   │   ├── auth/
│   │   │   ├── jwt.py                 # HS256 tokens + bcrypt
│   │   │   └── permissions.py         # Bitmask constants (0x01..0xFF)
│   │   ├── models/                    # SQLAlchemy 2.x ORM (8 tablas)
│   │   ├── schemas/                   # Pydantic v2 (Base/Create/Out)
│   │   ├── crud/                      # Generic CRUD<T> + específicos
│   │   ├── routers/                   # 6 routers (auth, clientes, vehiculos, ordenes, servicios, ia)
│   │   ├── middleware/
│   │   │   └── logging_middleware.py  # Request logging (method, path, status, ms)
│   │   └── services/
│   │       └── claude_service.py      # AI integration (OpenAI)
│   ├── db/
│   │   ├── create_taller_app_user.sql # Usuario con privilegios mínimos
│   │   └── README.md                  # ← Documentación de base de datos
│   ├── alembic/
│   │   ├── versions/
│   │   │   ├── 0a18a24c8531_tablas_iniciales.py
│   │   │   ├── a1b2c3d4e5f6_correcciones_modelo.py
│   │   │   ├── b2c3d4e5f6a7_plsql_objects.py
│   │   │   └── c3d4e5f6a7b8_seed_data.py
│   │   ├── env.py
│   │   └── script.py.mako
│   ├── seeds/
│   ├── wallet/                        # Oracle Wallet (mTLS)
│   ├── requirements.txt
│   ├── alembic.ini
│   └── .env
│
├── auto-hub-pro/                      # Frontend (React 19 + TS)
│   ├── src/
│   │   ├── api/                       # Axios instance + 5 módulos API
│   │   ├── components/
│   │   │   ├── ui/                    # 46 shadcn/ui components
│   │   │   ├── DataTable.tsx          # Tabla genérica con paginación
│   │   │   ├── Layout.tsx             # Layout principal (Sidebar + Topbar)
│   │   │   ├── Modal.tsx              # Modal reutilizable
│   │   │   ├── Sidebar.tsx            # Navegación lateral
│   │   │   ├── StatusBadge.tsx        # Badge de estado (ABIERTA/EN_PROCESO/CERRADA)
│   │   │   └── Topbar.tsx             # Barra superior
│   │   ├── routes/                    # 8 páginas (TanStack Router)
│   │   ├── types/                     # TypeScript interfaces
│   │   ├── hooks/                     # Custom hooks
│   │   ├── lib/                       # Utilidades (cn, error-page, mock-data)
│   │   ├── styles.css                 # Tailwind v4 + glass design tokens
│   │   ├── router.tsx
│   │   └── start.ts
│   ├── package.json
│   ├── vite.config.ts
│   └── .env
│
├── _specs/                            #  Especificaciones técnicas
│   └── maximizar-calificacion-rubrica.md
├── _plans/                            #  Planes de implementación
│   └── maximizar-calificacion-rubrica.md
├── .ai/                               #  Configuración de agentes opencode
│   ├── agents/                        # 6 agentes especializados
│   ├── skills/                        # 4 skills (Oracle, bitmask, etc.)
│   └── persistence/
│       └── lessons-learned.md
└── graphify-out/                      #  Dependency graph (AST)
```

---

## Guía de Inicio Rápido

### 1. Clonar y preparar entorno

```bash
# Backend
cd taller-mecanico-api

# Crear entorno virtual
python -m venv venv

# Activar (Windows PowerShell)
.\venv\Scripts\Activate.ps1

# Instalar dependencias
pip install -r requirements.txt
```

### 2. Configurar variables de entorno

Crear archivo `taller-mecanico-api/.env`:

```env
DB_USER=taller_app
DB_PASS=TuPasswordSeguro
DB_HOST=adb.us-ashburn-1.oraclecloud.com
DB_PORT=1522
DB_SERVICE=xxxxxxxxxxxxxx_low.adb.oraclecloud.com
DB_DSN=...

WALLET_LOCATION=wallet
WALLET_PASSWORD=WalletPassword

SECRET_KEY=your-super-secret-key-change-me
OPENAI_API_KEY=sk-...
```

### 3. Conectar Wallet de Oracle

```bash
# Descargar Wallet desde Oracle Cloud Console
# Extraer en: taller-mecanico-api/wallet/
# El wallet contiene: tnsnames.ora, sqlnet.ora, cwallet.sso, ewallet.p12, etc.

# Verificar conexión
python test_oracle.py
```

### 4. Ejecutar migraciones

```bash
alembic upgrade head
```

### 5. Iniciar servidor backend

```bash
uvicorn app.main:app --reload --port 8000
# API disponible en: http://localhost:8000/api/v1
# Docs: http://localhost:8000/docs
```

### 6. Iniciar frontend

```bash
cd auto-hub-pro
npm install
npm run dev
# UI disponible en: http://localhost:5173
```

---

## Rúbrica de Evaluación

| Criterio | Estado | Implementación |
|----------|--------|----------------|
| **Procedimiento Almacenado** | `sp_cerrar_orden` — SAVEPOINT, COMMIT/ROLLBACK, EXCEPTION, OUT param |
| **Función PL/SQL** |  `fn_calcular_total_orden` — suma de totales por orden |
| **Compound Trigger** | `trg_validar_cierre_orden` — evita cerrar orden con servicios pendientes |
| **Trigger Auditoría** |  6 triggers `trg_*_ses_id` — auto-asignación de `ses_id` vía `SYS_CONTEXT` |
| **Vista Compleja** |  `vw_resumen_clientes` — JOINs + subconsulta + CASE |
| **Vista Simple** |  `vw_ordenes_activas` — filtro WHERE |
| **Vista Materializada** |  `vm_metricas_taller` — refresh on demand |
| **CHECK Constraints** |  `ck_orden_estado`, `ck_vehiculo_anio`, `ck_ords_horas_positivas`, `ck_servicio_precio` |
| **Índice Funcional** |  `idx_vehiculo_upper_plate` on `UPPER(veh_plate)` |
| **Índice Compuesto** |  `idx_ordenes_status_fecha` on `(ord_status, ord_date)` |
| **Roles y Permisos (Bitmask)** |  `permisos_rol` tabla + `fn_tiene_permiso` con `BITAND` + hex masks (0xFF, 0x07, 0x03, 0x01) |
| **JWT Auth** |  HS256 tokens + bcrypt + interceptor Axios |
| **Multitenancy (ses_id)** |  Trigger asigna automáticamente; columnas en todas las tablas de negocio |
| **Transacciones (SAVEPOINT)** |  `db.begin_nested()` en `crud/orden.py` y SAVEPOINT en `sp_cerrar_orden` |
| **Usuario Mínimos Privilegios** |  `taller_app` — solo DML + EXECUTE, sin DDL |
| **Seed Data** | 3 sesiones, 3 clientes, 5 vehículos, 4 servicios, 3 órdenes, 3 usuarios |

---

## Créditos

| | |
|---|---|
| **Desarrollado por** | **Jose Andres Monrreal Muñoz** |
| **Institución** | Instituto Tecnológico de Linares (ITL) |
| **Tecnologías** | FastAPI · Oracle 23ai · React 19 · TypeScript · Tailwind v4 · TanStack Router |
| **Base de Datos** | Oracle Autonomous Database (23ai) con wallet mTLS |
| **Año** | 2026 |

---

<div align="center">
  <sub>Proyecto académico — Sistema Integral de Gestión para Taller Mecánico</sub>
</div>
