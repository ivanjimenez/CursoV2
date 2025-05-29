# Tema 6. ARQUITECTURA HEXAGONAL Y APLICACIÓN DE DDD

- [Tema 6. ARQUITECTURA HEXAGONAL Y APLICACIÓN DE DDD](#tema-6-arquitectura-hexagonal-y-aplicación-de-ddd)
  
  - [Contenidos](#tabla-de-contenidos)
  - [6.1 Comprender el patrón de puertos y adaptadores](#61-comprender-el-patrón-de-puertos-y-adaptadores)
  - [6.2 Identificar las capas: dominio, aplicación, infraestructura, interfaces](#62-identificar-las-capas-dominio-aplicación-infraestructura-interfaces)
  - [6.3 Diseñar interfaces para cada puerto (entrada y salida)](#63-diseñar-interfaces-para-cada-puerto-entrada-y-salida)
  - [6.4 Implementar adaptadores HTTP como controladores REST o WebSocket](#64-implementar-adaptadores-http-como-controladores-rest-o-websocket)
  - [6.5 Separar repositorios del dominio usando interfaces](#65-separar-repositorios-del-dominio-usando-interfaces)
  - [6.6 Diseñar pruebas para el núcleo sin depender de infraestructuras](#66-diseñar-pruebas-para-el-núcleo-sin-depender-de-infraestructuras)
  - [6.7 Integrar eventos de dominio desde la capa interna](#67-integrar-eventos-de-dominio-desde-la-capa-interna)
  - [6.8 Implementar casos de uso en la capa de aplicación](#68-implementar-casos-de-uso-en-la-capa-de-aplicación)   
  - [6.9 Configurar inyecciones de dependencia de adaptadores externos](#69-configurar-inyecciones-de-dependencia-de-adaptadores-externos)
  - [6.10 Ejemplo de microservicio hexagonal completo con FastAPI](#610-ejemplo-de-microservicio-hexagonal-completo-con-fastapi)



---


**Objetivo del Tema:** Aprender a diseñar y construir aplicaciones FastAPI siguiendo los principios de la Arquitectura Hexagonal y DDD, logrando un código desacoplado, enfocado en el dominio del negocio y fácilmente adaptable a cambios tecnológicos o de requisitos.

**Tecnologías Base:** Python 3.12, FastAPI, Docker, Docker Compose, MariaDB.

---

## 6.1 Comprender el patrón de puertos y adaptadores


¡Bienvenido/a al núcleo de la Arquitectura Hexagonal! Este patrón, también conocido como "Puertos y Adaptadores" (Ports and Adapters), fue concebido por Alistair Cockburn y su objetivo principal es **proteger el corazón de tu aplicación (el dominio) de las dependencias externas y la tecnología.**

Imagina tu aplicación como una ciudadela medieval. El tesoro más valioso, el **Dominio** (donde reside la lógica de negocio pura), está en el centro, bien protegido. No queremos que las preocupaciones sobre cómo se muestran los datos (interfaz de usuario), cómo se almacenan (base de datos) o cómo se comunican con otros sistemas (servicios externos) contaminen o compliquen este núcleo.

### ¿Cómo lo logramos?

Aquí es donde entran en juego los "Puertos" y los "Adaptadores".

### 1. El Hexágono (La Aplicación):

|!["figure"](img/image.png )|


Piensa en tu aplicación no como una estructura en capas tradicional (presentación, lógica, datos), sino como un hexágono (la forma es solo una metáfora para indicar que hay múltiples puntos de entrada y salida, no necesariamente seis).

* **Interior del Hexágono (Dominio y Lógica de Aplicación):** Aquí reside la lógica de negocio y las reglas que son independientes de cualquier tecnología o interfaz externa. Es el "qué" hace tu aplicación.
* **Exterior del Hexágono:** Todo lo que interactúa con tu aplicación desde el exterior: interfaces de usuario, bases de datos, sistemas de mensajería, APIs de terceros, pruebas automatizadas, etc.

### Gráfico 1: El Hexágono Conceptual


```mermaid
graph TD
    subgraph Exterior
        A[Infraestructura]
    end

    subgraph Interior
        C[Dominio y Lógica de Aplicación]
    end

    A --> B[Puertos]
    B --> C


```
### 2. Puertos (Ports):

Los puertos son la **especificación** de cómo el exterior puede interactuar con el interior de la aplicación, o cómo la aplicación puede interactuar con el exterior. Son como los enchufes en una pared: definen una interfaz, pero no la implementación.

* **Puertos de Entrada (Driving/Input Ports):** Definen cómo los actores externos (usuarios, otros sistemas) pueden *invocar* la lógica de la aplicación. Piensa en ellos como las APIs del núcleo de tu aplicación. Generalmente, se definen mediante interfaces que los casos de uso de la capa de aplicación implementarán.
    * *Ejemplo:* Una interfaz `ServicioDePedidos` con un método `crearPedido(datosDelPedido)`.

* **Puertos de Salida (Driven/Output Ports):** Definen cómo la aplicación se comunica con los servicios externos que necesita para realizar su trabajo (por ejemplo, persistencia de datos, envío de notificaciones, obtención de información de otros servicios). Son interfaces que la aplicación *espera* que el mundo exterior implemente.
    * *Ejemplo:* Una interfaz `RepositorioDePedidos` con métodos como `guardar(pedido)` o `buscarPorId(idPedido)`.

### Gráfico 2: Puertos de Entrada y Salida


```mermaid
graph TD
    UI[Adaptador_UI]
    Entrada[Puerto_Entrada]
    Logica[Logica_Aplicacion]
    Salida[Puerto_Salida]
    Persistencia[Adaptador_Persistencia]

    UI -->|Llama a| Entrada
    Entrada --> Logica
    Logica --> Salida
    Salida -->|Implementado por| Persistencia

```

### 3. Adaptadores (Adapters):

Los adaptadores son la **implementación** concreta de los puertos. Son los puentes que conectan el mundo exterior con los puertos de la aplicación. Traducen las señales del mundo exterior al lenguaje que entiende el puerto y viceversa.

* **Adaptadores de Entrada (Driving Adapters):** Toman la entrada de un actor externo y la dirigen hacia un puerto de entrada.
    * *Ejemplo:* Un controlador REST de FastAPI que recibe una petición HTTP, extrae los datos y llama al método `crearPedido` del `ServicioDePedidos`. Otros ejemplos podrían ser un manejador de eventos de WebSocket, un cliente de línea de comandos (CLI) o incluso pruebas de aceptación.

* **Adaptadores de Salida (Driven Adapters):** Implementan los puertos de salida para interactuar con herramientas o servicios externos específicos.
    * *Ejemplo:* Una clase `RepositorioDePedidosPostgreSQL` que implementa la interfaz `RepositorioDePedidos` y utiliza SQLAlchemy para guardar y recuperar pedidos de una base de datos PostgreSQL. Otro adaptador podría ser `RepositorioDePedidosEnMemoria` para pruebas, o un `ServicioDeNotificacionesEmail` que implemente una interfaz `PuertoDeNotificaciones`.

### ¿Por qué este patrón es tan poderoso?

1.  **Aislamiento del Dominio:** La lógica de negocio permanece pura y no se contamina con detalles tecnológicos. Esto facilita la comprensión y la evolución del dominio.
2.  **Testeabilidad:**
    * El núcleo de la aplicación (dominio y lógica de aplicación) se puede probar de forma aislada, sin necesidad de levantar bases de datos, servidores web o servicios externos. Puedes usar "mocks" o adaptadores falsos para los puertos de salida.
    * Los adaptadores también se pueden probar de forma independiente.
3.  **Flexibilidad Tecnológica:** Puedes cambiar de base de datos, de framework web, o de proveedor de servicios de mensajería sin (idealmente) tocar el núcleo de tu aplicación. Solo necesitas escribir un nuevo adaptador.
    * *Ejemplo:* Si empiezas con MongoDB y luego decides migrar a PostgreSQL, solo cambias el adaptador de persistencia. La lógica de tu aplicación no se entera.
4.  **Mantenibilidad:** El código está mejor organizado, con responsabilidades claras. Es más fácil entender dónde realizar cambios y cuál será su impacto.
5.  **Desarrollo Paralelo:** Diferentes equipos pueden trabajar en diferentes adaptadores (por ejemplo, un equipo en la interfaz web y otro en la integración con un servicio de pagos) una vez que los puertos están definidos.
6.  **Aplazamiento de Decisiones Técnicas:** Puedes empezar a desarrollar la lógica de negocio sin haber decidido aún qué base de datos o framework de mensajería usarás. Puedes empezar con adaptadores en memoria para pruebas y desarrollo temprano.

### Relación con DDD (Domain-Driven Design):

La Arquitectura Hexagonal es un excelente habilitador para DDD.

* El **Dominio** de DDD reside en el corazón del hexágono.
* Los **Servicios de Aplicación** (Application Services) de DDD a menudo implementan los puertos de entrada, orquestando los objetos de dominio para cumplir con los casos de uso.
* Los **Repositorios** de DDD son un ejemplo clásico de puertos de salida, definiendo cómo se persiste y recupera la información del dominio, siendo los adaptadores las implementaciones concretas para una base de datos específica.
* Permite proteger el Modelo de Dominio de las complejidades de la infraestructura.

### Interacción Dinámica (¡Pensemos Juntos!):

* **Pregunta para ti:** Imagina que estás construyendo un sistema de gestión de biblioteca con FastAPI. ¿Cuáles podrían ser algunos puertos de entrada y salida? ¿Y ejemplos de adaptadores para ellos?
    * *Pista (Entrada):* ¿Cómo un usuario podría añadir un nuevo libro? ¿O buscar libros?
    * *Pista (Salida):* ¿Dónde se guardarán los datos de los libros? ¿Cómo se podría notificar a un usuario si un libro que reservó está disponible?

* **Escenario:** Decides que tu API FastAPI es un adaptador de entrada. El usuario hace una petición `POST /libros` para crear un nuevo libro.
    1.  ¿Qué puerto de entrada invocaría este adaptador?
    2.  Dentro de la lógica de aplicación que implementa ese puerto, supongamos que necesitas guardar el libro. ¿Qué puerto de salida se utilizaría?
    3.  ¿Qué adaptador concreto podría implementar ese puerto de salida si usas PostgreSQL?

### Gráfico 3: Flujo de una Petición en Arquitectura Hexagonal

# Diagrama ASCII

```
+----------+     +-----------------+     +----------------------+     +-------------------+     +---------------------+
|  Usuario | --> | Adaptador HTTP  | --> | Puerto de Entrada    | --> | Lógica de         | --> | Puerto de Salida    |
| (Navegador|     | (FastAPI        |     | (Interfaz Caso de Uso|     | Aplicación        |     | (Interfaz          |
| o Cliente)|     | Controller)     |     | CrearLibro)          |     | (ServicioLibros)  |     | RepositorioLibros)  |
+----------+     +-----------------+     +----------------------+     +-------------------+     +----------+----------+
                                                                                                           |
                                                                                                           | (Implementado por)
                                                                                                           v
                                                                                                +---------------------+
                                                                                                | Adaptador de        |
                                                                                                | Persistencia        |
                                                                                                | (SQLAlchemy +       |
                                                                                                |  PostgreSQL)        |
                                                                                                +---------------------+
```

### Puntos Clave a Recordar:

* **Dependencias hacia adentro:** Las dependencias siempre apuntan hacia el interior del hexágono. El dominio no sabe nada sobre los adaptadores. Los adaptadores conocen los puertos (interfaces) del dominio/aplicación. Esto se logra a menudo con el **Principio de Inversión de Dependencias (DIP)** de SOLID.
* **Abstracciones (Puertos):** Los puertos son la clave. Definen contratos, no implementaciones.
* **Concreciones (Adaptadores):** Los adaptadores son los que se "ensucian las manos" con la tecnología específica.

---

## 6.2 Identificar las capas: dominio, aplicación, infraestructura, interfaces

Aunque la Arquitectura Hexagonal no prescribe capas estrictas como la arquitectura en capas tradicional, conceptualmente podemos identificar estas áreas:

![alt text](image.png)

1.  **Dominio (El Corazón del Hexágono):**
    *   Contiene la lógica de negocio, las reglas y el estado.
    *   **Entidades:** Objetos con identidad que persisten a lo largo del tiempo (ej. `Usuario`, `Producto`).
    *   **Objetos de Valor (Value Objects):** Objetos inmutables sin identidad conceptual, definidos por sus atributos (ej. `Direccion`, `Moneda`).
    *   **Agregados:** Clúster de entidades y objetos de valor tratados como una unidad (ej. `Pedido` con sus `LineasDePedido`).
    *   **Servicios de Dominio:** Lógica de negocio que no encaja naturalmente en una entidad u objeto de valor.
    *   **Eventos de Dominio:** Representan algo significativo que ocurrió en el dominio (ej. `PedidoCreado`).
    *   **Interfaces de Repositorio:** Definen cómo se persisten y recuperan los agregados (puertos de salida).
    *   *Dependencias: Ninguna hacia el exterior.*

2.  **Aplicación (Orquestación de Casos de Uso):**
    *   Contiene los casos de uso de la aplicación (servicios de aplicación).
    *   Orquesta la lógica del dominio para cumplir con las solicitudes.
    *   Utiliza las interfaces de repositorio para interactuar con la persistencia.
    *   Puede despachar eventos de dominio.
    *   Define los DTOs (Data Transfer Objects) para la comunicación con las interfaces de usuario, si es necesario.
    *   *Dependencias: Hacia el Dominio.*

3.  **Interfaces/Adaptadores (Interacción con el Mundo Exterior):**
    *   **Adaptadores de Entrada (Driving Adapters):**
        *   Controladores REST (FastAPI).
        *   Controladores WebSocket.
        *   Comandos CLI.
        *   Consumidores de colas de mensajes.
        *   *Llaman a los Servicios de Aplicación.*
    *   **Adaptadores de Salida (Driven Adapters):**
        *   Implementaciones de Repositorios (ej. para MariaDB con SQLAlchemy).
        *   Clientes para servicios externos (HTTP, gRPC).
        *   Productores de colas de mensajes.
        *   Implementaciones de servicios de notificación (email, SMS).
        *   *Implementan interfaces definidas en el Dominio o la Aplicación.*

4.  **Infraestructura (Detalles Técnicos):**
    *   Configuración de la base de datos, logging, frameworks, etc.
    *   Contiene la implementación concreta de los adaptadores de salida.
    *   Mecanismos de Inyección de Dependencias.
    *   *Aquí residen las implementaciones concretas de los adaptadores.*

**Regla de Dependencia:** Las dependencias siempre apuntan hacia el interior (hacia el Dominio). El Dominio no conoce las otras capas.

---

## 6.3 Diseñar interfaces para cada puerto (entrada y salida)

Una vez que hemos comprendido el concepto de Puertos y Adaptadores (Sección 6.1) e idealmente identificado las capas principales de nuestra aplicación (Sección 6.2), el siguiente paso crucial es **diseñar las interfaces para nuestros puertos**. Estas interfaces son los contratos formales que definen cómo el núcleo de la aplicación interactúa con el mundo exterior y viceversa.

### ¿Por qué son tan importantes las interfaces para los puertos?

Las interfaces son fundamentales en la Arquitectura Hexagonal por varias razones:

  * **Definen Contratos Claros:** Especifican qué métodos están disponibles, qué parámetros esperan y qué resultados devuelven. Esto elimina la ambigüedad y establece expectativas claras para cualquier implementación.
  * **Habilitan el Principio de Inversión de Dependencias (DIP):** El núcleo de la aplicación (que contiene los puertos) no depende de los detalles de la infraestructura (adaptadores), sino de estas abstracciones (interfaces). Son los adaptadores los que dependen de las interfaces definidas por el núcleo. Esto invierte la dirección tradicional de las dependencias hacia las capas de infraestructura.
  * **Facilitan la Testeabilidad:** Podemos crear fácilmente "mocks" o dobles de prueba que implementen estas interfaces para probar el núcleo de la aplicación de forma aislada, sin necesidad de infraestructura real (bases de datos, servicios externos, etc.).
  * **Permiten la Intercambiabilidad de Adaptadores:** Si la interfaz está bien definida, podemos cambiar una implementación de un adaptador (por ejemplo, cambiar de una base de datos MySQL a PostgreSQL, o de un servicio de envío de emails a otro) sin modificar el núcleo de la aplicación. Solo necesitamos un nuevo adaptador que cumpla con el contrato de la interfaz.
  * **Desacoplamiento:** Son la clave para lograr el desacoplamiento entre la lógica de negocio y las preocupaciones tecnológicas (frameworks, bibliotecas específicas, etc.).

En Python, estas interfaces se definen comúnmente usando Clases Base Abstractas (`ABC` del módulo `abc`), lo que permite definir métodos abstractos que las clases concretas (adaptadores) deberán implementar.

### A. Diseñando Interfaces para Puertos de Entrada (Driving Ports / Input Ports)

Los puertos de entrada definen cómo los actores externos (como controladores HTTP de FastAPI, consumidores de mensajes, scripts CLI, o incluso pruebas de aceptación) pueden invocar la lógica de la aplicación. Generalmente, estos puertos son implementados por los **Casos de Uso** o **Servicios de Aplicación** (Application Services en terminología DDD).

**Consideraciones al diseñar interfaces de puertos de entrada:**

1.  **Orientados a Casos de Uso:** Cada interfaz de puerto de entrada suele corresponder a un caso de uso específico o a un conjunto cohesionado de operaciones que un actor puede realizar con la aplicación. El nombre del puerto o de la interfaz a menudo refleja este caso de uso.

      * *Ejemplo:* `IGestionInventarioInputPort` (o `GestionInventarioUseCase`), `IProcesamientoPedidosInputPort`.

2.  **Nombres de Métodos Claros e Intencionales:** Los métodos dentro de la interfaz deben reflejar la acción específica que el actor quiere realizar, utilizando el Lenguaje Ubicuo del dominio.

      * *Ejemplo:* `registrar_nuevo_producto()`, `actualizar_stock_producto()`, `crear_pedido_cliente()`.

3.  **Data Transfer Objects (DTOs) o Comandos para la Entrada:** Para pasar datos al puerto, es una buena práctica usar objetos simples de transferencia de datos (DTOs) o "Comandos". Estos son objetos inmutables (o casi) que encapsulan los datos necesarios para ejecutar la operación. Esto desacopla el puerto de los detalles específicos del adaptador de entrada (ej. no pasar directamente el objeto `Request` de FastAPI o un diccionario genérico). Pydantic es excelente para definir estos DTOs en el contexto de FastAPI.

      * *Ejemplo de DTO de entrada con Pydantic:*
  
```python
class ProductoCreadoDTO(BaseModel):
            id_producto: UUID
            nombre: str
            precio_con_iva: float # Ejemplo de lógica que podría aplicar el caso de uso
            mensaje_confirmacion: str = "Producto registrado exitosamente."
```

**Ejemplo de Interfaz de Puerto de Entrada (Python):**

Imaginemos un caso de uso para registrar un nuevo producto en nuestro sistema.

```python
from abc import ABC, abstractmethod
from uuid import UUID
from typing import Optional # Usado más adelante

# Suponiendo que DatosNuevoProductoDTO y ProductoCreadoDTO están definidos como arriba
# (o importados desde el módulo correspondiente)

class IGestionInventarioInputPort(ABC): # "I" es una convención común para Interfaces
    """
    Puerto de entrada para gestionar el inventario.
    Estos métodos serían implementados por un Servicio de Aplicación (Caso de Uso).
    """

    @abstractmethod
    async def registrar_nuevo_producto(self, datos_producto: DatosNuevoProductoDTO) -> ProductoCreadoDTO:
        """
        Registra un nuevo producto en el sistema según los datos proporcionados.
        Devuelve información del producto creado, incluyendo su ID asignado.
        Puede lanzar excepciones específicas del dominio o aplicación en caso de error de validación de negocio.
        """
        pass

    @abstractmethod
    async def actualizar_stock(self, id_producto: UUID, cantidad_ajuste: int) -> None:
        """
        Actualiza el stock de un producto existente.
        'cantidad_ajuste' puede ser positivo (añadir) o negativo (reducir).
        No devuelve nada si la operación es exitosa.
        Puede lanzar excepciones si el producto no existe o si las reglas de negocio no se cumplen (ej. stock negativo no permitido).
        """
        pass

    # Podríamos tener un DTO específico para ProductoDetalleDTO
    class ProductoDetalleDTO(BaseModel): # Ejemplo anidado o definido externamente
        id_producto: UUID
        nombre: str
        descripcion: Optional[str]
        precio: float
        stock_actual: int

    @abstractmethod
    async def obtener_detalles_producto(self, id_producto: UUID) -> Optional[ProductoDetalleDTO]:
        """
        Obtiene los detalles de un producto específico por su ID.
        Devuelve un DTO con los detalles del producto o None si no se encuentra.
        """
        pass
```

*Nota: Los DTOs (`DatosNuevoProductoDTO`, `ProductoCreadoDTO`, `ProductoDetalleDTO`) son parte integral del "contrato" del puerto y deben estar bien definidos. Se recomienda definirlos en un módulo accesible tanto por la capa de aplicación como por los adaptadores de entrada.*

### B. Diseñando Interfaces para Puertos de Salida (Driven Ports / Output Ports)

Los puertos de salida definen cómo la aplicación interactúa con herramientas y servicios externos que *ella necesita* para funcionar. La aplicación define la interfaz (el "qué necesita"), y la infraestructura proporciona la implementación concreta (el "cómo se obtiene/hace"). Ejemplos comunes incluyen repositorios de persistencia, servicios de mensajería, gateways de pago, APIs de terceros, etc.

**Consideraciones al diseñar interfaces de puertos de salida:**

1.  **Perspectiva de la Necesidad de la Aplicación (Application's Need):** La interfaz se define según lo que la aplicación *necesita* de la herramienta externa, no según todas las capacidades que la herramienta podría ofrecer. La aplicación "manda" y dicta el contrato.

      * *Ejemplo:* Si la aplicación solo necesita guardar y buscar pedidos por ID, la interfaz `IRepositorioPedidos` solo tendrá esos métodos (`guardar(pedido: Pedido)` y `buscar_por_id(id_pedido: UUID) -> Optional[Pedido]`), incluso si la base de datos subyacente (implementada por el adaptador) puede realizar búsquedas por otros criterios o realizar operaciones más complejas.

2.  **Abstracción de la Tecnología:** Los nombres de los puertos y sus métodos deben ser agnósticos a la tecnología específica que los implementará. El objetivo es abstraer los detalles de la infraestructura.

      * *Incorrecto (acoplado a la tecnología):* `GuardarPedidoEnMongoDB()`, `ConsultarUsuarioEnTablaSQL()`.
      * *Correcto (abstracto):* `guardar_pedido(pedido: Pedido)`, `buscar_usuario_por_email(email: str)`. El adaptador luego se encargará de la lógica específica de MongoDB o SQL.

3.  **Uso de Objetos del Dominio:** Generalmente, los puertos de salida (especialmente los Repositorios en DDD) trabajan directamente con los objetos del dominio (Entidades, Agregados, Objetos de Valor). Esto mantiene el Lenguaje Ubicuo del dominio consistente a través de la capa de aplicación y facilita que la lógica de negocio opere con sus propios artefactos.

      * *Ejemplo:* Un método `guardar_pedido(pedido: Pedido)` donde `Pedido` es una entidad del dominio, rica en comportamiento y reglas.

4.  **Granularidad Adecuada (Principio de Segregación de Interfaces - ISP de SOLID):** Es preferible tener interfaces más pequeñas y específicas para las necesidades de un cliente (en este caso, un caso de uso de la aplicación) que una interfaz grande y genérica. Si un caso de uso solo necesita leer datos de productos, no debería depender de una interfaz que también incluye métodos para escribir o eliminar productos si no los usa.

      * *Ejemplo:* Podrías tener `IProductoQueryRepository` (solo métodos de lectura) y `IProductoCommandRepository` (métodos de escritura), aunque a menudo se agrupan en una única interfaz de repositorio si la cohesión es alta y los casos de uso suelen necesitar ambas capacidades para una entidad dada.

**Ejemplo de Interfaz de Puerto de Salida (Python):**

Imaginemos un puerto de salida para persistir y recuperar entidades `Producto` (que sería una entidad del dominio).

```python
from abc import ABC, abstractmethod
from typing import Optional, List # List es usado más abajo
from uuid import UUID

# Suponemos que existe una entidad de dominio llamada "Producto"
# Esta entidad viviría en la capa de Dominio.
# class Producto:
#     id: UUID
#     nombre: str
#     descripcion: Optional[str]
#     precio: float
#     stock: int
#     # ... más lógica de dominio, reglas de negocio y métodos
#     def cambiar_precio(self, nuevo_precio: float) -> None:
#         if nuevo_precio <= 0:
#             raise ValueError("El precio debe ser positivo.")
#         self.precio = nuevo_precio
#     def ajustar_stock(self, cantidad: int) -> None:
#         if self.stock + cantidad < 0:
#             raise ValueError("El stock no puede ser negativo.")
#         self.stock += cantidad

# Para simular la entidad de dominio Producto en este contexto:
class Producto:
    def __init__(self, id_producto: UUID, nombre: str, precio: float, stock: int, descripcion: Optional[str] = None):
        self.id: UUID = id_producto
        self.nombre: str = nombre
        self.descripcion: Optional[str] = descripcion
        if precio <= 0:
            raise ValueError("El precio debe ser positivo.")
        self.precio: float = precio
        if stock < 0:
            raise ValueError("El stock no puede ser negativo.")
        self.stock: int = stock

    def __repr__(self):
        return f"<Producto id={self.id} nombre='{self.nombre}' stock={self.stock}>"


class IRepositorioProductos(ABC): # Output Port
    """
    Puerto de salida para la persistencia de la entidad Producto.
    Esta interfaz será implementada por un adaptador en la capa de infraestructura
    (ej. AdaptadorSQLAlchemyProductoRepository, AdaptadorEnMemoriaProductoRepository).
    Define las operaciones de persistencia que la aplicación necesita para los productos.
    """

    @abstractmethod
    async def guardar(self, producto: Producto) -> None:
        """
        Guarda un producto (ya sea uno nuevo o actualiza uno existente).
        La lógica para determinar si es nuevo o existente puede residir en el adaptador
        o ser una expectativa sobre el estado del objeto Producto.
        """
        pass

    @abstractmethod
    async def obtener_por_id(self, id_producto: UUID) -> Optional[Producto]:
        """
        Obtiene un producto por su ID.
        Devuelve la entidad Producto si se encuentra, o None si no existe.
        """
        pass

    @abstractmethod
    async def obtener_todos(self, limite: int = 100, offset: int = 0) -> List[Producto]:
        """
        Obtiene una lista de todos los productos, con paginación opcional.
        """
        pass

    @abstractmethod
    async def eliminar(self, id_producto: UUID) -> bool:
        """
        Elimina un producto por su ID.
        Devuelve True si el producto fue encontrado y eliminado, False en caso contrario.
        """
        pass

# Otro ejemplo: Puerto de salida para enviar notificaciones
class INotificador(ABC): # Output Port
    """
    Puerto de salida para enviar notificaciones a los usuarios u otros sistemas.
    Abstrae el mecanismo de notificación (email, SMS, push notification, etc.).
    """
    @abstractmethod
    async def enviar_notificacion(self, destinatario: str, mensaje: str, asunto: Optional[str] = None) -> None:
        """
        Envía una notificación al destinatario especificado.
        'asunto' es opcional y puede ser relevante para notificaciones tipo email.
        """
        pass
```
### C. Ubicación de las Interfaces de los Puertos

En una estructura de proyecto típica de Arquitectura Hexagonal, la ubicación de estas interfaces es importante para mantener la dirección correcta de las dependencias:

  * **Interfaces de Puertos de Entrada:** Generalmente residen dentro de la **capa de Aplicación**. Definen los casos de uso y son, en efecto, el "API" del núcleo de la aplicación.
  * **Interfaces de Puertos de Salida:** También residen dentro de la **capa de Aplicación** (o a veces en la **capa de Dominio** si son muy genéricas y estrechamente ligadas a las necesidades fundamentales del dominio, como suelen ser las interfaces de Repositorio en DDD). Lo crucial es que el núcleo (Dominio y/o Aplicación) define estas interfaces, y la capa de Infraestructura las implementa.

Una estructura de directorios conceptual podría verse así:

```
mi_proyecto/
├── aplicacion/  <-- Capa de Aplicación (Casos de Uso, DTOs, Interfaces de Puertos)
│   ├── __init__.py
│   ├── puertos/
│   │   ├── __init__.py
│   │   ├── entrada/
│   │   │   ├── __init__.py
│   │   │   └── igestion_inventario_input_port.py  # Contiene IGestionInventarioInputPort
│   │   └── salida/
│   │       ├── __init__.py
│   │       └── irepositorio_productos.py          # Contiene IRepositorioProductos
│   │       └── inotificador.py                    # Contiene INotificador
│   ├── servicios/                                 # O casos_de_uso/
│   │   ├── __init__.py
│   │   └── servicio_gestion_inventario.py         # Implementa IGestionInventarioInputPort
│   │                                              # y USA IRepositorioProductos, INotificador
│   └── dtos.py                                    # Contiene DatosNuevoProductoDTO, etc.
│
├── dominio/     <-- Capa de Dominio (Entidades, Objetos de Valor, Servicios de Dominio)
│   ├── __init__.py
│   └── modelos/
│       ├── __init__.py
│       └── producto.py                            # Definición de la entidad Producto
│
├── infraestructura/ <-- Capa de Infraestructura (Adaptadores que implementan/usan puertos)
│   ├── __init__.py
│   ├── adaptadores_entrada/                       # O http_api/, cli_app/
│   │   └── __init__.py
│   │   └── api_fastapi/
│   │       └── __init__.py
│   │       └── routers_productos.py               # Adaptador que USA IGestionInventarioInputPort
│   └── adaptadores_salida/                        # O persistencia/, mensajeria/
│       ├── __init__.py
│       ├── persistencia_sqlalchemy/
│       │   ├── __init__.py
│       │   └── repositorio_productos_impl.py      # Adaptador que IMPLEMENTA IRepositorioProductos
│       └── notificacion_email/
│           └── __init__.py
│           └── servicio_email_impl.py             # Adaptador que IMPLEMENTA INotificador
│
└── main.py # Punto de entrada, configuración y orquestación de la inyección de dependencias
```

*(Nota: La estructura de directorios es solo un ejemplo y puede variar según las convenciones del equipo y el tamaño del proyecto.)*

### D. Consideraciones Clave Adicionales al Diseñar Interfaces de Puertos:

  * **Lenguaje Ubicuo (Ubiquitous Language):** Los nombres de las interfaces, sus métodos y los DTOs que utilizan deben reflejar el lenguaje del dominio de tu problema. Esto es un pilar de DDD y ayuda a la comunicación y comprensión.
  * **Contratos, no Implementaciones:** Recuerda que las interfaces definen el *qué* (qué operaciones se pueden realizar y qué datos se intercambian), no el *cómo* (cómo se implementan esas operaciones). El *cómo* es responsabilidad de los adaptadores en la capa de infraestructura.
  * **Evitar Fugas de Abstracción:** Ten cuidado de no incluir detalles específicos de la infraestructura (nombres de tablas SQL, endpoints de API externas concretas, formatos de datos de bibliotecas específicas) en las definiciones de los puertos. La interfaz debe ser una abstracción pura.
  * **Simplicidad (YAGNI - You Ain't Gonna Need It):** Diseña interfaces que sean lo más simples posible para cumplir su propósito actual. No añadas métodos o complejidad previendo necesidades futuras que quizás nunca se materialicen. Es más fácil añadir a una interfaz que quitar.
  * **Estabilidad de la Interfaz:** Una vez definida y en uso, cambiar una interfaz de puerto puede tener un impacto significativo, ya que todos sus adaptadores (implementaciones) y clientes (casos de uso o adaptadores de entrada) podrían necesitar cambios. Por ello, es importante pensarlas bien.


> Diseñar interfaces efectivas para tus puertos es un arte que se refina con la práctica y la comprensión profunda del dominio. Son la base para un sistema modular, testeable, flexible y mantenible bajo la Arquitectura Hexagonal, permitiendo que tu lógica de negocio evolucione independientemente de las tecnologías externas.
---

## 6.4 Implementar adaptadores HTTP como controladores REST o WebSocket

Una vez que hemos definido las interfaces para nuestros puertos de entrada (Sección 6.3), necesitamos implementar los **Adaptadores de Entrada** (Driving Adapters) que permitirán al mundo exterior interactuar con el núcleo de nuestra aplicación. En aplicaciones web modernas, los adaptadores HTTP son omnipresentes, manifestándose comúnmente como controladores RESTful o, para comunicación bidireccional en tiempo real, como manejadores de WebSocket.

Estos adaptadores actúan como la capa más externa que recibe las solicitudes de los clientes (navegadores, aplicaciones móviles, otros servicios) y las traduce en llamadas a los casos de uso (Servicios de Aplicación) definidos por nuestros puertos de entrada.

### El Papel del Adaptador HTTP

Un adaptador HTTP, en el contexto de la Arquitectura Hexagonal, tiene las siguientes responsabilidades principales:

1.  **Recepción y Enrutamiento de Peticiones:** Escuchar en rutas (endpoints) específicas y para métodos HTTP concretos (GET, POST, PUT, DELETE, etc.). FastAPI maneja esto de forma muy eficiente con sus decoradores de ruta.
2.  **Deserialización y Validación de la Entrada:** Convertir los datos crudos de la petición HTTP (ej. JSON del cuerpo, parámetros de consulta, variables de ruta) en los objetos de transferencia de datos (DTOs) o comandos que espera el puerto de entrada correspondiente. FastAPI, con su integración con Pydantic, automatiza gran parte de esta tarea, incluyendo la validación de tipos y restricciones.
3.  **Invocación del Puerto de Entrada (Caso de Uso):** Una vez que la entrada es válida y está en el formato correcto (DTO), el adaptador llama al método apropiado del Servicio de Aplicación (que implementa el puerto de entrada). Esto se logra a menudo mediante inyección de dependencias.
4.  **Serialización de la Salida:** Tomar el resultado devuelto por el Servicio de Aplicación (generalmente otro DTO o nada) y transformarlo en una respuesta HTTP adecuada (ej. un cuerpo JSON, un código de estado HTTP). FastAPI también facilita esto con los `response_model`.
5.  **Manejo de Errores y Excepciones:** Capturar excepciones lanzadas por las capas de aplicación o dominio (ej. `ProductoNoEncontradoError`, `ValidacionReglaDeNegocioError`) y traducirlas en respuestas HTTP con códigos de estado apropiados (ej. 404 Not Found, 400 Bad Request, 409 Conflict, 500 Internal Server Error). FastAPI permite definir manejadores de excepciones personalizados.
6.  **Gestión de la Sesión HTTP:** Aspectos como cabeceras, cookies, códigos de estado, etc.

Es crucial que el adaptador HTTP se mantenga "delgado". Su función es la de un traductor y orquestador de la comunicación HTTP, **no debe contener lógica de negocio**. La lógica de negocio reside en la capa de aplicación y, fundamentalmente, en la capa de dominio.

### Implementando un Controlador REST con FastAPI

FastAPI es un framework excelente para implementar adaptadores HTTP en una Arquitectura Hexagonal debido a su uso de type hints, Pydantic para validación y serialización, y su sistema de inyección de dependencias.

Continuemos con el ejemplo de `IGestionInventarioInputPort` y los DTOs que definimos en la sección 6.3 (`DatosNuevoProductoDTO`, `ProductoCreadoDTO`, `ProductoDetalleDTO`).

**1. Estructura del Adaptador (Router de FastAPI):**

Normalmente, organizarás tus endpoints en `APIRouter`s.


```python
# En infraestructura/adaptadores_entrada/api_fastapi/routers_productos.py

from fastapi import APIRouter, Depends, HTTPException, status
from uuid import UUID
from typing import List # Si tuviéramos un endpoint para listar productos

# Importaciones de la capa de aplicación (puertos y DTOs)
# Estas rutas muestran cómo el adaptador DEPENDE de las abstracciones de la aplicación
from aplicacion.puertos.entrada.igestion_inventario_input_port import IGestionInventarioInputPort
from aplicacion.dtos import DatosNuevoProductoDTO, ProductoCreadoDTO # Suponiendo que ProductoDetalleDTO también está aquí
# Para el ejemplo, si ProductoDetalleDTO fue definido anidado, se importaría de donde esté.
# from aplicacion.puertos.entrada.igestion_inventario_input_port import ProductoDetalleDTO

# Importación para la inyección de dependencias (se verá en detalle en la sección 6.9)
# Por ahora, asumimos una función que provee la instancia del servicio
from mi_proyecto.main import obtener_servicio_gestion_inventario # Esto es un placeholder

router = APIRouter(
    prefix="/api/v1/productos",
    tags=["Productos"]
)

# Placeholder para los DTOs si no están en aplicacion.dtos
# from pydantic import BaseModel
# class ProductoDetalleDTO(BaseModel):
#     id_producto: UUID
#     nombre: str
#     # ... otros campos

@router.post(
    "/",
    response_model=ProductoCreadoDTO,
    status_code=status.HTTP_201_CREATED,
    summary="Registrar un nuevo producto",
    description="Crea un nuevo producto en el inventario con los datos proporcionados."
)
async def registrar_producto_endpoint(
    datos_producto_api: DatosNuevoProductoDTO, # FastAPI usa esto para validar el cuerpo de la petición
    servicio_inventario: IGestionInventarioInputPort = Depends(obtener_servicio_gestion_inventario) # Inyección de dependencia
):
    """
    Endpoint para registrar un nuevo producto.
    - Recibe: Datos del nuevo producto en el cuerpo de la petición.
    - Llama: Al servicio de aplicación para registrar el producto.
    - Devuelve: Los datos del producto creado.
    """
    try:
        # El adaptador llama al puerto de entrada (Servicio de Aplicación)
        producto_creado_dto = await servicio_inventario.registrar_nuevo_producto(
            datos_producto=datos_producto_api # El DTO ya validado por FastAPI
        )
        return producto_creado_dto
    except ValueError as ve: # Ejemplo de excepción de validación de negocio
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=str(ve)
        )
    except Exception as e: # Captura genérica para otros errores inesperados
        # En un sistema real, querrías logging más específico y quizás tipos de error personalizados
        # provenientes de la capa de aplicación/dominio con mapeos a códigos HTTP.
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Ocurrió un error inesperado: {str(e)}"
        )

@router.get(
    "/{producto_id}",
    response_model=ProductoDetalleDTO, # Asumiendo que ProductoDetalleDTO está definido e importado
    summary="Obtener detalles de un producto",
    description="Recupera los detalles de un producto específico por su ID."
)
async def obtener_producto_endpoint(
    producto_id: UUID, # FastAPI extrae y valida el path parameter
    servicio_inventario: IGestionInventarioInputPort = Depends(obtener_servicio_gestion_inventario)
):
    """
    Endpoint para obtener los detalles de un producto.
    - Recibe: ID del producto como path parameter.
    - Llama: Al servicio de aplicación para obtener el producto.
    - Devuelve: Los detalles del producto o un error 404 si no se encuentra.
    """
    try:
        producto_dto = await servicio_inventario.obtener_detalles_producto(id_producto=producto_id)
        if producto_dto is None:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"Producto con ID '{producto_id}' no encontrado."
            )
        return producto_dto
    except HTTPException: # Re-lanzar HTTPExceptions para que FastAPI las maneje
        raise
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Ocurrió un error inesperado al obtener el producto: {str(e)}"
        )

# Aquí podrían ir otros endpoints: PUT para actualizar, DELETE para eliminar, GET para listar, etc.
# Cada uno seguiría un patrón similar:
# 1. Definir la ruta y el método HTTP.
# 2. Especificar los DTOs de entrada (FastAPI los usa para el cuerpo, parámetros, etc.) y salida (response_model).
# 3. Inyectar y usar el puerto de entrada (Servicio de Aplicación).
# 4. Manejar los resultados y las excepciones, traduciéndolos a respuestas HTTP.
```

**2. Inyección de Dependencias (Adelanto de la Sección 6.9):**

La línea `servicio_inventario: IGestionInventarioInputPort = Depends(obtener_servicio_gestion_inventario)` es clave. `Depends` es el mecanismo de FastAPI para la inyección de dependencias. La función `obtener_servicio_gestion_inventario` sería responsable de construir (o recuperar de un contenedor) la instancia concreta del servicio de aplicación que implementa `IGestionInventarioInputPort`. Esta instancia, a su vez, tendrá sus propias dependencias (como los repositorios) inyectadas.

**3. Manejo de Errores Específicos:**

En lugar de `ValueError` genérico, la capa de aplicación podría lanzar excepciones más específicas como `ReglaDeNegocioVioladaError` o `ProductoConNombreDuplicadoError`. El adaptador HTTP podría tener manejadores de excepciones globales de FastAPI para mapear estas excepciones personalizadas a respuestas HTTP específicas.

```python
# En main.py o un módulo de configuración de la app FastAPI
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

class ProductoNoEncontradoErrorAplicacion(Exception):
    def __init__(self, producto_id: UUID):
        self.producto_id = producto_id
        super().__init__(f"Producto con ID {producto_id} no encontrado.")

def configurar_manejadores_excepcion(app: FastAPI):
    @app.exception_handler(ProductoNoEncontradoErrorAplicacion)
    async def handle_producto_no_encontrado_error(request: Request, exc: ProductoNoEncontradoErrorAplicacion):
        return JSONResponse(
            status_code=status.HTTP_404_NOT_FOUND,
            content={"message": str(exc)}
        )
    # ... otros manejadores
```
### Adaptadores WebSocket con FastAPI

Si tu aplicación requiere comunicación bidireccional persistente (ej. notificaciones en tiempo real, chats, juegos), puedes implementar adaptadores WebSocket. Los principios son similares:

1.  **Establecimiento de la Conexión:** FastAPI maneja la negociación del handshake WebSocket.
2.  **Recepción de Mensajes:** El adaptador recibe mensajes del cliente a través de la conexión WebSocket.
3.  **Traducción e Invocación:** El mensaje se traduce a un comando o consulta para un puerto de entrada. Se invoca el servicio de aplicación.
4.  **Envío de Respuestas/Broadcasts:** El resultado del servicio de aplicación (o eventos generados) se envía de vuelta al cliente o a múltiples clientes a través de sus conexiones WebSocket.

<!-- end list -->

```python
# Ejemplo conceptual de un endpoint WebSocket en FastAPI
# from fastapi import WebSocket, WebSocketDisconnect

@router.websocket("/ws/inventario_updates/{client_id}")
async def websocket_endpoint_inventario(
    websocket: WebSocket,
    client_id: str,
    servicio_notificaciones_inventario: IServicioNotificacionesInventario = Depends(...) # Puerto de entrada para manejar lógica de WS
):
    await websocket.accept()
    await servicio_notificaciones_inventario.conectar_cliente(client_id, websocket) # Registrar el cliente
    try:
        while True:
            data = await websocket.receive_text() # O receive_json()
            # Aquí, 'data' podría ser un comando como "SUSCRIBIR_PRODUCTO_X"
            # O un mensaje que el servicio de aplicación debe procesar.
            await servicio_notificaciones_inventario.procesar_mensaje_cliente(client_id, data)
    except WebSocketDisconnect:
        await servicio_notificaciones_inventario.desconectar_cliente(client_id)
    except Exception as e:
        # Manejar otros errores, quizás enviar un mensaje de error por WS antes de cerrar.
        await websocket.close(code=status.WS_1011_INTERNAL_ERROR)

# El IServicioNotificacionesInventario sería un puerto de entrada específico para la lógica de WebSocket,
# que a su vez podría usar otros servicios de aplicación o puertos de salida para obtener datos o enviar notificaciones.
```
### Consideraciones Clave para Adaptadores HTTP:

  * **Mantenerlos "Delgados" (Thin Adapters):** Reitero, la lógica de negocio no va aquí. Su responsabilidad es la adaptación del protocolo HTTP a las llamadas de los casos de uso.
  * **Contrato API Explícito:** FastAPI genera automáticamente una especificación OpenAPI (Swagger UI / ReDoc), lo cual es una gran ventaja. Asegúrate de que tus DTOs (modelos Pydantic) y `response_model` reflejen fielmente el contrato deseado.
  * **Seguridad (Autenticación y Autorización):** FastAPI ofrece mecanismos robustos (`Security` con `Depends`) para integrar la autenticación (quién eres) y la autorización (qué tienes permitido hacer). Estas verificaciones suelen realizarse en el adaptador o en middleware antes de invocar el servicio de aplicación. La identidad del usuario verificado puede pasarse al servicio de aplicación si es necesaria para la lógica de negocio.
  * **Versión de API:** Si necesitas versionar tu API (ej. `/api/v1/productos`, `/api/v2/productos`), los `APIRouter` de FastAPI facilitan esta organización.
  * **No Acoplarse a Detalles del Dominio:** El adaptador conoce los DTOs definidos por la capa de aplicación, pero no debería necesitar conocer los detalles internos de las entidades del dominio. La transformación (si es necesaria) entre entidades del dominio y DTOs de respuesta ocurre en la capa de aplicación o en el límite entre esta y el adaptador.

> Implementar adaptadores HTTP correctamente es fundamental para exponer las capacidades del núcleo de tu aplicación de una manera estándar, segura y fácil de usar para los clientes. FastAPI proporciona herramientas poderosas para construir estos adaptadores de forma eficiente y alineada con los principios de la Arquitectura Hexagonal.
---

## 6.5 Separar repositorios del dominio usando interfaces

Un aspecto fundamental de la Arquitectura Hexagonal, especialmente cuando se combina con Domain-Driven Design (DDD), es la gestión de la persistencia de datos. El patrón **Repository** de DDD nos ofrece una abstracción crucial para esto: una interfaz similar a una colección para acceder a los objetos de nuestro dominio (Entidades y Raíces de Agregados). En la Arquitectura Hexagonal, estas interfaces de repositorio actúan como **Puertos de Salida** (Driven Ports).

El objetivo principal de separar los repositorios del dominio mediante interfaces es **aislar la lógica de negocio y de aplicación de las preocupaciones y tecnologías específicas de la persistencia de datos**. El dominio y la capa de aplicación no deben "saber" si los datos se guardan en una base de datos PostgreSQL, MongoDB, un archivo local o en memoria. Solo deben conocer el contrato (la interfaz del repositorio) que necesitan para obtener y guardar sus objetos.

### El Papel Clave de las Interfaces de Repositorio

Las interfaces de repositorio son el pilar de esta separación:

1.  **Definen el Contrato de Persistencia:** La interfaz especifica *qué* operaciones de persistencia necesita la aplicación o el dominio. Estas operaciones se expresan en términos del modelo de dominio.

      * *Ejemplos de métodos:* `obtener_por_id(id: UUID) -> Optional[Producto]`, `guardar(producto: Producto) -> None`, `buscar_por_nombre(nombre: str) -> List[Producto]`.

2.  **Pertenecen a la Capa de Aplicación/Dominio:** Este es un punto crucial para lograr la Inversión de Dependencias (la 'D' en SOLID). Las interfaces de los repositorios son definidas por las capas internas (aplicación o incluso dominio si son muy genéricas y fundamentales para la entidad). La capa de infraestructura (donde residen las implementaciones concretas de los repositorios) *depende* de estas interfaces, no al revés.

      * *Ubicación típica:* `aplicacion/puertos/salida/` o `dominio/puertos/` (si la interfaz es considerada parte intrínseca de las necesidades del dominio para gestionar sus agregados).

3.  **Son Agnósticas a la Tecnología:** Las firmas de los métodos en la interfaz utilizan objetos del dominio (Entidades, Raíces de Agregados, Objetos de Valor) y tipos de datos primitivos o DTOs genéricos si es necesario, pero nunca exponen detalles de la tecnología de base de datos subyacente (como objetos de conexión SQL, cursores, o tipos específicos de un ORM en la propia firma de la interfaz).

4.  **Enfocadas en Raíces de Agregados (DDD):** Según los principios de DDD, los repositorios se diseñan típicamente para gestionar Raíces de Agregados. Esto significa que obtienes y guardas el agregado completo a través de su raíz, asegurando la consistencia interna del agregado.

### Beneficios de esta Separación

Utilizar interfaces para desacoplar los repositorios del dominio y la aplicación ofrece ventajas significativas:

  * **Testeabilidad Mejorada:**

      * Los Servicios de Aplicación (casos de uso) pueden ser probados unitaria o funcionalmente de forma aislada, inyectando implementaciones "falsas" o "en memoria" (mocks/stubs) de las interfaces de repositorio. Esto elimina la necesidad de una base de datos real para muchas pruebas, haciéndolas más rápidas y fiables.
      * La lógica de las entidades del dominio puede ser probada sin ninguna consideración de cómo se persisten.

  * **Flexibilidad y Mantenibilidad Tecnológica:**

      * Puedes cambiar la tecnología de base de datos subyacente (ej. de SQLite a PostgreSQL, o de un ORM a otro) sin modificar el código del dominio o de la capa de aplicación. Solo necesitas escribir una nueva clase de adaptador que implemente la interfaz del repositorio existente.
      * Facilita la evolución de la capa de persistencia (ej. optimizar consultas, cambiar esquemas) con un impacto mínimo en el resto del sistema.

  * **Clara Separación de Responsabilidades:**

      * Se establece un límite explícito y fuerte entre la lógica de negocio/aplicación y la lógica de acceso a datos. Esto hace que el sistema sea más fácil de entender, razonar y mantener.

  * **Desarrollo Paralelo:**

      * Una vez que las interfaces de repositorio están definidas, los equipos pueden trabajar en paralelo: un equipo en la lógica de negocio y de aplicación, y otro en la implementación de la persistencia.

### Diseñando Interfaces de Repositorio Efectivas

Al diseñar estas interfaces, considera lo siguiente:

1.  **Nomenclatura:** Utiliza nombres claros y descriptivos que reflejen la entidad o agregado que gestionan. Es común usar el prefijo `I` (para Interfaz) o el sufijo `Repository`.

      * *Ejemplo:* `IProductoRepository`, `IPedidoRepository`, `ClienteRepository` (si la convención es no usar prefijos/sufijos explícitos para interfaces en Python y confiar en la type hinting).

2.  **Métodos Significativos:**

      * Incluye métodos para las operaciones CRUD básicas si son necesarias (`guardar`, `obtener_por_id`, `eliminar`). Nota: `guardar` a menudo maneja tanto la creación como la actualización.
      * Añade métodos de consulta (finders) que reflejen las necesidades específicas de los casos de uso de tu aplicación. Estos métodos deben devolver entidades del dominio o colecciones de ellas.
          * *Ejemplo:* `buscar_pedidos_por_cliente_y_estado(cliente_id: UUID, estado: EstadoPedido) -> List[Pedido]`.
      * Las firmas de los métodos deben usar objetos y tipos del dominio.
          * *Correcto:* `async def guardar(self, producto: Producto) -> None:`
          * *Evitar:* `async def guardar(self, datos_producto: dict) -> None:` (Esto introduce acoplamiento a una estructura de datos genérica en lugar del tipo de dominio).

3.  **Abstracción de la Unidad de Trabajo (Unit of Work):**

      * En algunos casos, especialmente con ORMs como SQLAlchemy, el concepto de "unidad de trabajo" (manejo de sesiones y transacciones) es importante. La interfaz del repositorio generalmente no expone directamente la sesión del ORM. La gestión de la transacción suele ser responsabilidad del Servicio de Aplicación (o un decorador/middleware que envuelve al caso de uso) que coordina uno o más repositorios. El repositorio opera dentro de la transacción que le proporciona el contexto de la aplicación.

**Ejemplo de Interfaz de Repositorio en Python:**

Revisitemos y afinemos la interfaz `IRepositorioProductos` que vimos anteriormente, asumiendo que tenemos una entidad `Producto` en nuestro dominio.


```python
# En dominio/modelos/producto.py (o similar)
from uuid import UUID, uuid4
from typing import Optional

class Producto: # Entidad del Dominio
    def __init__(self, nombre: str, precio: float, stock: int, descripcion: Optional[str] = None, id_producto: Optional[UUID] = None):
        self.id: UUID = id_producto or uuid4()
        self.nombre: str = nombre # Podría tener validaciones
        self.descripcion: Optional[str] = descripcion
        
        if precio <= 0:
            raise ValueError("El precio debe ser positivo.")
        self.precio: float = precio
        
        if stock < 0:
            raise ValueError("El stock no puede ser negativo.")
        self.stock: int = stock

    def cambiar_precio(self, nuevo_precio: float) -> None:
        if nuevo_precio <= 0:
            raise ValueError("El precio debe ser positivo.")
        self.precio = nuevo_precio

    def ajustar_stock(self, cantidad: int) -> None:
        nuevo_stock = self.stock + cantidad
        if nuevo_stock < 0:
            # Dependiendo de las reglas de negocio, esto podría lanzar un error o ser manejado de otra forma.
            raise ValueError("El ajuste de stock no puede resultar en stock negativo.")
        self.stock = nuevo_stock
    
    def __eq__(self, other):
        if not isinstance(other, Producto):
            return NotImplemented
        return self.id == other.id

    def __hash__(self):
        return hash(self.id)

# ---------------------------------------------------------------------------
# En aplicacion/puertos/salida/irepositorio_productos.py (o dominio/puertos/)
from abc import ABC, abstractmethod
from typing import List, Optional
from uuid import UUID
from dominio.modelos.producto import Producto # Importa la entidad del dominio

class IRepositorioProductos(ABC): # Puerto de Salida
    """
    Interfaz que define las operaciones de persistencia necesarias para la entidad Producto.
    Esta interfaz es implementada por adaptadores en la capa de infraestructura.
    """

    @abstractmethod
    async def guardar(self, producto: Producto) -> None:
        """
        Persiste un objeto Producto. Si el producto ya tiene un ID,
        se asume una actualización; si no, o si el ID no existe en la persistencia,
        se asume una creación. La lógica exacta puede depender de la implementación.
        """
        pass

    @abstractmethod
    async def obtener_por_id(self, id_producto: UUID) -> Optional[Producto]:
        """
        Recupera un Producto por su identificador único.
        Devuelve el Producto si se encuentra, o None en caso contrario.
        """
        pass

    @abstractmethod
    async def obtener_todos(self, limite: int = 100, offset: int = 0) -> List[Producto]:
        """
        Recupera una lista de todos los Productos, con paginación opcional.
        """
        pass

    @abstractmethod
    async def eliminar(self, id_producto: UUID) -> bool:
        """
        Elimina un Producto de la persistencia por su ID.
        Devuelve True si el producto fue encontrado y eliminado, False en caso contrario.
        """
        pass

    @abstractmethod
    async def buscar_por_nombre(self, nombre_parcial: str) -> List[Producto]:
        """
        Busca productos cuyo nombre contenga la cadena 'nombre_parcial'.
        Este es un ejemplo de un método de búsqueda más específico.
        """
        pass
```

### Relación con la Capa de Dominio

Es vital que las entidades y objetos de valor de la capa de dominio permanezcan puros y no tengan dependencias directas de las interfaces de repositorio para su lógica interna. Las entidades gestionan su estado y aplican sus reglas de negocio. Son los **Servicios de Aplicación** (casos de uso) los que utilizan las interfaces de repositorio para obtener estas entidades, invocar sus métodos de negocio y luego usar nuevamente los repositorios para persistir los cambios de estado.

En escenarios muy específicos, un **Servicio de Dominio** podría necesitar usar una interfaz de repositorio (por ejemplo, para verificar una invariante que cruza agregados, como la unicidad de un nombre de usuario antes de crear uno nuevo). Sin embargo, esto debe manejarse con cuidado para no diluir la cohesión de los agregados o introducir demasiadas dependencias en el dominio.

### La Implementación del Adaptador (Siguiente Paso)

Esta sección se ha centrado en *definir la interfaz* del repositorio para lograr la separación. La implementación concreta de esta interfaz (por ejemplo, `SQLAlchemyProductoRepository` o `MongoDBProductoRepository`) es un **adaptador** que reside en la capa de infraestructura. Este adaptador contendrá el código específico para interactuar con la base de datos elegida, traduciendo las llamadas a los métodos de la interfaz en operaciones de base de datos.

> Al separar el *qué* (la interfaz del repositorio definida por las necesidades de la aplicación/dominio) del *cómo* (la implementación del adaptador en la infraestructura), ganamos una enorme flexibilidad y mantenemos nuestro núcleo de negocio limpio y desacoplado de las preocupaciones tecnológicas de la persistencia.
---

## 6.6 Diseñar pruebas para el núcleo sin depender de infraestructuras

Una de las ventajas más significativas de la Arquitectura Hexagonal es su inherente **testeabilidad**. Al desacoplar el núcleo de la aplicación (la lógica de dominio y de aplicación) de las preocupaciones de la infraestructura (bases de datos, APIs externas, frameworks web), podemos diseñar pruebas robustas, rápidas y fiables para este núcleo sin necesidad de levantar entornos complejos o depender de componentes externos.

El "núcleo" en este contexto se refiere a:

  * **Capa de Dominio:** Entidades, Objetos de Valor, Raíces de Agregados, Servicios de Dominio y Eventos de Dominio. Aquí reside la lógica de negocio pura.
  * **Capa de Aplicación:** Servicios de Aplicación (o Casos de Uso) que orquestan la lógica de dominio para cumplir con las solicitudes de los usuarios o sistemas externos. Estos servicios implementan los puertos de entrada y utilizan los puertos de salida.

Probar "sin depender de infraestructuras" significa que nuestras pruebas para estas capas no requerirán una base de datos activa, no harán llamadas HTTP reales a servicios externos, no interactuarán con un bus de mensajes, ni necesitarán un servidor web en ejecución.

### 1\. Probando la Capa de Dominio

La capa de dominio, al ser (idealmente) Python puro sin referencias a frameworks o tecnologías externas, es la más sencilla de probar unitariamente.

  * **Enfoque:** Verificar la lógica de negocio intrínseca, las transiciones de estado de las entidades, las reglas de validación dentro de las entidades y los Objetos de Valor, y el comportamiento de los Servicios de Dominio.
  * **Características de las Pruebas de Dominio:**
      * Son **pruebas unitarias** en su forma más pura.
      * Generalmente no requieren *mocks* o *stubs* para dependencias externas, ya que el dominio no debería tenerlas (salvo, quizás, abstracciones muy simples como un proveedor de fecha/hora actual, que también se puede falsear fácilmente).
      * Son extremadamente rápidas de ejecutar.
      * Proporcionan una alta confianza en la corrección de las reglas de negocio.

**Ejemplo (usando `pytest` y la entidad `Producto` de secciones anteriores):**

```python
# En tests/dominio/test_producto.py

import pytest
from uuid import uuid4, UUID
from dominio.modelos.producto import Producto # Asumiendo la entidad Producto definida previamente

class TestProducto:

    def test_crear_producto_exitosamente(self):
        id_producto = uuid4()
        producto = Producto(
            id_producto=id_producto,
            nombre="Laptop Gamer",
            descripcion="Laptop con tarjeta gráfica dedicada",
            precio=1200.50,
            stock=10
        )
        assert producto.id == id_producto
        assert producto.nombre == "Laptop Gamer"
        assert producto.precio == 1200.50
        assert producto.stock == 10

    def test_crear_producto_precio_invalido_lanza_excepcion(self):
        with pytest.raises(ValueError, match="El precio debe ser positivo."):
            Producto(nombre="Teclado", precio=0, stock=5)

    def test_crear_producto_stock_invalido_lanza_excepcion(self):
        with pytest.raises(ValueError, match="El stock no puede ser negativo."):
            Producto(nombre="Mouse", precio=25.00, stock=-1)

    def test_cambiar_precio_exitosamente(self):
        producto = Producto(nombre="Monitor", precio=300.00, stock=5)
        producto.cambiar_precio(275.00)
        assert producto.precio == 275.00

    def test_cambiar_precio_invalido_lanza_excepcion(self):
        producto = Producto(nombre="Monitor", precio=300.00, stock=5)
        with pytest.raises(ValueError, match="El precio debe ser positivo."):
            producto.cambiar_precio(-50.00)

    def test_ajustar_stock_exitosamente(self):
        producto = Producto(nombre="SSD 1TB", precio=100.00, stock=20)
        
        producto.ajustar_stock(5) # Añadir stock
        assert producto.stock == 25
        
        producto.ajustar_stock(-10) # Quitar stock
        assert producto.stock == 15

    def test_ajustar_stock_resulta_en_negativo_lanza_excepcion(self):
        producto = Producto(nombre="RAM 16GB", precio=80.00, stock=5)
        with pytest.raises(ValueError, match="El ajuste de stock no puede resultar en stock negativo."):
            producto.ajustar_stock(-10)

    def test_productos_con_mismo_id_son_iguales(self):
        id_comun = uuid4()
        p1 = Producto(id_producto=id_comun, nombre="Prod A", precio=10, stock=1)
        p2 = Producto(id_producto=id_comun, nombre="Prod B", precio=20, stock=2) # Mismo ID, diferente data
        assert p1 == p2 # Basado en la implementación de __eq__ por ID
        assert hash(p1) == hash(p2)

    def test_productos_con_diferente_id_no_son_iguales(self):
        p1 = Producto(id_producto=uuid4(), nombre="Prod", precio=10, stock=1)
        p2 = Producto(id_producto=uuid4(), nombre="Prod", precio=10, stock=1)
        assert p1 != p2
```

### 2\. Probando la Capa de Aplicación (Servicios de Aplicación / Casos de Uso)

Los Servicios de Aplicación orquestan la lógica de dominio y coordinan la interacción con los puertos de salida (repositorios, notificadores, etc.). Probarlos sin infraestructura implica reemplazar las implementaciones reales de estos puertos de salida con **dobles de prueba** (test doubles).

  * **Enfoque:** Verificar que el servicio de aplicación maneja correctamente los datos de entrada (DTOs/Comandos), interactúa apropiadamente con los puertos de salida (llamando a los métodos correctos con los argumentos correctos) y produce el resultado esperado (DTOs de salida o excepciones).
  * **Técnica Clave: Mocking/Stubbing de Puertos de Salida:**
      * Se utilizan bibliotecas como `unittest.mock` de Python (o `pytest-mock`) para crear versiones simuladas (mocks o stubs) de las interfaces de los puertos de salida.
      * Estos mocks permiten controlar el comportamiento de las dependencias externas (ej. simular que un repositorio devuelve un producto específico o `None`) y verificar que el servicio de aplicación interactúa con ellas como se espera.

**Ejemplo (probando un `ServicioGestionInventario` que usa `IRepositorioProductos` e `INotificador`):**

Primero, definamos conceptualmente un servicio de aplicación y los DTOs/interfaces que usa (algunos ya los hemos visto):

```python
# --- En aplicacion/dtos.py ---
from pydantic import BaseModel, Field
from uuid import UUID

class DatosNuevoProductoDTO(BaseModel):
    nombre: str = Field(..., min_length=3)
    descripcion: str | None = None
    precio: float = Field(..., gt=0)
    stock_inicial: int = Field(..., ge=0)

class ProductoCreadoDTO(BaseModel):
    id_producto: UUID
    nombre: str
    mensaje_bienvenida: str | None = None


# --- En aplicacion/puertos/entrada/igestion_inventario_input_port.py ---
# (Ya definida en 6.3, la incluimos para contexto)
# from abc import ABC, abstractmethod
# class IGestionInventarioInputPort(ABC):
#     @abstractmethod
#     async def registrar_nuevo_producto(self, datos_producto: DatosNuevoProductoDTO) -> ProductoCreadoDTO:
#         pass


# --- En aplicacion/puertos/salida/irepositorio_productos.py --- (ya definida)
# --- En aplicacion/puertos/salida/inotificador.py --- (ya definida)


# --- En aplicacion/servicios/servicio_gestion_inventario.py ---
from dominio.modelos.producto import Producto
from aplicacion.puertos.entrada.igestion_inventario_input_port import IGestionInventarioInputPort # Asumimos que la interfaz se define así
from aplicacion.puertos.salida.irepositorio_productos import IRepositorioProductos
from aplicacion.puertos.salida.inotificador import INotificador
from aplicacion.dtos import DatosNuevoProductoDTO, ProductoCreadoDTO
from uuid import UUID

class NombreProductoDuplicadoError(Exception): # Excepción personalizada de la capa de aplicación
    pass

class ServicioGestionInventario(IGestionInventarioInputPort):
    def __init__(self, repositorio_productos: IRepositorioProductos, notificador: INotificador):
        self.repositorio_productos = repositorio_productos
        self.notificador = notificador

    async def registrar_nuevo_producto(self, datos_producto: DatosNuevoProductoDTO) -> ProductoCreadoDTO:
        # Simular verificación de nombre duplicado
        productos_existentes = await self.repositorio_productos.buscar_por_nombre(datos_producto.nombre)
        if any(p.nombre.lower() == datos_producto.nombre.lower() for p in productos_existentes):
            raise NombreProductoDuplicadoError(f"Ya existe un producto con el nombre '{datos_producto.nombre}'.")

        nuevo_producto = Producto(
            nombre=datos_producto.nombre,
            descripcion=datos_producto.descripcion,
            precio=datos_producto.precio,
            stock=datos_producto.stock_inicial
            # El ID se autogenera en la entidad Producto si no se pasa
        )
        
        await self.repositorio_productos.guardar(nuevo_producto)
        
        mensaje_notificacion = f"¡Nuevo producto '{nuevo_producto.nombre}' registrado con ID {nuevo_producto.id}!"
        await self.notificador.enviar_notificacion(
            destinatario="admin@example.com", # Destinatario de ejemplo
            mensaje=mensaje_notificacion,
            asunto="Nuevo Producto Registrado"
        )
        
        return ProductoCreadoDTO(
            id_producto=nuevo_producto.id,
            nombre=nuevo_producto.nombre,
            mensaje_bienvenida="Producto registrado con éxito y notificación enviada."
        )
```

Ahora, las pruebas para `ServicioGestionInventario` usando `pytest` y `unittest.mock`:

```python
# En tests/aplicacion/test_servicio_gestion_inventario.py

import pytest
from unittest.mock import Mock, AsyncMock # AsyncMock para métodos de corutina
from uuid import uuid4

from aplicacion.servicios.servicio_gestion_inventario import ServicioGestionInventario, NombreProductoDuplicadoError
from aplicacion.dtos import DatosNuevoProductoDTO, ProductoCreadoDTO
from aplicacion.puertos.salida.irepositorio_productos import IRepositorioProductos # Para type hinting del mock
from aplicacion.puertos.salida.inotificador import INotificador # Para type hinting del mock
from dominio.modelos.producto import Producto


@pytest.fixture
def mock_repositorio_productos() -> Mock:
    # Creamos un mock que simula la interfaz IRepositorioProductos
    # Para métodos async, usamos AsyncMock como el mock en sí o para `return_value` o `side_effect`
    mock = Mock(spec=IRepositorioProductos)
    mock.guardar = AsyncMock() # Especificamos que 'guardar' es una corutina mockeada
    mock.buscar_por_nombre = AsyncMock(return_value=[]) # Por defecto no encuentra duplicados
    return mock

@pytest.fixture
def mock_notificador() -> Mock:
    mock = Mock(spec=INotificador)
    mock.enviar_notificacion = AsyncMock() # 'enviar_notificacion' es una corutina mockeada
    return mock

@pytest.fixture
def servicio_inventario(mock_repositorio_productos: Mock, mock_notificador: Mock) -> ServicioGestionInventario:
    # Inyectamos los mocks en el servicio
    return ServicioGestionInventario(
        repositorio_productos=mock_repositorio_productos,
        notificador=mock_notificador
    )

@pytest.mark.asyncio # Necesario para probar funciones async con pytest
async def test_registrar_nuevo_producto_exitosamente(
    servicio_inventario: ServicioGestionInventario,
    mock_repositorio_productos: Mock,
    mock_notificador: Mock
):
    # Arrange (Preparar)
    datos_entrada = DatosNuevoProductoDTO(
        nombre="Nuevo Teclado Mecánico",
        descripcion="Teclado con switches Cherry MX",
        precio=150.75,
        stock_inicial=25
    )
    # Configuramos el mock del repositorio para la búsqueda de duplicados
    mock_repositorio_productos.buscar_por_nombre.return_value = []


    # Act (Actuar)
    resultado_dto = await servicio_inventario.registrar_nuevo_producto(datos_entrada)

    # Assert (Verificar)
    # 1. Verificar que el repositorio fue llamado para guardar
    mock_repositorio_productos.guardar.assert_called_once()
    producto_guardado_args = mock_repositorio_productos.guardar.call_args[0][0] # El primer argumento posicional
    
    assert isinstance(producto_guardado_args, Producto)
    assert producto_guardado_args.nombre == datos_entrada.nombre
    assert producto_guardado_args.precio == datos_entrada.precio
    assert producto_guardado_args.stock == datos_entrada.stock_inicial

    # 2. Verificar que el notificador fue llamado
    mock_notificador.enviar_notificacion.assert_called_once()
    args_notificacion, _ = mock_notificador.enviar_notificacion.call_args
    assert args_notificacion[0] == "admin@example.com" # destinatario
    assert datos_entrada.nombre in args_notificacion[1] # mensaje
    assert "Nuevo Producto Registrado" in args_notificacion[2] # asunto


    # 3. Verificar el DTO de respuesta
    assert isinstance(resultado_dto, ProductoCreadoDTO)
    assert resultado_dto.id_producto is not None # Se debe haber generado un ID
    assert resultado_dto.nombre == datos_entrada.nombre
    assert "éxito" in resultado_dto.mensaje_bienvenida.lower()

@pytest.mark.asyncio
async def test_registrar_nuevo_producto_nombre_duplicado_lanza_excepcion(
    servicio_inventario: ServicioGestionInventario,
    mock_repositorio_productos: Mock,
    mock_notificador: Mock
):
    # Arrange
    nombre_existente = "Producto Existente"
    datos_entrada = DatosNuevoProductoDTO(
        nombre=nombre_existente, # Nombre que ya existe
        precio=100.00,
        stock_inicial=10
    )
    # Simulamos que el repositorio encuentra un producto con el mismo nombre
    producto_mock_existente = Producto(id_producto=uuid4(), nombre=nombre_existente, precio=90.00, stock=5)
    mock_repositorio_productos.buscar_por_nombre.return_value = [producto_mock_existente]
    
    # Act & Assert
    with pytest.raises(NombreProductoDuplicadoError, match=f"Ya existe un producto con el nombre '{nombre_existente}'."):
        await servicio_inventario.registrar_nuevo_producto(datos_entrada)
    
    # Verificar que no se intentó guardar ni notificar
    mock_repositorio_productos.guardar.assert_not_called()
    mock_notificador.enviar_notificacion.assert_not_called()
```

### 3\. Tipos de Dobles de Prueba Comunes

  * **Mocks (Simulacros):** Se centran en la verificación de interacciones. Se configuran con expectativas sobre qué métodos serán llamados, cuántas veces y con qué argumentos (como en `assert_called_once()`).
  * **Stubs (Retornos Fijos):** Proveen respuestas predefinidas a las llamadas. Por ejemplo, un stub de repositorio podría devolver siempre una lista específica de productos cuando se llama a `obtener_todos()`. `mock_repositorio_productos.buscar_por_nombre.return_value = []` es un ejemplo de configuración de un stub.
  * **Fakes (Falsificaciones):** Tienen implementaciones funcionales, pero simplificadas y no aptas para producción. Un ejemplo clásico es un repositorio en memoria (una lista o diccionario Python) que implementa la interfaz del repositorio. Los Fakes pueden ser muy útiles para pruebas de integración más complejas de la capa de aplicación sin la sobrecarga de una base de datos real.

### 4\. Estructura de las Pruebas

Es una buena práctica organizar las pruebas en un directorio `tests/` en la raíz del proyecto, con una estructura interna que refleje la de tu código fuente:

```
mi_proyecto/
├── aplicacion/
├── dominio/
├── infraestructura/
└── tests/
    ├── __init__.py
    ├── dominio/
    │   ├── __init__.py
    │   └── test_producto.py
    │   └── test_otro_objeto_dominio.py
    ├── aplicacion/
    │   ├── __init__.py
    │   └── test_servicio_gestion_inventario.py
    └── conftest.py  # Para fixtures globales de pytest
```

Adopta un patrón como **Arrange-Act-Assert** (AAA) o **Given-When-Then** (GWT) para estructurar cada caso de prueba, haciéndolos más legibles.

### Beneficios de esta Estrategia de Pruebas para el Núcleo

  * **Ciclo de Retroalimentación Rápido:** Las pruebas se ejecutan en milisegundos o segundos, ya que no hay operaciones lentas de E/S ni configuración de sistemas externos.
  * **Fiabilidad:** Las pruebas son deterministas y menos propensas a fallos esporádicos ("flakiness") causados por problemas de red, estado de bases de datos, etc.
  * **Localización Precisa de Errores:** Cuando una prueba del núcleo falla, indica un problema directamente en la lógica de negocio o de aplicación, no en una capa de infraestructura.
  * **Confianza para Refactorizar:** Un conjunto sólido de pruebas para el núcleo permite refactorizar con mayor seguridad tanto el propio núcleo como las implementaciones de los adaptadores.
  * **Documentación Viva:** Las pruebas sirven como ejemplos ejecutables de cómo se espera que funcione y se utilice el dominio y los servicios de aplicación.

### ¿Qué NO se Prueba Aquí?

Este enfoque se centra en probar el núcleo *aislado*. Las siguientes pruebas son distintas y también importantes, pero se abordan de otra manera:

  * **Pruebas de los Adaptadores:** Por ejemplo, probar que un adaptador de repositorio SQLAlchemy realmente interactúa correctamente con una base de datos (esto sería una prueba de integración más lenta).
  * **Pruebas de Integración Completas (End-to-End):** Probar el flujo completo desde una petición HTTP hasta la base de datos y viceversa.

Al diseñar pruebas para el núcleo sin depender de infraestructuras, estás invirtiendo en la calidad y mantenibilidad a largo plazo de tu software, asegurando que la lógica más crítica de tu aplicación sea robusta y correcta.

---

## 6.7 Integrar eventos de dominio desde la capa interna

Imagina que el núcleo de tu aplicación (tu capa de Dominio y Aplicación) no es solo un ejecutor de tareas, sino un narrador de historias. Cada vez que ocurre algo importante, algo que cambia el estado del negocio, tu aplicación "anuncia" esa noticia. Esas "noticias" son los **Eventos de Dominio**.

**¿Qué es un Evento de Dominio, en cristiano?**

Piensa en ello como una notificación de un hecho consumado, algo relevante para el negocio que acaba de suceder.

  * Se nombran en **tiempo pasado**: `ProductoCreado`, `PedidoConfirmado`, `StockAjustado`.
  * Son **inmutables**: Lo que pasó, pasó. No se puede cambiar el evento.
  * Llevan **información útil**: Los datos necesarios para entender qué sucedió (ej. el ID del producto, la nueva cantidad de stock, la fecha).

**Gráfico Conceptual 1: El Flujo Básico de un Evento**

```
+-------------------+      +---------------------+      +----------------------+
| Algo Significativo|----->| Se genera un EVENTO |----->| Alguien (o algo)     |
| Sucede en el      |      | (La "Noticia")      |      | Reacciona al Evento  |
| Dominio           |      +---------------------+      +----------------------+
+-------------------+
  (Ej: Se crea un
   nuevo producto)
```

**Pausa para Reflexionar (¡Tu Turno\!):** interactivity

> En tu aplicación FastAPI (quizás la tienda online que estamos imaginando), si un usuario finaliza una compra:
>
>   * ¿Qué "noticia importante" (Evento de Dominio) crees que se generaría inmediatamente después de que el pago se confirma y el pedido se marca como "Pagado"?
>   * ¿Qué información mínima necesitaría llevar ese evento?

-----

### La Anatomía de un Evento: ¿Qué lleva la "Noticia"?

Para que un evento sea útil, debe contener la información esencial sobre lo que ocurrió. Siguiendo con Python, una `dataclass` (idealmente `frozen=True` para inmutabilidad) o un modelo Pydantic son perfectos para esto.

**Ejemplo: `ProductoRegistradoEvento`**

```python
from dataclasses import dataclass, field
from uuid import UUID, uuid4
from datetime import datetime, timezone

# Una clase base opcional para metadatos comunes a todos los eventos
@dataclass(frozen=True)
class EventoBase:
    id_evento: UUID = field(default_factory=uuid4)
    fecha_ocurrencia: datetime = field(default_factory=lambda: datetime.now(timezone.utc))

@dataclass(frozen=True)
class ProductoRegistradoEvento(EventoBase):
    id_producto: UUID
    nombre_producto: str
    precio_inicial: float
    stock_inicial: int
    categoria_id: UUID | None # Podríamos añadir más detalles relevantes
```
**Gráfico Conceptual 2: Estructura de nuestro `ProductoRegistradoEvento`**

```
+----------------------------------+
| ProductoRegistradoEvento         |
+----------------------------------+
| - id_evento: UUID                |  <-- Heredado de EventoBase
| - fecha_ocurrencia: datetime     |  <-- Heredado de EventoBase
|----------------------------------|
| - id_producto: UUID              |  <-- Datos específicos del evento
| - nombre_producto: str           |
| - precio_inicial: float          |
| - stock_inicial: int             |
| - categoria_id: UUID | None     |
+----------------------------------+
```

**¡Manos a la Obra\! (Pequeño Reto):** interactivity

> Si tuvieras un evento `UsuarioCambioEmailEvento`, ¿qué campos cruciales debería tener, además de los heredados de `EventoBase`? Piénsalo un momento.
>
> *Pista: Necesitarás saber de quién es el email, cuál era el antiguo (quizás) y cuál es el nuevo.*

-----

### ¿Quién Grita "¡Noticia\!"? - Originando Eventos desde el Corazón del Dominio

Los eventos no surgen de la nada. Nacen en el momento en que una Entidad (especialmente una Raíz de Agregado) o un Servicio de Dominio realiza una acción que cambia el estado de una manera significativa.

**El Proceso dentro del Agregado:**

1.  Un Agregado (ej. `Producto`) recibe una orden (ej. `ajustar_stock()`).
2.  Realiza su lógica interna y valida las reglas de negocio.
3.  Cambia su propio estado (ej. actualiza la cantidad de `self.stock`).
4.  **Y aquí está la magia:** El Agregado crea una instancia del evento correspondiente (ej. `StockProductoAjustadoEvento`) y lo *registra* en una lista interna de "eventos pendientes".

**Gráfico Conceptual 3: El Agregado Colecciona sus Propias Noticias**

```
                      Comando: "Ajustar stock en -5 unidades"
                                     |
                                     v
+------------------------------------------------------------------+
| Agregado: Producto (ID: XYZ, Stock Actual: 20)                   |
|------------------------------------------------------------------|
| metodo: ajustar_stock(cantidad: -5, motivo: "Venta ABC")         |
|   1. Validar (stock - 5 >= 0) -> OK                              |
|   2. self.stock = 15                                             |
|   3. evento = StockProductoAjustadoEvento(id_producto=XYZ,        |
|                                       nuevo_stock=15,        |
|                                       cambio=-5,             |
|                                       motivo="Venta ABC")     |
|   4. self._eventos_pendientes.append(evento)                     |  <-- ¡Aquí!
|                                                                  |
+------------------------------------------------------------------+
      |
      v
  (El estado del producto ha cambiado Y un evento ha sido registrado internamente)
```

**Punto Clave para Debatir:** interactivity

> El Agregado *colecciona* sus eventos, pero **no los envía directamente** al exterior (no llama a un servicio de email, no publica en Kafka). ¿Por qué crees que se hace esta separación?
>
> *Pista: Piensa en la responsabilidad única del Agregado y en la consistencia transaccional (¿qué pasa si guardar en la BBDD falla después de enviar el email?).*

-----

### El Cartero de Eventos: Despachando las Noticias

Si el Agregado solo guarda las "noticias" en su mochila, ¿quién se encarga de repartirlas? ¡El **Servicio de Aplicación** (o Caso de Uso)\!

**El Flujo Típico con el Servicio de Aplicación:**

1.  El Servicio de Aplicación recibe un comando (ej. desde un controlador FastAPI).
2.  Utiliza un Repositorio (Puerto de Salida) para cargar el Agregado necesario.
3.  Llama al método de negocio del Agregado (ej. `producto.ajustar_stock(...)`). *Este es el momento en que el Agregado registra internamente sus eventos.*
4.  Utiliza el Repositorio para persistir los cambios en el Agregado. ¡Este paso es crucial que ocurra **antes** de despachar los eventos\!
5.  **¡Ahora sí\!** El Servicio de Aplicación le pide al Agregado su lista de `_eventos_pendientes`.
6.  Entrega estos eventos a un componente llamado **Despachador de Eventos** (Event Dispatcher).

**Gráfico Conceptual 4: El Viaje del Evento desde el Agregado hasta el Despachador**

```
                                     +-------------------------+
                                     |     FastAPI Controller  |
                                     +------------+------------+
                                                  | (1. Comando)
                                                  v
+---------------------------------------------------------------------------------------------+
| Servicio de Aplicación (ej. `ServicioInventario.ajustar_stock_producto`)                    |
|---------------------------------------------------------------------------------------------|
| 1. producto = self.repo_productos.obtener_por_id(id_producto)  <---- (2. Carga Agregado) ---+---> Repositorio
| 2. producto.ajustar_stock(cantidad, motivo)  <------------------- (3. Lógica de Dominio, Agregado registra evento)
| 3. self.repo_productos.guardar(producto) <----------------------- (4. Persiste Agregado) ---+---> Repositorio
| 4. eventos_ocurridos = producto.obtener_eventos_pendientes()                                |
| 5. self.despachador_eventos.despachar_todos(eventos_ocurridos) ---- (5. Despacha!) ---------> Despachador
+---------------------------------------------------------------------------------------------+  de Eventos
```

**El Despachador de Eventos (Event Dispatcher / Mediator):**

Este es un componente (puede ser una simple clase en tu capa de aplicación) cuya responsabilidad es:

  * Recibir eventos.
  * Saber quiénes están interesados en cada tipo de evento (los "Manejadores" o "Subscriptores").
  * Entregar cada evento a todos sus manejadores correspondientes.

**Pregunta para la Audiencia (¡Tú\!):** interactivity

> Siguiendo el Gráfico 4, ¿qué problemas podríamos tener si el paso `self.repo_productos.guardar(producto)` falla por alguna razón (ej. la base de datos está caída), pero *antes* ya hubiéramos intentado despachar los eventos? ¿Cómo ayuda el orden mostrado a la consistencia?

-----

### Los Oídos Atentos: Manejadores de Eventos (Event Handlers)

Cuando el "cartero" (Despachador) entrega la "noticia" (Evento), alguien tiene que estar escuchando. Esos son los **Manejadores de Eventos** (Event Handlers o Subscribers).

Un Manejador es una pieza de código que:

  * Se suscribe a uno o más tipos de eventos.
  * Ejecuta una lógica específica cuando recibe un evento del tipo al que está suscrito.

**Gráfico Conceptual 5: El Despachador y sus Manejadores**

```
+-----------------------+
| Despachador de Eventos|
+-----------+-----------+
            |
            +---- Evento: ProductoRegistradoEvento ----+
            |                                          |
            v                                          v
+-------------------------+              +-----------------------------+
| ManejadorA:             |              | ManejadorB:                 |
| NotificarAdminSistema   |              | ActualizarCacheDeProductos  |
| (Usa INotificadorPort)  |              | (Usa ICachePort)            |
+-------------------------+              +-----------------------------+

            +---- Evento: PedidoConfirmadoEvento ------+
            |                                          |
            v                                          v
+-------------------------+              +-----------------------------+
| ManejadorC:             |              | ManejadorD:                 |
| EnviarEmailConfirmacion |              | ReservarStockEnInventario   |
| (Usa IEmailPort)        |              | (Usa IServicioInventario)   |
+-------------------------+              +-----------------------------+
```

**¿Dónde viven estos Manejadores?**

  * **En la Capa de Aplicación:** Si la reacción al evento es una lógica que pertenece al mismo sistema/contexto delimitado.
      * Ejemplo: Cuando se crea un `ProductoRegistradoEvento`, un manejador podría enviar un email de notificación al administrador (usando un puerto de salida `INotificadorPort`).
      * Otro manejador podría actualizar una caché de productos populares.
  * **En la Capa de Infraestructura (como Adaptadores):** Si la reacción implica comunicarse con un sistema externo.
      * Ejemplo: Cuando ocurre un `PedidoPagadoEvento`, un manejador (que es un adaptador) podría publicar este evento en un bus de mensajes como Kafka o RabbitMQ para que otros microservicios lo consuman.

**Ejemplo Interactivo de Diseño:** interactivity

> Para nuestro `ProductoRegistradoEvento`, imaginemos que queremos dos reacciones:
>
> 1.  **Notificar al equipo de marketing** para que preparen material promocional.
> 2.  **Añadir el producto a un índice de búsqueda** para que los clientes lo encuentren.
>
> ¿Cómo llamarías a estos dos manejadores? ¿Qué puertos de salida (abstracciones de infraestructura) necesitaría cada uno aproximadamente?

**Un Vistazo a un Manejador (Python Conceptual):**

# En aplicacion/manejadores_eventos/producto_handlers.py
from dominio.eventos.producto_eventos import ProductoRegistradoEvento # El evento que nos interesa
from aplicacion.puertos.salida.inotificador import INotificador # Puerto para enviar emails/slack, etc.

```python
class NotificarMarketingSobreNuevoProductoHandler:
    def __init__(self, notificador: INotificador, destinatario_marketing: str):
        self._notificador = notificador
        self._destinatario_marketing = destinatario_marketing

    async def __call__(self, evento: ProductoRegistradoEvento) -> None:
        # El despachador llamará a este método cuando ocurra un ProductoRegistradoEvento
        mensaje = (
            f"¡Atención Marketing! Nuevo producto para promocionar:\n"
            f"ID: {evento.id_producto}\n"
            f"Nombre: {evento.nombre_producto}\n"
            f"Precio: {evento.precio_inicial}"
        )
        print(f"[HANDLER DEBUG] Preparando notificación para marketing: {evento.nombre_producto}")
        await self._notificador.enviar_notificacion(
            destinatario=self._destinatario_marketing,
            mensaje=mensaje,
            asunto=f"Nuevo Producto Disponible: {evento.nombre_producto}"
        )
```

*(Este manejador se "registraría" en el Despachador de Eventos durante la inicialización de la aplicación).*

-----

### ¿Por Qué Tanta "Ceremonia" con los Eventos? ¡Los Beneficios\!

Usar eventos de dominio puede parecer que añade más piezas al puzzle, ¡pero los beneficios suelen merecer la pena\!

1.  **DESACOPLAMIENTO MÁXIMO:** Esta es la joya de la corona.

      * El código que registra un producto **no necesita saber** quién o cuántos están interesados en ese hecho. No llama directamente al servicio de email, ni al de caché, ni al de búsqueda.
      * ¡Imagina que mañana quieres enviar un SMS además de un email\! El código original del registro de producto **no se toca**. Solo añades un nuevo `ManejadorEnviarSmsNuevoProducto`.

2.  **MAYOR FLEXIBILIDAD Y EXTENSIBILIDAD:**

      * Añadir nuevas reacciones a eventos existentes es fácil: creas un nuevo manejador y lo registras. ¡Listo\!
      * El sistema puede crecer orgánicamente añadiendo comportamientos sin modificar el código central probado.

3.  **TRAZABILIDAD Y AUDITORÍA:**

      * Los eventos son un registro natural de las cosas importantes que han sucedido. Puedes guardarlos para auditoría, análisis de negocio o depuración.

4.  **COMUNICACIÓN ENTRE MÓDULOS O MICROSERVICIOS:**

      * Los eventos son una forma excelente de comunicar cambios entre diferentes partes de un sistema grande o entre microservicios (generalmente a través de un bus de mensajes).

**Momento de Revelación:** interactivity

> Piensa en una funcionalidad que hayas implementado recientemente o que tengas en mente para tu proyecto. Si la hubieras diseñado con eventos de dominio, ¿qué parte del código se habría simplificado o habría sido más fácil de extender después?

-----

### Algunas Consideraciones Avanzadas (Para los Curiosos)

A medida que te adentras en los eventos de dominio, surgen algunos temas más profundos:

  * **Atomicidad (Consistencia Fuerte):** ¿Cómo te aseguras de que el cambio de estado del Agregado Y la publicación (o al menos el encolamiento seguro) de sus eventos ocurren como una unidad atómica (todo o nada)?
      * **Patrón Outbox:** Una técnica común es guardar los eventos en una tabla especial de la misma base de datos que tus agregados, dentro de la misma transacción. Un proceso separado luego lee de esta "bandeja de salida" y los publica de forma fiable.
  * **Manejo Síncrono vs. Asíncrono:**
      * **Síncrono:** El servicio de aplicación espera a que todos los manejadores (in-process) terminen antes de devolver la respuesta al cliente. Simple, pero puede afectar la latencia si los manejadores son lentos.
      * **Asíncrono:** El servicio de aplicación despacha el evento y continúa. Los manejadores se ejecutan en segundo plano (ej. en un `asyncio.Task`, o encolados para un worker). Mejor para la latencia, pero añade complejidad (manejo de errores, reintentos).
  * **Idempotencia de los Manejadores:** Especialmente en sistemas distribuidos o con manejo asíncrono, un evento podría ser entregado a un manejador más de una vez. El manejador debería ser **idempotente**, es decir, procesar el mismo evento múltiples veces debería tener el mismo resultado final que procesarlo una sola vez.

-----

### ¡Tu Turno de Nuevo\! Integrando el Conocimiento interactivity

Imagina el siguiente escenario en tu aplicación: **Un usuario cancela su suscripción (`SuscripcionCanceladaEvento`).**

1.  **¿Quién origina este evento?** (¿Una entidad `Suscripcion`? ¿Un servicio de aplicación directamente?)
2.  **¿Qué información llevaría `SuscripcionCanceladaEvento`?**
3.  **¿Qué manejadores podrían estar interesados en este evento?** Piensa en al menos dos.
      * *Ejemplo 1:* Enviar un email de "Lamentamos verte ir" con una encuesta.
      * *Ejemplo 2:* Revocar el acceso del usuario a funcionalidades premium.
      * *Ejemplo 3:* Notificar al equipo de retención de clientes.
4.  **Para cada manejador que pensaste, ¿sería parte de la capa de aplicación o un adaptador de infraestructura? ¿Qué puertos de salida necesitaría?**

Intenta dibujar (mentalmente o en papel) el flujo completo, desde que el usuario hace clic en "Cancelar Suscripción" en la interfaz, pasando por el adaptador de entrada, el servicio de aplicación, la entidad, el despachador de eventos, y finalmente llegando a los manejadores.

-----

¡Y eso es un vistazo dinámico a los eventos de dominio\! Son una pieza clave para construir aplicaciones que no solo funcionan, sino que también son adaptables, resilientes y cuentan la historia de tu negocio de manera efectiva.
---

## 6.8 Implementar casos de uso en la capa de aplicación

Ya hemos hablado de los puertos de entrada (nuestras APIs del núcleo) y de cómo los adaptadores HTTP (como los controladores FastAPI) los invocan. Pero, ¿quién implementa realmente la lógica detrás de estos puertos de entrada? ¡Bienvenidos a la **Capa de Aplicación** y a sus protagonistas: los **Servicios de Aplicación** o **Manejadores de Casos de Uso**\!

Si la Capa de Dominio es el cerebro con las reglas de negocio puras, la Capa de Aplicación es como el director de orquesta. No toca los instrumentos (esa es tarea del dominio y de los adaptadores de infraestructura), pero se asegura de que todos trabajen juntos en armonía para producir una melodía coherente: el cumplimiento de un caso de uso específico del usuario.

### ¿Qué es un Servicio de Aplicación (o Caso de Uso)?

Un Servicio de Aplicación:

1.  **Implementa un Puerto de Entrada:** Es la materialización concreta de una interfaz de puerto de entrada. Si el puerto define `async def crear_pedido(self, datos_pedido: PedidoDTO) -> ConfirmacionPedidoDTO:`, el servicio de aplicación tendrá un método con esa firma.
2.  **Orquesta el Flujo:** Coordina los diferentes componentes para realizar una tarea. Esto implica:
      * Recibir datos de entrada (generalmente DTOs o Comandos).
      * Utilizar Puertos de Salida (como Repositorios) para obtener o persistir Entidades del Dominio.
      * Invocar lógica en las Entidades del Dominio o en Servicios de Dominio.
      * Utilizar otros Puertos de Salida para efectos colaterales (ej. enviar notificaciones, publicar eventos).
      * Despachar Eventos de Dominio que hayan sido generados por los agregados.
      * Devolver un resultado (generalmente un DTO) o nada si es un comando puro.
3.  **Es Cliente del Dominio y de los Puertos de Salida:** Depende de las abstracciones del dominio y de las interfaces de los puertos de salida, no de implementaciones concretas de infraestructura.
4.  **Mantiene la Lógica de Aplicación, no de Negocio Pura:** La lógica de negocio fundamental (reglas que siempre aplican a una entidad, invariantes) debe estar en el Dominio. El servicio de aplicación contiene la lógica de *flujo de trabajo* o *coordinación* específica de un caso de uso. Intenta mantenerlos delgados ("thin services").

### Responsabilidades Clave de un Servicio de Aplicación

Vamos a desglosar lo que hace un servicio de aplicación típico:

1.  **Recepción de Datos de Entrada:** A través de los métodos definidos por el puerto de entrada que implementa, recibe DTOs (Data Transfer Objects) o Comandos.

      * *Ejemplo:* `async def registrar_nuevo_producto(self, datos_producto: DatosNuevoProductoDTO) -> ProductoCreadoDTO:`

2.  **Validación a Nivel de Aplicación (Opcional):** Aunque la validación de formato de los DTOs la puede hacer el adaptador (FastAPI con Pydantic es genial para esto), el servicio de aplicación podría realizar validaciones más específicas del caso de uso que no encajan ni en el dominio ni en el adaptador. Por ejemplo, verificar permisos si no se delega a un middleware, o una regla que cruce múltiples agregados si es parte del flujo del caso de uso (con cuidado).

3.  **Recuperación de Objetos de Dominio:** Usa las interfaces de los Repositorios (Puertos de Salida) para cargar las Entidades o Raíces de Agregados necesarios desde la persistencia.

      * *Ejemplo:* `producto_existente = await self.repositorio_productos.obtener_por_id(id_producto)`

4.  **Orquestación de la Lógica de Dominio:** Llama a métodos en las Entidades, Agregados o Servicios de Dominio para ejecutar las reglas de negocio.

      * *Ejemplo:* `producto_existente.ajustar_stock(cantidad, motivo_ajuste)`
      * `resultado_calculo = self.servicio_calculo_impuestos_dominio.calcular_para(pedido)`

5.  **Persistencia de Cambios:** Después de que los objetos de dominio han sido modificados, utiliza los Repositorios para guardar estos cambios.

      * *Ejemplo:* `await self.repositorio_productos.guardar(producto_modificado)`

6.  **Coordinación de Otros Puertos de Salida:** Si el caso de uso requiere enviar una notificación, interactuar con un sistema externo, etc., el servicio de aplicación utilizará la interfaz del puerto de salida correspondiente.

      * *Ejemplo:* (Aunque a menudo esto se delega a manejadores de eventos) `await self.notificador.enviar_confirmacion_pedido(datos_confirmacion)`

7.  **Gestión de Transacciones (Conceptual):** El servicio de aplicación es, conceptualmente, el límite de una transacción. Todas las operaciones dentro de un método de caso de uso (leer, modificar dominio, guardar, despachar eventos in-process) deberían idealmente ejecutarse como una unidad atómica (o todo tiene éxito, o todo se revierte). La implementación técnica de esto (ej. un decorador de transacción, middleware) puede estar fuera del propio servicio, pero el servicio define el alcance.

8.  **Despacho de Eventos de Dominio:** Después de persistir los cambios (¡importante el orden\!), obtiene los eventos de dominio acumulados por los agregados y los pasa a un Despachador de Eventos.

      * *Ejemplo:* `eventos = producto_modificado.obtener_eventos_pendientes()`
      * `await self.despachador_eventos.despachar_multiples(eventos)`

9.  **Devolución de Resultados:** Retorna un DTO con la información de salida relevante para el invocador, o `None` si la operación no produce datos de respuesta.

      * *Ejemplo:* `return ProductoActualizadoDTO(id=producto_modificado.id, stock_nuevo=producto_modificado.stock)`

### Estructura Típica de un Servicio de Aplicación

Generalmente, se implementan como clases.

  * **Constructor:** Recibe sus dependencias (implementaciones de puertos de salida, despachador de eventos) a través de inyección de dependencias. Estas dependencias son siempre **interfaces**, no clases concretas de infraestructura.
  * **Métodos:** Implementan las operaciones definidas en una o más interfaces de puerto de entrada. Cada método representa un caso de uso o una acción específica.
  * **Asincronía:** Si los puertos de salida (repositorios, etc.) son asíncronos (común en Python con I/O), los métodos del servicio de aplicación también serán `async`.

### Ejemplo de Implementación (Continuando con `ServicioGestionInventario`)

Vamos a expandir nuestro `ServicioGestionInventario` para ilustrar estos puntos. Asumiremos que tenemos las interfaces `IRepositorioProductos`, `INotificador`, `IDespachadorEventos` y los DTOs y Eventos definidos en secciones anteriores.

```python
# En aplicacion/servicios/servicio_gestion_inventario.py

from uuid import UUID, uuid4

# Entidades y Eventos del Dominio
from dominio.modelos.producto import Producto
# (Se asume que Producto.py ahora puede registrar ProductoCreadoEvento y StockProductoActualizadoEvento)
# from dominio.eventos.producto_eventos import ProductoCreadoEvento, StockProductoActualizadoEvento

# Puertos de Entrada y Salida, DTOs
from aplicacion.puertos.entrada.igestion_inventario_input_port import IGestionInventarioInputPort
from aplicacion.puertos.salida.irepositorio_productos import IRepositorioProductos
# from aplicacion.puertos.salida.inotificador import INotificador # Podría usarse aquí o vía manejador de evento
from aplicacion.despachadores.despachador_eventos_simple import DespachadorEventosSimple # O la interfaz IDespachadorEventos
from aplicacion.dtos import (
    DatosNuevoProductoDTO, ProductoCreadoDTO, 
    DatosAjusteStockDTO, ProductoStockActualizadoDTO # Nuevos DTOs para el ejemplo
)

# Excepciones personalizadas de la capa de aplicación
class ProductoNoEncontradoError(Exception):
    def __init__(self, id_producto: UUID):
        self.id_producto = id_producto
        super().__init__(f"Producto con ID '{id_producto}' no encontrado.")

class NombreProductoDuplicadoError(Exception):
    def __init__(self, nombre: str):
        self.nombre = nombre
        super().__init__(f"Ya existe un producto con el nombre '{nombre}'.")

class StockInsuficienteError(ValueError): # Hereda de ValueError para semántica
    pass


class ServicioGestionInventario(IGestionInventarioInputPort):
    def __init__(
        self,
        repositorio_productos: IRepositorioProductos,
        despachador_eventos: DespachadorEventosSimple # Idealmente una interfaz IDespachadorEventos
        # notificador: INotificador # Opcional, si se notifica directamente y no vía evento
    ):
        self.repositorio_productos = repositorio_productos
        self.despachador_eventos = despachador_eventos
        # self.notificador = notificador

    async def registrar_nuevo_producto(self, datos_producto: DatosNuevoProductoDTO) -> ProductoCreadoDTO:
        # 1. Validación a nivel de aplicación (ejemplo: unicidad del nombre)
        #    (Esta lógica podría estar también en un Servicio de Dominio si es una regla de negocio invariante)
        productos_existentes = await self.repositorio_productos.buscar_por_nombre(datos_producto.nombre)
        if any(p.nombre.lower() == datos_producto.nombre.lower() for p in productos_existentes):
            raise NombreProductoDuplicadoError(nombre=datos_producto.nombre)

        # 2. Orquestación de lógica de dominio (creación de la entidad)
        #    La entidad Producto se encarga de sus validaciones internas (precio > 0, stock >= 0)
        #    y de registrar el ProductoCreadoEvento internamente.
        try:
            nuevo_producto = Producto(
                nombre=datos_producto.nombre,
                descripcion=datos_producto.descripcion,
                precio=datos_producto.precio,
                stock=datos_producto.stock_inicial
                # El ID se autogenera en la entidad Producto si no se pasa
            )
        except ValueError as e: # Capturar errores de validación de la entidad
            raise # O envolverla en una excepción de aplicación si se quiere más control

        # 3. Persistencia de cambios
        await self.repositorio_productos.guardar(nuevo_producto)

        # 4. Despacho de Eventos de Dominio
        eventos_dominio = nuevo_producto.obtener_eventos_pendientes()
        await self.despachador_eventos.despachar_multiples(eventos_dominio)
        
        # 5. Devolución de resultado (DTO)
        #    Podríamos añadir más lógica aquí si el DTO de salida necesita más info que la del evento
        return ProductoCreadoDTO(
            id_producto=nuevo_producto.id,
            nombre=nuevo_producto.nombre,
            # mensaje_bienvenida se podría formar aquí o ser añadido por un manejador de evento
            mensaje_bienvenida=f"Producto '{nuevo_producto.nombre}' registrado exitosamente."
        )

    async def ajustar_stock_producto(self, datos_ajuste: DatosAjusteStockDTO) -> ProductoStockActualizadoDTO:
        # 1. Recuperación del objeto de dominio
        producto = await self.repositorio_productos.obtener_por_id(datos_ajuste.id_producto)
        if not producto:
            raise ProductoNoEncontradoError(id_producto=datos_ajuste.id_producto)

        # 2. Orquestación de lógica de dominio
        #    El método 'ajustar_stock' de la entidad Producto valida y registra el evento.
        id_operacion_ficticio = uuid4() # En un caso real, esto vendría del comando o contexto
        try:
            producto.ajustar_stock(datos_ajuste.cantidad_ajuste, id_operacion=id_operacion_ficticio)
        except ValueError as e: # Capturar error de stock insuficiente de la entidad
            raise StockInsuficienteError(str(e))


        # 3. Persistencia de cambios
        await self.repositorio_productos.guardar(producto)

        # 4. Despacho de Eventos de Dominio
        eventos_dominio = producto.obtener_eventos_pendientes()
        await self.despachador_eventos.despachar_multiples(eventos_dominio)
        
        # 5. Devolución de resultado (DTO)
        return ProductoStockActualizadoDTO(
            id_producto=producto.id,
            nombre_producto=producto.nombre,
            stock_actual=producto.stock,
            ajuste_realizado=datos_ajuste.cantidad_ajuste,
            mensaje="Stock actualizado correctamente."
        )

# --- DTOs adicionales para el ejemplo de ajustar_stock_producto ---
# (Normalmente en aplicacion/dtos.py)
# from pydantic import BaseModel
# from uuid import UUID

# class DatosAjusteStockDTO(BaseModel):
#     id_producto: UUID
#     cantidad_ajuste: int # Positivo para añadir, negativo para quitar
#     motivo: str | None = None

# class ProductoStockActualizadoDTO(BaseModel):
#     id_producto: UUID
#     nombre_producto: str
#     stock_actual: int
#     ajuste_realizado: int
#     mensaje: str
```

### Principios Clave para Implementar Servicios de Aplicación

  * **Servicios Delgados, Dominio Rico:** La mayor parte de las reglas de negocio complejas deben residir en el Dominio (Entidades, VOs, Servicios de Dominio). Los servicios de aplicación actúan como coordinadores.
  * **Depender de Abstracciones:** Siempre depender de las interfaces de los puertos (entrada y salida), no de implementaciones concretas. Esto es clave para la testeabilidad y flexibilidad.
  * **Una Responsabilidad (por método):** Cada método público del servicio de aplicación debería idealmente corresponder a un único caso de uso o una acción atómica del usuario/sistema.
  * **Sin Lógica de Presentación:** Los servicios de aplicación no deben saber nada sobre HTTP, JSON, HTML, etc. Reciben y devuelven DTOs o tipos de datos simples/del dominio. La "traducción" a formatos de presentación es tarea de los adaptadores de entrada.
  * **Manejo de Errores Específico:** Pueden capturar excepciones del dominio y, si es necesario, envolverlas en excepciones más específicas de la capa de aplicación para que los adaptadores las interpreten y conviertan en respuestas adecuadas (ej. códigos de estado HTTP).

### ¿Cómo Encaja Todo?

```
+-------------------+      +---------------------+      +-------------------------+
| Adaptador de      | --1--> | Puerto de Entrada   | --2--> | Servicio de Aplicación  |
| Entrada (FastAPI) |      | (Interfaz)          |      | (Implementa Puerto Ent.)|
+-------------------+      +---------------------+      +------------+------------+
                                                                     | 3. Usa
                                                                     v
                                                 +-------------------+   +----------------------+
                                                 | Dominio (Entidades,|   | Puertos de Salida    |
                                                 | Agregados, S.Dom) |   | (Interfaces Repo,   |
                                                 +-------------------+   | Notificador, etc.)   |
                                                      ^       ^          +----------------------+
                                                      |       | 4. Usa             | 5. Implementado por
                                                      +-------+--------------------+
                                                               (para persistir,    (en Infraestructura)
                                                                notificar, etc.)
```

1.  El Adaptador de Entrada (ej. controlador FastAPI) recibe una petición, la valida y convierte en un DTO.
2.  Llama al método correspondiente del Servicio de Aplicación (que implementa el Puerto de Entrada), pasándole el DTO.
3.  El Servicio de Aplicación utiliza objetos del Dominio y...
4.  ...Puertos de Salida (interfaces) para realizar el trabajo.
5.  Las implementaciones concretas de los Puertos de Salida están en la Infraestructura, pero el Servicio de Aplicación no las conoce directamente.

Implementar correctamente los casos de uso en la capa de aplicación es fundamental para tener un sistema bien estructurado, fácil de probar, mantener y evolucionar, donde la lógica de negocio está bien protegida y orquestada de manera clara.
---

## 6.9 Configurar inyecciones de dependencia de adaptadores externos

**Imaginemos nuestro hexágono...**

Recordemos rápidamente:

  * **Núcleo (Dominio y Aplicación):** Lógica pura, reglas de negocio, casos de uso. ¡El corazón de nuestra aplicación\!
  * **Exterior (Infraestructura):** Todo lo que interactúa con el mundo exterior: bases de datos, APIs de terceros, sistemas de mensajería, y también ¡nuestra API HTTP con FastAPI\!

Los **adaptadores externos** son esos componentes en la capa de infraestructura que implementan los **puertos** definidos por nuestro núcleo para comunicarse con herramientas específicas (la base de datos PostgreSQL, una API de pagos, un servicio de envío de correos, etc.).

**(Piensa 🤔):** Si nuestro `ServicioDePedidos` (en la capa de aplicación) necesita guardar un pedido, ¿debería saber si se guarda en PostgreSQL, MongoDB o un archivo CSV? ¡Claro que no\! Solo debe conocer una interfaz, un "contrato" (el puerto `RepositorioDePedidos`).

Ahí es donde la **Inyección de Dependencias (DI)** se convierte en nuestra mejor aliada.

-----

### **Acto 1: ¿Por Qué Inyectar Dependencias de Adaptadores Externos? El Poder del "Desacople Maestro"**

La Inyección de Dependencias es un patrón de diseño en el que un objeto recibe sus dependencias (otros objetos con los que trabaja) desde una fuente externa, en lugar de crearlas internamente.

**Analogía Dinámica: ¡El Chef Estrella y sus Utensilios\!** 🍳🔪

  * **Chef Estrella (Nuestro Caso de Uso / Servicio de Aplicación):** Sabe la receta (lógica de negocio) a la perfección. Necesita ciertos utensilios para cocinar (ej: una sartén antiadherente, un horno de convección).
  * **Utensilios Específicos (Adaptadores Externos):** La sartén "SuperCook 3000" (nuestro `AdaptadorPostgreSQL`) o el horno "MegaBake Pro" (nuestro `AdaptadorServicioEmail`).
  * **Ayudante de Cocina Inteligente (Sistema de Inyección de Dependencias de FastAPI):** En lugar de que el Chef vaya a la tienda a comprar o construya cada utensilio específico (acoplamiento fuerte), el Ayudante le proporciona el utensilio exacto que cumple con la especificación ("necesito algo para saltear", "necesito algo para hornear a 200°C") justo cuando lo necesita.

**Ventajas Clave para Nuestros Adaptadores Externos:**

1.  **Desacoplamiento PURO:**

      * Nuestro núcleo (Dominio/Aplicación) solo depende de abstracciones (interfaces/puertos). ¡No sabe nada de PostgreSQL, Kafka o AWS S3\!
      * Los adaptadores externos dependen del núcleo (para implementar los puertos), pero el núcleo NO depende de los adaptadores. ¡Esta es la inversión de dependencia en acción\!

2.  **Testabilidad Suprema:**

      * ¿Queremos probar un servicio de aplicación sin tocar la base de datos real? ¡Fácil\! Inyectamos un adaptador "falso" (un mock o un `InMemoryRepository`) que cumpla el mismo contrato (puerto). ¡Tests unitarios y de integración más rápidos y fiables\!

3.  **Flexibilidad y Mantenibilidad:**

      * ¿Migramos de PostgreSQL a MySQL? ¿Cambiamos de proveedor de servicio de email?
          * Creamos un nuevo adaptador (`AdaptadorMySQL`, `AdaptadorSendgrid`).
          * Cambiamos la "configuración" de la inyección de dependencias para que ahora proporcione el nuevo adaptador.
          * ¡El núcleo de la aplicación ni se entera\! ✨

-----

### **Acto 2: FastAPI al Rescate con `Depends` - ¡Tu Ayudante de Cocina Personal\!**

Ya conocéis `Depends` de FastAPI. Es ese sistema elegante y potente que nos permite declarar dependencias para nuestros path operations, y FastAPI se encarga de resolverlas y proporcionarlas.

```python
# src/domain/ports/user_repository_port.py
from abc import ABC, abstractmethod
from typing import Optional
from src.domain.entities.user import User # Asumiendo una entidad User

class UserRepositoryPort(ABC):
    @abstractmethod
    async def get_by_id(self, user_id: str) -> Optional[User]:
        raise NotImplementedError

    @abstractmethod
    async def save(self, user: User) -> User:
        raise NotImplementedError
```

**(Pregunta para vosotros 🤔):** ¿Por qué `ABC` y `@abstractmethod` son tan importantes aquí para la Arquitectura Hexagonal? *(Respuesta: Definen el contrato estricto que cualquier adaptador DEBE cumplir, garantizando la intercambiabilidad).*

**Paso 2: El Adaptador (Implementación Concreta) en Infraestructura (¡También\!)**

Este adaptador necesitará, por ejemplo, una sesión de base de datos para funcionar.

```python
# src/infrastructure/adapters/postgres_user_repository_adapter.py
from typing import Optional
from sqlalchemy.ext.asyncio import AsyncSession # Ejemplo con SQLAlchemy
from src.domain.entities.user import User
from src.domain.ports.user_repository_port import UserRepositoryPort

class PostgresUserRepositoryAdapter(UserRepositoryPort):
    def __init__(self, db_session: AsyncSession): # ¡Dependencia de una sesión de BD!
        self._db_session = db_session

    async def get_by_id(self, user_id: str) -> Optional[User]:
        # Lógica para buscar en PostgreSQL con self._db_session
        # ...ejemplo: user_db = await self._db_session.get(UserModel, user_id)
        print(f"ADAPTADOR POSTGRES: Buscando usuario {user_id}")
        if user_id == "1": # Simulación
            return User(id="1", email="user@example.com", name="Usuario Real de BD")
        return None

    async def save(self, user: User) -> User:
        # Lógica para guardar/actualizar en PostgreSQL con self._db_session
        print(f"ADAPTADOR POSTGRES: Guardando usuario {user.name}")
        # ...ejemplo: self._db_session.add(user_model); await self._db_session.commit()
        return user
```
**Paso 3: El "Cableado" Mágico - Los Proveedores de Dependencias en Infraestructura** 🔌

Aquí es donde le decimos a FastAPI: "Oye, cuando alguien pida un `UserRepositoryPort`, quiero que le des una instancia de `PostgresUserRepositoryAdapter`, y para construirla, necesitarás una `AsyncSession`".

Primero, necesitamos un proveedor para la sesión de base de datos (esto es estándar en FastAPI con BBDD):

```python
# src/infrastructure/database/db_config.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "postgresql+asyncpg://user:password@host:port/db" # ¡Configuración!

async_engine = create_async_engine(DATABASE_URL, echo=True)
AsyncSessionFactory = sessionmaker(bind=async_engine, class_=AsyncSession, expire_on_commit=False)

async def get_db_session() -> AsyncSession: # Este es un proveedor de dependencia
    async with AsyncSessionFactory() as session:
        try:
            yield session
            await session.commit() # Opcional, depende de tu estrategia de commit
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
```

Ahora, el proveedor para nuestro repositorio:

```python
# src/infrastructure/dependencies/repository_providers.py
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession

from src.domain.ports.user_repository_port import UserRepositoryPort
from src.infrastructure.adapters.postgres_user_repository_adapter import PostgresUserRepositoryAdapter
from src.infrastructure.database.db_config import get_db_session # Importamos el proveedor de sesión

# ¡Este es el proveedor clave para nuestro adaptador!
def get_user_repository(
    db_session: AsyncSession = Depends(get_db_session) # FastAPI inyectará la sesión aquí
) -> UserRepositoryPort: # ¡Retorna el tipo del PUERTO!
    return PostgresUserRepositoryAdapter(db_session=db_session)
```

**¡Observad la belleza\!**

  * `get_user_repository` depende de `get_db_session`.
  * Retorna `UserRepositoryPort` (la abstracción), pero internamente instancia `PostgresUserRepositoryAdapter` (la concreción).

**Paso 4: Usando el Puerto Inyectado en un Caso de Uso (Capa de Aplicación)**

Nuestros servicios de aplicación deben depender de la abstracción (el puerto).


```python
# src/application/use_cases/user_service.py
from typing import Optional
from fastapi import Depends # Para inyectar el puerto en el servicio

from src.domain.entities.user import User
from src.domain.ports.user_repository_port import UserRepositoryPort
# ¡IMPORTANTE! El servicio de aplicación no debería importar directamente get_user_repository
# La inyección se configurará más arriba, al definir el servicio o al usarlo en un endpoint.

class UserService:
    def __init__(self, user_repo: UserRepositoryPort): # Depende de la ABSTRACCIÓN
        self._user_repo = user_repo

    async def find_user_by_id(self, user_id: str) -> Optional[User]:
        # Lógica del caso de uso...
        return await self._user_repo.get_by_id(user_id)

    async def register_new_user(self, name: str, email: str) -> User:
        # ... más lógica, validaciones ...
        new_user = User(id="generated_id", name=name, email=email) # La generación de ID podría ser otro servicio/puerto
        return await self._user_repo.save(new_user)

# ¿Cómo proporcionamos UserService con su dependencia?
# Opción A: El servicio se construye con su dependencia inyectada directamente en el router.
# Opción B (más encapsulada): Creamos un proveedor para el propio servicio.

# src/infrastructure/dependencies/service_providers.py
from src.application.use_cases.user_service import UserService
from src.domain.ports.user_repository_port import UserRepositoryPort
from .repository_providers import get_user_repository # El proveedor de repo que definimos antes

def get_user_service(
    user_repo: UserRepositoryPort = Depends(get_user_repository) # Inyecta la implementación del repo
) -> UserService:
    return UserService(user_repo=user_repo)
```

**Paso 5: Inyección Final en los Endpoints de FastAPI (Capa de Interfaces)**

Nuestros controladores/routers en la capa de interfaces pedirán el servicio de aplicación, y FastAPI, a través de los proveedores, ensamblará todo.

```python
# src/interfaces/http/user_controller.py
from fastapi import APIRouter, Depends, HTTPException
from typing import Optional

from src.application.use_cases.user_service import UserService
from src.domain.entities.user import User as UserEntity # Para el response_model
from src.infrastructure.dependencies.service_providers import get_user_service # Proveedor del servicio

router = APIRouter(prefix="/users", tags=["Users"])

@router.get("/{user_id}", response_model=Optional[UserEntity])
async def get_user_endpoint(
    user_id: str,
    user_service: UserService = Depends(get_user_service) # ¡Aquí ocurre la magia!
):
    user = await user_service.find_user_by_id(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

# ... otros endpoints ...
```
**Visualicemos el Flujo de Inyección para `get_user_endpoint`:**

1.  Llega una petición a `/users/{user_id}`.
2.  FastAPI ve que `get_user_endpoint` necesita `UserService` vía `Depends(get_user_service)`.
3.  Ejecuta `get_user_service()`:
      * Este a su vez necesita `UserRepositoryPort` vía `Depends(get_user_repository)`.
4.  Ejecuta `get_user_repository()`:
      * Este necesita `AsyncSession` vía `Depends(get_db_session)`.
5.  Ejecuta `get_db_session()`:
      * Crea y proporciona una `AsyncSession`.
6.  La `AsyncSession` se inyecta en `get_user_repository()`, que crea y retorna un `PostgresUserRepositoryAdapter`.
7.  El `PostgresUserRepositoryAdapter` se inyecta en `get_user_service()`, que crea y retorna un `UserService`.
8.  El `UserService` se inyecta en `get_user_endpoint`.
9.  ¡El endpoint se ejecuta con todas sus dependencias listas\!

**(¡Momento Didáctico\! 🤯):** ¿Veis cómo el `user_controller` y `user_service` son completamente ajenos a `Postgres` o `SQLAlchemy`? Solo conocen `UserRepositoryPort` y `UserService`. ¡Pura Arquitectura Hexagonal\!

-----

### **Acto 4: Configuraciones Avanzadas y Mejores Prácticas**

  * **Cambiando Implementaciones (ej. para Pruebas): `app.dependency_overrides`**

    Una de las joyas de FastAPI para la testabilidad en arquitecturas como esta. En tus tests (ej. con `pytest`), puedes sobreescribir un proveedor de dependencia por otro.

    Imagina un `InMemoryUserRepositoryAdapter` para no tocar la BD en los tests:


```python
# src/infrastructure/adapters/in_memory_user_repository_adapter.py (simplificado)
    from src.domain.ports.user_repository_port import UserRepositoryPort
    # ...
    class InMemoryUserRepositoryAdapter(UserRepositoryPort):
        _users: dict[str, User] = {}
        async def get_by_id(self, user_id: str) -> Optional[User]: return self._users.get(user_id)
        async def save(self, user: User) -> User: self._users[user.id] = user; return user
```
En tu `conftest.py` o fichero de setup de tests:

```python
# tests/conftest.py
    import pytest
    from fastapi.testclient import TestClient
    from src.main import app # Tu aplicación FastAPI
    from src.domain.ports.user_repository_port import UserRepositoryPort
    from src.infrastructure.adapters.in_memory_user_repository_adapter import InMemoryUserRepositoryAdapter
    from src.infrastructure.dependencies.repository_providers import get_user_repository # El proveedor original

    @pytest.fixture(scope="function")
    def client_with_in_memory_repo():
        # Proveedor "override"
        def get_override_in_memory_user_repository() -> UserRepositoryPort:
            return InMemoryUserRepositoryAdapter()

        # ¡La magia!
        app.dependency_overrides[get_user_repository] = get_override_in_memory_user_repository
        
        with TestClient(app) as c:
            yield c
        
        app.dependency_overrides.clear() # Limpiar para otros tests
```
Ahora, cualquier test que use el fixture `client_with_in_memory_repo` usará el repositorio en memoria sin cambiar una línea del código de aplicación o de los controladores.

  * **Organización de los Proveedores:**

      * Crear un directorio `src/infrastructure/dependencies/` es una buena práctica.
      * Dentro, puedes tener ficheros como `repository_providers.py`, `service_providers.py`, `external_api_providers.py`, etc.
      * Esto mantiene el "pegamento" de la infraestructura bien organizado.

  * **¿Dependencias de Configuración?**
    Si un adaptador necesita, por ejemplo, una API key de una variable de entorno:

```python
# src/config.py
    from pydantic_settings import BaseSettings
    class Settings(BaseSettings):
        external_service_api_key: str = "DEFAULT_KEY"
        # ... otras configuraciones ...
        class Config:
            env_file = ".env"
    settings = Settings()

    # src/infrastructure/adapters/some_external_service_adapter.py
    class SomeExternalServiceAdapter:
        def __init__(self, api_key: str):
            self.api_key = api_key
        async def do_something(self): print(f"Using API key: {self.api_key}")

    # src/infrastructure/dependencies/external_api_providers.py
    from src.config import settings
    def get_external_service_adapter() -> SomeExternalServiceAdapter:
        return SomeExternalServiceAdapter(api_key=settings.external_service_api_key)
```

FastAPI puede inyectar `Settings` también si lo necesitas, pero a menudo es más simple acceder a un objeto `settings` global como el de Pydantic.

-----

### **Acto 5: El Momento "¡Ajá\!" - Por Qué Esto es Oro Puro para tu Hexágono**

Configurar la Inyección de Dependencias de esta manera:

1.  **Refuerza los Límites del Hexágono:** El núcleo se mantiene agnóstico a la infraestructura. Los detalles de implementación de los adaptadores están encapsulados y son intercambiables.

2.  **Habilita la "Conectividad Flexible":** Cambiar una base de datos, un servicio de mensajería, o la forma en que te comunicas con una API externa, se convierte en:

      * Implementar un nuevo adaptador que cumpla el puerto existente.
      * Actualizar la función proveedora en `infrastructure/dependencies` para que devuelva la nueva implementación.
      * ¡Y ya está\! El dominio y la aplicación no se tocan.

3.  **Simplifica la Evolución del Sistema:** A medida que tu aplicación crece o cambian los requisitos tecnológicos, esta separación y configuración explícita de dependencias te ahorrará incontables dolores de cabeza.

-----

**🚀 Desafío Dinámico para el Equipo:**

Imaginad que necesitamos integrar un **servicio de notificaciones**.

1.  **Puerto:** `NotificationServicePort` (en `domain/ports`) con un método `send_notification(user_id: str, message: str)`.
2.  **Adaptador:** `EmailNotificationAdapter` (en `infrastructure/adapters`) que usa una (hipotética) librería `cool_email_sender`.
3.  **Preguntas:**
      * ¿Qué dependencias podría tener `EmailNotificationAdapter` en su `__init__` (ej. un API key, una URL de servicio SMTP)?
      * ¿Cómo crearíais el proveedor `get_notification_service()` en `infrastructure/dependencies/notification_providers.py`?
      * Si vuestro `UserService` necesita enviar una notificación tras registrar un usuario, ¿cómo le inyectaríais `NotificationServicePort`?

¡Discutidlo en grupos pequeños y presentad vuestra solución de "cableado"\!

-----

**Conclusión y Próximos Pasos:**

Dominar la inyección de dependencias con FastAPI no es solo una técnica de FastAPI; es la forma de dar vida real y pragmática a los principios de la Arquitectura Hexagonal. Permite que el núcleo de tu aplicación sea robusto, testable e independiente de los detalles cambiantes del mundo exterior.

Recuerda: **Abstracciones en el núcleo, concreciones en la infraestructura, y FastAPI `Depends` como el maestro de ceremonias que los une elegantemente.**

**Referencias Bibliográficas y Lecturas Recomendadas:**

  * **FastAPI Dependency Injection:** [https://fastapi.tiangolo.com/tutorial/dependencies/](https://fastapi.tiangolo.com/tutorial/dependencies/) (¡La fuente oficial\!)
  * **Alistair Cockburn - Hexagonal Architecture:** [https://alistair.cockburn.us/hexagonal-architecture/](https://alistair.cockburn.us/hexagonal-architecture/) (El origen)
  * **Martin Fowler - Dependency Injection:** [https://martinfowler.com/articles/injection.html](https://martinfowler.com/articles/injection.html) (Conceptos clave)
  * **Robert C. Martin (Uncle Bob) - Clean Architecture:** Aunque no es Hexagonal per se, comparte muchos principios sobre la inversión de dependencias. (Libro "Clean Architecture: A Craftsman's Guide to Software Structure and Design")

En la próxima sesión, pondremos todo esto aún más en práctica al diseñar pruebas para el núcleo sin depender de infraestructuras (¡ya hemos visto un adelanto con `dependency_overrides`\!).

¡Gracias y a seguir construyendo hexágonos sólidos\!

## 6.10 Ejemplo de microservicio hexagonal completo con FastAPI

Ahora, juntemos todo en una estructura de proyecto.

**Estructura de Directorios (Simplificada):**

```
hexagonal_fastapi_ddd/
├── app/
│   ├── __init__.py
│   ├── application/                 # Casos de Uso, DTOs, Puertos de Entrada (Servicios App)
│   │   ├── __init__.py
│   │   ├── dtos/
│   │   │   └── product_dto.py
│   │   ├── ports/
│   │   │   └── product_service.py   # Interfaz ProductServicePort
│   │   └── services/
│   │       └── product_app_service.py # Implementación ProductApplicationService
│   ├── domain/                      # Lógica de Negocio Pura
│   │   ├── __init__.py
│   │   ├── entities/
│   │   │   └── product.py           # Entidad Product
│   │   ├── events/
│   │   │   └── product_events.py    # Eventos de Dominio
│   │   └── ports/
│   │       └── product_repository.py # Interfaz ProductRepositoryPort
│   ├── infrastructure/              # Implementaciones Concretas de Adaptadores de Salida y Config
│   │   ├── __init__.py
│   │   ├── database.py              # Configuración de DB (SQLAlchemy, get_session)
│   │   ├── db_models/
│   │   │   └── product_model.py     # Modelo SQLAlchemy para Product
│   │   ├── event_dispatcher.py
│   │   ├── event_handlers.py
│   │   └── repositories/
│   │       └── mariadb_product_repository.py # Implementación MariaDBProductRepository
│   └── interfaces/                  # Adaptadores de Entrada (API, CLI, etc.)
│       ├── __init__.py
│       └── api/
│           ├── __init__.py
│           ├── dependencies.py      # Factorías para Inyección de Dependencias
│           └── product_routes.py    # Endpoints FastAPI para Productos
├── tests/                           # Pruebas (unitarias, integración)
│   ├── __init__.py
│   ├── application/
│   │   └── test_product_service.py
│   └── domain/
│       └── test_product_entity.py
├── .env                             # Variables de entorno (ej. DATABASE_URL)
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── main.py                          # Punto de entrada de la aplicación FastAPI
└── requirements.txt
```

**`app/domain/entities/product.py`:** (Como se mostró antes, con `_events` y métodos)

**`app/domain/ports/product_repository.py`:** (Como se mostró antes)

**`app/domain/events/product_events.py`:** (Como se mostró antes)

**`app/application/dtos/product_dto.py`:**
```python
from pydantic import BaseModel, Field
from typing import Optional
from app.domain.entities.product import Product as DomainProduct

class ProductBaseDTO(BaseModel):
    name: str = Field(..., min_length=3, max_length=50)
    description: Optional[str] = Field(None, max_length=255)
    price: float = Field(..., gt=0)

class ProductCreateDTO(ProductBaseDTO):
    pass

class ProductDTO(ProductBaseDTO):
    id: str

    class Config:
        orm_mode = True # Pydantic v1
        # from_attributes = True # Pydantic v2

    @classmethod
    def from_domain(cls, product: DomainProduct) -> "ProductDTO":
        return cls(
            id=product.id,
            name=product.name,
            description=product.description,
            price=product.price
        )
```

**`app/application/ports/product_service.py`:** (Como se mostró antes)

**`app/application/services/product_app_service.py`:** (Como se mostró antes, con `EventDispatcher`)

**`app/infrastructure/database.py`:** (Como se mostró antes, con `Base`)

**`app/infrastructure/db_models/product_model.py`:**
```python
from sqlalchemy import Column, Integer, String, Float
from app.infrastructure.database import Base

class Product(Base):
    __tablename__ = "products"

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    name = Column(String(50), index=True, nullable=False)
    description = Column(String(255), nullable=True)
    price = Column(Float, nullable=False)
```

**`app/infrastructure/repositories/mariadb_product_repository.py`:** (Como se mostró antes)

**`app/infrastructure/event_dispatcher.py`:** (Como se mostró antes)

**`app/infrastructure/event_handlers.py`:** (Como se mostró antes, pero registrar el handler se hace en `main.py`)

**`app/interfaces/api/dependencies.py`:** (Como se mostró antes)

**`app/interfaces/api/product_routes.py`:** (Como se mostró antes)

**`main.py`:**
```python
from fastapi import FastAPI
from contextlib import asynccontextmanager

from app.interfaces.api import product_routes
from app.infrastructure.database import create_tables
from app.infrastructure.event_dispatcher import event_dispatcher
from app.infrastructure.event_handlers import send_product_creation_notification # Import handler
from app.domain.events.product_events import ProductCreatedEvent # Import event type

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Código a ejecutar antes de que la aplicación empiece a aceptar peticiones
    print("Application startup: Creating database tables...")
    await create_tables() # Crear tablas al inicio
    print("Database tables created (if they didn't exist).")
    
    # Registrar manejadores de eventos
    print("Registering event handlers...")
    event_dispatcher.register(ProductCreatedEvent, send_product_creation_notification)
    print("Event handlers registered.")
    
    yield
    # Código a ejecutar después de que la aplicación termine
    print("Application shutdown...")

app = FastAPI(
    title="Hexagonal FastAPI DDD Service",
    description="Un ejemplo de microservicio con Arquitectura Hexagonal, DDD y FastAPI.",
    version="0.1.0",
    lifespan=lifespan # Usar el nuevo lifespan para startup/shutdown
)

app.include_router(product_routes.router, prefix="/api/v1")

@app.get("/health", tags=["Health"])
async def health_check():
    return {"status": "ok"}

if __name__ == "__main__":
    import uvicorn
    # Para desarrollo, uvicorn puede manejar el reload.
    # Para producción, se usaría un gestor de procesos como Gunicorn + Uvicorn workers.
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=True)
```

**`requirements.txt`:**
```
fastapi>=0.100.0
uvicorn[standard]>=0.23.0
pydantic>=1.10.0,<2.0.0 # O >=2.0.0 si ajustas los DTOs (orm_mode -> from_attributes)
sqlalchemy[asyncio]>=2.0.0
aiomysql>=0.2.0 # Driver para MariaDB/MySQL con asyncio
python-dotenv>=1.0.0 # Para .env
# Para pruebas
pytest
pytest-asyncio
httpx # Para cliente HTTP en pruebas de integración
# Opcional: para mocks
# unittest.mock # Ya viene con Python
```
*Ajusta las versiones de Pydantic si usas v2.*

**`.env` (ejemplo):**
```
DATABASE_URL="mysql+aiomysql://myuser:mypassword@mariadb_hexagonal_service:3306/mydatabase"
# Para desarrollo local sin Docker, podrías tener:
# DATABASE_URL="mysql+aiomysql://root:example@localhost:3306/hexagonal_db"
```
*Nota: `mariadb_hexagonal_service` es el nombre del servicio de MariaDB en `docker-compose.yml`.*

**`Dockerfile`:**
```dockerfile
FROM python:3.12-slim

WORKDIR /app

# Variables de entorno para Python
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Instalar dependencias del sistema si fueran necesarias
# RUN apt-get update && apt-get install -y --no-install-recommends some-package && rm -rf /var/lib/apt/lists/*

COPY ./requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY ./app /app/app
COPY ./main.py /app/

# Copiar .env si lo usas para configuración en el contenedor (mejor usar variables de entorno de Docker Compose)
# COPY ./.env /app/

EXPOSE 8000

# Comando para ejecutar la aplicación.
# Para producción, se recomienda Gunicorn con Uvicorn workers.
# CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
# Para desarrollo, uvicorn con reload es útil, pero el reload no funcionará bien dentro de Docker
# si los archivos no se montan como volumen.
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**`docker-compose.yml`:**
```yaml
version: '3.8'

services:
  app:
    build: .
    container_name: hexagonal_fastapi_app
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app/app         # Montar código para desarrollo (permite hot-reloading con uvicorn --reload)
      - ./main.py:/app/main.py
      # - ./.env:/app/.env # Opcional si el .env se copia en Dockerfile
    environment:
      # Sobrescribir o definir variables de entorno aquí es preferible a copiar .env
      - DATABASE_URL=mysql+aiomysql://myuser:mypassword@mariadb_hexagonal_service:3306/mydatabase
      # - PYTHONUNBUFFERED=1 # Ya está en el Dockerfile
      # - PYTHONDONTWRITEBYTECODE=1 # Ya está en el Dockerfile
    depends_on:
      mariadb_hexagonal_service:
        condition: service_healthy # Espera a que MariaDB esté lista
    networks:
      - hexagonal_network

  mariadb_hexagonal_service:
    image: mariadb:10.11 # Usar una versión específica de MariaDB
    container_name: mariadb_hexagonal_db
    ports:
      - "3307:3306" # Exponer MariaDB en el puerto 3307 del host para acceso externo si es necesario
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword # No usar en producción
      MYSQL_DATABASE: mydatabase
      MYSQL_USER: myuser
      MYSQL_PASSWORD: mypassword
    volumes:
      - mariadb_data:/var/lib/mysql # Persistir datos de MariaDB
    healthcheck: # Para que 'depends_on' funcione mejor
      test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost", "-u", "root", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - hexagonal_network

volumes:
  mariadb_data:

networks:
  hexagonal_network:
    driver: bridge
```

**Para ejecutar:**
1.  Crea los archivos y directorios como se describe.
2.  Asegúrate de tener Docker y Docker Compose instalados.
3.  Desde la raíz del proyecto `hexagonal_fastapi_ddd/`, ejecuta:
    ```bash
    docker-compose up --build
    ```
4.  Una vez levantado, podrás acceder a la API en `http://localhost:8000/docs`.

    Puedes probar a crear un producto con un POST a `http://localhost:8000/api/v1/products/` con un JSON como:
    ```json
    {
      "name": "Super Product",
      "description": "An amazing product from Hexagonal Arch",
      "price": 199.99
    }
    ```
    Deberías ver en la consola de Docker Compose los logs, incluyendo el mensaje del `ProductCreatedEvent` siendo manejado.

---


