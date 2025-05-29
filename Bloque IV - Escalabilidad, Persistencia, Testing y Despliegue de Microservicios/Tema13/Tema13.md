# Tema 13. PERSISTENCIA DE DATOS EN MICROSERVICIOS

* [Tema 13. PERSISTENCIA DE DATOS EN MICROSERVICIOS](Tema13.md#tema-13-persistencia-de-datos-en-microservicios)
  * [13.1 Integración de SQLAlchemy ORM](Tema13.md#131-integración-de-sqlalchemy-orm)
  * [13.2 Modelos desacoplados del dominio (DTO vs Entity)](Tema13.md#132-modelos-desacoplados-del-dominio-dto-vs-entity)
  * [13.3 Patrones de Repositorio con SQLAlchemy: Guardianes Elegantes de Tu Persistencia](Tema13.md#133-patrones-de-repositorio-con-sqlalchemy-guardianes-elegantes-de-tu-persistencia)
  * [13.4 Gestión de Transacciones Locales: Asegurando la Atomicidad en Tus Operaciones de Datos](Tema13.md#134-gestión-de-transacciones-locales-asegurando-la-atomicidad-en-tus-operaciones-de-datos)
  * [13.5 Transacciones distribuidas: sagas y outbox](Tema13.md#135-transacciones-distribuidas-sagas-y-outbox)
  * [13.6 Rollback coordinado con eventos](Tema13.md#136-rollback-coordinado-con-eventos)
  * [13.7 Conexión a MongoDB con `motor`](Tema13.md#137-conexión-a-mongodb-con-motor)
  * [13.8 Esquemas flexibles en MongoDB](Tema13.md#138-esquemas-flexibles-en-mongodb)
  * [13.9 Bases de datos por servicio y separación](Tema13.md#139-bases-de-datos-por-servicio-y-separación)
  * [13.9 Bases de Datos por Servicio y Separación de Datos: Autonomía y Desacoplamiento en la Persistencia de `Microservices`](Tema13.md#139-bases-de-datos-por-servicio-y-separación-de-datos-autonomía-y-desacoplamiento-en-la-persistencia-de-microservices)
  * [13.10 Pools de conexión y timeouts](Tema13.md#1310-pools-de-conexión-y-timeouts)
  * [13.10 `Pools` de Conexión y `Timeouts`: Optimizando el Flujo Ininterrumpido de Datos](Tema13.md#1310-pools-de-conexión-y-timeouts-optimizando-el-flujo-ininterrumpido-de-datos)
  * [Bibliografía](Tema13.md#bibliografía)

## 13.1 Integración de SQLAlchemy ORM

***

En el desarrollo de `microservices` robustos con FastAPI, la persistencia de datos es un pilar. Cuando se trata de bases de datos relacionales (como PostgreSQL, MySQL, SQLite), **SQLAlchemy** se erige como la biblioteca Python por excelencia, ofreciendo un poderoso **Object Relational Mapper (ORM)**. Esta sección se enfoca en su integración **asíncrona** con FastAPI, una sinergia crucial para construir aplicaciones `backend` de alto rendimiento y no bloqueantes.

**SQLAlchemy ORM: Tu Intérprete Experto entre el Mundo de Objetos Python y las Tablas SQL 📜**

* **¿Qué es SQLAlchemy?** Es un `toolkit` SQL y ORM para Python. El ORM permite a los desarrolladores interactuar con la base de datos usando clases y objetos Python, abstrayendo gran parte del SQL subyacente.
* **Beneficios Clave del ORM:**
  * **Abstracción del Dialecto SQL:** Escribes código Python que puede funcionar (con ajustes mínimos) sobre diferentes motores de BBDD.
  * **Productividad:** Simplifica enormemente las operaciones CRUD (Create, Read, Update, Delete) y las consultas complejas.
  * **Mantenibilidad:** El código orientado a objetos suele ser más fácil de entender y mantener para muchos desarrolladores Python.
  * **Seguridad:** Ayuda a prevenir vulnerabilidades de `SQL injection` cuando se usa correctamente (al no construir SQL con `strings` directamente).
* **La Revolución Asíncrona de SQLAlchemy (v1.4+ y especialmente v2.0+):**\
  SQLAlchemy ha abrazado `asyncio`, proporcionando una API asíncrona completa. Esto es _fundamental_ para FastAPI, ya que permite que las operaciones de base de datos (que son inherentemente I/O-bound) no bloqueen el `event loop` principal, permitiendo a tu aplicación manejar miles de `requests` concurrentes.

**Los Pilares de la Integración Asíncrona: Componentes Esenciales ⚙️**

Para una integración asíncrona exitosa, estos son los actores principales de SQLAlchemy:

| Componente                | Importación (`from sqlalchemy...`)                       | Propósito Principal                                                                                                                           |
| ------------------------- | -------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `create_async_engine`     | `from sqlalchemy.ext.asyncio import create_async_engine` | Crea el "motor" de conexión asíncrona a la BBDD. Gestiona el `connection pool` y el dialecto específico de la BBDD.                           |
| `AsyncSession`            | `from sqlalchemy.ext.asyncio import AsyncSession`        | La `session` es el corazón de la interacción ORM. Es tu `workspace` para todas las operaciones de BBDD en un contexto asíncrono.              |
| `async_sessionmaker`      | `from sqlalchemy.ext.asyncio import async_sessionmaker`  | Una factoría para crear instancias de `AsyncSession` de manera consistente y preconfigurada.                                                  |
| `DeclarativeBase`         | `from sqlalchemy.orm import DeclarativeBase`             | (Estilo SQLAlchemy 2.0+) La clase base de la cual heredarán tus modelos ORM para definir el mapeo tabla-clase de forma declarativa.           |
| `Mapped`, `mapped_column` | `from sqlalchemy.orm import Mapped, mapped_column`       | (Estilo SQLAlchemy 2.0+) Utilizados dentro de tus modelos ORM para definir atributos mapeados a columnas de tabla, con `type hints` robustos. |

* **`Database Drivers` Asíncronos Indispensables:**\
  El `async_engine` necesita un `driver` de base de datos que soporte `asyncio`.
  * PostgreSQL: `asyncpg` (ej. `DATABASE_URL = "postgresql+asyncpg://user:pass@host/db"`)
  * MySQL: `aiomysql` o `asyncmy` (ej. `DATABASE_URL = "mysql+aiomysql://user:pass@host/db"`)
  * SQLite: `aiosqlite` (ej. `DATABASE_URL = "sqlite+aiosqlite:///./mi_app.db"`)

**Definiendo Tus Entidades: Modelos SQLAlchemy con la Elegancia Declarativa 🏛️**

Tus modelos ORM son clases Python que SQLAlchemy mapeará a tablas.

1.  **La `Base` para Tus Modelos:**

    ```python
    # app/db/base_class.py
    from sqlalchemy.orm import DeclarativeBase

    class Base(DeclarativeBase):
        pass
        # Aquí podrías definir metadata común, como una columna 'id' o timestamps,
        # o un type_annotation_map para tipos personalizados.
    ```
2.  **Ejemplo de Modelo ORM (`ProductDB`):**

    ```python
    # app/models/product_db.py
    from sqlalchemy import String, Float, Boolean, Integer
    from sqlalchemy.orm import Mapped, mapped_column
    from typing import Optional

    from app.db.base_class import Base # Importar tu Base

    class ProductDB(Base):
        __tablename__ = "products" # Nombre de la tabla en la base de datos

        id: Mapped[int] = mapped_column(Integer, primary_key=True, index=True, autoincrement=True)
        name: Mapped[str] = mapped_column(String(200), index=True, nullable=False)
        description: Mapped[Optional[str]] = mapped_column(String(500), nullable=True)
        price: Mapped[float] = mapped_column(Float, nullable=False) # Para dinero, considera Numeric o Decimal
        is_available: Mapped[bool] = mapped_column(Boolean, default=True)

        # __repr__ para una representación legible (opcional, pero útil para debugging)
        def __repr__(self) -> str:
            return f"<ProductDB(id={self.id}, name='{self.name}', price={self.price})>"
    ```

**Gestión de `AsyncSession` en FastAPI: El Patrón `Dependency` Como Guardián 🔗**

El manejo correcto del ciclo de vida de la `AsyncSession` es crítico. Cada `request` HTTP debe, idealmente, operar con su propia `session`, la cual debe ser cerrada al finalizar.

1.  **Configuración del `Engine` y `async_sessionmaker` (ej. en `app/db/session.py`):**

    ```python
    # app/db/session.py
    from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

    # Deberías cargar esto desde variables de entorno o un archivo de configuración
    SQLALCHEMY_DATABASE_URL = "postgresql+asyncpg://user:password@localhost/mydatabase"
    # O para SQLite (útil para pruebas y desarrollo rápido):
    # SQLALCHEMY_DATABASE_URL = "sqlite+aiosqlite:///./test_app_definitive.db"

    async_engine = create_async_engine(
        SQLALCHEMY_DATABASE_URL,
        echo=True,  # Log SQL generado; DESACTIVAR EN PRODUCCIÓN
        future=True # Habilitar comportamiento de SQLAlchemy 2.0
    )

    # Crear una factoría de sesiones configurada
    # expire_on_commit=False es crucial para FastAPI para poder acceder a los atributos
    # de los objetos ORM después de que la sesión haya hecho commit y antes de que el request termine.
    AsyncSessionFactory = async_sessionmaker(
        bind=async_engine,
        class_=AsyncSession,
        expire_on_commit=False
    )

    # (Opcional) Función para inicializar la base de datos y crear tablas
    # from app.db.base_class import Base
    # async def init_db():
    #     async with async_engine.begin() as conn:
    #         # await conn.run_sync(Base.metadata.drop_all) # Descomentar para borrar y recrear
    #         await conn.run_sync(Base.metadata.create_all)
    ```

    _En tu `main.py`, podrías llamar a `init_db()` en un evento `startup` si necesitas que las tablas se creen al iniciar la app._
2.  **La Dependencia `get_session` Definitiva:**\
    Esta dependencia es la forma canónica de proporcionar una `AsyncSession` a tus `endpoints`.

    ```python
    # app/api/dependencies.py (o donde organices tus dependencias)
    from typing import AsyncGenerator
    from sqlalchemy.ext.asyncio import AsyncSession
    from app.db.session import AsyncSessionFactory # Importa tu factoría

    async def get_session() -> AsyncGenerator[AsyncSession, None]:
        """
        Dependency provider for an SQLAlchemy AsyncSession.
        It ensures the session is properly closed after the request.
        Transaction management (commit/rollback) is the responsibility
        of the code using the session (e.g., service layer or endpoint).
        """
        async with AsyncSessionFactory() as session:
            # El bloque 'async with' se encarga de:
            # 1. session = AsyncSessionFactory() -> obtener una nueva sesión del pool
            # 2. Al final del bloque (incluso si hay excepciones): await session.close()
            try:
                yield session # La sesión se inyecta aquí
            except Exception:
                # Si una excepción no manejada se propaga hasta aquí,
                # la transacción (si estaba activa y no se hizo commit/rollback)
                # será implícitamente rollbackeada por la BBDD al cerrar la conexión
                # o al finalizar la sesión si la transacción no fue commiteada.
                # Ser explícito con rollback aquí puede ser una opción si se desea
                # controlar el loggeo o acciones específicas antes de re-lanzar.
                # await session.rollback() # Opcional, ya que el cierre de sesión sin commit lo haría.
                raise # Re-lanzar para que los manejadores de excepción de FastAPI actúen
    ```

    * **Responsabilidad Transaccional:** La dependencia `get_session` provee la `session`. La decisión de hacer `await session.commit()` o `await session.rollback()` reside en la lógica de la `path operation` o, preferiblemente, en una capa de servicio que utiliza la `session`.

**Operaciones ORM CRUD Asíncronas con la `Session` Inyectada ⚡**

Veamos cómo usar esta `session` en `endpoints` CRUD (los modelos Pydantic `ProductCreate`, `ProductUpdate`, `ProductResponse` se asumirían definidos como en 11.5, adaptados a `Product`).

```python
# app/api/endpoints/products.py
from fastapi import APIRouter, Depends, HTTPException, status, Response
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.future import select # O simplemente 'from sqlalchemy import select' en SA 2.0
from typing import List

# from app.db.session import get_session # La dependencia
# from app.models.product_db import ProductDB # El modelo ORM
# from app.schemas.product_api import ProductCreate, ProductUpdate, ProductResponse # Modelos Pydantic

router = APIRouter()

@router.post("/", response_model=ProductResponse, status_code=status.HTTP_201_CREATED)
async def create_product(
    product_in: ProductCreate,
    db: AsyncSession = Depends(get_session)
):
    db_product = ProductDB(**product_in.model_dump())
    db.add(db_product)
    try:
        await db.commit()
        await db.refresh(db_product) # Para obtener el ID autogenerado y otros defaults
    except Exception as e: # Ser más específico con excepciones de BBDD (ej. IntegrityError)
        await db.rollback()
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail=f"Error al crear producto: {str(e)}")
    return db_product

@router.get("/{product_id}", response_model=ProductResponse)
async def read_product(product_id: int, db: AsyncSession = Depends(get_session)):
    product = await db.get(ProductDB, product_id) # Elegante y directo para PKs
    if product is None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Producto no encontrado")
    return product

@router.get("/", response_model=List[ProductResponse])
async def read_products(skip: int = 0, limit: int = 100, db: AsyncSession = Depends(get_session)):
    result = await db.execute(select(ProductDB).offset(skip).limit(limit))
    products = result.scalars().all()
    return products

# ... (PUT, PATCH, DELETE seguirían un patrón similar, obteniendo el objeto, modificando/borrando, y commiteando) ...
# Para PUT/PATCH:
# product_to_update = await db.get(ProductDB, product_id)
# if not product_to_update: raise HTTPException(404)
# for key, value in product_update_data.model_dump(exclude_unset=True).items():
#     setattr(product_to_update, key, value)
# db.add(product_to_update) # SQLAlchemy rastrea cambios
# await db.commit()
# await db.refresh(product_to_update)
# return product_to_update
```

**Diagrama `Mermaid`: El Flujo de Vida de una `AsyncSession` en un `Request` FastAPI**

```mermaid
sequenceDiagram
    participant Client
    participant FastAPI_EP as FastAPI Endpoint <br> (ej. POST /products/)
    participant Dep_get_session as Dependency: get_session()
    participant ASessFact as AsyncSessionFactory
    participant ASess as AsyncSession (yielded)
    participant OrmOps as ORM Operations <br> (en Endpoint o Service)
    participant DB as Database

    Client->>+FastAPI_EP: HTTP Request (Payload)
    Note over FastAPI_EP: Pydantic valida Payload
    FastAPI_EP->>+Dep_get_session: Solicita `AsyncSession`
    Dep_get_session->>+ASessFact: `AsyncSessionFactory()`
    ASessFact-->>-Dep_get_session: Nueva `AsyncSession`
    Note over Dep_get_session: `async with ... as session:`
    Dep_get_session-->>FastAPI_EP: `yield session` (inyecta `ASess`)
    
    FastAPI_EP->>+OrmOps: `db.add(obj)`, `await db.commit()`, etc.
    OrmOps->>+ASess: Ejecuta operaciones ORM
    ASess->>DB: Envía SQL a la BBDD
    DB-->>ASess: Resultados de BBDD
    ASess-->>-OrmOps: ORM objects/confirmación
    OrmOps-->>-FastAPI_EP: Resultado (ej. objeto ORM creado)
    
    Note over FastAPI_EP: Serializa resultado a JSON
    FastAPI_EP-->>-Client: HTTP Response
    
    Note over Dep_get_session: `async with` finaliza, `session.close()` es llamado automáticamente.
```

**Reflexiones Profesionales para una Integración Impecable:**

* **Manejo de Transacciones Explícito:** Para operaciones que involucran múltiples cambios que deben ser atómicos, envuélvelos en un bloque `try/except` con `await db.commit()` al final del `try` y `await db.rollback()` en el `except`. Considera el patrón `Unit of Work` (que se verá en 13.4) para una gestión más estructurada.
* **Capa de Servicio/Repositorio:** La lógica de interactuar con `db_item_orm = ProductDB(...)`, `db.add()`, `await db.commit()`, etc., idealmente se abstrae en una **capa de Repositorio o Servicio** (ver 13.3). Tus `endpoints` FastAPI se mantendrán más delgados y centrados en HTTP, delegando la lógica de datos a esta capa, que a su vez usaría la `session` inyectada.
* **Optimización de Consultas:** Aprende a usar `select()` con `options(selectinload(Model.relationship_field))` para `eager loading` de relaciones y evitar el problema N+1. SQLAlchemy 2.0 ha mejorado mucho la expresividad de `select()`.
* **Testing:** La dependencia `get_session` es tu mejor amiga para las pruebas. Puedes sobrescribirla (`app.dependency_overrides[get_session] = ...`) para usar una base de datos de prueba en memoria (como SQLite `async`) o para `mockear` completamente la capa de base de datos.

**Conclusión (Versión Definitiva): SQLAlchemy y FastAPI – Una Sinergia Asíncrona para la Persistencia Profesional**

La integración asíncrona de SQLAlchemy ORM con FastAPI, apuntalada por el sistema de `Dependency Injection` para una gestión impecable de `AsyncSession`, es la piedra angular para construir `backends` Python modernos, eficientes y robustos que interactúan con bases de datos relacionales.

Al internalizar el flujo `async`, el ciclo de vida de la `session` gestionado por `Depends`, y las responsabilidades transaccionales, tus profesionales estarán equipados no solo para realizar operaciones CRUD, sino para construir capas de acceso a datos complejas, mantenibles y altamente performantes. Esta base sólida es esencial antes de adentrarnos en patrones más avanzados de persistencia y diseño de dominio.

## 13.2 Modelos desacoplados del dominio (DTO vs Entity)

Con una base sólida sobre cómo integrar SQLAlchemy ORM de forma asíncrona con FastAPI (nuestro 13.1 definitivo), es el momento de abordar una distinción conceptual y práctica crucial en el diseño de aplicaciones robustas: el desacoplamiento entre los modelos que usamos para la persistencia (las `entities` ORM) y los modelos que usamos para la comunicación API (los `Data Transfer Objects - DTOs`, que en FastAPI son nuestros modelos Pydantic).

En la sección anterior (13.1), definimos `ProductDB` como nuestro modelo SQLAlchemy ORM (nuestra **`Entity`** de base de datos) y también usamos modelos Pydantic como `ProductCreatePydantic` y `ProductResponsePydantic` (nuestros **`Data Transfer Objects - DTOs`**) para los `request/response payloads` de nuestra API. Esta separación, aunque pueda parecer una duplicación de definiciones al principio, es una práctica de diseño fundamental y altamente beneficiosa para la mantenibilidad, flexibilidad y robustez de nuestras aplicaciones.

**¿Qué es una `Entity` (Entidad ORM) y qué es un `DTO`?**

1. **`Entity` (Modelo ORM - Ej. `ProductDB` de SQLAlchemy):**
   * **Propósito:** Representar la estructura de una tabla (o un conjunto de tablas relacionadas) en tu base de datos. Está directamente ligada a tu capa de persistencia.
   * **Características:**
     * Define el mapeo objeto-relacional: columnas, tipos de datos SQL, claves primarias/foráneas, relaciones (`relationships`) con otras `entities`.
     * Contiene la lógica de cómo los datos se almacenan y se recuperan de la base de datos.
     * Su ciclo de vida está gestionado por la `session` del ORM (ej. `db.add()`, `await db.commit()`).
     * **Está optimizada para la persistencia y la integridad de los datos en la base de datos.**
   *   **Ejemplo (SQLAlchemy):**

       ```python
       # class ProductDB(Base):
       #     __tablename__ = "products"
       #     id: Mapped[int] = mapped_column(Integer, primary_key=True)
       #     name: Mapped[str] = mapped_column(String(200))
       #     _confidential_storage_detail: Mapped[str] = mapped_column(String(50)) # Campo interno de BBDD
       #     # ... otros campos y relaciones ...
       ```
2. **`DTO` (`Data Transfer Object` - Ej. `ProductResponsePydantic`, `ProductCreatePydantic`):**
   * **Propósito:** Representar la estructura de los datos tal como se envían o reciben a través de la interfaz de tu API (o entre capas de tu aplicación). Son los "contratos de datos" de tu API.
   * **Características:**
     * Definidos usando Pydantic en el contexto de FastAPI.
     * Validados automáticamente por FastAPI en los `requests` y usados para serializar `responses`.
     * **Están optimizados para la comunicación y las necesidades específicas de los `clients` de la API.**
     * Pueden ser un subconjunto, un superconjunto o una transformación de los campos de una `Entity` ORM.
     * No contienen lógica de persistencia; son simples contenedores de datos.
   *   **Ejemplo (Pydantic):**

       ```python
       # class ProductResponsePydantic(BaseModel):
       #     product_id: int # Podríamos querer renombrar 'id' a 'product_id' para la API
       #     product_name: str # Renombrar 'name' a 'product_name'
       #     description: Optional[str]
       #     price: float
       #     # Nótese la ausencia de '_confidential_storage_detail'
       #     model_config = {"from_attributes": True} # Para Pydantic V2 (era orm_mode)
       ```

**¿Por Qué Desacoplar `DTOs` de `Entities`? Las Ventajas Estratégicas**

Separar estos dos tipos de modelos es una forma de aplicar el **Principio de Responsabilidad Única (`Single Responsibility Principle - SRP`)** y el **Principio de Separación de Incumbencias (`Separation of Concerns - SoC`)**.

1. **API Estable vs. Esquema de Base de Datos Evolutivo:**
   * El esquema de tu base de datos (`Entities` ORM) puede necesitar cambiar por razones internas (optimización, nuevas relaciones, normalización/desnormalización).
   * Tu API pública (`DTOs`) debe aspirar a una mayor estabilidad para no romper `clients` existentes.
   * **Desacoplamiento:** Puedes refactorizar tu `Entity` ORM (ej. dividir un campo, añadir tablas internas) sin cambiar necesariamente el `DTO` que expones, siempre que puedas seguir mapeando los datos.
2. **Exposición Selectiva de Datos:**
   * Tus `Entities` ORM pueden contener campos que no quieres (o no debes) exponer a través de la API (ej. `hashed_passwords`, campos de auditoría interna, detalles de configuración de almacenamiento como `_confidential_storage_detail` en el ejemplo anterior).
   * Los `DTOs` te permiten definir explícitamente qué campos son parte del contrato de la API.
3. **Optimización para Diferentes Casos de Uso:**
   * **`DTOs` de Creación/Actualización:** Pueden tener campos diferentes o validaciones más estrictas que la `Entity` (ej. `password` en `UserCreateDTO` vs. `hashed_password` en `UserEntity`).
   * **`DTOs` de Respuesta:** Pueden aplanar jerarquías, combinar datos de múltiples `Entities`, o calcular campos derivados que no existen directamente en la base de datos, todo para la conveniencia del `client`.
   * **Ejemplo:** Un `ProductSummaryDTO` para una lista podría tener menos campos que un `ProductDetailDTO` para la vista de un solo producto.
4. **Validación Específica de la API:**
   * Pydantic (`DTOs`) te permite aplicar validaciones que son relevantes para los datos de entrada de la API (ej. `min_length` para un `string` de `input`, formatos específicos) que podrían ser diferentes o adicionales a los `constraints` de la base de datos definidos en la `Entity` ORM.
5. **Prevención de Fugas de Detalles de Implementación del ORM/BBDD:**
   * Si devuelves instancias de `Entities` ORM directamente, podrías exponer accidentalmente detalles sobre tu esquema de base de datos o el comportamiento del ORM (ej. atributos de carga `lazy` que podrían causar consultas adicionales si el `client` los accede al serializar).
   * Los `DTOs` actúan como una barrera protectora y un contrato explícito.
6. **Mayor Testabilidad:**
   * Puedes probar tu lógica de API y tus `DTOs` independientemente de la capa de persistencia (usando `mocks` para la capa que produce/consume `Entities`).
   * Puedes probar tu capa de persistencia y tus `Entities` ORM independientemente de cómo se exponen los datos en la API.

**El Mapeo entre `DTOs` y `Entities` ORM: El Puente Necesario**

Si tenemos modelos separados, necesitamos una forma de mapear datos entre ellos:

1.  **De `DTO` de `Request` a `Entity` ORM (ej. en una operación `Create` o `Update`):**

    ```python
    # En tu endpoint o service layer
    # async def create_product_logic(product_create_dto: ProductCreatePydantic, db: AsyncSession):
    #     # Mapeo de DTO a Entity ORM
    #     db_product = ProductDB(
    #         name=product_create_dto.name,
    #         description=product_create_dto.description,
    #         price=product_create_dto.price,
    #         # ... otros campos ...
    #     )
    #     db.add(db_product)
    #     await db.commit()
    #     await db.refresh(db_product)
    #     return db_product # Devuelves la Entity ORM, que luego se mapeará a un DTO de respuesta
    ```

    Pydantic V2 con `model_dump()`: `ProductDB(**product_create_dto.model_dump())` puede simplificar esto si los nombres de campo coinciden.
2.  **De `Entity` ORM a `DTO` de `Response` (ej. en una operación `Read`):**\
    Aquí es donde la configuración `model_config = {"from_attributes": True}` (o `Config.orm_mode = True` en Pydantic V1) en tus modelos Pydantic de respuesta es crucial. Permite que Pydantic lea los datos directamente desde los atributos de la instancia de la `Entity` ORM.

    ```python
    # En tu endpoint
    # @router.get("/{product_id}", response_model=ProductResponsePydantic)
    # async def read_product(product_id: int, db: AsyncSession = Depends(get_session)):
    #     db_product_entity = await db.get(ProductDB, product_id)
    #     if not db_product_entity:
    #         raise HTTPException(status_code=404, detail="Producto no encontrado")
    #     # FastAPI, usando ProductResponsePydantic como response_model y su config from_attributes,
    #     # mapeará automáticamente db_product_entity (SQLAlchemy model) a ProductResponsePydantic.
    #     return db_product_entity 
    ```

    Si los nombres de los campos entre la `Entity` y el `DTO` no coinciden, o si necesitas lógica de transformación, tendrás que hacer el mapeo manualmente antes de retornar, o usar `computed_fields` o `@field_serializer` en Pydantic si es apropiado.

**Diagrama `Mermaid`: La Danza DTO-Entidad**

```mermaid

 graph LR

Client[Client API]

subgraph FastAPIAdapters
    API_EP[API Endpoint - FastAPI]
    PydanticReqDTO[Pydantic Request DTO - ProductCreate]
    PydanticRespDTO[Pydantic Response DTO - ProductResponse]
end

subgraph ApplicationLayer
    AppService[Application Service - Use Case]
end

subgraph InfrastructureLayer
    SQLAlchemyEntity[SQLAlchemy ORM Entity - ProductDB]
    DB[(Database)]
end

Client -->|HTTP Request JSON| API_EP

API_EP -->|Deserializa y valida| PydanticReqDTO
API_EP -->|Pasa DTO o datos| AppService

AppService -->|Mapea DTO a entidad| SQLAlchemyEntity
AppService -->|Usa ORM| DB_Ops[ORM Ops - add, commit, get]

DB_Ops --> SQLAlchemyEntity
SQLAlchemyEntity <--> DB

DB_Ops --> SQLAlchemyEntityData[SQLAlchemy Entity desde DB]
SQLAlchemyEntityData -->|Devuelta a| AppService
AppService -->|Devuelve entidad o datos| API_EP

API_EP -->|Mapea Entity a DTO| PydanticRespDTO
API_EP -->|Serializa a JSON| ClientResp[HTTP Response JSON]
ClientResp --> Client

style PydanticReqDTO fill:#D5F5E3,stroke:#2ECC71
style PydanticRespDTO fill:#D5F5E3,stroke:#2ECC71
style SQLAlchemyEntity fill:#AED6F1,stroke:#3498DB
style SQLAlchemyEntityData fill:#AED6F1,stroke:#3498DB



```

Este diagrama muestra cómo los `DTOs` son la moneda de cambio en la frontera de la API, mientras que las `Entities` ORM son específicas de la capa de persistencia. El `Application Service` (o el `endpoint` si la lógica es simple) actúa como el traductor.

**Desafíos y Consideraciones:**

* **`Boilerplate` de Mapeo:** Sí, hay un coste en escribir y mantener el código de mapeo si los `DTOs` y las `Entities` difieren significativamente. Herramientas como `AutoMapper` (en otros lenguajes) intentan aliviar esto, pero en Python, a menudo es un mapeo manual explícito o confiar en `model_dump()` y `from_attributes` de Pydantic. El beneficio del desacoplamiento suele superar este coste.
* **Mantener la Sincronización:** Cuando cambias una `Entity`, debes recordar si necesitas actualizar los `DTOs` correspondientes y el mapeo.

**Conclusión: `DTOs` y `Entities` ORM – Aliados Estratégicos, No Gemelos Idénticos**

Desacoplar tus `DTOs` (modelos de API Pydantic) de tus `Entities` ORM (modelos SQLAlchemy) es una práctica de diseño que te recompensa con:

* **Flexibilidad:** Evoluciona tu `database schema` y tu API pública de forma más independiente.
* **Seguridad:** Expón solo los datos que necesitas, protegiendo la información sensible.
* **Claridad:** Contratos de API explícitos y modelos de persistencia optimizados para sus respectivos propósitos.
* **Mantenibilidad:** Cambios en una capa tienen menos probabilidad de impactar directamente a la otra.

Aunque introduce una capa de mapeo, los beneficios en términos de robustez y adaptabilidad a largo plazo son inmensos, especialmente en aplicaciones y `microservices` que se espera evolucionen y se mantengan durante años. FastAPI y Pydantic te proporcionan las herramientas perfectas para implementar este patrón de forma elegante.

Continuamos con el **13.3**. Después de haber integrado SQLAlchemy ORM (13.1) y comprendido la importancia de desacoplar nuestros modelos de API (DTOs) de los modelos de base de datos (Entities ORM) (13.2), el siguiente paso lógico es introducir una abstracción que nos permita organizar y gestionar nuestro acceso a datos de una manera aún más limpia y estructurada: el **Patrón Repositorio (`Repository Pattern`)**.

Este patrón es un pilar en arquitecturas que buscan un alto grado de desacoplamiento y testabilidad, como la Arquitectura Hexagonal.

## 13.3 Patrones de Repositorio con SQLAlchemy: Guardianes Elegantes de Tu Persistencia

Imagina que tus `entities` ORM (como `ProductDB`) son tesoros valiosos guardados en una compleja bóveda (tu base de datos). En lugar de que cada parte de tu aplicación (tus `endpoints` o `services`) tenga que conocer los intrincados mecanismos de la cerradura y los pasadizos secretos de la bóveda (los detalles de SQLAlchemy, las sentencias `select`, cómo manejar `sessions`), designas a un **Guardián del Tesoro** o **Bibliotecario Experto** para cada tipo de tesoro. Este guardián es tu **Repositorio**.

**El Patrón Repositorio: Tu Intermediario Experto con la Capa de Persistencia 🧑‍💼**

* **Definición y Propósito:**\
  El Patrón Repositorio media entre la capa de dominio (o servicios de aplicación) y la capa de mapeo de datos (el ORM, en nuestro caso SQLAlchemy). Actúa como una **colección en memoria de objetos de dominio**, proporcionando una interfaz para añadir, eliminar, y buscar `entities` sin exponer los detalles de la tecnología de persistencia subyacente.
* **Analogía del Bibliotecario:**\
  Piensa en un `ProductRepository` como el bibliotecario jefe de la sección de "Productos".
  * Quieres un producto específico por su ID: `bibliotecario.get_product_by_id(123)`.
  * Quieres todos los productos de una categoría: `bibliotecario.list_products_by_category("electronics")`.
  * Tienes un nuevo producto para añadir al catálogo: `bibliotecario.add_product(nuevo_producto)`.\
    Tú no necesitas saber si el bibliotecario usa fichas, un sistema informático arcaico, o magia SQLAlchemy para encontrar o guardar el libro (producto). Solo interactúas con su interfaz clara.
* **Beneficios Clave:**
  1. **Desacoplamiento:** Tu lógica de negocio (`application services`, `use cases`) depende de una interfaz de repositorio, no directamente de SQLAlchemy. Esto significa que tu `core` de negocio no "sabe" que estás usando PostgreSQL o SQLAlchemy.
  2. **Centralización de la Lógica de Acceso a Datos:** Todas las `queries` y operaciones de persistencia para un tipo de `entity` (o `aggregate root` en DDD) se agrupan en su repositorio.
  3. **Testabilidad Mejorada:** En las pruebas unitarias de tus `services`, puedes fácilmente `mockear` la interfaz del repositorio para simular el acceso a datos sin necesidad de una base de datos real.
  4. **Mayor Legibilidad y Mantenibilidad:** Los `services` se vuelven más limpios, ya que delegan los detalles de la persistencia.

**1. Diseñando la Interfaz del Repositorio (El Contrato con el Bibliotecario) 📜**

Primero, definimos un contrato (una interfaz) para nuestro repositorio. En Python, esto se puede hacer con una clase base abstracta (`abc.ABC`) o, de forma más moderna y flexible para `type hinting`, con `typing.Protocol`.

```python
# app/repositories/protocols/product_repository_protocol.py
from typing import Protocol, List, Optional, Any
from sqlalchemy.ext.asyncio import AsyncSession # Importante para el contexto

# Asumimos que ProductDB es nuestro modelo SQLAlchemy (Entity)
# y ProductCreatePydantic, ProductUpdatePydantic son nuestros DTOs Pydantic
# from app.models.product_db import ProductDB 
# from app.schemas.product_api import ProductCreatePydantic, ProductUpdatePydantic 

# Para el ejemplo, definiremos placeholders si no los importamos realmente
class ProductDB: pass 
class ProductCreatePydantic: pass
class ProductUpdatePydantic: pass


class IProductRepository(Protocol):
    # La sesión se pasará a los métodos o al constructor de la implementación
    
    async def get_by_id(self, db: AsyncSession, product_id: int) -> Optional[ProductDB]:
        ...

    async def get_all(self, db: AsyncSession, skip: int = 0, limit: int = 100) -> List[ProductDB]:
        ...
    
    async def get_by_name(self, db: AsyncSession, name: str) -> Optional[ProductDB]: # Ejemplo de método específico
        ...

    async def create(self, db: AsyncSession, product_in: ProductCreatePydantic) -> ProductDB:
        ...

    async def update(
        self, db: AsyncSession, product_db_obj: ProductDB, product_in: ProductUpdatePydantic
    ) -> ProductDB: # Recibe la entidad ORM a actualizar y los datos del DTO
        ...

    async def delete(self, db: AsyncSession, product_id: int) -> Optional[ProductDB]: # Devuelve el objeto borrado o None
        ...
```

* **Nota sobre `AsyncSession`:** En este diseño de interfaz, estamos haciendo explícito que los métodos del repositorio operarán sobre una `AsyncSession` que se les provee. Esto permite que la gestión de la transacción (y el `commit/rollback`) ocurra fuera del repositorio, típicamente en la capa de servicio o `use case` que orquesta la `unit of work`.

**2. Implementando el Repositorio Concreto con SQLAlchemy (El Bibliotecario en Acción) 🛠️**

Ahora, creamos una clase que implementa esta interfaz usando SQLAlchemy.

```python
# app/repositories/implementations/sql_product_repository.py
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.future import select # O from sqlalchemy import select
from typing import List, Optional, Any

# from app.models.product_db import ProductDB
# from app.schemas.product_api import ProductCreatePydantic, ProductUpdatePydantic
# from app.repositories.protocols.product_repository_protocol import IProductRepository # Importar el Protocol

# Re-definimos placeholders si no se importan para que el ejemplo sea autocontenido
class ProductDB: # SQLAlchemy Model (placeholder)
    id: int
    name: str
    # ... otros campos

    def __init__(self, **kwargs): # Constructor simple para el placeholder
        for key, value in kwargs.items():
            setattr(self, key, value)

class ProductCreatePydantic(BaseModel): # Pydantic Model (placeholder)
    name: str
    description: Optional[str] = None
    price: float
    is_available: bool = True

class ProductUpdatePydantic(BaseModel): # Pydantic Model (placeholder)
    name: Optional[str] = None
    description: Optional[str] = None
    price: Optional[float] = None
    is_available: Optional[bool] = None

class SQLAlchemyProductRepository: # No necesita heredar explícitamente de IProductRepository si usamos Protocol
    
    async def get_by_id(self, db: AsyncSession, product_id: int) -> Optional[ProductDB]:
        return await db.get(ProductDB, product_id)

    async def get_all(self, db: AsyncSession, skip: int = 0, limit: int = 100) -> List[ProductDB]:
        result = await db.execute(
            select(ProductDB).offset(skip).limit(limit)
        )
        return list(result.scalars().all()) # Convertir a lista

    async def get_by_name(self, db: AsyncSession, name: str) -> Optional[ProductDB]:
        result = await db.execute(
            select(ProductDB).where(ProductDB.name == name) # Ajustar el where según el modelo real
        )
        return result.scalar_one_or_none()

    async def create(self, db: AsyncSession, product_in: ProductCreatePydantic) -> ProductDB:
        # El mapeo DTO -> Entity ORM ocurre aquí o en el service layer ANTES de llamar al repo.
        # Por simplicidad, asumimos que product_in puede usarse para crear ProductDB directamente.
        # En la práctica, el product_in sería un Domain Object o una Entity ya mapeada.
        # Si product_in es un DTO, el mapeo es responsabilidad del llamador o del repo mismo.
        # Aquí vamos a asumir que el repositorio toma el DTO y crea la Entity ORM.
        
        db_product = ProductDB(**product_in.model_dump())
        db.add(db_product)
        # El flush es opcional aquí, pero puede ser útil si necesitas el ID inmediatamente
        # ANTES del commit final que haría el service layer.
        # await db.flush()
        # await db.refresh(db_product) # Si flush generó el ID y quieres el objeto refrescado
        return db_product # Devuelve la instancia ORM, aún no commiteada por el repo

    async def update(
        self, db: AsyncSession, product_db_obj: ProductDB, product_in: ProductUpdatePydantic
    ) -> ProductDB:
        update_data = product_in.model_dump(exclude_unset=True) # Solo campos provistos en el DTO
        for key, value in update_data.items():
            setattr(product_db_obj, key, value)
        db.add(product_db_obj) # SQLAlchemy rastrea los cambios
        # await db.flush()
        # await db.refresh(product_db_obj)
        return product_db_obj # Devuelve la instancia ORM actualizada, aún no commiteada

    async def delete(self, db: AsyncSession, product_id: int) -> Optional[ProductDB]:
        product_to_delete = await db.get(ProductDB, product_id)
        if product_to_delete:
            await db.delete(product_to_delete)
            # await db.flush()
            return product_to_delete
        return None
```

* **Gestión de `Commit/Flush`:** Notarás que los métodos `create`, `update`, y `delete` modifican la `session` (`db.add`, `db.delete`) pero **no llaman a `await db.commit()`**. Esto es intencional y una práctica común. Permite que múltiples operaciones de repositorio (quizás para diferentes `entities`) se agrupen en una única transacción gestionada por una capa superior (un `Application Service` o una `Unit of Work`, que veremos en 13.4).
  * `await db.flush()`: Si necesitas que los cambios se envíen a la BBDD (ej. para obtener un ID autogenerado o para que se disparen `constraints` antes del `commit` final), puedes usar `flush()`.
  * El `commit` final lo hará quien orqueste la `unit of work`.

**3. Utilizando Repositorios en `Application Services` o `Endpoints` FastAPI 🚀**

Ahora, ¿cómo usamos este repositorio? A través de `Dependency Injection`.

*   **`Dependency Provider` para el Repositorio:**

    ```python
    # app/api/dependencies.py
    # from sqlalchemy.ext.asyncio import AsyncSession
    # from app.db.session import get_session # Nuestra dependencia de sesión
    # from app.repositories.implementations.sql_product_repository import SQLAlchemyProductRepository
    # from app.repositories.protocols.product_repository_protocol import IProductRepository

    def get_product_repository(db: AsyncSession = Depends(get_session)) -> IProductRepository: # Devuelve el Protocolo!
        return SQLAlchemyProductRepository() # La session 'db' no se pasa aquí, se pasará a cada método
                                             # o, alternativamente, pasar 'db' al constructor de SQLAlchemyProductRepository
                                             # y que el repo la guarde. ¡Esta segunda opción es más común!
    ```

    **Refinamiento para el `Provider` y Constructor del Repo:**\
    Es más idiomático que el repositorio reciba la `session` en su constructor.

    ```python
    # app/repositories/implementations/sql_product_repository.py (modificado)
    # class SQLAlchemyProductRepository:
    #     def __init__(self, db: AsyncSession): # Recibe la session aquí
    #         self.db = db
        
    #     async def get_by_id(self, product_id: int) -> Optional[ProductDB]: # Ya no necesita 'db' como arg
    #         return await self.db.get(ProductDB, product_id)
        # ... otros métodos modificados para usar self.db ...

    # app/api/dependencies.py (modificado)
    # def get_product_repository(db: AsyncSession = Depends(get_session)) -> IProductRepository:
    #     return SQLAlchemyProductRepository(db=db) # Inyectar la session al crear el repo
    ```
*   **Inyectando y Usando el Repositorio en un `Endpoint` (o `Service Layer`):**

    ```python
    # app/api/endpoints/products.py
    # from .schemas_api import ProductResponse # Pydantic model para la respuesta
    # from .dependencies import get_product_repository
    # from .protocols.product_repository_protocol import IProductRepository

    # @router.get("/{product_id}", response_model=ProductResponse)
    # async def get_product_details(
    #     product_id: int,
    #     product_repo: IProductRepository = Depends(get_product_repository) # Inyectar la INTERFAZ
    # ):
    #     db_product = await product_repo.get_by_id(product_id) # El repo usa la session que se le inyectó
    #     if db_product is None:
    #         raise HTTPException(status_code=404, detail="Product not found")
    #     return db_product # FastAPI mapeará ProductDB a ProductResponse
    ```

    _Observa que el `endpoint` depende de `IProductRepository` (el `protocol`), no de la implementación concreta. Esto es clave para el desacoplamiento y la testabilidad.El `commit` (si es una operación de escritura) lo haría el `endpoint` o el `service layer` después de llamar al método del repositorio, usando la misma `session` que se inyectó al repositorio._

    ```python
    # Ejemplo de creación en un endpoint usando el repo y manejando commit
    # @router.post("/", response_model=ProductResponse, status_code=status.HTTP_201_CREATED)
    # async def create_new_product(
    #     product_in: ProductCreatePydantic,
    #     product_repo: IProductRepository = Depends(get_product_repository),
    #     db: AsyncSession = Depends(get_session) # Obtener la session también aquí para commit/rollback
    # ):
    #     # Lógica para mapear product_in (DTO) a una entidad de dominio o directamente
    #     # al formato que espera product_repo.create si es necesario.
    #     # Asumimos que product_repo.create puede tomar el DTO.
    #     try:
    #         # El repositorio hace db.add(db_obj)
    #         created_product_orm = await product_repo.create(product_in) 
    #         await db.commit() # El endpoint (o service) es responsable del commit
    #         await db.refresh(created_product_orm) # Opcional, si create no lo hace
    #         return created_product_orm
    #     except Exception as e: # Idealmente, excepciones más específicas
    #         await db.rollback()
    #         raise HTTPException(status_code=500, detail=f"Error creating product: {str(e)}")
    ```

**4. Repositorios Genéricos vs. Específicos: Encontrando el Equilibrio ⚖️**

* **Repositorio Genérico (`GenericRepository`):** Una clase base con métodos CRUD comunes (ej. `add(entity)`, `get(id)`, `list_all()`, `delete(entity)`), parametrizada por el tipo de `entity`.
  * **Pros:** Reduce mucho `boilerplate` si tienes muchas `entities` con operaciones CRUD estándar.
  * **Cons:** Puede llevar a una interfaz demasiado genérica ("talla única no sirve para todos") o a la necesidad de `casting`. A veces, las `queries` no son tan simples.
* **Repositorio Específico (`SpecificRepository`):** Un repositorio para cada `entity` o `aggregate root` (ej. `ProductRepository`, `UserRepository`).
  * **Pros:** Interfaces claras y adaptadas a las necesidades de cada `entity`. Permite añadir métodos de consulta muy específicos (ej. `product_repo.find_by_category_and_price_range(...)`).
  * **Cons:** Más código si muchas `entities` comparten la misma lógica CRUD.
* **Solución Común:** Usar un `GenericRepository` como base y luego hacer que los `SpecificRepositories` hereden de él, añadiendo sus métodos especializados.

**Diagrama `Mermaid`: El Repositorio en la Arquitectura (Flujo Detallado) 🧱**

```mermaid
      sequenceDiagram
        participant Client
        participant FastAPI_EP as FastAPI Endpoint - ej. POST /products/
        participant Dep_get_prod_repo as Depends - get_product_repository
        participant Dep_get_sess as Depends - get_session
        participant ProdRepoImpl as SQLAlchemyProductRepository - Implementa IProductRepository
        participant ASess as AsyncSession
        participant DB as Database

        Client->>+FastAPI_EP: HTTP Request (Payload: ProductCreatePydantic)
        FastAPI_EP->>+Dep_get_prod_repo: Solicita IProductRepository
        Dep_get_prod_repo->>+Dep_get_sess: Solicita AsyncSession
        Dep_get_sess-->>-Dep_get_prod_repo: Inyecta session
        Dep_get_prod_repo->>ProdRepoImpl: Crear SQLAlchemyProductRepository(session)
        ProdRepoImpl-->>-Dep_get_prod_repo: repo_instance
        Dep_get_prod_repo-->>FastAPI_EP: Inyecta repo_instance como product_repo

        FastAPI_EP->>+FastAPI_EP: (Opcional) Mapea DTO a entidad
        FastAPI_EP->>+ProdRepoImpl: Ejecuta product_repo.create(entity o dto)
        ProdRepoImpl->>+ASess: session.add(db_entity)
        ASess-->>-ProdRepoImpl: Objeto añadido a la sesión
        ProdRepoImpl-->>-FastAPI_EP: created_orm_entity (sin commit)

        FastAPI_EP->>+ASess: session.commit() (commit directo o por Service Layer)
        ASess->>DB: SQL INSERT
        DB-->>ASess: Commit OK
        ASess->>FastAPI_EP: session.refresh(created_orm_entity)
        ASess-->>-FastAPI_EP: created_orm_entity con ID

        Note over FastAPI_EP: Serializa created_orm_entity a JSON
        FastAPI_EP-->>-Client: HTTP 201 Response

```

**Beneficios Reafirmados del Patrón Repositorio ✨**

* **Máximo Desacoplamiento:** La lógica de negocio (`services`) depende de una abstracción (`IProductRepository`), no de los detalles de SQLAlchemy.
* **Centralización de `Queries`:** Toda la lógica para acceder a los datos de `ProductDB` está en `SQLAlchemyProductRepository`. Si necesitas cambiar cómo se obtiene un producto, solo modificas un lugar.
* **Testabilidad Superior:** Puedes `mockear` `IProductRepository` en tus pruebas unitarias para los `services` o `endpoints`, eliminando la dependencia de una base de datos real.
* **Legibilidad del Código de Negocio:** Los `services` se enfocan en orquestar la lógica de negocio y las reglas, no en construir `queries` SQL o interactuar con la `session` del ORM directamente para cada operación.

**Conclusión: Repositorios como Guardianes de Tu Lógica de Datos, Protegiendo Tu Dominio 🏯**

El Patrón Repositorio es una abstracción poderosa y fundamental en la construcción de aplicaciones mantenibles y escalables. Cuando se implementa correctamente con SQLAlchemy y se integra en FastAPI mediante `Dependency Injection`:

* Eleva la calidad de tu código separando incumbencias.
* Facilita las pruebas y la evolución de tu `data access layer`.
* Permite que tu lógica de dominio permanezca limpia y agnóstica a la persistencia.

Es un paso esencial hacia una arquitectura más limpia y profesional, preparando el terreno para conceptos como la `Unit of Work` que exploraremos a continuación.

## 13.4 Gestión de Transacciones Locales: Asegurando la Atomicidad en Tus Operaciones de Datos

En cualquier aplicación que interactúe con una base de datos, especialmente para operaciones de escritura (crear, actualizar, eliminar), la **atomicidad** es un concepto crítico. Queremos que una serie de cambios relacionados en la base de datos se traten como una **única unidad de trabajo indivisible (`single, indivisible unit of work`)**: o todas las operaciones dentro de esa unidad tienen éxito y se aplican (`commit`), o si alguna falla, todas las operaciones anteriores dentro de esa unidad se deshacen (`rollback`), dejando la base de datos en el estado en que estaba antes de que comenzara la unidad. Esto es la esencia de una **transacción**.

Cuando hablamos de **transacciones locales**, nos referimos a transacciones que involucran a una **única base de datos** o un único `resource manager` transaccional.

**¿Por Qué Son Cruciales las Transacciones? El Principio ACID**

Las transacciones son un pilar de las bases de datos relacionales y se asocian a menudo con las propiedades ACID:

* **A**tomicidad (`Atomicity`): Todas las operaciones de la transacción se completan, o ninguna lo hace. No hay estados intermedios.
* **C**onsistencia (`Consistency`): La transacción lleva la base de datos de un estado válido a otro estado válido, manteniendo la integridad referencial y los `constraints`.
* **I**solamiento (`Isolation`): Las transacciones concurrentes no deben interferir entre sí. Cada transacción "siente" que se ejecuta de forma aislada. (Los niveles de aislamiento son un tema profundo: `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`).
* **D**urabilidad (`Durability`): Una vez que una transacción ha hecho `commit`, los cambios son permanentes y sobreviven a fallos del sistema (ej. reinicios).

**Gestión de Transacciones con `AsyncSession` de SQLAlchemy en FastAPI**

SQLAlchemy, a través de su objeto `AsyncSession`, proporciona los mecanismos para gestionar transacciones.

1. **Inicio de una Transacción:**
   * **Implícito:** Cuando realizas la primera operación de base de datos (ej. una `query`, `session.add()`) con una `AsyncSession` "fresca", SQLAlchemy a menudo inicia una transacción implícitamente si el `engine` de la base de datos subyacente opera de esa manera (la mayoría de las BBDD relacionales lo hacen).
   *   **Explícito con `session.begin()`:** Para un control más granular y claro del alcance de una transacción, puedes usar `async with session.begin():` (o `await session.begin()` si no usas el `context manager`). Esto inicia explícitamente una transacción. Si el bloque `async with` termina sin excepciones, la transacción hace `commit` automáticamente. Si ocurre una excepción dentro del bloque, hace `rollback` automáticamente.

       ```python
       # async def mi_operacion_transaccional(db: AsyncSession, data_a: Any, data_b: Any):
       #     async with db.begin(): # Inicia una transacción explícita
       #         # Operación 1: Crear objeto A
       #         obj_a = MiModeloA(**data_a)
       #         db.add(obj_a)
       #         await db.flush() # Para obtener obj_a.id si es autogenerado, por ejemplo
               
       #         # Operación 2: Crear objeto B que depende de A
       #         obj_b = MiModeloB(a_id=obj_a.id, **data_b)
       #         db.add(obj_b)
               
       #         # Si llegamos aquí sin errores, db.commit() se llamará automáticamente al salir del 'with'
           
       #     # Si hubo una excepción dentro del 'with', db.rollback() se llamó automáticamente
       #     return obj_a, obj_b 
       ```
2. **Confirmación de Cambios (`Commit`):**
   * `await session.commit()`: Persiste todos los cambios realizados en la `session` (objetos añadidos, modificados, eliminados) en la base de datos de forma atómica. Si `commit()` tiene éxito, los cambios son duraderos. Si falla (ej. por una violación de `constraint` que no se detectó con `flush`), lanza una excepción y la transacción debería hacerse `rollback`.
   * Es la operación que "guarda el juego".
3. **Deshacer Cambios (`Rollback`):**
   * `await session.rollback()`: Descarta todos los cambios realizados en la `session` desde el inicio de la transacción actual (o desde el último `commit`). La base de datos vuelve al estado anterior.
   * Esencial cuando ocurre un error durante una unidad de trabajo para mantener la consistencia.
4. **`Flush` (Sincronización Intermedia):**
   * `await session.flush()`: Envía las sentencias SQL pendientes a la base de datos _sin terminar la transacción_. Esto es útil para:
     * Obtener IDs autogenerados por la base de datos para objetos recién añadidos.
     * Disparar `constraints` de base de datos (`NOT NULL`, `UNIQUE`) antes del `commit` final.
     * Ver los efectos de tus cambios en `queries` subsecuentes _dentro de la misma transacción_.
   * Un `flush` puede fallar (ej. `IntegrityError`). Si esto ocurre, la transacción sigue activa y puedes hacer `rollback`.

**¿Dónde Gestionar las Transacciones? La Capa de Servicio o `Use Case`**

En el patrón Repositorio (13.3), dijimos que los métodos del repositorio típicamente no hacen `commit` ellos mismos. Esto es porque una **unidad de trabajo de negocio** puede involucrar operaciones en múltiples repositorios o múltiples llamadas a métodos del mismo repositorio.

* **La Responsabilidad de la Capa de Servicio (`Application Service` / `Use Case Handler`):**\
  Esta capa es la candidata ideal para definir el alcance de una transacción. Un método de servicio orquesta las operaciones necesarias para cumplir un caso de uso:
  1. Obtiene una `AsyncSession` (a través de `Depends` si el servicio mismo es una dependencia, o se le pasa como argumento).
  2. Inicia una transacción (explícita o implícitamente).
  3. Llama a uno o más métodos de uno o más repositorios, pasándoles la misma `session`.
  4. Si todas las operaciones son exitosas, hace `commit` a la `session`.
  5. Si alguna operación falla, hace `rollback` a la `session`.
*   **Ejemplo con una Capa de Servicio:**

    ```python
    # --- app/services/order_service.py ---
    from sqlalchemy.ext.asyncio import AsyncSession
    # from app.repositories.protocols import IOrderRepository, IOrderItemRepository, IProductRepository
    # from app.schemas.order_api import OrderCreateDTO
    # from app.models.order_db import OrderDB, OrderItemDB # SQLAlchemy models
    # from app.domain_exceptions import ProductOutOfStockError, InsufficientPaymentError

    class OrderService:
        def __init__(
            self, 
            order_repo: IOrderRepository, 
            item_repo: IOrderItemRepository,
            product_repo: IProductRepository # Para verificar stock, por ejemplo
        ):
            self.order_repo = order_repo
            self.item_repo = item_repo
            self.product_repo = product_repo

        async def place_order(self, db: AsyncSession, order_data: OrderCreateDTO, user_id: int) -> OrderDB:
            # Iniciar una transacción explícita es una buena práctica aquí para claridad
            async with db.begin(): # ¡Esto maneja commit/rollback automáticamente!
                # 1. Verificar stock de productos (usando product_repo)
                # for item_dto in order_data.items:
                #     product = await self.product_repo.get_by_id(db, item_dto.product_id)
                #     if not product or product.stock < item_dto.quantity:
                #         raise ProductOutOfStockError(product_id=item_dto.product_id)
                #     # Reducir stock (esto también sería parte de la transacción)
                #     product.stock -= item_dto.quantity
                #     await self.product_repo.update(db, product, {"stock": product.stock}) # Asumiendo que update lo permite

                # 2. Crear el objeto OrderDB (usando order_repo)
                # (Mapear order_data DTO a los campos necesarios para OrderDB)
                # new_order_orm = await self.order_repo.create(db, order_data_para_orm, user_id)
                
                # 3. Crear los OrderItemDB objects (usando item_repo)
                # for item_dto in order_data.items:
                #     await self.item_repo.create_for_order(db, new_order_orm.id, item_dto)
                
                # 4. (Opcional) Procesar pago o marcar para pago
                # if not await self.procesar_pago_simulado(db, new_order_orm.id, order_data.total_amount):
                #     raise InsufficientPaymentError(order_id=new_order_orm.id)

                # Si todas las operaciones dentro del 'async with db.begin():' son exitosas,
                # SQLAlchemy hará commit automáticamente al salir del bloque.
                # Si alguna de las operaciones (o validaciones) lanza una excepción,
                # SQLAlchemy hará rollback automáticamente al salir del bloque with por la excepción.
                
                # return new_order_orm # Devolver la orden creada y commiteada
                pass # Placeholder para la lógica real


    # --- En el endpoint FastAPI ---
    # from .dependencies import get_session, get_order_service # get_order_service inyectaría los repos

    # @router.post("/orders/", response_model=OrderResponseDTO, status_code=status.HTTP_201_CREATED)
    # async def create_order_endpoint(
    #     order_in: OrderCreateDTO,
    #     current_user: User = Depends(get_current_authenticated_user), # Asumiendo auth
    #     order_service: OrderService = Depends(get_order_service),
    #     db: AsyncSession = Depends(get_session) # La sesión es necesaria para el commit/rollback aquí
    #                                            # o el servicio podría manejarlo si recibe la session factory
    # ):
    #     try:
    #         # El método place_order ahora usa 'async with db.begin()' internamente
    #         # o el servicio gestiona commit/rollback de la sesión que se le pasa.
    #         # Si el servicio toma la sesión como argumento y la gestiona él mismo:
    #         new_order = await order_service.place_order(db, order_in, current_user.id)
    #         # No se necesita db.commit() aquí si place_order usa 'async with db.begin()'
    #         # o si el servicio hace commit de la sesión que se le pasó.
    #         return new_order 
    #     except ProductOutOfStockError as e:
    #         # await db.rollback() # No es necesario si place_order usa 'async with db.begin()'
    #         raise HTTPException(status_code=status.HTTP_409_CONFLICT, detail=str(e))
    #     except InsufficientPaymentError as e:
    #         # await db.rollback()
    #         raise HTTPException(status_code=status.HTTP_402_PAYMENT_REQUIRED, detail=str(e))
    #     except Exception as e: # Captura general para otros errores inesperados
    #         # await db.rollback()
    #         # Loggear el error 'e'
    #         raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail="Ocurrió un error procesando el pedido.")

    ```

    * **El Patrón `Unit of Work`:** El concepto de que una capa de servicio gestiona el inicio, `commit` y `rollback` de una transacción que puede abarcar múltiples operaciones de repositorio es la esencia del patrón `Unit of Work`. El `async with db.begin():` de SQLAlchemy es una forma de implementar una UoW simple.

**Diagrama `Mermaid`: Transacción Local Orquestada por un `Service Layer`**

```mermaid
sequenceDiagram
    participant EP as FastAPI Endpoint
    participant Dep_Service as Depends(get_order_service)
    participant OrderServ as OrderService
    participant Dep_Session as Depends(get_session)
    participant ASess as AsyncSession
    participant OrderRepo as IOrderRepository
    participant ItemRepo as IOrderItemRepository
    participant ProductRepo as IProductRepository
    participant DB as Database

    EP->>+Dep_Service: Solicita `OrderService`
    Dep_Service->>+Dep_Session: (Implicitly) `OrderService` necesita `AsyncSession` para sus repositorios
    Dep_Session-->>Dep_Service: Inyecta `session` a `OrderService` (o a sus repos)
    Dep_Service-->>-EP: Inyecta `order_service_instance`

    EP->>+Dep_Session: (Si el endpoint también toma la sesión directamente para commit/rollback)
    Dep_Session-->>-EP: Inyecta `session` como `db`

    EP->>+OrderServ: `await order_service.place_order(db, order_data, user_id)`
    Note over OrderServ: `async with db.begin():` o `await db.begin_nested()` si ya hay transacción
    OrderServ->>+ProductRepo: `await product_repo.check_stock_and_update(db, item)`
    ProductRepo->>DB: SQL SELECT/UPDATE
    DB-->>ProductRepo: Stock OK
    ProductRepo-->>-OrderServ: 
    
    OrderServ->>+OrderRepo: `await order_repo.create(db, new_order_data)`
    OrderRepo->>DB: SQL INSERT (Order)
    DB-->>OrderRepo: Order Creada
    OrderRepo-->>-OrderServ: `new_order_orm`
    
    OrderServ->>+ItemRepo: `await item_repo.create_for_order(db, order_id, item_data)`
    ItemRepo->>DB: SQL INSERT (OrderItems)
    DB-->>ItemRepo: Items Creados
    ItemRepo-->>-OrderServ: 
    
    Note over OrderServ: Si todo OK, `async with db.begin()` hace `COMMIT` al salir. <br/> Si hay excepción, hace `ROLLBACK`.
    OrderServ-->>-EP: `new_order_orm` (o lanza excepción)
    
    EP-->>Client: HTTP Response
```

**Consideraciones Adicionales:**

* **Niveles de Aislamiento (`Isolation Levels`):** SQLAlchemy y tu base de datos permiten configurar el nivel de aislamiento de las transacciones. El nivel por defecto (`READ COMMITTED` en muchas BBDD) es a menudo suficiente, pero para ciertos casos de uso podrías necesitar niveles más estrictos (ej. `REPEATABLE READ`, `SERIALIZABLE`), lo cual puede tener implicaciones en la concurrencia y el rendimiento.
* **`Savepoints` (Transacciones Anidadas):** SQLAlchemy soporta `savepoints` o transacciones anidadas (`session.begin_nested()`). Esto permite hacer `rollback` a un punto intermedio de una transacción más grande sin deshacer toda la transacción. Útil para lógica compleja con pasos opcionales o recuperables.
* **Manejo de `Deadlocks`:** En sistemas con alta concurrencia, los `deadlocks` en la base de datos pueden ocurrir. Tu aplicación debe estar preparada para capturar excepciones de `deadlock` y potencialmente reintentar la transacción (con cuidado y `backoff`).

**Conclusión: La Transacción Local Como Red de Seguridad para la Integridad de Tus Datos transact**

La gestión adecuada de transacciones locales es no negociable para cualquier aplicación que modifique datos. Asegura que tus operaciones de negocio se apliquen de manera atómica y consistente, preservando la integridad de tu base de datos.

En FastAPI, al usar SQLAlchemy asíncrono:

* La `AsyncSession` es tu herramienta principal para el control transaccional (`commit`, `rollback`, `begin`).
* La capa de **Servicio de Aplicación** o **`Use Case Handler`** es el lugar natural para orquestar estas unidades de trabajo, a menudo utilizando el `context manager` `async with session.begin():` para una gestión declarativa y segura de la transacción.
* Los **Repositorios** participan en la transacción proporcionando los métodos de acceso a datos, pero usualmente no gestionan el `commit/rollback` final ellos mismos.

Dominar este flujo es esencial para construir `microservices` que no solo sean rápidos, sino también fiables y correctos en su manejo de datos.

## 13.5 Transacciones distribuidas: sagas y outbox

Entramos en uno de los territorios más desafiantes pero cruciales de los `microservices`: las **transacciones distribuidas**. Si el 13.4 trató sobre asegurar la atomicidad dentro de _una única_ base de datos (transacciones locales), el **13.5** aborda el complejo problema de mantener la consistencia cuando una operación de negocio abarca _múltiples_ servicios, cada uno potencialmente con su propia base de datos.

En un sistema monolítico con una única base de datos, las transacciones ACID locales (13.4) son nuestros grandes aliados para mantener la consistencia. Pero en una arquitectura de `microservices`, una única "operación de negocio" puede requerir la coordinación de cambios de estado a través de varios servicios independientes, cada uno con su propia base de datos.

**El Problema: La Imposibilidad de las Transacciones ACID Distribuidas Tradicionales**

Los protocolos de `commit` en dos fases (2PC - `Two-Phase Commit`) que intentan extender las propiedades ACID a través de múltiples bases de datos son notorios por su complejidad, fragilidad y el impacto negativo en la disponibilidad y el rendimiento. En la práctica, **no se suelen recomendar para arquitecturas de `microservices` modernas** que priorizan el desacoplamiento y la resiliencia.

Entonces, ¿cómo mantenemos una consistencia de datos razonable a través de los servicios? Aquí es donde entran en juego patrones como **Sagas** y el patrón **Transactional Outbox**. Estos patrones abrazan la **consistencia eventual (`eventual consistency`)** pero proporcionan mecanismos para asegurar que el sistema, como un todo, llegue a un estado consistente.

**1. El Patrón `Saga`: Orquestando una Danza de Transacciones Locales**

Una `Saga` es una secuencia de transacciones locales donde cada transacción actualiza los datos dentro de un único servicio. Si una transacción local falla en la secuencia, la `Saga` ejecuta una serie de **transacciones compensatorias (`compensating transactions`)** para deshacer los cambios realizados por las transacciones locales previas que sí tuvieron éxito.

* **Concepto Clave:** En lugar de un `commit` atómico global, tenemos una serie de "pasos" (transacciones locales) y "contra-pasos" (transacciones compensatorias).
* **Tipos de Sagas:**
  * **Coreografía (`Choreography`):** No hay un coordinador central. Cada servicio participante publica `events` cuando completa su transacción local. Otros servicios escuchan estos `events` y ejecutan sus propias transacciones locales (o compensatorias) en respuesta.
    * **Pros:** Desacoplado, sin punto único de fallo en el coordinador.
    * **Cons:** Puede ser difícil de rastrear el estado global de la `Saga`. El flujo de control está distribuido. `Cyclic dependencies` pueden ser un problema.
  * **Orquestación (`Orchestration`):** Un servicio coordinador (el "orquestador de la `Saga`") dirige la secuencia de transacciones locales, diciéndole a cada servicio participante qué operación realizar. Si un paso falla, el orquestador es responsable de invocar las transacciones compensatorias necesarias en el orden correcto.
    * **Pros:** Flujo de control centralizado, más fácil de entender y monitorizar el estado de la `Saga`.
    * **Cons:** El orquestador puede ser un punto único de fallo (necesita ser resiliente). Introduce más acoplamiento (aunque es de orquestación).
* **Ejemplo de `Saga` Coreografiada (Creación de Pedido):**
  1. **`Orders Service`:**
     * `Transaction L1:` Crea el pedido en estado `PENDING`.
     * Publica `OrderCreatedEvent`.
  2. **`Customers Service`:**
     * Escucha `OrderCreatedEvent`.
     * `Transaction L2:` Verifica el crédito del cliente y reserva el crédito.
     * Si OK: Publica `CustomerCreditReservedEvent`.
     * Si Falla: Publica `CustomerCreditLimitExceededEvent`.
       * **Compensación (escuchada por `Orders Service`):** `Orders Service` cambia el estado del pedido a `FAILED_NO_CREDIT`.
  3. **`Stock Service`:**
     * Escucha `CustomerCreditReservedEvent`.
     * `Transaction L3:` Reserva el stock de los productos.
     * Si OK: Publica `StockReservedEvent`.
     * Si Falla: Publica `StockUnavailableEvent`.
       * **Compensación (escuchada por `Customers Service`):** `Customers Service` libera el crédito reservado (Transacción Compensatoria C2). Publica `CustomerCreditReleasedEvent`.
       * **Compensación (escuchada por `Orders Service`):** `Orders Service` cambia el estado del pedido a `FAILED_NO_STOCK`.
  4. **`Orders Service`:**
     * Escucha `StockReservedEvent`.
     * `Transaction L4:` Cambia el estado del pedido a `APPROVED` o `CONFIRMED`.
  5. **Diagrama de una `Saga` Coreografiada (Simplificada):**

```mermaid
   
sequenceDiagram
    participant Client
    participant OS as Orders Service
    participant CS as Customers Service
    participant SS as Stock Service
    participant Bus as Event Bus

    Client->>+OS: Crear Pedido
    OS->>OS: LTX1: Guardar Pedido (PENDING)
    OS-->>Bus: Publica OrderCreatedEvent
    Bus-->>CS: OrderCreatedEvent
    CS->>CS: LTX2: Reservar Crédito
    alt Crédito OK
        CS-->>Bus: CustomerCreditReservedEvent
        Bus-->>SS: CustomerCreditReservedEvent
        SS->>SS: LTX3: Reservar Stock
        alt Stock OK
            SS-->>Bus: StockReservedEvent
            Bus-->>+OS: StockReservedEvent
            OS->>OS: LTX4: Confirmar Pedido (APPROVED)
            OS-->>-Client: Pedido Confirmado
        else Stock NO OK
            SS-->>Bus: StockUnavailableEvent
            Bus-->>CS: StockUnavailableEvent (Compensación)
            CS->>CS: CTX2: Liberar Crédito
            CS-->>Bus: CustomerCreditReleasedEvent
            Bus-->>+OS: StockUnavailableEvent / CreditReleased
            OS->>OS: CTX1: Pedido FAILED_NO_STOCK
            OS-->>-Client: Pedido Fallido (Sin Stock)
        end
    else Crédito NO OK
        CS-->>Bus: CustomerCreditLimitExceededEvent
        Bus-->>+OS: CustomerCreditLimitExceededEvent
        OS->>OS: CTX1: Pedido FAILED_NO_CREDIT
        OS-->>-Client: Pedido Fallido (Sin Crédito)
    end
```

* **Transacciones Compensatorias:** Deben ser idempotentes y, idealmente, no deberían fallar (o tener una estrategia de reintento muy robusta). Diseñar compensaciones puede ser complejo.

**2. El Patrón `Transactional Outbox`: Asegurando la Entrega de `Events`**

Un problema común al publicar `events` después de una transacción local es: ¿qué pasa si la transacción de base de datos (ej. guardar el pedido) tiene éxito, pero la publicación del `event` al `message broker` falla (ej. el `broker` está caído)? Terminas con un estado inconsistente: el pedido se creó, pero nadie más se enteró.

El patrón `Transactional Outbox` resuelve esto asegurando que la acción de guardar el estado de negocio y la acción de publicar el `event` ocurran de forma **atómica** desde la perspectiva del servicio.

* **Cómo Funciona:**
  1. Dentro de la **misma transacción de base de datos local** donde el servicio modifica su propio estado (ej. crea un `Order` en la tabla `orders`), también inserta una representación del `event` o `message` que necesita ser publicado en una tabla especial en la misma base de datos, llamada `outbox` (ej. `events_to_publish`).
  2. Si la transacción local hace `commit`, tanto los cambios de negocio como el `event` en la tabla `outbox` se guardan atómicamente.
  3. Si la transacción local hace `rollback`, ninguno de los dos se guarda.
  4. Un **proceso separado** (o un `thread` en segundo plano, o un `CDC - Change Data Capture system` como Debezium) monitorea la tabla `outbox`.
  5. Este proceso lee los `events` de la tabla `outbox` y los publica de manera fiable en el `message broker` (Kafka, RabbitMQ).
  6. Una vez que el `event` ha sido publicado exitosamente en el `broker`, el proceso marca el `event` en la tabla `outbox` como "publicado" o lo elimina.
* **Beneficios:**
  * **Garantiza "At-Least-Once Delivery" de `Events`:** Previene la pérdida de `events` si la publicación directa al `broker` falla después de un `commit` de negocio.
  * **Consistencia Atómica:** El `state change` y la intención de publicar el `event` son atómicos.
* **Diagrama `Mermaid` del Patrón `Outbox`:**

```mermaid
sequenceDiagram
    participant AppService as Application Service
    participant DB_TX as Local DB Transaction
    participant DB_Orders as Orders Table
    participant DB_Outbox as Outbox Table
    participant MsgRelay as Message Relay Process
    participant MsgBroker as Message Broker (Kafka / RabbitMQ)

    AppService->>+DB_TX: BEGIN TRANSACTION
    DB_TX->>DB_Orders: INSERT INTO orders (...)
    DB_TX->>DB_Outbox: INSERT INTO outbox_events (event_payload)
    DB_TX-->>-AppService: COMMIT TRANSACTION

    Note right of AppService: Estado y evento guardados atómicamente

    loop Polling o CDC
        MsgRelay->>+DB_Outbox: Leer eventos no publicados
        DB_Outbox-->>-MsgRelay: Nuevos eventos
        alt Hay eventos
            MsgRelay->>+MsgBroker: Publicar evento
            MsgBroker-->>-MsgRelay: Publicación OK
            MsgRelay->>+DB_Outbox: Marcar evento como publicado
            DB_Outbox-->>-MsgRelay: Actualización OK
        end
    end
```

* **Implementación:**
  * La tabla `outbox` típicamente almacena el `payload` del `event`, el tipo de `event`, el `topic/exchange` destino, y un `status` (ej. `PENDING`, `PUBLISHED`).
  * El `Message Relay` puede ser un `worker` separado, un `scheduled job`, o un `listener` de `Change Data Capture (CDC)` si tu base de datos lo soporta (ej. Debezium con PostgreSQL o MySQL). `CDC` es a menudo la solución más eficiente y de menor latencia para esto.

**Combinando Sagas y `Outbox`:**

El patrón `Outbox` es a menudo un compañero esencial del patrón `Saga`, especialmente en Sagas coreografiadas. Cada servicio que participa en la `Saga` y necesita publicar un `event` después de su transacción local puede usar el patrón `Outbox` para asegurar que su `event` se publique de manera fiable.

**Desafíos de las Transacciones Distribuidas y Estos Patrones:**

* **Complejidad:** Implementar Sagas y el patrón `Outbox` (especialmente el `Message Relay` o `CDC`) añade una complejidad considerable a tu sistema en comparación con las transacciones locales.
* **Consistencia Eventual:** Debes estar cómodo con la idea de que el sistema global no será consistente de forma instantánea. Habrá ventanas de tiempo donde un servicio ha completado su parte, pero otros aún no han reaccionado.
* **Diseño de Transacciones Compensatorias:** Pueden ser difíciles de diseñar correctamente y deben ser infalibles o altamente resilientes.
* **Pruebas y Depuración:** Testear y depurar flujos de `Saga` puede ser complicado debido a su naturaleza distribuida y asíncrona. La trazabilidad (como vimos en 9.6) es vital.
* **Idempotencia:** Los `consumers` de `events` (y los `handlers` de transacciones compensatorias) deben ser idempotentes, ya que los `events` podrían ser entregados más de una vez por el `Message Relay` si hay reintentos.

**Conclusión: Aceptando la Complejidad por la Resiliencia y el Desacoplamiento ⚖️**

Las transacciones distribuidas son uno de los problemas más difíciles en `microservices`. Mientras que las transacciones ACID locales son la norma para una única base de datos, no escalan bien entre servicios.

* El patrón **`Saga`** (ya sea coreografiado o orquestado) ofrece un modelo para lograr la consistencia de negocio a través de múltiples servicios mediante una secuencia de transacciones locales y compensaciones.
* El patrón **`Transactional Outbox`** es una técnica crucial para asegurar que los `events` (que a menudo impulsan las Sagas) se publiquen de manera fiable después de que un servicio haya completado su transacción local.

Estos patrones no son triviales de implementar, pero son herramientas esenciales para construir sistemas de `microservices` que sean resilientes, escalables y mantengan la consistencia de los datos a un nivel de negocio, incluso en ausencia de transacciones distribuidas tradicionales. La elección de usarlos debe basarse en una necesidad clara de consistencia a través de los límites del servicio.

## 13.6 Rollback coordinado con eventos

Tras explorar las Sagas y el patrón `Outbox` en el 13.5 como mecanismos para gestionar operaciones que abarcan múltiples `microservices`, es natural que nos preguntemos: ¿qué sucede cuando uno de los pasos de esta compleja coreografía distribuida falla? No podemos simplemente hacer un `ROLLBACK DATABASE` global como en una transacción ACID local.

Aquí es donde el concepto de **`rollback` coordinado con `events`** se vuelve esencial. Es la forma en que las Sagas manejan los fallos, deshaciendo el trabajo previo para mantener la consistencia del negocio a un nivel aceptable.

Imagina una `Saga` como una delicada hilera de fichas de dominó. Cada ficha que cae (una transacción local exitosa en un servicio) desencadena la siguiente. Pero, ¿qué pasa si una ficha a mitad de camino se niega a caer o cae de forma incorrecta (una transacción local falla)? No podemos simplemente ignorarlo; necesitamos una forma de "deshacer" el efecto de las fichas que ya cayeron para evitar un estado inconsistente en todo el sistema. Este "deshacer" en un sistema distribuido no es un `rollback` mágico, sino una serie de **transacciones compensatorias (`compensating transactions`)** orquestadas, a menudo, por `events`.

**El `Rollback` en un Mundo Distribuido: No es un Deshacer Mágico, es una Compensación Inteligente 🧠**

En el contexto de Sagas:

* Un **`rollback`** no significa que el estado de cada base de datos involucrada vuelve _exactamente_ al punto preciso antes de que la `Saga` comenzara (aunque ese es el ideal).
* Significa que se ejecutan acciones para **neutralizar o revertir el impacto de las transacciones locales que ya se completaron con éxito** dentro de la `Saga`.
* El objetivo es llevar el sistema global a un estado que sea **consistente desde la perspectiva del negocio**, incluso si eso significa que ciertos registros fueron creados y luego marcados como "cancelados" o "reembolsados".

**`Failure Events`: Las Señales de Alarma que Desencadenan la Retirada Estratégica 🚩**

Cuando una transacción local crucial dentro de una `Saga` falla (y los reintentos locales no resuelven el problema), el servicio responsable de esa transacción debe:

1. Deshacer sus propios cambios locales (si es posible dentro de su propia transacción atómica).
2. **Publicar un `failure event` específico.** Este `event` es la señal para el resto de los participantes de la `Saga` de que algo ha ido mal y que podrían necesitar tomar acciones compensatorias.

* **Ejemplos de `Failure Events`:**
  * `PaymentFailedEvent { orderId: "123", reason: "Insufficient funds" }`
  * `StockReservationFailedEvent { orderId: "123", productId: "P456", reason: "Not enough stock" }`
  * `CustomerVerificationFailedEvent { customerId: "C789", sagaId: "SAGA_XYZ" }`
* **Contenido del `Failure Event`:**
  * Un identificador de la `Saga` (`correlation_id`) para vincularlo con la operación de negocio original.
  * Información sobre el fallo (qué servicio falló, por qué).
  * Datos necesarios para que otros servicios puedan identificar qué compensar (ej. `order_id`, `customer_id`).

**Diseñando Transacciones Compensatorias: El Arte de Deshacer con Precisión Quirúrgica ⏪**

Una transacción compensatoria (`CTX` o `Compensating Transaction`) es una operación que revierte los efectos de una transacción local previa (`LTX` o `Local Transaction`) que fue parte de la `Saga`.

* **Idempotencia Absoluta:** Es **crítico** que las `compensating transactions` sean idempotentes. Un `failure event` podría ser procesado múltiples veces por un `subscriber` debido a reintentos en el `message bus`. La compensación debe poder ejecutarse varias veces sin causar efectos secundarios incorrectos después de la primera ejecución exitosa.
* **Fiabilidad Extrema:** Deben ser diseñadas para tener la mayor probabilidad de éxito posible. Un fallo en una transacción compensatoria es un problema serio que a menudo requiere intervención manual.
* **Lógica Inversa (Generalmente):**
  * Si LTX1 fue `DebitAccount(amount)` ➔ CTX1 podría ser `CreditAccount(amount)`.
  * Si LTX2 fue `ReserveInventory(productId, quantity)` ➔ CTX2 podría ser `ReleaseInventory(productId, quantity)`.
* **Datos para la Compensación:** El `failure event` o el estado actual del servicio deben contener la información necesaria para realizar la compensación (ej. el `order_id` para cancelar un pedido, la cantidad de `stock` que se reservó).
* **No Pueden Asumir que el `State` no ha Cambiado (Reto Avanzado):** Desde que la LTX original se ejecutó, otros procesos podrían haber interactuado con los mismos datos. La compensación debe ser lo suficientemente inteligente para manejar esto (ej. si se intenta liberar `stock` que ya fue consumido por otro pedido prioritario). A veces, esto puede implicar marcar algo como "compensación fallida, requiere revisión".

**Coordinación Basada en `Events` (Coreografía del `Rollback`): La Danza de la Reversión 💃🕺**

En una `Saga` coreografiada, los servicios se suscriben a los `failure events` de otros servicios para coordinar el `rollback`.

1. **Suscripción a `Failure Events`:** Los servicios que ejecutaron transacciones locales exitosas en los primeros pasos de la `Saga` escuchan los `failure events` de los servicios que actúan en pasos posteriores.
2. **Ejecución de la Compensación:** Al recibir un `failure event` relevante (identificado por el `Saga ID` / `correlation_id`):
   * El servicio verifica si su transacción original para esa `Saga` fue completada.
   * Si es así, ejecuta su transacción compensatoria local.
   * Es buena práctica que la transacción compensatoria también sea idempotente.
3. **(Opcional) Publicación de `Compensation Events`:** Después de ejecutar exitosamente una transacción compensatoria, un servicio puede publicar un `event` de confirmación (ej. `PaymentRefundedEvent`, `StockReleasedEvent`). Esto puede ayudar a otros servicios a rastrear el progreso del `rollback` o a desencadenar otras compensaciones en cascada si es necesario.

**Diagrama `Mermaid`: `Rollback` Coordinado en una `Saga` Coreografiada**

Imaginemos una `Saga` para procesar un pedido: 1. `Order Service` crea pedido. 2. `Payment Service` procesa pago. 3. `Inventory Service` reserva `stock`. Supongamos que la reserva de `stock` falla.

```mermaid
sequenceDiagram
    participant Client
    participant OS as Order Service
    participant PS as Payment Service
    participant IS as Inventory Service
    participant EvBus as Event Bus

    Client->>+OS: Place Order (SagaID: XYZ)
    OS->>OS: LTX1: Create Order (status: PENDING_PAYMENT)
    OS-->>EvBus: Event: OrderCreated (SagaID: XYZ)

    OS-->>-Client: ACK Place Order

    EvBus->>+PS: OrderCreated (SagaID: XYZ)
    PS->>PS: LTX2: Process Payment

    alt Payment Successful
        PS-->>EvBus: PaymentSuccessful (SagaID: XYZ)

        EvBus->>+IS: PaymentSuccessful (SagaID: XYZ)
        IS->>IS: LTX3: Reserve Inventory

        alt Inventory Reservation Fails
            IS-->>EvBus: InventoryReservationFailed (SagaID: XYZ)

            EvBus->>+PS: InventoryReservationFailed
            PS->>PS: CTX2: Refund Payment
            PS-->>EvBus: PaymentRefunded (SagaID: XYZ)
            PS-->>-EvBus: ACK Refund

            EvBus->>+OS: InventoryReservationFailed / PaymentRefunded
            OS->>OS: CTX1: Cancel Order (FAILED_INVENTORY)
            OS-->>EvBus: OrderFailed (SagaID: XYZ)
            OS-->>-Client: Order Failed (Out of Stock)
        else Inventory Reservation OK
            IS-->>EvBus: InventoryReserved (SagaID: XYZ)
            EvBus->>+OS: InventoryReserved
            OS->>OS: LTX4: Confirm Order (CONFIRMED)
            OS-->>EvBus: OrderConfirmed (SagaID: XYZ)
            OS-->>-Client: Order Confirmed
        end

    else Payment Failed
        PS-->>EvBus: PaymentFailed (SagaID: XYZ)
        EvBus->>+OS: PaymentFailed
        OS->>OS: CTX1: Cancel Order (FAILED_PAYMENT)
        OS-->>EvBus: OrderFailed (SagaID: XYZ)
        OS-->>-Client: Order Failed (Payment Declined)
    end
```

**Desafíos y Consideraciones Clave del `Rollback` Coordinado:**

* **Diseño de Compensaciones Robustas:** Esta es la parte más difícil. ¿Qué pasa si el `item` a "des-reservar" ya no está en el mismo estado? ¿Se pueden "des-enviar" emails? Las compensaciones deben ser diseñadas para el "peor caso esperado".
* **Fallos Durante la Compensación:** Un `nightmare scenario`. Si una transacción compensatoria falla, usualmente requiere intervención manual o alertas críticas. Por ello, deben ser lo más simples y fiables posible.
* **Latencia y `State` Intermedio:** El proceso de `rollback` completo a través de `events` puede llevar tiempo. Durante este periodo, diferentes partes del sistema estarán en estados inconsistentes desde una perspectiva global, hasta que la `Saga` se resuelva (ya sea con éxito o con una compensación completa).
* **Pruebas Exhaustivas:** Simular todos los posibles puntos de fallo y verificar que las compensaciones funcionen correctamente es un esfuerzo de prueba significativo pero indispensable.
* **`Distributed Tracing` y `Logging`:** Es vital tener un buen `tracing` (usando el `Saga ID` / `correlation_id`) y `logging` centralizado para poder diagnosticar qué falló y dónde está el proceso de compensación.

**Conclusión: La Resiliencia a Través de la Compensación Inteligente y Coordinada 🏗️**

El `rollback` coordinado por `events` es la manifestación del principio de "autocorrección" en arquitecturas de `microservices` que utilizan el patrón `Saga`. No nos da la atomicidad férrea de una transacción ACID distribuida (que evitamos por su complejidad y impacto en la disponibilidad), pero nos ofrece un camino pragmático hacia la **consistencia eventual del negocio** incluso frente a fallos.

Requiere un diseño cuidadoso, una atención meticulosa a la idempotencia y fiabilidad de las transacciones compensatorias, y una buena infraestructura de `eventing` y `observability`. Pero el resultado es un sistema que puede manejar fallos parciales con mayor gracia, deshaciendo operaciones de manera coordinada para evitar dejar el negocio en un estado corrupto o inconsistente a largo plazo. Es un pilar para construir `microservices` que no solo son autónomos, sino también colaborativamente resilientes.

## 13.7 Conexión a MongoDB con `motor`

Cambiamos de tercio en nuestra exploración de la persistencia. Si con SQLAlchemy (13.1) nos sumergimos en el mundo estructurado de las bases de datos relacionales, ahora con el **13.7** nos aventuramos en la flexibilidad y escalabilidad de las bases de datos NoSQL, específicamente **MongoDB**, utilizando **`Motor`**, su `driver` oficial asíncrono para Python.

Imagina que pasamos de construir con ladrillos perfectamente uniformes (SQL) a trabajar con arcilla increíblemente maleable (NoSQL con MongoDB). Ambos tienen su lugar y su poder, y `Motor` es la herramienta que nos permite modelar esa arcilla a alta velocidad en nuestras aplicaciones FastAPI.

Cuando tus necesidades de datos se inclinan hacia esquemas flexibles, desarrollo rápido, escalabilidad horizontal o el manejo de grandes volúmenes de datos no estructurados o semi-estructurados, **MongoDB** emerge como una opción NoSQL muy popular. Es una base de datos orientada a documentos, lo que significa que almacena datos en estructuras similares a `JSON` llamadas BSON (`Binary JSON`).

Para interactuar con MongoDB desde nuestro `backend` FastAPI asíncrono, el `driver` de elección es **`Motor`**.

**MongoDB: Una Breve Inmersión en el Mundo de los Documentos Flexibles 📄**

* **¿Qué es MongoDB?**
  * Una base de datos NoSQL, `document-oriented`.
  * Los datos se almacenan en **`collections`** (similares a las tablas SQL).
  * Cada `collection` contiene **`documents`** (similares a las filas SQL, pero son objetos BSON/JSON).
  * **`Schema-flexible`:** Los `documents` dentro de una misma `collection` no necesitan tener la misma estructura (aunque generalmente comparten una estructura similar por coherencia).
* **Casos de Uso Donde Brilla:**
  * Catálogos de productos con atributos variados.
  * Perfiles de usuario con campos opcionales y `nested data`.
  * `Content management systems (CMS)`.
  * `Caching` de datos.
  * `Logging` y analíticas en tiempo real.
  * Aplicaciones con requisitos de desarrollo ágil y esquemas evolutivos.
* **Ventajas:**
  * **Flexibilidad de Esquema:** Facilita la iteración rápida y la adaptación a cambios en los requisitos de datos.
  * **Escalabilidad Horizontal:** Diseñado para escalar mediante `sharding` y `replica sets`.
  * **Rendimiento para Ciertas Cargas de Trabajo:** Especialmente para grandes volúmenes de lecturas/escrituras de `documents` completos.
  * **Facilidad de Uso para Desarrolladores:** Trabajar con `JSON/BSON documents` se siente natural para muchos desarrolladores acostumbrados a `JavaScript` o Python `dicts`.

**`Motor`: Tu Conductor Asíncrono de Alto Rendimiento para MongoDB y FastAPI 🏍️**

* **`Motor`** es el `driver` oficial de MongoDB para Python que soporta `asyncio`. Esto lo hace el compañero perfecto para FastAPI.
* **Componentes Clave de `Motor`:**
  * `motor.motor_asyncio.AsyncIOMotorClient`: El punto de entrada para conectarse a una `instance` o `cluster` de MongoDB.
  * `AsyncIOMotorDatabase`: Representa una base de datos específica dentro de MongoDB. Se obtiene del `client`.
  * `AsyncIOMotorCollection`: Representa una `collection` específica dentro de una base de datos. Es el objeto con el que realizarás la mayoría de las operaciones CRUD.

**Estableciendo la Conexión: El Puente FastAPI-MongoDB 🌉**

1.  **Instalación:**

    ```bash
    pip install motor
    ```

    (Opcionalmente, `pip install "motor[snappy,gssapi,srv,tls]"` para `extra features`).
2.  **Creando el `AsyncIOMotorClient` y Obteniendo `Database`/`Collection`:**\
    Esto generalmente se hace en un módulo de configuración o de base de datos.

    ```python
    # app/db/mongo_db.py (o un nombre similar)
    from motor.motor_asyncio import AsyncIOMotorClient, AsyncIOMotorDatabase, AsyncIOMotorCollection
    from app.core.config import settings # Asumiendo que tienes una config para la URL

    # MONGO_CONNECTION_STRING = "mongodb://localhost:27017" # Ejemplo
    MONGO_CONNECTION_STRING = settings.MONGODB_URL # Desde tu config
    MONGO_DATABASE_NAME = settings.MONGO_DATABASE_NAME # Desde tu config

    class MongoDBConnection:
        client: AsyncIOMotorClient = None
        db: AsyncIOMotorDatabase = None

    db_connection = MongoDBConnection()

    async def connect_to_mongo():
        print(f"Attempting to connect to MongoDB at {MONGO_CONNECTION_STRING}...")
        db_connection.client = AsyncIOMotorClient(MONGO_CONNECTION_STRING)
        db_connection.db = db_connection.client[MONGO_DATABASE_NAME] # Seleccionar la base de datos
        print(f"Successfully connected to MongoDB, database: '{MONGO_DATABASE_NAME}'.")

    async def close_mongo_connection():
        if db_connection.client:
            db_connection.client.close()
            print("MongoDB connection closed.")

    # En tu main.py, llamarías a esto en los eventos startup/shutdown:
    # app = FastAPI()
    # @app.on_event("startup")
    # async def startup_event():
    #     await connect_to_mongo()
    # @app.on_event("shutdown")
    # async def shutdown_event():
    #     await close_mongo_connection()
    ```
3.  **`Dependency Injection` para `AsyncIOMotorDatabase` o `AsyncIOMotorCollection`:**\
    Para usar la base de datos o una `collection` específica en tus `endpoints`.

    ```python
    # app/api/dependencies.py
    # from app.db.mongo_db import db_connection # Importar la instancia de conexión global
    # from motor.motor_asyncio import AsyncIOMotorDatabase, AsyncIOMotorCollection

    async def get_mongo_database() -> AsyncIOMotorDatabase:
        if db_connection.db is None:
            # Esto no debería pasar si connect_to_mongo se llama en startup
            raise RuntimeError("MongoDB connection not established.")
        return db_connection.db

    # Dependencia para una colección específica (ej. 'items')
    async def get_items_collection(
        db: AsyncIOMotorDatabase = Depends(get_mongo_database)
    ) -> AsyncIOMotorCollection:
        return db["items_collection"] # El nombre de tu colección
    ```

**Operaciones CRUD con `Motor`: Hablando el Lenguaje de los Documentos (Asíncrono) ✍️🔍🔄🗑️**

Una vez que tienes un objeto `AsyncIOMotorCollection` (inyectado como `collection` en tu `endpoint`), puedes realizar operaciones CRUD.

*   **Tabla de Operaciones `Motor` Comunes (sobre una `collection`):**

    | Operación       | Método `Motor` (`await collection.METHOD_NAME(...)`)         | Descripción Breve                                                                                      | Devuelve (Típico)                                   |
    | --------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------- |
    | **Create**      | `insert_one(document: dict)`                                 | Inserta un único `document`.                                                                           | `InsertOneResult` (contiene `.inserted_id`)         |
    |                 | `insert_many(documents: List[dict])`                         | Inserta una lista de `documents`.                                                                      | `InsertManyResult` (contiene `.inserted_ids`)       |
    | **Read (One)**  | `find_one(filter: Optional[dict] = None, ...)`               | Encuentra el primer `document` que coincide con el `filter`.                                           | `dict` (el `document`) o `None` si no se encuentra. |
    | **Read (Many)** | `find(filter: Optional[dict] = None, ...)`                   | Devuelve un `AsyncIOMotorCursor` para iterar sobre los `documents` que coinciden.                      | `AsyncIOMotorCursor`                                |
    | **Update**      | `update_one(filter: dict, update: dict, upsert=False)`       | Actualiza el primer `document` que coincide. Usa operadores de `update` MongoDB (ej. `{"$set": ...}`). | `UpdateResult` (`.modified_count`, `.upserted_id`)  |
    |                 | `update_many(filter: dict, update: dict, upsert=False)`      | Actualiza todos los `documents` que coinciden.                                                         | `UpdateResult` (`.modified_count`, `.upserted_id`)  |
    |                 | `replace_one(filter: dict, replacement: dict, upsert=False)` | Reemplaza completamente el primer `document` que coincide.                                             | `UpdateResult` (`.modified_count`, `.upserted_id`)  |
    | **Delete**      | `delete_one(filter: dict)`                                   | Elimina el primer `document` que coincide.                                                             | `DeleteResult` (`.deleted_count`)                   |
    |                 | `delete_many(filter: dict)`                                  | Elimina todos los `documents` que coinciden.                                                           | `DeleteResult` (`.deleted_count`)                   |
* **Ejemplos de Código (dentro de un `endpoint` FastAPI con `collection: AsyncIOMotorCollection = Depends(get_items_collection)`):**
  *   **Crear un `Document`:**

      ```python
      # from pydantic import BaseModel
      # class ItemCreatePydantic(BaseModel): name: str; price: float
      # item_data_dict = item_create_dto.model_dump() # Convertir Pydantic model a dict
      # result = await collection.insert_one(item_data_dict)
      # inserted_id = result.inserted_id # Es un ObjectId
      # return {"id": str(inserted_id), **item_data_dict}
      ```
  *   **Leer un `Document` por `_id`:**\
      (Necesitarás `from bson import ObjectId` para convertir un `string ID` a `ObjectId` de MongoDB)

      ```python
      # from bson import ObjectId, errors as bson_errors
      # try:
      #     obj_id = ObjectId(item_id_str)
      # except bson_errors.InvalidId:
      #     raise HTTPException(status_code=400, detail="Invalid item ID format")
      # document = await collection.find_one({"_id": obj_id})
      # if document:
      #     document["_id"] = str(document["_id"]) # Convertir ObjectId a string para la respuesta
      #     return document
      # raise HTTPException(status_code=404, detail="Item not found")
      ```
  *   **Leer Múltiples `Documents` con Filtro:**

      ```python
      # filter_query = {"category": "electronics", "price": {"$lt": 500}} # Menor que 500
      # cursor = collection.find(filter_query).limit(10) # Aplicar limit
      # results = await cursor.to_list(length=None) # length=None para todos los resultados del cursor (cuidado con grandes datasets)
      # for doc in results: # Convertir ObjectId a string
      #     doc["_id"] = str(doc["_id"])
      # return results
      ```
  *   **Actualizar un `Document`:**

      ```python
      # from bson import ObjectId
      # # item_update_dto: Pydantic model con campos opcionales
      # update_data_dict = item_update_dto.model_dump(exclude_unset=True) # Solo campos provistos
      # if not update_data_dict: # No hay nada que actualizar
      #     raise HTTPException(status_code=400, detail="No update data provided")
      # result = await collection.update_one(
      #     {"_id": ObjectId(item_id_str)},
      #     {"$set": update_data_dict}
      # )
      # if result.matched_count == 0:
      #     raise HTTPException(status_code=404, detail="Item not found to update")
      # return {"message": f"Updated {result.modified_count} item(s)."}
      ```
  *   **Eliminar un `Document`:**

      ```python
      # from bson import ObjectId
      # result = await collection.delete_one({"_id": ObjectId(item_id_str)})
      # if result.deleted_count == 0:
      #     raise HTTPException(status_code=404, detail="Item not found to delete")
      # return Response(status_code=status.HTTP_204_NO_CONTENT)
      ```

**Pydantic y MongoDB: Un Matrimonio de Conveniencia (Validación y Mapeo) 💞**

Aunque MongoDB es `schema-flexible`, **sigue siendo crucial usar modelos Pydantic en tus `endpoints` FastAPI** para:

* Validar los `payloads` de `request`.
* Definir la estructura de los `response models`.
* Obtener documentación OpenAPI automática.
*   **Mapeando el `_id` de MongoDB:**\
    MongoDB usa `_id` (un tipo `bson.ObjectId`) como `primary key`. Al interactuar con Pydantic:

    * **Para `Responses`:** A menudo querrás exponer el `_id` como un `string` `id` en tu API.
    * **Para `Requests`:** Al crear `documents`, MongoDB genera el `_id`. Al actualizar/buscar, puedes recibir un `ID string` del `client` y necesitarás convertirlo a `ObjectId`.

    ```python
    from pydantic import BaseModel, Field
    from bson import ObjectId
    from typing import Optional, Any

    # Helper para manejar ObjectId en Pydantic, especialmente para validación y schema
    # Esta es una forma común de hacerlo, aunque Pydantic V2 tiene más opciones con Annotated_JsonSchemaValue
    class PyObjectId(ObjectId):
        @classmethod
        def __get_pydantic_core_schema__(cls, source_type: Any, handler: Any) -> Any:
            # Adaptado para Pydantic V2. Referencia: https://docs.pydantic.dev/latest/usage/types/custom/
            from pydantic_core import core_schema
            
            def validate(value: Any) -> ObjectId:
                if isinstance(value, ObjectId):
                    return value
                if ObjectId.is_valid(str(value)):
                    return ObjectId(str(value))
                raise ValueError("Invalid ObjectId")

            # Representa ObjectId como string en JSON schema, pero valida como ObjectId
            # Para la serialización, se necesitará un json_encoder en model_config
            return core_schema.json_or_python_schema(
                python_schema=core_schema.with_info_plain_validator_function(validate),
                json_schema=core_schema.str_schema(pattern=r'^[0-9a-fA-F]{24}$'), # Ejemplo de patrón regex
                serialization=core_schema.plain_serializer_function_ser_schema(lambda v: str(v))
            )

    class MongoProductBasePydantic(BaseModel):
        name: str = Field(..., min_length=3)
        description: Optional[str] = None
        price: float = Field(..., gt=0)

    class MongoProductCreatePydantic(MongoProductBasePydantic):
        pass

    class MongoProductResponsePydantic(MongoProductBasePydantic):
        # Pydantic V2: Usar Field con alias para mapear _id a id
        id: PyObjectId = Field(..., alias='_id') # _id es el campo en MongoDB

        model_config = {
            "from_attributes": True, # Para crear desde objetos con atributos (como dicts de Motor)
            "populate_by_name": True, # Permite que el alias '_id' funcione también en la inicialización
                                      # y para la serialización si el campo se llama 'id'.
            "json_encoders": {ObjectId: str} # Asegura que ObjectId se serialize a str en el JSON final
        }
    ```

    * Al insertar: `item_dict = pydantic_model.model_dump(by_alias=False)` (para no enviar `_id` si `id` es el campo Pydantic).
    * Al leer y convertir a Pydantic: `MongoProductResponsePydantic.model_validate(mongo_doc_dict)` (Pydantic V2) o `MongoProductResponsePydantic.parse_obj(mongo_doc_dict)` (Pydantic V1). La configuración `populate_by_name=True` y `from_attributes=True` ayuda mucho aquí si los `keys` del `dict` de MongoDB coinciden con los nombres de campo o sus `alias`.

**Diagrama `Mermaid`: Flujo de Integración MongoDB con `Motor` en FastAPI 🌊**

```mermaid
sequenceDiagram
    participant Client
    participant FastAPI_EP as FastAPI Endpoint <br> (ej. POST /mongo_products)
    participant DI_get_coll as Depends(get_items_collection)
    participant MotorColl as AsyncIOMotorCollection
    participant MongoDB_Srv as MongoDB Server

    Client->>+FastAPI_EP: HTTP Request (Payload JSON)
    Note over FastAPI_EP: Pydantic valida Payload a MongoProductCreatePydantic
    FastAPI_EP->>+DI_get_coll: Solicita `AsyncIOMotorCollection`
    DI_get_coll-->>-FastAPI_EP: Inyecta `collection`
    
    FastAPI_EP->>FastAPI_EP: `doc_to_insert = product_in.model_dump()`
    FastAPI_EP->>+MotorColl: `await collection.insert_one(doc_to_insert)`
    MotorColl->>+MongoDB_Srv: Operación BSON (INSERT)
    MongoDB_Srv-->>-MotorColl: Resultado (ej. `inserted_id`)
    MotorColl-->>-FastAPI_EP: `InsertOneResult`
    
    Note over FastAPI_EP: Construye respuesta (ej. con `inserted_id` y datos) <br/> Mapea a `MongoProductResponsePydantic`
    FastAPI_EP-->>-Client: HTTP Response (ej. 201 Created, JSON)
```

**Conclusión: Flexibilidad Asíncrona con MongoDB y `Motor` para FastAPI 🚀**

`Motor` es el `driver` esencial para conectar tus aplicaciones FastAPI asíncronas con el poder y la flexibilidad de MongoDB.

* Proporciona una API completamente `async` para todas las operaciones de base de datos, asegurando que tu `event loop` no se bloquee.
* La flexibilidad de esquema de MongoDB es una gran ventaja para ciertos tipos de aplicaciones, pero recuerda que **la validación rigurosa con Pydantic en los límites de tu API (DTOs) sigue siendo crucial** para la robustez y la claridad del contrato de tu API.
* El sistema de `Dependency Injection` de FastAPI te permite gestionar y proveer las `database connections` y `collections` de `Motor` a tus `endpoints` de manera limpia y eficiente.

Con esta integración, estás listo para construir `microservices` FastAPI que aprovechen las fortalezas de MongoDB para la persistencia de datos `document-oriented`.

## 13.8 Esquemas flexibles en MongoDB

Con nuestro `Motor` listo para la acción (13.7), es hora de sumergirnos en una de las características más distintivas y, a menudo, debatidas de MongoDB: su **esquema flexible**. Este es el corazón del **13.8**.

Si las bases de datos relacionales son como arquitectos que exigen planos detallados y aprobados antes de colocar un solo ladrillo, MongoDB a menudo se siente más como un escultor que puede moldear la arcilla sobre la marcha. Pero incluso la arcilla más flexible necesita una mano guía para tomar una forma útil y coherente.

Una de las principales razones por las que los desarrolladores eligen MongoDB es su **flexibilidad de esquema**. A diferencia de las bases de datos SQL tradicionales, donde debes definir una estructura de tabla rígida (`columns`, `data types`) antes de insertar datos, MongoDB permite que los `documents` dentro de una misma `collection` tengan estructuras diferentes.

**MongoDB y la Flexibilidad de Esquema: ¿Libertad Absoluta o Caos Controlado? 🤔**

* **No es "Schema-Less", es "Schema-Flexible" (o `Dynamically-Typed`):**\
  Es un error común referirse a MongoDB como "schema-less". Si bien no _impone_ un esquema a nivel de base de datos por defecto de la misma manera que SQL, casi siempre existe un **esquema implícito** definido y esperado por tu aplicación. Si tu aplicación espera un `field` "email" en los `documents` de la `collection` "users", ese es tu esquema de aplicación.\
  La "flexibilidad" radica en que:
  * No todos los `documents` en una `collection` necesitan tener exactamente los mismos `fields`.
  * Puedes añadir nuevos `fields` a nuevos `documents` sin tener que ejecutar un `ALTER TABLE` que modifique toda la `collection` (una operación que puede ser costosa y bloqueante en SQL).
* **El Poder de la Variación:**\
  Un `document` de producto podría tener `fields` para "tamaño\_pantalla" y "resolucion" si es un monitor, mientras que otro `document` de producto en la misma `collection` podría tener "material" y "talla" si es una camiseta.
* **Evolución sin Fricción (Aparente):**\
  Si decides que tus `products` ahora necesitan un `field` "tags", simplemente empiezas a incluirlo en los nuevos `documents` que insertas o actualizas. Los `documents` antiguos no se ven afectados y no se rompen.

**Ventajas Estratégicas de los Esquemas Flexibles 🌟**

Esta flexibilidad no es solo una característica técnica; ofrece beneficios tangibles:

1. **Agilidad en el Desarrollo (`Agile Development`):**
   * Permite iterar más rápidamente, especialmente en las primeras etapas de un proyecto cuando los requisitos de datos aún están evolucionando. Puedes empezar a almacenar datos sin tener un diseño de esquema perfectamente finalizado.
2. **Manejo de Datos Polimórficos y Diversos:**
   * Ideal para `product catalogs` donde diferentes tipos de productos tienen atributos completamente diferentes.
   * `User profiles` que pueden tener una variedad de `fields` opcionales o personalizados.
   * Datos de `IoT` donde diferentes sensores pueden enviar `payloads` con estructuras variables.
3. **Datos Escasos (`Sparse Data`):**
   * Si muchos de tus `fields` son opcionales y solo están presentes en un pequeño subconjunto de `documents`, no "malgastas" espacio (o la complejidad de `NULLs` en SQL) para esos `fields` en los `documents` donde no aplican.
4. **Integración de Datos de Múltiples Fuentes:**
   * Puede ser más fácil consolidar datos de diferentes fuentes que podrían tener ligeras variaciones en sus estructuras.

**Los Desafíos de la Libertad: Gestionando la Flexibilidad con Disciplina y Herramientas 🚧**

La gran flexibilidad de MongoDB también conlleva responsabilidades y desafíos:

1. **La Carga de la Validación se Desplaza (Principalmente) a la Aplicación:**
   * Dado que la base de datos no impone un esquema estricto por defecto, **tu aplicación se vuelve la principal responsable de validar la estructura y los tipos de datos** antes de escribirlos o al leerlos.
   * ¡Aquí es donde **Pydantic en FastAPI se convierte en tu mejor amigo y guardián!** Tus modelos Pydantic definen el esquema que tu API _espera_ y _produce_.
2. **Consistencia de Datos:**
   * Sin `constraints` a nivel de base de datos tan rígidos como en SQL (aunque MongoDB tiene algunas capacidades), asegurar la consistencia lógica de los datos (ej. que un `order_id` en una `collection` de `payments` realmente exista en la `collection` de `orders`) recae más en la lógica de tu aplicación.
3. **Complejidad en las `Queries`:**
   * Si los `documents` dentro de una `collection` varían drásticamente, tus `queries` deben ser lo suficientemente robustas para manejar la ausencia de `fields` (usando el operador `$exists`) o variaciones en los `data types` (usando `$type`).
   * Ejemplo: `await collection.find_one({"user_id": 123, "profile_picture_url": {"$exists": True}})`
4. **Estrategias de Indexación (`Indexing`):**
   * Los `indexes` siguen siendo absolutamente vitales para el rendimiento de las `queries` en MongoDB.
   * Un esquema muy flexible puede hacer que la estrategia de `indexing` sea más compleja si los `fields` sobre los que necesitas consultar varían mucho entre `documents` o cambian con frecuencia.
5. **`Schema Drift` y Documentación:**
   * El "esquema implícito" puede evolucionar orgánicamente con el tiempo (`schema drift`). Si no se gestiona, diferentes partes de la aplicación o diferentes miembros del equipo podrían tener expectativas distintas sobre la estructura de los `documents`.
   * Es crucial **documentar la estructura "canónica" o "esperada"** de tus `documents` para cada `collection`, incluso si MongoDB no la impone.

**MongoDB al Rescate: `Schema Validation` del Lado del Servidor 🛡️**

Reconociendo la necesidad de un mayor control, MongoDB (desde la versión 3.6) ofrece **`schema validation` a nivel de `collection`**. Esto te permite definir reglas (usando la sintaxis de `JSON Schema`) que los `documents` deben cumplir para ser insertados o actualizados.

* **¿Cómo Funciona?**
  * Defines un `validator` (un objeto `JSON Schema`) al crear una `collection` o modificar una existente (`collMod` command).
  * Puedes especificar:
    * `required`: Lista de `fields` obligatorios.
    * `properties`: Definiciones para `fields` específicos, incluyendo `bsonType`, `pattern` (para `strings`), `minimum`/`maximum` (para números), `minItems`/`maxItems` (para `arrays`), etc.
    * `additionalProperties: false` para prohibir `fields` no definidos en el `schema`.
  * Puedes configurar el `validationLevel`:
    * `off`: Sin validación.
    * `moderate`: Aplica validación solo a `inserts` y a `updates` de `documents` que ya cumplen el `schema`. No afecta a `updates` de `documents` que ya eran inválidos.
    * `strict`: Aplica validación a todos los `inserts` y `updates`.
  * Y el `validationAction`:
    * `warn`: Loguea una advertencia si la validación falla, pero permite la operación.
    * `error` (default): Rechaza la operación si la validación falla.
*   **Ejemplo Conceptual de `JSON Schema` para una `collection` "users" en MongoDB:**

    ```javascript
    // Comando para aplicar en mongo shell
    // db.runCommand({
    //   collMod: "users", // Nombre de la colección
    //   validator: {
    //     $jsonSchema: {
    //       bsonType: "object",
    //       title: "User Object Validation",
    //       required: ["email", "username", "hashed_password", "created_at"],
    //       properties: {
    //         email: {
    //           bsonType: "string",
    //           pattern: "^.+@.+\\..+$", // Simple regex para email
    //           description: "must be a string and match the regular expression pattern"
    //         },
    //         username: {
    //           bsonType: "string",
    //           minLength: 3,
    //           description: "must be a string of at least 3 chars and is required"
    //         },
    //         hashed_password: {
    //           bsonType: "string",
    //           description: "is required"
    //         },
    //         full_name: {
    //           bsonType: "string",
    //           description: "optional full name"
    //         },
    //         is_active: {
    //           bsonType: "bool",
    //           description: "defaults to true if not provided by application"
    //         },
    //         created_at: {
    //           bsonType: "date",
    //           description: "is required"
    //         },
    //         preferences: {
    //           bsonType: ["object", "null"], // Puede ser un objeto o null
    //           properties: {
    //             receive_newsletter: { bsonType: "bool" },
    //             theme: { enum: ["light", "dark", "system"] }
    //           }
    //         }
    //       }
    //     }
    //   },
    //   validationAction: "warn" // O "error" para producción
    // })
    ```
* **El Equilibrio Perfecto:** La `schema validation` de MongoDB te ofrece un punto intermedio: puedes seguir beneficiándote de la flexibilidad para ciertos `fields` o `sub-documents` mientras impones reglas estrictas para las partes críticas de tu `schema`.

**Pydantic en FastAPI: Tu Contrato y Guardián del Esquema a Nivel de Aplicación 👮**

Incluso si usas la `schema validation` de MongoDB, tus modelos Pydantic en FastAPI siguen siendo **esenciales**:

* **Contrato de API:** Definen la estructura de datos que tu API _expone_ y _espera_, independientemente de las ligeras variaciones que puedan existir en la base de datos a lo largo del tiempo.
* **Validación en el Límite:** Pydantic valida los datos en el momento en que entran a tu aplicación (en `requests`) y antes de que salgan (para `responses` si usas `response_model`). Esto protege tu lógica de negocio de datos inesperados.
* **Manejo de `Optional Fields` y `Defaults`:** `Optional[SomeType]` en Pydantic y `default` values se alinean bien con la naturaleza de los `fields` que pueden o no estar presentes en los `documents` de MongoDB.
* **Manejo de `Extra Fields`:**
  * En Pydantic V2, la configuración por defecto es `model_config = {"extra": "ignore"}`, lo que significa que si MongoDB devuelve `fields` que no están en tu modelo Pydantic, Pydantic los ignorará al parsear.
  * Puedes cambiarlo a `extra = "allow"` (para incluirlos como atributos) o `extra = "forbid"` (para lanzar un error si hay `fields` inesperados).
* **`model_dump(exclude_none=True, by_alias=True)`:** Es muy útil al preparar datos para insertar o actualizar en MongoDB, ya que omite los `fields` que son `None` (evitando almacenarlos explícitamente si no es necesario) y usa `aliases` (como `_id` para `id`).

**Diagrama `Mermaid`: Pydantic y MongoDB – Colaboración entre Flexibilidad y Contrato**

```mermaid
    graph TD

        subgraph API_Layer_FastAPI
            HTTP_Req[HTTP Request - JSON Payload] --> PYD_Req_Val[Pydantic Request Model - Entrada]
            PYD_Req_Val --> AppLogic[Lógica de Aplicación / Servicios]
            AppLogic --> Motor_Driver[Motor Driver]

            Motor_Driver_Read[Motor Driver - Lectura] --> AppLogic_Read[Lógica Aplicación - Lectura]
            AppLogic_Read --> PYD_Resp_Val[Pydantic Response Model - Salida]
            PYD_Resp_Val --> HTTP_Resp[HTTP Response - JSON Payload]
        end

        subgraph Persistencia_MongoDB
            Motor_Driver --> MongoDB_Coll[(MongoDB Collection)]
            MongoDB_Coll --> MongoDB_Coll
            MongoDB_Coll --> Motor_Driver_Read
        end

        style PYD_Req_Val fill:#D5F5E3,stroke:#2ECC71
        style PYD_Resp_Val fill:#D5F5E3,stroke:#2ECC71
        style MongoDB_Coll fill:#A9DFBF,stroke:#27AE60
        style AppLogic fill:#AED6F1,stroke:#3498DB
        style AppLogic_Read fill:#AED6F1,stroke:#3498DB

```

El diagrama ilustra cómo Pydantic actúa como el "guardián del `schema`" en la capa de API, mientras que MongoDB ofrece flexibilidad en la capa de almacenamiento, con una opción de validación a nivel de base de datos.

**Conclusión: La Flexibilidad como Herramienta Estratégica, No Como Excusa para el Desorden 🦉**

El esquema flexible de MongoDB es una de sus características más potentes, ofreciendo una agilidad de desarrollo sin igual para ciertos tipos de aplicaciones y datos. Sin embargo, esta libertad conlleva la responsabilidad de gestionar la estructura y la calidad de los datos a nivel de aplicación.

* **FastAPI con Pydantic** te proporciona las herramientas perfectas para definir, validar y hacer cumplir un "esquema de aplicación" robusto en los límites de tu API, trayendo orden y previsibilidad al mundo flexible de MongoDB.
* Considera usar la **`schema validation` nativa de MongoDB** como una red de seguridad adicional para imponer `constraints` críticos directamente en la base de datos, logrando un equilibrio óptimo entre flexibilidad y control.

Al entender y gestionar conscientemente la flexibilidad de esquema, puedes aprovechar al máximo MongoDB sin caer en el caos de datos inconsistentes.

## 13.9 Bases de datos por servicio y separación

¡Continuamos con el **13.9**! Este punto nos lleva directamente al corazón de la arquitectura de `microservices` y cómo se relaciona con la persistencia: el principio de **"una base de datos por servicio"** y la importancia de la **separación de datos**.

Si en las secciones anteriores exploramos _cómo_ conectarnos a diferentes tipos de bases de datos (SQL con SQLAlchemy, NoSQL con MongoDB), ahora nos preguntaremos _dónde_ deben residir esos datos en un ecosistema de `microservices`.

Uno de los principios fundamentales y más transformadores en el diseño de arquitecturas de `microservices` es la idea de que **cada `microservice` debe poseer y gestionar su propia base de datos, y esta no debe ser compartida directamente por otros servicios.** Este concepto es crucial para lograr la verdadera autonomía, el desacoplamiento y la escalabilidad independiente que prometen los `microservices`.

**El Anti-Patrón: La Base de Datos Monolítica Compartida monolithic**

En arquitecturas monolíticas tradicionales, es común tener una única base de datos centralizada a la que acceden todos los módulos de la aplicación. Intentar llevar este modelo directamente a una arquitectura de `microservices` (donde múltiples servicios independientes leen y escriben en la misma base de datos compartida) es un `anti-pattern` que conduce a múltiples problemas:

* **Acoplamiento Fuerte (`Tight Coupling`):**
  * Si varios servicios comparten el mismo `schema` de base de datos, un cambio en ese `schema` requerido por un servicio (ej. añadir una tabla, modificar una columna) puede romper otros servicios.
  * La coordinación de migraciones de `schema` se vuelve una pesadilla.
* **Escalabilidad Limitada:** La base de datos compartida se convierte en un cuello de botella. Escalar la base de datos para satisfacer las necesidades de todos los servicios es complejo y costoso. Diferentes servicios pueden tener patrones de acceso a datos muy distintos (ej. uno intensivo en lecturas, otro en escrituras) que son difíciles de optimizar en un único `datastore`.
* **Autonomía Reducida del Equipo:** Los equipos no pueden evolucionar el `schema` de persistencia de su servicio de forma independiente. Deben coordinarse con otros equipos, ralentizando el desarrollo.
* **Elección Tecnológica Restringida:** Todos los servicios se ven forzados a usar la misma tecnología de base de datos, incluso si un servicio particular se beneficiaría enormemente de un tipo diferente de `datastore` (ej. un servicio de catálogo de productos podría preferir MongoDB, mientras que un servicio de transacciones financieras podría necesitar PostgreSQL con sus garantías ACID).
* **Resiliencia Disminuida:** Si la base de datos compartida falla, todos los servicios que dependen de ella fallan.

**El Principio Guía: Una Base de Datos por `Microservice` (Database per Service) 🛡️**

El patrón "Database per Service" aboga por que cada `microservice` sea el único propietario de sus datos y de su `schema` de base de datos.

* **¿Qué Significa "Poseer"?**
  * El `microservice` es el único que puede leer y escribir directamente en su base de datos.
  * Otros servicios que necesiten acceder a esos datos deben hacerlo **a través de la API expuesta por el `microservice` propietario**. No deben hacer `queries` directas a la base de datos de otro servicio.
* **Diagrama `Mermaid`: `Database per Service` vs. `Shared Database`**

**Beneficios de la Separación de Datos y `Database per Service`:**

1. **Autonomía del Servicio y del Equipo (`Service and Team Autonomy`):**
   * Cada equipo puede elegir la tecnología de base de datos que mejor se adapte a las necesidades de su `microservice` (SQL, NoSQL, `graph database`, `time-series database`, etc.).
   * Pueden evolucionar el `schema` de su base de datos de forma independiente sin afectar a otros servicios.
   * Ciclos de desarrollo y despliegue más rápidos.
2. **Desacoplamiento Fuerte (`Loose Coupling`):**
   * Los servicios están desacoplados a nivel de persistencia. La única forma de interactuar con los datos de otro servicio es a través de su API bien definida. Esto refuerza los límites del servicio.
3. **Escalabilidad Mejorada (`Improved Scalability`):**
   * Cada base de datos puede escalarse independientemente según las necesidades específicas del `microservice` al que sirve.
   * Puedes optimizar el rendimiento de cada base de datos para los patrones de acceso de su servicio.
4. **Resiliencia Aumentada (`Increased Resilience`):**
   * Un fallo en la base de datos de un `microservice` no afecta directamente a otros `microservices` (a menos que dependan de su API, en cuyo caso se aplican patrones como `Circuit Breakers`).
5. **Seguridad de Datos Más Granular (`Granular Data Security`):**
   * Es más fácil aplicar políticas de seguridad específicas a cada base de datos según la sensibilidad de los datos que contiene.

**Desafíos y Consideraciones de `Database per Service`:**

Si bien los beneficios son significativos, este patrón también introduce sus propios desafíos:

1. **Consistencia de Datos Entre Servicios (`Data Consistency Across Services`):**
   * Este es el mayor desafío. Si una operación de negocio requiere actualizar datos en múltiples `microservices` (cada uno con su propia base de datos), ¿cómo aseguras la consistencia?
   * **Solución:** Patrones como **Sagas** (13.5), `eventual consistency` (10.7), y `event-driven architecture` son necesarios. No puedes tener transacciones ACID distribuidas fácilmente.
2. **`Queries` que Abarcan Múltiples Servicios (`Queries Spanning Multiple Services`):**
   * Si necesitas obtener datos que residen en las bases de datos de varios `microservices` para satisfacer una única petición del `client`, no puedes simplemente hacer un `JOIN` SQL.
   * **Soluciones:**
     * **Composición de APIs (`API Composition`):** Un servicio "agregador" o el propio `API Gateway` realiza múltiples llamadas a las APIs de los `microservices` individuales y combina los resultados.
     * **`Command Query Responsibility Segregation (CQRS)`:** Puedes crear vistas materializadas (`materialized views`) o `read models` específicos para `queries` que combinan datos de múltiples servicios. Estos `read models` se actualizan de forma asíncrona a través de `events`.
     * **Replicación de Datos (con cuidado):** A veces, un servicio puede mantener una copia local (y de solo lectura) de datos que necesita frecuentemente de otro servicio, actualizada mediante `events`.
3. **Gestión de Múltiples Bases de Datos:**
   * Mayor `overhead` operativo: necesitas desplegar, monitorizar, hacer `backup` y gestionar múltiples bases de datos.
   * Requiere más experiencia en DevOps y gestión de infraestructura.
4. **Complejidad Transaccional para Operaciones de Negocio Completas:**
   * Como se mencionó, las Sagas son más complejas de implementar que las transacciones ACID locales.

**Estrategias de Implementación y Variaciones:**

* **`Private Tables` dentro de un `Schema` Compartido (Paso Intermedio):**\
  A veces, como un paso intermedio o en organizaciones donde la gestión de muchas bases de datos es un obstáculo, los equipos pueden acordar usar `schemas` separados o un conjunto de tablas estrictamente privadas para cada servicio dentro de una misma `instance` de base de datos física. Esto ofrece cierto aislamiento lógico, pero aún comparte la infraestructura física y los riesgos de rendimiento. No es "Database per Service" puro, pero puede ser un compromiso.
* **Elección de la Base de Datos Correcta para el Trabajo (`Polyglot Persistence`):**\
  Una vez que cada servicio tiene su propia base de datos, puedes elegir la tecnología de base de datos que mejor se adapte a sus necesidades:
  * `Orders Service`: Podría usar PostgreSQL por sus transacciones ACID robustas.
  * `Product Catalog Service`: Podría usar MongoDB o Elasticsearch por su flexibilidad de `schema` y capacidades de búsqueda.
  * `User Session Service`: Podría usar Redis por su velocidad.
  * `Analytics Service`: Podría usar una `columnar database` o `data warehouse`.

**Diagrama `Mermaid`: Integración de Datos en `Microservices`**

```mermaid
graph TD
    User_Client[User Client] --> APIGW[API Gateway]

    APIGW -->|/orders| OS[Orders Service API]
    APIGW -->|/products| PS[Product Catalog API]
    APIGW -->|/users| US[Users Service API]

    subgraph "Orders Microservice"
        OS --> DB_OS[(PostgreSQL OrdersDB)]
    end
    subgraph "Product Catalog Microservice"
        PS --> DB_PS[(MongoDB ProductDB)]
    end
    subgraph "Users Microservice"
        US --> DB_US[(MySQL UsersDB)]
    end

    OS -. Event (OrderCreated) .-> EventBus[(Event Bus <br> Kafka/RabbitMQ)]
    EventBus -.-> PS_Consumer(Product Catalog Consumer)
    EventBus -.-> US_Consumer(Users Service Consumer)
    
    PS_Consumer -. Actualiza Read Model .-> DB_PS
    US_Consumer -. Actualiza Read Model .-> DB_US

    style APIGW fill:#D5F5E3,stroke:#2ECC71
    style EventBus fill:#FAD7A0,stroke:#F39C12
```

Este diagrama ilustra cómo cada `microservice` tiene su propia base de datos y cómo pueden comunicarse (e indirectamente compartir datos para `read models`) a través de `events` y un `event bus`, o mediante llamadas API directas (no mostradas para la sincronización de datos aquí, pero sí para la interacción inicial).

**Conclusión: Autonomía de Datos como Pilar del Diseño de `Microservices` ⚖️**

El principio de "una base de datos por servicio" es fundamental para lograr los beneficios de desacoplamiento, autonomía y escalabilidad que se buscan con una arquitectura de `microservices`. Si bien introduce desafíos, particularmente en torno a la consistencia de datos y las `queries` distribuidas, estos se abordan con patrones como Sagas, CQRS y `event-driven architecture`.

* **No compartas tablas directamente entre servicios.** Obliga a la comunicación a través de APIs bien definidas.
* **Elige la tecnología de base de datos adecuada para cada servicio.** Adopta la "persistencia políglota" (`polyglot persistence`).
* **Prepárate para gestionar la consistencia eventual.**

Para tus profesionales que desarrollan `microservices` con FastAPI, entender y aplicar este principio de separación de datos es tan importante como diseñar los `endpoints` de la API o la lógica de negocio interna de cada servicio. Es la base sobre la cual se construyen sistemas distribuidos verdaderamente robustos y evolutivos.

### 13.10 Pools de conexión y timeouts

El **13.10** se centra en dos aspectos operativos absolutamente cruciales para la salud, el rendimiento y la resiliencia de cualquier aplicación que interactúe con una base de datos: los **`connection pools` (pools de conexiones)** y la gestión de **`timeouts` (tiempos de espera)**.

Si tus `database connections` son las autopistas hacia tus datos, los `connection pools` son los carriles inteligentemente gestionados para evitar atascos, y los `timeouts` son las señales de emergencia y los desvíos para cuando el tráfico se detiene inesperadamente.

Conectar a una base de datos es una operación costosa en términos de recursos y tiempo. Establecer una nueva `connection` para cada `query` o `request` es ineficiente y no escala. Del mismo modo, esperar indefinidamente a que una base de datos responda puede dejar tu aplicación colgada y no responsiva. Aquí es donde los `connection pools` y una gestión inteligente de `timeouts` se vuelven indispensables.

**1. `Connection Pooling`: El Arte de Reutilizar Conexiones Eficientemente 🚗🅿️**

* **El Coste Oculto de una Nueva `Database Connection`:**\
  Cada vez que tu aplicación necesita hablar con la base de datos y establece una nueva `connection`, ocurren varias cosas:
  1. `TCP Handshake`: Negociación de red para establecer la `socket connection`.
  2. `Database Authentication`: El `server` de base deatos verifica las credenciales.
  3. `Resource Allocation`: El `server` de base de datos asigna memoria y procesos para manejar la nueva `connection`.\
     Repetir esto para cada `query` o `user request` es un enorme cuello de botella.
* **¿Qué es un `Connection Pool`?**\
  Un `connection pool` es, esencialmente, una **caché de `database connections`** mantenida por tu `driver` de base de datos o tu ORM (como SQLAlchemy o `Motor`). En lugar de crear una `connection` desde cero cada vez:
  1. La aplicación solicita una `connection` del `pool`.
  2. Si hay una `connection` disponible (inactiva) en el `pool`, se reutiliza.
  3. Si no hay ninguna disponible y el `pool` no ha alcanzado su tamaño máximo, se puede crear una nueva y añadirla al `pool`.
  4. Cuando la aplicación termina de usar la `connection`, en lugar de cerrarla completamente, la **devuelve al `pool`** para que otros puedan usarla.
* **Beneficios Incuestionables:**
  * **Rendimiento Drásticamente Mejorado:** Se reduce significativamente la latencia asociada con el establecimiento de `connections`, ya que se reutilizan las existentes.
  * **Gestión Eficiente de Recursos:** Se controla el número máximo de `connections` que tu aplicación abre a la base de datos, previniendo que se agoten los recursos en el `database server` (que tiene un límite finito de `connections` que puede manejar).
  * **Mayor `Throughput` de la Aplicación:** Al reducir la latencia por `request`, la aplicación puede manejar más `requests` por segundo.
*   **`Connection Pooling` en SQLAlchemy Asíncrono (con `create_async_engine`):**\
    SQLAlchemy gestiona el `pooling` automáticamente cuando creas un `AsyncEngine`. Puedes configurarlo con varios parámetros:

    | Parámetro (`create_async_engine`) | Descripción                                                                                                                                                                                                                                | Ejemplo de Valor |
    | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------- |
    | `pool_size`                       | Número de `connections` que el `pool` intentará mantener abiertas y disponibles.                                                                                                                                                           | `10`             |
    | `max_overflow`                    | Número de `connections` adicionales que se pueden abrir temporalmente más allá de `pool_size` si hay alta demanda.                                                                                                                         | `5`              |
    | `pool_timeout`                    | Segundos a esperar por una `connection` del `pool` si todas (`pool_size` + `max_overflow`) están en uso.                                                                                                                                   | `30`             |
    | `pool_recycle`                    | Segundos después de los cuales una `connection` se cierra y se reabre. Previene `stale connections` debido a `firewalls` o `timeouts` de red/BBDD.                                                                                         | `1800` (30 min)  |
    | `pool_pre_ping`                   | Si es `True`, SQLAlchemy enviará un "ping" simple (una `query` ligera como `SELECT 1`) a la BBDD antes de entregar una `connection` del `pool` para asegurar que está viva. Impacta ligeramente el rendimiento pero mejora la resiliencia. | `True`           |

    ```python
    # from sqlalchemy.ext.asyncio import create_async_engine
    # SQLALCHEMY_DATABASE_URL = "postgresql+asyncpg://user:pass@host/db"
    # async_engine = create_async_engine(
    #     SQLALCHEMY_DATABASE_URL,
    #     pool_size=10,
    #     max_overflow=20,
    #     pool_recycle=3600, # Reciclar conexiones cada hora
    #     pool_pre_ping=True
    # )
    ```
*   **`Connection Pooling` en `Motor` (MongoDB Driver):**`AsyncIOMotorClient` gestiona un `connection pool` internamente por defecto. Puedes influir en su comportamiento a través de parámetros en el `connection string` o en el constructor de `AsyncIOMotorClient`.

    | Parámetro (`AsyncIOMotorClient` o `Connection String`) | Descripción                                                                                                                            | Ejemplo de Valor (`string` o `int`) |
    | ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
    | `maxPoolSize`                                          | Máximo número de `connections` en el `pool` por cada `server` en el `cluster` MongoDB. Default: `100`.                                 | `maxPoolSize=50`                    |
    | `minPoolSize`                                          | Mínimo número de `connections` a mantener por `server`. Default: `0`. Requiere `heartbeatFrequencyMS` para que se mantengan.           | `minPoolSize=5`                     |
    | `waitQueueTimeoutMS`                                   | Milisegundos que una operación esperará por una `connection` si el `pool` ha alcanzado `maxPoolSize`. Default `0` (espera indefinida). | `waitQueueTimeoutMS=10000` (10s)    |
    | `connectTimeoutMS`                                     | Milisegundos para establecer una nueva `connection` con un `server` MongoDB. Default `20000` (20s).                                    | `connectTimeoutMS=5000` (5s)        |
    | `heartbeatFrequencyMS`                                 | Con qué frecuencia el `driver` revisa el estado de los `servers` MongoDB (ms). Default `10000` (10s).                                  | `heartbeatFrequencyMS=30000` (30s)  |

    ```python
    # from motor.motor_asyncio import AsyncIOMotorClient
    # MONGO_URL = "mongodb://localhost:27017/?maxPoolSize=50&waitQueueTimeoutMS=5000"
    # client = AsyncIOMotorClient(MONGO_URL)
    # # O al crear el cliente:
    # # client = AsyncIOMotorClient("mongodb://localhost:27017", maxPoolSize=50, waitQueueTimeoutMS=5000)
    ```
* **Diagrama `Mermaid`: El Flujo de un `Connection Pool`**

**2. `Timeouts`: Los Guardianes Contra la Espera Infinita ⏰**

Los `timeouts` son mecanismos de seguridad esenciales para prevenir que tu aplicación se quede colgada indefinidamente esperando una respuesta de la base de datos, lo cual podría ocurrir debido a problemas de red, una base de datos sobrecargada, o una `query` muy ineficiente.

*   **Tipos de `Timeouts` y su Configuración:**

    | Tipo de `Timeout`              | SQLAlchemy (Ejemplo Configuración)                                                                                                                                                                       | `Motor` (MongoDB) (Ejemplo Configuración)                                                              | Propósito                                                                             |
    | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------- |
    | **`Connection Timeout`**       | (Driver-specific) `create_async_engine(..., connect_args={'timeout': 5})` para `asyncpg`.                                                                                                                | `connectTimeoutMS=5000` en `conn_str` o `Client`.                                                      | Tiempo máximo para establecer una _nueva_ `connection` con la BBDD.                   |
    | **`Statement/Query Timeout`**  | (`Query-level`) `stmt.execution_options(timeout=10)` o (`Session-level`) `session.execute(stmt, execution_options={"timeout": 10})`. También `statement_timeout` en `connect_args` para algunos drivers. | `socketTimeoutMS=10000` (inactividad en `socket`), `maxTimeMS` en la `query` para el `server` MongoDB. | Tiempo máximo para que una `query` o `statement` individual se complete.              |
    | **`Pool Wait Timeout`**        | `create_async_engine(..., pool_timeout=30)`                                                                                                                                                              | `waitQueueTimeoutMS=30000`                                                                             | Tiempo máximo a esperar por una `connection` del `pool` si está lleno.                |
    | **`Server Selection Timeout`** | (No aplica directamente a SQLAlchemy Core/ORM `pool`)                                                                                                                                                    | `serverSelectionTimeoutMS=30000`                                                                       | (MongoDB `replica sets/sharded clusters`) Tiempo para encontrar un `server` adecuado. |
*   **Manejando Excepciones de `Timeout` en el Código:**\
    Cuando un `timeout` ocurre, la librería (`SQLAlchemy` o `Motor`) o `asyncio` lanzará una excepción.

    * SQLAlchemy: `sqlalchemy.exc.TimeoutError` (a menudo envolviendo una excepción del `driver` o `asyncio.TimeoutError`).
    * `Motor`: `pymongo.errors.ConnectionFailure` (puede envolver `socket.timeout`), `pymongo.errors.ServerSelectionTimeoutError`.
    * `asyncio`: `asyncio.TimeoutError` si usas `asyncio.wait_for()` explícitamente.

    Tu código debe estar preparado para capturar estas excepciones:

    ```python
    # try:
    #     # ... operación de base de datos ...
    #     results = await db.execute(select(MyModel).execution_options(timeout=5)) # Timeout por query
    # except sqlalchemy.exc.TimeoutError:
    #     # Manejar el timeout de la query (log, reintento, error HTTP 503/504)
    #     raise HTTPException(status_code=504, detail="Database query timed out")
    # except asyncio.TimeoutError: # Si se usa asyncio.wait_for()
    #     raise HTTPException(status_code=504, detail="Operation timed out")
    # except Exception as e: # Otros errores de BBDD
    #     # ...
    ```

    * La estrategia de manejo puede incluir reintentos (con `exponential backoff`), o devolver un error HTTP apropiado al `client` (ej. `503 Service Unavailable` o `504 Gateway Timeout`).

**3. Buenas Prácticas para `Pools` y `Timeouts` 🎯**

* **Dimensionar el `Pool` Correctamente (`Sizing`):**
  * Esto es más un arte que una ciencia exacta y depende de:
    * El número de `workers` de tu aplicación FastAPI (ej. `workers` Uvicorn). Cada `worker` tiene su propio `event loop` y podría tener su propio `engine` y `pool` si se inicializan por `worker`.
    * La naturaleza de tus `requests` (cuántas `queries` hacen, cuánto duran).
    * La capacidad máxima de `connections` de tu `database server`.
  * Un `pool` demasiado pequeño se convierte en un cuello de botella. Uno demasiado grande desperdicia recursos en la aplicación y puede sobrecargar la base de datos.
  * **Fórmula Común (Punto de Partida):** `((core_count * 2) + effective_spindle_count)` para `pool_size` en BBDD tradicionales. Pero para `async` y `microservices`, a menudo es más empírico y se ajusta con pruebas de carga. Comienza con un valor conservador (ej. 5-10 por `instance`/`worker` de app) y monitoriza.
* **`pool_recycle` / `Keepalives`:**
  * ¡Esencial! Las `network devices` (firewalls, `load balancers`) o el propio `database server` pueden cerrar `connections` TCP que consideran inactivas después de un cierto tiempo. `pool_recycle` (SQLAlchemy) o `heartbeatFrequencyMS` (`Motor`, que impulsa el `keepalive` de `sockets`) aseguran que las `connections` se refresquen o se mantengan activas.
* **`Timeouts` Realistas pero Firmes:**
  * No configures `timeouts` tan agresivamente cortos que operaciones válidas pero ligeramente lentas fallen constantemente.
  * No los dejes tan largos (o infinitos) que tu aplicación pueda colgarse indefinidamente.
  * Basa los `timeouts` en el rendimiento esperado de tus `queries` y la latencia de tu red.
* **Monitorización Continua:**
  * Monitoriza el uso de tu `connection pool` (ej. `connections` activas, en espera, tamaño del `pool`). Muchas librerías de APM pueden ayudar.
  * Vigila las tasas de error y `timeout` de la base de datos en tus `logs` y sistemas de `metrics`.
* **Pruebas Bajo Carga (`Load Testing`):**\
  La mejor forma de validar y afinar tus configuraciones de `pool` y `timeout` es sometiendo tu aplicación a pruebas de carga realistas y de estrés.

**Conclusión del Tema 13: Construyendo una Persistencia Resiliente, Performante y Profesional 🚀**

Hemos recorrido un largo camino en este Tema 13, desde la integración de SQLAlchemy ORM y `Motor` con FastAPI, pasando por el desacoplamiento de modelos, el patrón Repositorio, la gestión de transacciones locales, y los complejos desafíos de las transacciones distribuidas con Sagas y el patrón `Outbox`.

Este último punto sobre **`connection pools` y `timeouts`** cierra el círculo, enfocándose en los aspectos operativos que hacen que toda esa lógica de persistencia funcione de manera eficiente y fiable en el mundo real. Una configuración cuidadosa de estos parámetros no es glamurosa, pero es la diferencia entre una aplicación que se arrastra o se cae bajo carga y una que es rápida, estable y escalable.

Con estas herramientas y conocimientos, estás equipado para diseñar e implementar la capa de persistencia de tus `microservices` FastAPI de una manera que sea no solo funcionalmente correcta, sino también performante, resiliente y profesional. ¡La base de datos es el cimiento de muchas aplicaciones, y ahora sabes cómo construir ese cimiento con solidez!

***

¡Excelente idea! Después de absorber tanta información técnica y práctica, tener una guía de referencias clave es fundamental para que tus profesionales puedan profundizar, consultar y seguir aprendiendo.

Aquí te presento una **Bibliografía Compacta y Estratégica**, seleccionada para complementar los temas que hemos cubierto (especialmente los más recientes sobre persistencia, diseño de APIs, WebSockets y mensajería), manteniendo un enfoque profesional y práctico.

***

## Bibliografía

Esta selección no pretende ser exhaustiva, sino ofrecer puntos de partida de alta calidad para la autoformación continua y la consulta experta.

**I. SQLAlchemy (Persistencia Relacional Asíncrona)**

1. **SQLAlchemy Official Documentation (especialmente la sección 2.0 y AsyncIO):**
   * _Enlace:_ [www.sqlalchemy.org/docs/](https://www.sqlalchemy.org/docs/) (Navegar a la "SQLAlchemy 2.0 ORM" y "Asynchronous IO (aio)" sections).
   * _Por qué:_ La referencia definitiva. La documentación de SQLAlchemy 2.0 es excelente para entender el nuevo estilo de `querying` y el soporte `async`. Relevante para el Tema 13.1, 13.3, 13.4, 13.10.
2. **Libro: "Essential SQLAlchemy, 2nd Edition"** (por Jason Myers y Rick Copeland, aunque puede que no cubra el `async` más reciente en profundidad, los conceptos ORM son sólidos).
   * _Por qué:_ Aunque la documentación es clave, un libro bien estructurado puede ayudar a asentar conceptos ORM fundamentales. Verificar la cobertura de `async` o complementar con la documentación.

**II. MongoDB y `Motor` (Persistencia NoSQL Asíncrona)**

1. **MongoDB Official Documentation:**
   * _Enlace:_ [www.mongodb.com/docs/](https://www.mongodb.com/docs/)
   * _Por qué:_ Para entender los conceptos de MongoDB, su `query language`, `aggregation framework`, `indexing`, y `schema design` (flexible). Relevante para el Tema 13.7, 13.8, 13.10.
2. **`Motor` Official Documentation:**
   * _Enlace:_ [motor.readthedocs.io](https://motor.readthedocs.io/)
   * _Por qué:_ La guía esencial para el `driver` asíncrono oficial de MongoDB para Python. Cubre la API completa, ejemplos y mejores prácticas. Relevante para 13.7, 13.10.

**III. Diseño de APIs y Principios RESTful**

1. **Libro: "REST API Design Rulebook"** (por Mark Masse)
   * _Por qué:_ Ofrece un conjunto de reglas y directrices concisas para el diseño de APIs RESTful. Muy práctico. Relevante para el Tema 11.1, 11.2, 11.6.
2. **Microsoft REST API Guidelines:**
   * _Enlace:_ [github.com/microsoft/api-guidelines](https://github.com/microsoft/api-guidelines)
   * _Por qué:_ Un conjunto muy completo y bien razonado de directrices para el diseño de APIs REST, mantenido por un equipo con mucha experiencia. Cubre `versioning`, `collections`, errores, etc.
3. **OpenAPI Specification Documentation:**
   * _Enlace:_ [spec.openapis.org/oas/v3.1.0.html](https://spec.openapis.org/oas/v3.1.0.html) (o la última versión)
   * _Por qué:_ Para entender el estándar que FastAPI usa para generar la documentación. Relevante para 11.4.

**IV. `Microservices`, Patrones de Datos y Mensajería**

1. **Libro: "Building Microservices"** (por Sam Newman)
   * _Enlace:_ O'Reilly
   * _Por qué:_ Un clásico que cubre los principios fundamentales, desafíos y patrones para construir arquitecturas de `microservices`. Relevante para el contexto general y temas como "Database per Service" (13.9), Sagas (13.5).
2. **Libro: "Designing Data-Intensive Applications"** (por Martin Kleppmann)
   * _Enlace:_ O'Reilly
   * _Por qué:_ Una obra maestra que profundiza en los fundamentos de los sistemas de datos, incluyendo transacciones, consistencia, replicación, `event sourcing`, y `stream processing`. Esencial para entender los `trade-offs` en persistencia y sistemas distribuidos. Relevante para 10.7, 13.4, 13.5, 13.6.
3. **Microservices.io (Patrones de Diseño):**
   * _Enlace:_ [microservices.io/patterns/](https://microservices.io/patterns/) (especialmente las secciones de `data management` y `communication`).
   * _Por qué:_ Una excelente colección online de patrones de diseño para `microservices`, incluyendo `Database per Service`, Sagas, `Transactional Outbox`, `API Composition`, CQRS, `Event Sourcing`.
4. **Kafka Official Documentation:**
   * _Enlace:_ [kafka.apache.org/documentation/](https://kafka.apache.org/documentation/)
   * _Por qué:_ Para profundizar en Kafka, sus conceptos (`topics`, `partitions`, `brokers`, `consumer groups`), y su ecosistema. Relevante para el Tema 9 y partes del 10.
5. **RabbitMQ Official Documentation:**
   * _Enlace:_ [www.rabbitmq.com/documentation.html](https://www.rabbitmq.com/documentation.html)
   * _Por qué:_ Para aprender sobre AMQP, `exchanges`, `queues`, `bindings`, y las `features` avanzadas de RabbitMQ. Relevante para el Tema 9 y partes del 10.

**V. `WebSockets` y Comunicación en Tiempo Real**

1. **MDN Web Docs - WebSockets:**
   * _Enlace:_ [developer.mozilla.org/en-US/docs/Web/API/WebSockets\_API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
   * _Por qué:_ Una referencia excelente y accesible para la API `WebSocket` del lado del `client` (JavaScript) y los conceptos del protocolo. Relevante para el Tema 10 y 11.
2. **RFC 6455 - The WebSocket Protocol:**
   * _Enlace:_ [tools.ietf.org/html/rfc6455](https://tools.ietf.org/html/rfc6455)
   * _Por qué:_ El estándar oficial. Para una comprensión profunda del protocolo subyacente (más técnico).

**VI. Seguridad de APIs**

1. **OWASP API Security Top 10:**
   * _Enlace:_ [owasp.org/API-Security/](https://owasp.org/API-Security/) (buscar la lista más reciente)
   * _Por qué:_ Para estar al tanto de las vulnerabilidades más comunes en APIs y cómo mitigarlas. Relevante para 11.8, 11.7, 10.9.
2. **JWT.io (Introduction to JSON Web Tokens):**
   * _Enlace:_ [jwt.io/introduction/](https://jwt.io/introduction/)
   * _Por qué:_ Una buena introducción a los `JWTs`, su estructura y casos de uso. Relevante para 11.8.
3. **OAuth 2.0 Official Website:**
   * _Enlace:_ [oauth.net/2/](https://oauth.net/2/)
   * _Por qué:_ Para entender el `framework` OAuth2 en profundidad. Relevante para 11.8.

***
