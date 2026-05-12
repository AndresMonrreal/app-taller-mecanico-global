# oracle-sqlalchemy

Skill para trabajar con Oracle Autonomous Database vía SQLAlchemy 2.x + oracledb en modo thin. Aplica siempre que toques modelos, migraciones o consultas de base de datos.

## Conexión

- Driver: `oracledb` en **modo thin** (sin Oracle Client instalado)
- ORM: SQLAlchemy 2.x con sintaxis declarativa moderna
- La sesión se obtiene SIEMPRE vía dependency de FastAPI, nunca se instancia manualmente en routers ni services

```python
# db/session.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase
from app.core.config import settings

engine = create_engine(
    settings.DATABASE_URL,
    echo=False,          # True solo en desarrollo
    pool_pre_ping=True,  # evita conexiones muertas
    pool_size=5,
    max_overflow=10,
)

SessionLocal = sessionmaker(bind=engine, autoflush=False, autocommit=False)

class Base(DeclarativeBase):
    pass

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

## Convenciones de modelos Oracle

Oracle usa MAYÚSCULAS para nombres de tablas y columnas. Siempre mapea explícitamente:

```python
from sqlalchemy import Column, String, Integer, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from app.db.session import Base
import datetime

class Client(Base):
    __tablename__ = "CLIENTS"

    id = Column("CLIENT_ID", Integer, primary_key=True, index=True)
    name = Column("NAME", String(200), nullable=False)
    email = Column("EMAIL", String(255), unique=True, nullable=False)
    phone = Column("PHONE", String(20), nullable=True)
    created_at = Column("CREATED_AT", DateTime, default=datetime.datetime.utcnow)

    vehicles = relationship("Vehicle", back_populates="client")
```

## IDs: secuencias Oracle (patrón actual)

Los IDs son INTEGER con secuencias Oracle. NO uses `autoincrement=True` genérico — Oracle necesita una secuencia explícita en Alembic:

```python
# En la migración Alembic:
from alembic import op
import sqlalchemy as sa

def upgrade():
    op.execute("CREATE SEQUENCE clients_seq START WITH 1 INCREMENT BY 1")
    op.create_table(
        'CLIENTS',
        sa.Column('CLIENT_ID', sa.Integer,
                  sa.Sequence('clients_seq'), primary_key=True),
        ...
    )

def downgrade():
    op.drop_table('CLIENTS')
    op.execute("DROP SEQUENCE clients_seq")
```

## Migración futura a UUID v4

Cuando migres IDs a UUID (objetivo del proyecto), el patrón será:

```python
from sqlalchemy import String
from sqlalchemy.orm import mapped_column
import uuid

# En Oracle, UUID se guarda como VARCHAR2(36) o RAW(16)
id = mapped_column("CLIENT_ID", String(36), primary_key=True,
                   default=lambda: str(uuid.uuid4()))
```

No mezcles INTEGER y UUID en el mismo modelo. Migra modelo completo de una vez.

## Tipos de datos Oracle → Python

| Oracle | SQLAlchemy | Python |
|--------|-----------|--------|
| VARCHAR2(n) | String(n) | str |
| NUMBER | Integer / Numeric | int / Decimal |
| DATE / TIMESTAMP | DateTime | datetime |
| CLOB | Text | str |
| RAW(16) | LargeBinary(16) | bytes |

- **Nunca uses** `Text` para campos cortos — usa `String(n)` con longitud explícita
- **Fechas**: Oracle no tiene `BOOLEAN` nativo — usa `Integer` con 0/1 o `String(1)` con 'Y'/'N'

## Alembic con Oracle

Configuración crítica en `alembic/env.py`:

```python
# Asegura que Alembic use el dialect de Oracle correctamente
from sqlalchemy.dialects import oracle

def run_migrations_online():
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )
    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            # Oracle no soporta transacciones DDL — esto es necesario:
            transaction_per_migration=True,
            render_as_batch=False,  # batch mode es para SQLite, NO Oracle
        )
```

## Reglas críticas

- ❌ NO uses `render_as_batch=True` — es solo para SQLite
- ❌ NO uses `Boolean` de SQLAlchemy directamente — Oracle no lo soporta nativamente
- ❌ NO ejecutes DDL manual en producción — siempre Alembic
- ❌ NO crees sesiones fuera de `get_db()`
- ✅ Usa `pool_pre_ping=True` siempre — Oracle Autonomous cierra conexiones idle
- ✅ Nombra todas las constraints explícitamente (Oracle tiene límite de 30 chars en nombres)
- ✅ Usa `String(n)` con longitud en todos los campos de texto