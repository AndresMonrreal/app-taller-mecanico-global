    # permissions-bitmask

    Skill para implementar y evolucionar el sistema de permisos bitmask del proyecto taller mecánico. Aplica cuando toques autenticación, usuarios, roles o protección de endpoints.

    ## Concepto: ¿qué es un bitmask de permisos?

    En lugar de una tabla de roles compleja, cada permiso es una potencia de 2. Un usuario tiene un número entero que representa todos sus permisos combinados con OR binario.

    ```
    PERM_VER    = 1   → 0001
    PERM_CREAR  = 2   → 0010
    PERM_EDITAR = 4   → 0100
    PERM_BORRAR = 8   → 1000

    Usuario con VER + CREAR + EDITAR = 1 + 2 + 4 = 7 → 0111
    ```

    Para verificar si tiene un permiso: `(permisos_usuario & permiso_requerido) == permiso_requerido`

    ## Implementación: app/core/permissions.py

    ```python
    # Permisos base del sistema de taller mecánico
    PERM_VER    = 1    # Ver registros (clientes, vehículos, órdenes)
    PERM_CREAR  = 2    # Crear nuevos registros
    PERM_EDITAR = 4    # Editar registros existentes
    PERM_BORRAR = 8    # Eliminar registros
    PERM_ADMIN  = 16   # Acceso administrativo (usuarios, configuración)

    # Combinaciones predefinidas de roles comunes
    ROLE_VIEWER    = PERM_VER                              # = 1
    ROLE_MECANICO  = PERM_VER | PERM_CREAR | PERM_EDITAR  # = 7
    ROLE_ADMIN     = PERM_VER | PERM_CREAR | PERM_EDITAR | PERM_BORRAR | PERM_ADMIN  # = 31

    def has_permission(user_perms: int, required: int) -> bool:
        """Verifica si user_perms incluye el permiso requerido."""
        return (user_perms & required) == required

    def has_any_permission(user_perms: int, *required: int) -> bool:
        """Verifica si user_perms incluye AL MENOS UNO de los permisos."""
        return any(has_permission(user_perms, p) for p in required)

    def has_all_permissions(user_perms: int, *required: int) -> bool:
        """Verifica si user_perms incluye TODOS los permisos."""
        return all(has_permission(user_perms, p) for p in required)
    ```

    ## Modelo User con permisos

    ```python
    # models/user.py
    from sqlalchemy import Column, String, Integer, DateTime, Boolean
    from app.db.session import Base
    import datetime

    class User(Base):
        __tablename__ = "USERS"

        id          = Column("USER_ID", Integer, primary_key=True, index=True)
        username    = Column("USERNAME", String(100), unique=True, nullable=False)
        email       = Column("EMAIL", String(255), unique=True, nullable=False)
        hashed_password = Column("HASHED_PASSWORD", String(255), nullable=False)
        permissions = Column("PERMISSIONS", Integer, default=1, nullable=False)  # PERM_VER por defecto
        is_active   = Column("IS_ACTIVE", Integer, default=1, nullable=False)    # 1=activo, 0=inactivo
        created_at  = Column("CREATED_AT", DateTime, default=datetime.datetime.utcnow)
    ```

    ## JWT con permisos en el payload

    ```python
    # core/security.py
    from datetime import datetime, timedelta
    from jose import JWTError, jwt
    from passlib.context import CryptContext
    from app.core.config import settings

    pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

    def create_access_token(user_id: int, permissions: int) -> str:
        expire = datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
        payload = {
            "sub": str(user_id),
            "permissions": permissions,  # el bitmask va aquí
            "exp": expire,
        }
        return jwt.encode(payload, settings.SECRET_KEY, algorithm=settings.ALGORITHM)

    def decode_token(token: str) -> dict:
        return jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])

    def verify_password(plain: str, hashed: str) -> bool:
        return pwd_context.verify(plain, hashed)

    def hash_password(plain: str) -> str:
        return pwd_context.hash(plain)
    ```

    ## Dependency de usuario actual + verificación de permisos

    ```python
    # dependencies/auth.py
    from fastapi import Depends, HTTPException, status
    from fastapi.security import OAuth2PasswordBearer
    from sqlalchemy.orm import Session
    from app.db.session import get_db
    from app.core.security import decode_token
    from app.core.permissions import has_permission
    from app.models.user import User

    oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/login")

    async def get_current_user(
        token: str = Depends(oauth2_scheme),
        db: Session = Depends(get_db)
    ) -> User:
        credentials_exception = HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token inválido o expirado",
            headers={"WWW-Authenticate": "Bearer"},
        )
        try:
            payload = decode_token(token)
            user_id = int(payload.get("sub"))
        except Exception:
            raise credentials_exception

        user = db.query(User).filter(User.id == user_id, User.is_active == 1).first()
        if not user:
            raise credentials_exception
        return user

    def require_permission(permission: int):
        """Factory que devuelve una dependency que exige un permiso específico."""
        async def _check(current_user: User = Depends(get_current_user)) -> User:
            if not has_permission(current_user.permissions, permission):
                raise HTTPException(
                    status_code=status.HTTP_403_FORBIDDEN,
                    detail="No tienes permisos para esta acción",
                )
            return current_user
        return _check
    ```

    ## Uso en routers

    ```python
    # routers/clients.py
    from fastapi import APIRouter, Depends
    from app.dependencies.auth import require_permission
    from app.core.permissions import PERM_VER, PERM_CREAR, PERM_EDITAR, PERM_BORRAR

    router = APIRouter()

    @router.get("/")
    def list_clients(current_user = Depends(require_permission(PERM_VER))):
        ...

    @router.post("/")
    def create_client(current_user = Depends(require_permission(PERM_CREAR))):
        ...

    @router.put("/{id}")
    def update_client(current_user = Depends(require_permission(PERM_EDITAR))):
        ...

    @router.delete("/{id}")
    def delete_client(current_user = Depends(require_permission(PERM_BORRAR))):
        ...
    ```

    ## Reglas del sistema

    - ✅ Cada permiso es potencia de 2: 1, 2, 4, 8, 16, 32...
    - ✅ Nuevos permisos se agregan multiplicando por 2 el último
    - ✅ El valor por defecto de un usuario nuevo es `PERM_VER` (1)
    - ✅ Solo usuarios con `PERM_ADMIN` pueden cambiar permisos de otros
    - ❌ NO uses strings como "admin", "viewer" — siempre constantes numéricas
    - ❌ NO hardcodees números en los routers — importa las constantes
    - ❌ NO guardes el nombre del rol en la BD — solo el integer de permisos