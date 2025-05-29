# Tema 16. PROYECTO FINAL: APLICACIÓN COMPLETA BASADA EN MICROSERVICIOS CON FASTAPI

* [Tema 16. PROYECTO FINAL: APLICACIÓN COMPLETA BASADA EN MICROSERVICIOS CON FASTAPI](Tema16.md#tema-16-proyecto-final-aplicación-completa-basada-en-microservicios-con-fastapi)

    * [16.1 Definición de dominio y bounded contexts](Tema16.md#161-definición-de-dominio-y-bounded-contexts)
    * [16.2 División en microservicios independientes](Tema16.md#162-división-en-microservicios-independientes)
    * [16.3 Contratos REST, eventos y gRPC entre servicios](Tema16.md#163-contratos-rest-eventos-y-grpc-entre-servicios)
    * [16.4 DDD + arquitectura hexagonal en cada servicio](Tema16.md#164-ddd--arquitectura-hexagonal-en-cada-servicio)
    * [16.5 Desarrollo de API Gateway y orquestador de eventos](Tema16.md#165-desarrollo-de-api-gateway-y-orquestador-de-eventos)
    * [16.6 Integración de DB relacional y no relacional](Tema16.md#166-integración-de-db-relacional-y-no-relacional)
    * [16.7 Seguridad, validación y pruebas completas](Tema16.md#167-seguridad-validación-y-pruebas-completas)
    * [16.8 Colas con Kafka y WebSockets para real-time](Tema16.md#168-colas-con-kafka-y-websockets-para-real-time)
    * [16.9 Despliegue automatizado con CI/CD en Kubernetes](Tema16.md#169-despliegue-automatizado-con-cicd-en-kubernetes)
    * [16.10 Documentación y repositorio versionado](Tema16.md#1610-documentación-y-repositorio-versionado)


## 16.1 Definición de dominio y bounded contexts

Este es el momento de cristalizar todo lo aprendido en un proyecto cohesivo. Siguiendo tu directriz, no nos extenderemos en teoría abstracta, sino que **plantearemos un proyecto con una idea actual y describiremos cómo se abordaría cada punto de este tema dentro de ese proyecto**.

El proyecto se desarrollará conceptualmente con **Python 3.12, FastAPI y MariaDB** como base relacional, pero incorporaremos otros servicios y tecnologías según sea necesario para ilustrar una arquitectura de `microservices` moderna y completa.

**Nombre del Proyecto Propuesto:** 🅿️ **ParkWise - Smart Urban Parking Platform**

**Visión del Proyecto ParkWise:**\
Una plataforma integral que conecta a conductores que buscan estacionamiento con proveedores de espacios (particulares, garajes comerciales), optimizando el uso del espacio urbano, facilitando reservas, pagos, y ofreciendo `features` en tiempo real como disponibilidad y notificaciones. Incluirá soporte para `EV charging spots` y potencial para `dynamic pricing`.

Comencemos a desglosar ParkWise a través de los puntos del Tema 16.

***

Para el proyecto **ParkWise**, el primer paso es sumergirnos en su dominio y delimitar sus fronteras conceptuales mediante `Bounded Contexts`, un concepto clave de `Domain-Driven Design (DDD)`. Esto nos ayudará a entender las diferentes partes del problema y a sentar las bases para una división lógica en `microservices`.

**A. Dominio Principal de ParkWise:**

El **dominio central** de ParkWise gira en torno a la **gestión eficiente y la intermediación de espacios de estacionamiento urbano**. Su objetivo es resolver los problemas de:

* **Conductores (`Drivers`):** Encontrar y reservar estacionamiento de forma fácil y fiable, accediendo a información en tiempo real sobre disponibilidad y precio.
* **Proveedores de Estacionamiento (`Providers`):** Maximizar la utilización de sus espacios (ya sean privados, comerciales o públicos gestionados), gestionar listados, precios y recibir pagos de forma segura.
* **La Plataforma ParkWise:** Facilitar esta interacción, asegurar la confianza, gestionar transacciones y proveer valor añadido (como `smart features`, notificaciones, analíticas).

Las `core capabilities` incluirán:

* Búsqueda geoespacial de `parking spots`.
* Visualización de disponibilidad en tiempo real.
* Sistema de reservas y `booking management`.
* Procesamiento de pagos y `payouts`.
* Gestión de perfiles de `users` (`drivers` y `providers`).
* Notificaciones instantáneas.
* Soporte para características especiales como `EV charging spots` y potencial para `dynamic pricing` basado en demanda u hora.

**B. `Bounded Contexts` Identificados para ParkWise:**

Un `Bounded Context` es un límite conceptual dentro del cual un modelo de dominio particular es consistente y aplicable. Para ParkWise, podemos identificar los siguientes `Bounded Contexts` iniciales:

1. **👤 Identity & Access Management (IAM) Context:**
   * **Responsabilidades:** Gestión de `users` (conductores, proveedores, administradores de la plataforma), `authentication` (registro, `login`, `logout`), `authorization` (roles, `permissions`), gestión de perfiles de `user`.
   * **Lenguaje Ubicuo (`Ubiquitous Language`) Clave:** `User`, `DriverProfile`, `ProviderProfile`, `Role`, `Permission`, `Credential`, `AuthToken`.
   * **Lógica Central:** Validación de credenciales, gestión de `sessions` o `tokens`, asignación de `roles`.
2. **🅿️ Parking Spot & Availability Context:**
   * **Responsabilidades:** Gestión del inventario de `parking spots` por parte de los `Providers`. Esto incluye la creación y actualización de listados de `spots` (ubicación, tamaño, tipo - regular, `EV charging`, cubierto, etc.), definición de reglas de precios base, y, crucialmente, la gestión de su **disponibilidad en tiempo real**.
   * **Lenguaje Ubicuo Clave:** `ParkingSpot`, `Location` (GeoJSON), `AvailabilitySlot`, `PricingRule`, `SpotType` (EV, Handicap), `ProviderListing`.
   * **Lógica Central:** Algoritmos de actualización de disponibilidad, validación de datos de `spots`, gestión de `amenities`.
3. **📅 Booking & Reservation Context:**
   * **Responsabilidades:** Manejar el proceso completo de reserva de un `parking spot` por un `Driver`. Esto incluye la búsqueda de `spots` disponibles (posiblemente interactuando con el `Parking Spot Context`), la creación de una reserva para un horario específico, la gestión del ciclo de vida de la reserva (`PENDING`, `CONFIRMED`, `ACTIVE`, `COMPLETED`, `CANCELLED`), y el manejo de modificaciones o cancelaciones.
   * **Lenguaje Ubicuo Clave:** `Booking`, `ReservationRequest`, `TimeSlot`, `BookingStatus`, `CancellationPolicy`.
   * **Lógica Central:** Lógica de validación de reservas (ej. no `double booking`), aplicación de políticas de cancelación, cálculo de costes preliminares.
4. **💳 Payment & Billing Context:**
   * **Responsabilidades:** Procesar los pagos de los `Drivers` por las reservas, gestionar los `payouts` a los `Providers`, manejar facturas, reembolsos y disputas. Integración con `payment gateways` externos.
   * **Lenguaje Ubicuo Clave:** `Payment`, `Transaction`, `Invoice`, `Refund`, `Payout`, `PaymentMethod`, `GatewayResponse`.
   * **Lógica Central:** Interacción segura con `payment gateways`, cálculo de comisiones, seguimiento de transacciones financieras.
5. **📣 Notification Context:**
   * **Responsabilidades:** Enviar comunicaciones proactivas a los `users` sobre eventos importantes.
   * **Lenguaje Ubicuo Clave:** `Notification`, `NotificationChannel` (Email, SMS, `WebSocket Push`), `NotificationTemplate`, `UserPreferences` (para notificaciones).
   * **Lógica Central:** `Templating` de mensajes, enrutamiento a través de diferentes `channels`, gestión de intentos de entrega, respeto de las preferencias del `user`. (Este `context` podría evolucionar a un `Notification Microservice` dedicado, como vimos en 10.6).
6. **🌟 Review & Rating Context:**
   * **Responsabilidades:** Permitir a los `Drivers` calificar y escribir reseñas sobre los `parking spots` y/o `Providers` después de una reserva completada. Permitir a los `Providers` ver sus calificaciones.
   * **Lenguaje Ubicuo Clave:** `Review`, `RatingScore`, `Comment`, `ReviewableEntity` (Spot, Provider).
   * **Lógica Central:** Validación de reseñas, cálculo de puntuaciones promedio, moderación (opcional).
7. **(Futuro Potencial) 📈 Analytics & Reporting Context:**
   * **Responsabilidades:** Recopilar datos de los otros `contexts` para generar informes y analíticas para los `Providers` (ej. ocupación, ingresos) y para los administradores de la plataforma ParkWise (ej. crecimiento de `users`, `revenue streams`, `patterns` de uso).
   * **Lenguaje Ubicuo Clave:** `Report`, `Metric`, `DataPoint`, `AnalyticsDashboard`.
   * **Lógica Central:** Agregación de datos, generación de informes, `data visualization APIs`.

**Diagrama de `Bounded Contexts` de ParkWise (Simplificado):**

```mermaid
graph TD
    subgraph ParkWise_Platform
        IAM[Identity and Access Management Context]
        PSA[Parking Spot and Availability Context]
        BRC[Booking and Reservation Context]
        PBC[Payment and Billing Context]
        NC[Notification Context]
        RRC[Review and Rating Context]
        ARC[Analytics and Reporting Context - Futuro]

        DriverUser[Driver - Usuario]
        ProviderUser[Provider - Usuario]
        AdminUser[Admin - Usuario]

        DriverUser --> IAM
        ProviderUser --> IAM
        AdminUser --> IAM

        ProviderUser --> PSA
        DriverUser -->|Busca disponibilidad| PSA
        
        DriverUser -->|Reserva| BRC
        BRC -->|Consulta disponibilidad| PSA
        BRC -->|Consulta usuario| IAM
        
        BRC -->|Inicia pago| PBC
        PBC -->|Consulta usuario| IAM
        
        PBC -->|Notifica resultado| NC
        BRC -->|Notifica estado| NC
        PSA -->|Notifica disponibilidad| NC
        NC -->|Notifica a| DriverUser
        NC -->|Notifica a| ProviderUser
        
        DriverUser -->|Escribe reseña| RRC
        RRC -->|Relacionado con| PSA
        ProviderUser -->|Lee reseñas| RRC

        PSA --> ARC
        BRC --> ARC
        PBC --> ARC
        ARC --> AdminUser
    end

    style IAM fill:#D5F5E3,stroke:#2ECC71
    style PSA fill:#AED6F1,stroke:#3498DB
    style BRC fill:#FAD7A0,stroke:#F39C12
    style PBC fill:#FFCDD2,stroke:#C62828
    style NC fill:#E8DAEF,stroke:#8E44AD
    style RRC fill:#FFF9C4,stroke:#F1C40F
    style ARC fill:#CFD8DC,stroke:#37474F


```

**Consideraciones Adicionales:**

* **Evolución:** Estos `bounded contexts` pueden evolucionar. Algunos podrían subdividirse más a medida que la complejidad crece. Por ejemplo, la "Gestión de Precios" dentro de `Parking Spot & Availability` podría volverse lo suficientemente compleja (con `dynamic pricing`, reglas promocionales) como para justificar su propio `bounded context` (y `microservice`).
* **Superposición y Comunicación:** Aunque los `bounded contexts` buscan el desacoplamiento, inevitablemente necesitan comunicarse. Esto se gestionará a través de APIs bien definidas y/o `event-driven architecture` (que veremos en 16.3).

Esta definición inicial del dominio y sus `bounded contexts` nos da un marco sólido para empezar a pensar en la división en `microservices` (16.2) y los contratos entre ellos. Es un ejercicio iterativo; a medida que profundicemos en cada aspecto del proyecto, podríamos refinar estos límites.



## 16.2 División en microservicios independientes



La meta aquí es traducir cada `bounded context` (o un grupo coherente de ellos) en un `microservice` con responsabilidades claras y límites bien definidos.



Basándonos en los `Bounded Contexts` identificados para **ParkWise** en la sección 16.1, proponemos la siguiente división inicial en `microservices` independientes. Cada servicio será responsable de su propio dominio de datos y lógica de negocio, exponiendo sus capacidades a través de APIs.

1. **`Identity Service` (Basado en IAM Context)**
   * **Responsabilidades Principales:**
     * Gestión de `user accounts` (`drivers`, `providers`, `admins`).
     * `Authentication` (`user registration`, `login`, `token issuance` - ej. `JWT`).
     * `Authorization` básica (gestión de `roles` y `permissions`).
     * Gestión de perfiles de `user` (datos básicos, preferencias de contacto no relacionadas con notificaciones específicas).
   * **`Database` Propia:** MariaDB (para `user credentials`, perfiles, `roles`).
   * **API Expuesta:** `Endpoints` para `register`, `login`, `get_user_profile`, `update_profile`, gestión de `roles` (para `admins`).
2. **`Parking Lot Service` (Basado en Parking Spot & Availability Context)**
   * **Responsabilidades Principales:**
     * Gestión del `inventory` de `parking spots` por parte de los `Providers`.
     * Creación, actualización, y eliminación de `listings` de `spots` (ubicación `GeoJSON`, tamaño, tipo - regular, `EV charging`, cubierto, `amenities`).
     * Gestión de la disponibilidad de `spots` en tiempo real (marcar como `disponible/ocupado/reservado`, `bloqueos temporales`).
     * Definición de `pricing rules` base asociadas a los `spots` o `lots`.
   * **`Database` Propia:**
     * MariaDB para datos estructurales de `spots` y `providers`.
     * (Potencial) Redis para `tracking` de disponibilidad en tiempo real de alta velocidad y/o MongoDB para `GeoJSON queries` si MariaDB no es suficiente.
   * **API Expuesta:** `Endpoints` para `providers` para gestionar sus `spots`, `endpoints` para buscar `spots` (con filtros geoespaciales, por tipo, etc.), `endpoints` para actualizar disponibilidad (puede ser interno o llamado por otros servicios).
3. **`Booking Service` (Basado en Booking & Reservation Context)**
   * **Responsabilidades Principales:**
     * Orquestar el proceso de reserva: buscar `spots` disponibles (interactuando con `Parking Lot Service`), validar la solicitud de reserva.
     * Crear y gestionar el ciclo de vida de las reservas (`PENDING_PAYMENT`, `CONFIRMED`, `ACTIVE`, `COMPLETED`, `CANCELLED`).
     * Aplicar políticas de cancelación y modificación.
     * Calcular costes de reserva.
   * **`Database` Propia:** MariaDB (para `bookings`, `reservation details`, `status history`).
   * **API Expuesta:** `Endpoints` para `drivers` para crear y gestionar sus reservas, `endpoints` para `providers` para ver las reservas de sus `spots`.
4. **`Payment Service` (Basado en Payment & Billing Context)**
   * **Responsabilidades Principales:**
     * Integración con `external payment gateways` (Stripe, PayPal, etc.).
     * Procesar pagos para reservas.
     * Gestionar reembolsos.
     * Orquestar `payouts` a los `Providers`.
     * Generación de `invoices` (simplificado inicialmente).
   * **`Database` Propia:** MariaDB (para `transaction records`, `payment status`, `payout records`).
   * **API Expuesta:** `Endpoints` internos para ser llamados por `Booking Service` para iniciar pagos, `endpoints` para manejar `webhooks` de `payment gateways`, `endpoints` para `providers` para ver su historial de `payouts`.
5. **`Notification Service` (Basado en Notification Context)**
   * **Responsabilidades Principales:**
     * Consumir `events` de negocio de otros servicios (ej. `BookingConfirmedEvent`, `PaymentSuccessfulEvent`, `NewSpotAvailableEvent`).
     * Consultar preferencias de notificación del `user` (podría interactuar con `Identity Service`).
     * Formatear y enviar notificaciones a través de múltiples `channels` (`WebSockets`, Email, SMS - inicialmente `WebSockets` y Email).
   * **`Database` Propia:** MariaDB (para `notification templates`, historial de notificaciones enviadas, `user notification preferences` si no están en `Identity Service`).
   * **API Expuesta:** Principalmente consume `events`. Podría tener una API interna para `trigger` notificaciones específicas si es necesario, pero el `pattern` principal es `event-driven`.
6. **`Review Service` (Basado en Review & Rating Context)**
   * **Responsabilidades Principales:**
     * Permitir a los `drivers` enviar `reviews` y `ratings` para `parking spots` después de una reserva completada.
     * Calcular `average ratings`.
     * Exponer `reviews` y `ratings`.
     * (Opcional) Moderación de `reviews`.
   * **`Database` Propia:** MariaDB o MongoDB (MongoDB podría ser bueno aquí por la flexibilidad de los `reviews` y `nested comments` si se añaden). Inicialmente MariaDB.
   * **API Expuesta:** `Endpoints` para crear `reviews`, obtener `reviews` para un `spot` o `provider`.

**Diagrama `Mermaid` de la División en `Microservices` para ParkWise:**

```mermaid
graph TD
    APIGW[API Gateway / BFFs]

    subgraph Service Cluster
        IdentitySvc["Identity Service<br/>(MariaDB)"]
        ParkingLotSvc["Parking Lot Service<br/>(MariaDB, opc. Redis/Mongo)"]
        BookingSvc["Booking Service<br/>(MariaDB)"]
        PaymentSvc["Payment Service<br/>(MariaDB)"]
        NotificationSvc["Notification Service<br/>(MariaDB)"]
        ReviewSvc["Review Service<br/>(MariaDB)"]
    end

    APIGW --> IdentitySvc
    APIGW --> ParkingLotSvc
    APIGW --> BookingSvc
    APIGW --> ReviewSvc
    
    BookingSvc -- Llama API / Eventos --> ParkingLotSvc
    BookingSvc -- Llama API / Eventos --> PaymentSvc
    BookingSvc -- Publica Evento --> EventBus[(Event Bus<br/>Kafka/RabbitMQ)]
    PaymentSvc -- Publica Evento --> EventBus
    ParkingLotSvc -- Publica Evento --> EventBus
    ReviewSvc -- Publica Evento --> EventBus 
    IdentitySvc -- Publica Evento UserUpdated --> EventBus

    EventBus --> NotificationSvc

    style IdentitySvc fill:#82E0AA,stroke:#1E8449
    style ParkingLotSvc fill:#AED6F1,stroke:#3498DB
    style BookingSvc fill:#FAD7A0,stroke:#F39C12
    style PaymentSvc fill:#FFCDD2,stroke:#C62828
    style NotificationSvc fill:#E8DAEF,stroke:#8E44AD
    style ReviewSvc fill:#FFF9C4,stroke:#F1C40F
    style APIGW fill:#D6DBDF,stroke:#2C3E50
    style EventBus fill:#E5E7E9,stroke:#5D6D7E
```

**Consideraciones Clave para esta División:**

* **Autonomía:** Cada servicio gestiona su propio `schema` de base de datos y puede ser desplegado y escalado independientemente.
* **Comunicación:**
  * **Síncrona (API Calls):** En algunos casos, un servicio podría necesitar llamar a la API de otro directamente para obtener información que necesita _inmediatamente_ para completar una operación (ej. `Booking Service` podría consultar la disponibilidad en `Parking Lot Service`). Esto debe usarse con moderación para evitar acoplamiento fuerte.
  * **Asíncrona (`Event-Driven`):** Para la mayoría de las interacciones de "reacción" y para desacoplar servicios, se usará un `Event Bus` (Kafka o RabbitMQ). Cuando un servicio completa una acción significativa, publica un `event`. Otros servicios interesados se suscriben a estos `events`. (Más en 16.3 y 16.8).
* **`API Gateway`:** Un `API Gateway` (o `Backend-For-Frontend - BFFs`) se situará frente a estos `microservices` para exponer una API unificada y adecuada a los `clients` (web, móvil). (Más en 16.5).
* **Evolución:** Esta es una división inicial. A medida que ParkWise crezca, algunos servicios podrían subdividirse más (ej. `Pricing Service` separado) o podrían surgir nuevos servicios.

Esta división busca un equilibrio entre cohesión (agrupar funcionalidades relacionadas) y desacoplamiento (permitir desarrollo y despliegue independiente).



## 16.3 Contratos REST, eventos y gRPC entre servicios

Vamos directos al **16.3** para el proyecto **ParkWise**. Habiendo definido el dominio, los `bounded contexts` (16.1) y la división inicial en `microservices` (16.2), ahora es crucial definir **cómo estos servicios van a "hablar" entre sí y con el mundo exterior**. Los contratos de comunicación son las reglas del lenguaje que aseguran que la colaboración sea posible y fiable.



En un ecosistema de `microservices` como ParkWise, no existe una única forma de comunicación. Se elegirá el estilo más adecuado (síncrono, asíncrono, `request/response`, `event-driven`) según la naturaleza de la interacción. Para cada estilo, un **contrato bien definido** es esencial.

**1. Contratos `RESTful API` para Comunicación Síncrona (Interna y Externa) 📞**

* **Uso en ParkWise:**
  * **APIs Públicas (vía `API Gateway`):** Los `clients` (aplicaciones móviles, `web frontends`) interactuarán con ParkWise principalmente a través de `endpoints` RESTful expuestos por un `API Gateway`. Este `gateway` enrutará las peticiones a los `microservices` internos correspondientes.
  * **Comunicación `Service-to-Service` Síncrona (Selectiva):** En algunos casos donde un servicio necesita una respuesta inmediata de otro para completar su operación.
    * Ejemplo: El `Booking Service`, al intentar crear una reserva, podría necesitar consultar en tiempo real al `Parking Lot Service` sobre la disponibilidad exacta de un `spot` específico para el `timeslot` solicitado.
    * Otro ejemplo: El `API Gateway` podría necesitar obtener detalles del `user` del `Identity Service` antes de procesar una petición.
* **Definición del Contrato REST para cada `Microservice` ParkWise:**
  * **OpenAPI Specification (Generada por FastAPI):** Cada `microservice` FastAPI (ej. `IdentityService`, `ParkingLotService`, `BookingService`) expondrá su propia `OpenAPI specification` (disponible en `/openapi.json`). Este `schema` es el contrato formal.
  *   **Pydantic Models (`DTOs - Data Transfer Objects`):** Como vimos en el Tema 11.3, los modelos Pydantic definirán rigurosamente los `schemas` para los `request bodies` y `response models`. Estos son la `source of truth` para la estructura de los datos intercambiados.

      ```python
      # Ejemplo: DTO para solicitar disponibilidad en Parking Lot Service
      # (usado por Booking Service o API Gateway)
      # from pydantic import BaseModel
      # from datetime import datetime

      # class SpotAvailabilityRequest(BaseModel):
      #     spot_id: str
      #     start_time: datetime
      #     end_time: datetime

      # class SpotAvailabilityResponse(BaseModel):
      #     spot_id: str
      #     is_available: bool
      #     estimated_price: Optional[float] = None
      ```
  * **HTTP Methods y `Status Codes`:** Se seguirán estrictamente las buenas prácticas RESTful (Tema 11.1 y 11.6) para el uso de `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, y los `status codes` apropiados.
  * **Versioning de API (vía URI Path):** Se usará `versioning` en la URI para gestionar la evolución de los contratos API de cada servicio (ej. `/parking/v1/spots/{spot_id}/availability`). (Tema 11.2).
* **Consideraciones para ParkWise:**
  * Para las llamadas `service-to-service` síncronas, se utilizarán `HTTP clients` asíncronos (como `httpx`) dentro de los servicios FastAPI para no bloquear el `event loop`.
  * Se implementarán patrones de resiliencia como `retries` (con `exponential backoff`) y `circuit breakers` (ej. usando librerías como `pybreaker` o `resilience4j`-like patterns) para estas llamadas internas.

**2. Contratos de `Events` para Desacoplamiento y Flujos Asíncronos 📣**

La comunicación `event-driven` será la espina dorsal para el desacoplamiento y la reactividad en ParkWise, utilizando **Kafka** como `message broker` principal (como se anticipa en 16.8).

* **Uso en ParkWise (Comunicación Asíncrona Primaria):**
  * `IdentityService` publica `UserRegisteredEvent`, `UserProfileUpdatedEvent`.
  * `ParkingLotService` publica `ParkingSpotListedEvent`, `ParkingSpotUpdatedEvent`, `ParkingSpotAvailabilityChangedEvent`.
  * `BookingService` publica `BookingRequestedEvent`, `BookingConfirmedEvent`, `BookingCancelledEvent`, `BookingCompletedEvent`.
  * `PaymentService` publica `PaymentProcessedEvent`, `PaymentFailedEvent`, `PayoutInitiatedEvent`.
  * `ReviewService` publica `ReviewSubmittedEvent`.
  * El `NotificationService` será un consumidor clave de muchos de estos `events` para informar a los `users`. Otros servicios también se suscribirán a los `events` relevantes para mantener su propio `state` o `trigger` lógica de negocio.
* **Definición del Contrato de `Event`:**
  *   **`Event Schema` (Pydantic Models):** Cada tipo de `event` se definirá con un modelo Pydantic, asegurando una estructura clara y validable.

      ```python
      # from pydantic import BaseModel, Field
      # from typing import List, Dict, Any, Literal
      # from uuid import UUID, uuid4
      # from datetime import datetime, timezone

      # class EventMetadata(BaseModel):
      #     event_id: UUID = Field(default_factory=uuid4)
      #     event_type: str # El nombre del tipo de evento, ej. "BookingConfirmed"
      #     event_version: str = "1.0"
      #     timestamp: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))
      #     source_service: str # Nombre del servicio que originó el evento
      #     correlation_id: Optional[UUID] = None

      # class BookingConfirmedEventPayload(BaseModel):
      #     booking_id: str
      #     user_id: str
      #     spot_id: str
      #     start_time: datetime
      #     end_time: datetime
      #     total_price: float

      # class BookingConfirmedEvent(BaseModel):
      #     metadata: EventMetadata
      #     payload: BookingConfirmedEventPayload
      ```
  * **Serialización:** Los `events` se serializarán a **`JSON`** para su publicación en Kafka.
  * **`Broker` (Kafka):**
    * **`Topics`:** Se definirán `topics` Kafka bien nombrados y con una granularidad adecuada (ej. `parkwise.booking.events`, `parkwise.parking.events`, `parkwise.user.events`). Se podría considerar un `topic` por `aggregate type` o por `bounded context`.
    * **`Message Keys`:** Se usarán `keys` apropiadas en los mensajes Kafka (ej. `booking_id`, `spot_id`, `user_id`) para asegurar que los `events` relacionados con la misma `entity` vayan a la misma `partition` (garantizando el orden para esa `entity`).
  * **Versioning de `Events`:**
    * El `field` `event_version` en `EventMetadata` es crucial.
    * Se necesitarán estrategias para la evolución del `schema` de `events` (ej. `upcasting` de `events` antiguos, consumidores que pueden manejar múltiples versiones, o un `schema registry` como Confluent Schema Registry si se usa Avro/Protobuf en lugar de JSON para `payloads` más complejos y con `schema evolution` robusta).
* **Consideraciones para ParkWise:**
  * Todos los `event consumers` _deben_ ser idempotentes (14.7).
  * Se configurarán `Dead Letter Topics (DLTs)` en Kafka para manejar `events` que no se pueden procesar.
  * La trazabilidad (`correlation_id` propagado a través de `events`) será fundamental.

**3. Contratos `gRPC` para Comunicación Interna de Alto Rendimiento (Uso Selectivo y Avanzado) 🚀**

`gRPC` (Google Remote Procedure Call) podría considerarse para interacciones `service-to-service` internas muy específicas donde el rendimiento (baja latencia, alto `throughput`) y un contrato fuertemente tipado son primordiales, y la sobrecarga de HTTP/JSON de REST es significativa.

* **Uso Potencial en ParkWise (Ejemplos Especulativos):**
  * Si el `Parking Lot Service` necesita proveer un `stream` de `updates` de disponibilidad de `spots` de muy alta frecuencia a un `Booking Service` (o a un `caching layer` especializado que el `Booking Service` consulta), `gRPC streaming` podría ser una opción.
  * Llamadas internas entre un `API Gateway` y un `microservice` en una `hot path` si se demuestra que REST es un cuello de botella.
* **Definición del Contrato (`.proto` files):**
  *   Los servicios, métodos `RPC`, y `message types` (equivalentes a `DTOs`) se definen usando **`Protocol Buffers`**.

      ```protobuf
      // Ejemplo: parking_spot_availability.proto
      // syntax = "proto3";

      // package parkwise.parking.v1; // Namespace

      // service ParkingSpotAvailabilityService {
      //   // Un RPC para obtener el estado de múltiples spots de una vez
      //   rpc GetSpotsAvailability(SpotsAvailabilityRequest) returns (SpotsAvailabilityResponse);
      //   // Un RPC para un stream de updates (más avanzado)
      //   rpc SubscribeToSpotUpdates(SpotUpdateRequest) returns (stream SpotAvailabilityStreamItem);
      // }

      // message SpotAvailabilityRequest {
      //   repeated string spot_ids = 1;
      // }

      // message SpotAvailability {
      //   string spot_id = 1;
      //   bool is_available = 2;
      //   int64 last_updated_utc_timestamp = 3;
      // }

      // message SpotsAvailabilityResponse {
      //   repeated SpotAvailability spots = 1;
      // }

      // message SpotUpdateRequest {
      //   string spot_id = 1;
      // }
      // message SpotAvailabilityStreamItem {
      //    SpotAvailability spot_update = 1;
      // }
      ```
  * A partir de estos archivos `.proto`, se genera código `stub` (tanto para el `client` como para el `server`) en Python usando el compilador `protoc` y los `plugins` de `gRPC` para Python. FastAPI puede integrarse con `servers` `gRPC` (ej. el `server` `gRPC` corre junto a FastAPI, o FastAPI llama a `clients` `gRPC`).
* **Consideraciones para ParkWise:**
  * **Complejidad:** `gRPC` y `Protocol Buffers` añaden una nueva tecnología al `stack` y tienen su propia curva de aprendizaje.
  * **Tooling:** Requiere `protoc` y gestión de los `stubs` generados.
  * **No `Browser-Friendly` Directo:** La comunicación directa desde un `browser` a un `endpoint` `gRPC` generalmente requiere un `proxy` como `gRPC-Web`.
  * **Decisión:** Para ParkWise, `gRPC` se consideraría **solo para `hot paths` internos muy específicos** después de un `profiling` que demuestre que REST/eventos son un cuello de botella significativo. La prioridad inicial será REST y `events` sobre Kafka.

**Tabla Resumen: Estilos de Comunicación y Contratos en ParkWise**

| Estilo de Comunicación   | Protocolo Principal        | Formato de Datos Común | Definición de Contrato Principal                                    | Casos de Uso Clave en ParkWise                                                                                                                                        |
| ------------------------ | -------------------------- | ---------------------- | ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`REST API`**           | HTTP/1.1, HTTP/2           | `JSON`                 | OpenAPI `Spec` (generada por FastAPI desde Pydantic `DTOs`)         | Exposición de `capabilities` a `clients` externos (vía `API Gateway`); algunas llamadas `service-to-service` síncronas (ej. consulta de disponibilidad).              |
| **`Events` (Asíncrono)** | Kafka Protocol (sobre TCP) | `JSON` (inicialmente)  | `Schema` de `Event` (Pydantic `models`), `Topic Naming Conventions` | Desacoplamiento máximo, `broadcast` de cambios de estado, `trigger` de `workflows` asíncronos, notificaciones (ej. `BookingConfirmedEvent`, `PaymentProcessedEvent`). |
| **`gRPC` (Opcional)**    | HTTP/2                     | `Protocol Buffers`     | Archivos `.proto`                                                   | (Especulativo/Avanzado) Comunicación interna `service-to-service` de muy baja latencia o `streaming` para `hot paths` específicos.                                    |

**Conclusión: Estableciendo el Lenguaje Común para una Colaboración Fluida entre `Microservices`**

La elección del estilo de comunicación y la definición rigurosa de sus contratos son decisiones arquitectónicas fundamentales en ParkWise. Una mezcla pragmática será la clave:

* **REST APIs** (definidas con OpenAPI/Pydantic) para la mayoría de las interacciones síncronas y la exposición externa.
* **`Events` sobre Kafka** (definidos con Pydantic/JSON) como el mecanismo principal para la comunicación asíncrona, el desacoplamiento y la resiliencia.
* **`gRPC`** se mantendrá como una opción especializada a considerar solo si surgen requisitos de rendimiento interno extremo que no puedan ser satisfechos por los otros dos.

Esta estrategia multi-modal, con contratos claros para cada modo, permitirá que los `microservices` de ParkWise colaboren eficazmente mientras mantienen su autonomía.

***

## 16.4 DDD + arquitectura hexagonal en cada servicio

Si el 16.3 definió _cómo_ se comunican nuestros `microservices` de ParkWise, el **16.4** se sumerge en _cómo estructuramos internamente cada uno de esos `microservices`_. Para construir servicios que sean robustos, mantenibles y evolucionables, aplicaremos los principios de **`Domain-Driven Design (DDD)`** y la **Arquitectura Hexagonal (también conocida como `Ports and Adapters`)**.

Imagina cada `microservice` no como una simple caja negra, sino como una fortaleza bien diseñada: un núcleo de lógica de negocio invaluable, protegido del mundo exterior por murallas (la arquitectura hexagonal) y hablando un lenguaje preciso y especializado (el `ubiquitous language` de DDD).

***

Para cada `microservice` identificado en ParkWise (Identity, Parking Lot, Booking, Payment, Notification, Review), adoptaremos un enfoque interno que priorice la claridad del dominio y el desacoplamiento de la infraestructura.

**1. `Domain-Driven Design (DDD)` en el Corazón de Cada `Microservice` ParkWise ❤️**

Dentro de los límites de cada `microservice` (que ya se alinean con nuestros `Bounded Contexts` de 16.1), aplicaremos los patrones tácticos de DDD:

* **`Ubiquitous Language` Específico del `Bounded Context`:**
  * El `Booking Service` hablará de `Bookings`, `TimeSlots`, `ReservationStatus`.
  * El `Parking Lot Service` hablará de `ParkingSpots`, `Availability`, `GeoCoordinates`, `Amenities`.
  * Este lenguaje se reflejará en los nombres de clases, métodos y variables dentro del `core` de cada servicio.
* **Modelado con `Entities`, `Value Objects`, y `Aggregates`:**
  * **`Entities`:** Objetos con una identidad que persiste a lo largo del tiempo y pueden cambiar de estado (ej. una `Booking` específica, un `ParkingSpot` específico). En SQLAlchemy, estos serán nuestras clases ORM (como `ProductDB` en 13.1).
  * **`Value Objects`:** Objetos inmutables definidos por sus atributos, sin una identidad conceptual (ej. un `TimeSlot` con `start_time` y `end_time`, una `Money` `value object` con `amount` y `currency`, una `GeoCoordinate`).
  * **`Aggregates`:** Un clúster de `entities` y `value objects` relacionados que se tratan como una única unidad de consistencia transaccional. Tienen una raíz (`Aggregate Root`) que es la única `entity` del `aggregate` a la que se puede hacer referencia desde el exterior.
    * **`BookingService`:** `Booking` podría ser un `Aggregate Root`, conteniendo `BookingItem` `entities` (si un `booking` puede tener múltiples `spots` o `services` asociados) y `TimeSlot` `value objects`. Todas las operaciones que modifican un `booking` pasarían a través del `Booking aggregate root`.
    * **`ParkingLotService`:** `ParkingSpot` sería un `Aggregate Root`.
    * **`IdentityService`:** `User` sería un `Aggregate Root`.
* **`Domain Events` Internos:**
  * Cuando un `Aggregate` cambia de estado debido a una operación de negocio, genera `Domain Events` para señalar ese cambio. Estos son `events` _internos_ al servicio, que describen algo que ya ha sucedido.
  * Ejemplo: El método `confirm_booking()` en el `Booking Aggregate` podría generar un `BookingConfirmedDomainEvent`.
  * Estos `domain events` pueden ser luego usados por la capa de aplicación para desencadenar lógica adicional o ser traducidos a `integration events` para otros `microservices` (como vimos en 16.3).
* **`Repositories` (Interfaces Definidas por el Dominio/Aplicación):**
  * El dominio define cómo necesita persistir y recuperar sus `Aggregates`, a través de interfaces de `Repository` (ej. `IBookingRepository`). La implementación concreta (SQLAlchemy, Motor) reside en la capa de infraestructura. (Como vimos en 13.3).
* **`Domain Services`:**
  * Para lógica de dominio que no encaja naturalmente en una `Entity` o `Aggregate` (ej. un `PricingCalculationService` que involucra múltiples `entities` o reglas complejas).

**2. Arquitectura Hexagonal (`Ports and Adapters`) por `Microservice` 🏰**

Esta arquitectura protege nuestro `domain model` y `application logic` (el hexágono) de las dependencias de la infraestructura (bases de datos, `message brokers`, APIs externas, `frameworks` web).

* **El Hexágono (Núcleo del `Microservice`):**
  * Contiene el **`Domain Model`** (las `Entities`, `Aggregates`, `Value Objects`, `Domain Events`, `Domain Services` de DDD).
  * Contiene los **`Application Services`** (o `Use Case Handlers`): Orquestan los `use cases`, interactuando con los `Aggregates` del dominio y los `Repositories`. Esta capa es el punto de entrada principal al `core logic`.
  * **Importante:** El código aquí es puro Python, sin referencias a FastAPI, SQLAlchemy, Kafka, etc.
* **Puertos (`Ports` - Interfaces Definidas por el Núcleo):**\
  Son las "puertas" y "enchufes" del hexágono. Son interfaces (ej. `Python Protocols` o `ABCs`).
  *   **`Driving Ports` (Puertos de Entrada / `Use Case Interfaces`):**\
      Definen cómo el mundo exterior invoca la lógica del núcleo. Son las APIs que exponen los `Application Services`.

      ```python
      # Ejemplo para el BookingService: application/ports/input/booking_use_cases.py
      from typing import Protocol, Optional # Necesitaríamos DTOs definidos también
      # from ..dtos import BookingRequestDTO, BookingDetailsDTO # (DTOs de la capa de aplicación)

      class ICreateBookingUseCase(Protocol):
          async def execute(self, user_id: str, request_data: "BookingRequestDTO") -> "BookingDetailsDTO":
              ...

      class ICancelBookingUseCase(Protocol):
          async def execute(self, user_id: str, booking_id: str) -> None:
              ...
      ```
  *   **`Driven Ports` (Puertos de Salida / Interfaces de Infraestructura):**\
      Definen lo que el núcleo necesita del mundo exterior (abstracciones de infraestructura).

      ```python
      # Ejemplo para el BookingService: domain/ports/output/booking_repository.py
      # from ..aggregates.booking_aggregate import BookingAggregate # El Aggregate de dominio
      # from typing import Optional, Protocol

      class IBookingRepository(Protocol):
          async def save(self, booking: "BookingAggregate") -> None: ...
          async def get_by_id(self, booking_id: str) -> Optional["BookingAggregate"]: ...
          # ... otros métodos necesarios ...

      # Ejemplo: domain/ports/output/event_publisher.py
      # from ..events import DomainEvent # Clase base para eventos de dominio

      class IEventPublisher(Protocol):
          async def publish(self, domain_event: "DomainEvent") -> None: ...
      ```
* **Adaptadores (`Adapters` - En la Capa de Infraestructura, fuera del Hexágono):**\
  Son las implementaciones concretas de los `Ports`, conectando el núcleo con tecnologías específicas.
  * **`Driving Adapters` (Invocan los `Input Ports` del Núcleo):**
    * **Controladores FastAPI REST:** Convierten `HTTP requests` (validados con Pydantic `DTOs` de API) en llamadas a los métodos de las `Use Case Interfaces` (`ICreateBookingUseCase.execute(...)`).
    * **`Handlers`** **`WebSocket` FastAPI:** Convierten `messages` `WebSocket` en llamadas a `Use Case Interfaces`.
    * **Consumidores de Kafka/RabbitMQ:** Toman `messages` del `broker`, los deserializan y los mapean a llamadas a `Use Case Interfaces`.
  * **`Driven Adapters` (Implementan los `Output Ports` que el Núcleleo Necesita):**
    * **Repositorios SQLAlchemy/Motor:** Clases que implementan `IBookingRepository` usando `AsyncSession` para interactuar con MariaDB o MongoDB.
    * **Publicadores de Eventos Kafka/RabbitMQ:** Clases que implementan `IEventPublisher` para tomar `Domain Events` (o `Integration Events` derivados de ellos) y publicarlos en el `message broker`.
    * **Clientes HTTP/gRPC:** Si el `BookingService` necesita llamar, por ejemplo, al `ParkingLotService` de forma síncrona, podría hacerlo a través de un `Port` como `IParkingLotQueryPort`, y habría un `ParkingLotHttpClientAdapter` que implementaría este `port` haciendo una llamada HTTP real.

**3. Estructura de Directorios Sugerida para un `Microservice` ParkWise (ej. `BookingService`):**\
Esta estructura busca reflejar la separación hexagonal.

```
booking_service/
├── app/
│   ├── main.py             # FastAPI app setup, lifespan events, DI para adaptadores
│   ├── core/               # Configuración global del servicio, settings
│   │   └── config.py
│   │
│   ├── domain/             # EL CORAZÓN: Lógica y modelos de dominio puros
│   │   ├── __init__.py
│   │   ├── aggregates/     # ej. booking_aggregate.py (clase Booking)
│   │   ├── entities/       # Si hay entities que no son aggregate roots
│   │   ├── value_objects/  # ej. time_slot.py, money.py
│   │   ├── domain_services/ # Lógica de dominio que no encaja en un aggregate
│   │   ├── events.py       # Definiciones de Domain Events (ej. BookingConfirmedDomainEvent)
│   │   └── ports/          # --- PUERTOS DE SALIDA (Interfaces que el dominio necesita) ---
│   │       └── output/
│   │           ├── __init__.py
│   │           ├── booking_repository.py  (Define IBookingRepository)
│   │           └── event_publisher.py     (Define IEventPublisher)
│   │
│   ├── application/        # Orquesta el dominio, implementa casos de uso
│   │   ├── __init__.py
│   │   ├── use_cases/      # Implementaciones concretas de los casos de uso
│   │   │   ├── __init__.py
│   │   │   └── create_booking_use_case.py (Implementa ICreateBookingUseCase)
│   │   ├── services.py     # (Opcional) Application services si son distintos a los use cases
│   │   └── ports/          # --- PUERTOS DE ENTRADA (Interfaces de los casos de uso) ---
│   │       └── input/
│   │           ├── __init__.py
│   │           └── booking_use_cases.py (Define ICreateBookingUseCase, etc.)
│   │   └── dtos.py         # DTOs usados por los casos de uso (internos a la app core, no necesariamente los de API)
│   │
│   └── infrastructure/     # --- ADAPTADORES e implementación de detalles externos ---
│       ├── __init__.py
│       ├── api/            # Adaptadores de entrada HTTP y WebSocket (FastAPI routers)
│       │   ├── __init__.py
│       │   └── v1/
│       │       ├── __init__.py
│       │       └── booking_endpoints.py # Usa ICreateBookingUseCase, etc.
│       ├── persistence/    # Adaptadores de salida para persistencia
│       │   ├── __init__.py
│       │   └── sqlalchemy_booking_repository.py # Implementa IBookingRepository
│       ├── messaging/      # Adaptadores para Pub/Sub
│       │   ├── __init__.py
│       │   ├── kafka_event_publisher.py   # Implementa IEventPublisher
│       │   └── kafka_event_consumers.py # Adaptadores de entrada si consume eventos
│       └── dependencies.py # Configuración de DI de FastAPI para inyectar adaptadores a los endpoints,
                            # y adaptadores de infraestructura a los casos de uso (si aplica).
├── tests/
│   ├── unit/
│   │   ├── domain/
│   │   └── application/
│   ├── integration/
│   │   ├── persistence/
│   │   └── api/
│   └── conftest.py
└── pyproject.toml
```

* **Flujo de Dependencias:** `Infrastructure` depende de `Application` y `Domain`. `Application` depende de `Domain`. **`Domain` no depende de nada más que de sí mismo (y quizás tipos Python estándar).**

**4. Diagrama `Mermaid`: Arquitectura Hexagonal para un `Microservice` ParkWise (ej. `BookingService`)**

```mermaid
graph TD
    subgraph OutsideWorld [Mundo Exterior / Otros Servicios]
        direction LR
        HTTPClient["HTTP Client<br/>(API Gateway / User Browser)"]
        KafkaConsumerTrigger["Kafka Broker<br/>(Eventos de otros servicios)"]
    end

    subgraph BookingMicroservice ["Booking Microservice (Hexágono)"]
        direction LR
        
        subgraph Adapters_Infra ["Capa de Infraestructura (Adaptadores)"]
            direction TB
            FastAPIAdapter["FastAPI REST/WS Adapter<br/>(booking_endpoints.py)"]
            KafkaConsumerAdapter["Kafka Consumer Adapter<br/>(Consumidor de eventos externos)"]
            SQLAlchemyRepoAdapter["SQLAlchemy Booking Repository Adapter<br/>(Implementa IBookingRepository)"]
            KafkaEventPubAdapter["Kafka Event Publisher Adapter<br/>(Implementa IEventPublisher)"]
            ParkingAPIClientAdapter["Parking Service HTTP Client Adapter<br/>(Implementa IParkingQueryPort)"]
        end

        subgraph AppCore ["Núcleo de Aplicación (Puertos y Lógica)"]
            direction TB
            PortIn_UseCase1["Input Port: ICreateBookingUseCase"]
            PortIn_UseCase2["Input Port: IProcessPaymentUpdateUseCase"]
            
            AppService_UseCaseImpl["Application Services / Use Cases<br/>(Implementan Input Ports, orquestan Dominio)"]
            
            DomainModel["Domain Model (DDD)<br/>BookingAggregate, Domain Events,<br/>Value Objects, Domain Services"]

            PortOut_Repo["Output Port: IBookingRepository"]
            PortOut_EventPub["Output Port: IEventPublisher"]
            PortOut_Parking["Output Port: IParkingLotQueryPort"]
        end
        
        FastAPIAdapter -- Invoca --> PortIn_UseCase1
        KafkaConsumerAdapter -- Invoca --> PortIn_UseCase2

        PortIn_UseCase1 --> AppService_UseCaseImpl
        PortIn_UseCase2 --> AppService_UseCaseImpl
        
        AppService_UseCaseImpl -- Usa --> DomainModel
        AppService_UseCaseImpl -- Usa --> PortOut_Repo
        AppService_UseCaseImpl -- Usa --> PortOut_EventPub
        AppService_UseCaseImpl -- Usa --> PortOut_Parking

        PortOut_Repo -- Es implementado por --> SQLAlchemyRepoAdapter
        PortOut_EventPub -- Es implementado por --> KafkaEventPubAdapter
        PortOut_Parking -- Es implementado por --> ParkingAPIClientAdapter

    end
    
    HTTPClient --> FastAPIAdapter
    KafkaConsumerTrigger --> KafkaConsumerAdapter

    SQLAlchemyRepoAdapter --> MariaDB_Booking[(MariaDB para Booking)]
    KafkaEventPubAdapter --> KafkaBroker_Egress["Kafka Broker (Publica Eventos de Dominio/Integración)"]
    ParkingAPIClientAdapter --> ParkingServiceAPI_Ext["Parking Lot Service API (Externa)"]

    style DomainModel fill:#E8DAEF,stroke:#8E44AD,stroke-width:3px,color:#000
    style AppService_UseCaseImpl fill:#FAD7A0,stroke:#F39C12,stroke-width:2px
    style Adapters_Infra fill:#D6DBDF,stroke:#2C3E50
```

**5. Beneficios de `DDD` + Hexagonal para Cada `Microservice` ParkWise:**

* **`Microservices` Cohesivos y Enfocados:** Cada servicio tiene un modelo de dominio claro para su `bounded context`.
* **Alta Testabilidad:** El `domain` y `application core` (el hexágono) se prueban unitariamente en completo aislamiento, `mockeando` los `output ports`. Los `adapters` se prueban en integración.
* **Flexibilidad Tecnológica en los `Adapters`:** Puedes cambiar de MariaDB a PostgreSQL (modificando el `repository adapter`) o de Kafka a RabbitMQ (modificando el `event publisher/consumer adapter`) con un impacto mínimo en el `core business logic`.
* **Mantenibilidad y Evolución:** La clara separación de incumbencias facilita entender, modificar y extender cada `microservice` de forma independiente.
* **Resiliencia:** Un `adapter` fallido (ej. no se puede conectar a Kafka) no debería corromper la lógica de `domain` (aunque el `use case` podría fallar y necesitar lógica de compensación).

**Conclusión: Construyendo Fortalezas de Lógica Interna – `DDD` y Hexagonal Como Escudo y Espada**

Aplicar los principios de `Domain-Driven Design` y la Arquitectura Hexagonal _dentro_ de cada `microservice` de ParkWise es una inversión estratégica. Asegura que, aunque el sistema general es distribuido y potencialmente complejo, cada componente individual es:

* **Robusto:** Con una lógica de negocio bien definida y protegida.
* **Bien Estructurado:** Con claras separaciones entre el dominio, la aplicación y la infraestructura.
* **Adaptable:** Capaz de evolucionar sus tecnologías de infraestructura sin reescrituras masivas del `core`.

FastAPI se integra perfectamente en este modelo, actuando como un `driving adapter` eficiente para las interacciones HTTP y `WebSocket`, mientras que su sistema de `Dependency Injection` facilita la conexión de estos `adapters` con los `ports` de entrada de tu `application core`.



## 16.5 Desarrollo de API Gateway y orquestador de eventos

Avanzamos al **16.5** del proyecto **ParkWise**. Habiendo definido la estructura interna de nuestros `microservices` con DDD y Arquitectura Hexagonal (16.4), ahora necesitamos pensar en cómo los `clients` externos acceden a nuestro ecosistema y cómo se coordinan los flujos de negocio más complejos que involucran múltiples servicios, especialmente los asíncronos.

Aquí es donde entran en juego dos componentes arquitectónicos clave: el **`API Gateway`** y un concepto que llamaremos **`Event Orchestrator/Processor`** para manejar flujos de `events` sofisticados.

***

En una arquitectura de `microservices` como la de ParkWise, tener docenas de servicios exponiendo sus propias APIs directamente a los `clients` (web, móviles) puede ser una pesadilla en términos de gestión, seguridad y experiencia del `client`. De manera similar, los flujos de negocio que requieren la coordinación de varios `microservices` a través de `events` pueden necesitar un punto de inteligencia más allá de simples reacciones `publish/subscribe`.

**1. El `API Gateway` de ParkWise: El Guardián y Conserje Principal del Ecosistema 🤵**

* **Propósito y Beneficios en ParkWise:**\
  Un `API Gateway` se situará como el **único punto de entrada (`single entry point`)** para todas las peticiones de los `clients` externos a la plataforma ParkWise.
  * **`Request Routing`:** Dirige las peticiones entrantes al `microservice` interno apropiado. Ej: `/api/v1/users/...` va al `IdentityService`, `/api/v1/spots/...` va al `ParkingLotService`.
  * **Abstracción y Fachada (`Facade`):** Oculta la complejidad de la arquitectura de `microservices` interna a los `clients`. Los `clients` interactúan con una API unificada.
  * **`Authentication` y `Authorization` Centralizadas (o Primer Nivel):** Puede manejar la validación inicial de `tokens` (ej. `JWTs`) y aplicar reglas de `authorization` globales antes de que la petición llegue a un `microservice`. (Ver 11.8).
  * **`Rate Limiting` y `Throttling`:** Protege los servicios `backend` de ser sobrecargados.
  * **`SSL Termination`:** Maneja la terminación SSL/TLS, liberando a los `microservices` internos de esta tarea.
  * **`Request/Response Transformation`:** Puede adaptar `payloads` o `headers` si es necesario.
  * **`Caching` de Respuestas Comunes:** Puede cachear respuestas de `endpoints` frecuentemente accedidos.
  * **`Logging` y `Tracing` Centralizados:** Un punto ideal para loguear todas las peticiones entrantes e iniciar `distributed traces`.
  * **`Backend-For-Frontend (BFF)` (Opcional):** El `API Gateway` podría incluso tener variantes específicas para diferentes tipos de `clients` (ej. un `Mobile BFF`, un `WebApp BFF`), cada uno exponiendo una API optimizada para las necesidades de ese `frontend`, agregando o adaptando datos de múltiples `microservices`.
* **Implementación Conceptual para ParkWise:**
  * **Tecnología:** Para mantener la coherencia con nuestro `stack` y para fines didácticos, podríamos conceptualizar el `API Gateway` de ParkWise como **una aplicación FastAPI dedicada**. Esta `app` FastAPI no contendría lógica de negocio `core`, sino que su principal función sería enrutar, autenticar y posiblemente transformar peticiones a los otros `microservices` ParkWise (usando un `HTTP client` asíncrono como `httpx`).
  * En un escenario de producción a gran escala, se podría optar por soluciones de `API Gateway` gestionadas o `software` especializado: Kong, Tyk, AWS API Gateway, Google Cloud API Gateway, Azure API Management, NGINX Plus, Envoy.
  *   **Rutas Ejemplo en el `FastAPI Gateway`:**

      ```
      /gateway/v1/auth/...                 -> IdentityService (para login, register)
      /gateway/v1/users/me                 -> IdentityService 
      /gateway/v1/parking-spots/search     -> ParkingLotService
      /gateway/v1/parking-spots/{spot_id}  -> ParkingLotService
      /gateway/v1/providers/spots/...      -> ParkingLotService (para gestión de providers)
      /gateway/v1/bookings/                -> BookingService
      /gateway/v1/payments/process         -> PaymentService
      /gateway/v1/reviews/spot/{spot_id}   -> ReviewService
      ```
* **Diagrama : `API Gateway` en ParkWise**

**2. El "Orquestador de `Events`" (`Event Orchestrator/Processor`) de ParkWise: Dirigiendo Procesos Asíncronos Complejos 🎭**

El término "orquestador de `events`" puede ser un poco ambiguo. No nos referimos al `message broker` (Kafka/RabbitMQ) en sí mismo, sino a un **componente lógico o servicio dedicado** que implementa flujos de negocio más complejos que se desencadenan por `events` y que pueden involucrar múltiples pasos o la coordinación de varios `microservices` de forma asíncrona.

* **Clarificando el Rol en ParkWise:**
  * **No es simple `Pub/Sub` reactivo:** Va más allá de que un servicio simplemente reaccione a un `event` de otro.
  * **Gestión de `Workflows`** **`Event-Driven`:** Si un `event` inicia un proceso de negocio que requiere una secuencia específica de acciones en otros servicios, o decisiones basadas en los resultados de pasos intermedios (todo de forma asíncrona).
  * **Sagas Orquestadas (como en 13.5):** Si optamos por el patrón de `Saga` orquestada para ciertas transacciones distribuidas (ej. un proceso de `onboarding` de `provider` muy complejo), el orquestador de la `Saga` sería un ejemplo de este componente.
  * **Procesamiento Complejo de `Events` (`Complex Event Processing - CEP`):** Consumir múltiples `streams` de `events`, correlacionarlos, detectar `patterns` y generar `events` de nivel superior o `trigger` acciones.
* **Implementación Conceptual para ParkWise:**
  * Podría ser un **`microservice` Python dedicado** (posiblemente usando FastAPI para una API de gestión interna si es necesario, o simplemente un consumidor robusto de Kafka/RabbitMQ).
  * Este servicio se suscribiría a `events` de negocio clave de otros `microservices` de ParkWise.
  * Su lógica interna implementaría la "coreografía inteligente" o la "orquestación" del `workflow`.
* **Ejemplo de Flujo Orquestado/Procesado en ParkWise: "Provider Onboarding & Spot Verification"**
  1. `IdentityService` publica `ProviderRegisteredEvent`.
  2. Un `ProviderOnboardingOrchestrator` consume `ProviderRegisteredEvent`.
  3. `Orchestrator` emite un `Command` (o `event`) para el (futuro) `VerificationService` para iniciar la verificación de documentos del `provider`.
  4. `ParkingLotService` publica `InitialSpotListingPendingReviewEvent` cuando un nuevo `provider` lista su primer `spot`.
  5. El `ProviderOnboardingOrchestrator` consume este `event`.
  6. Si la verificación del `provider` (del paso 3) ya fue exitosa, el `Orchestrator` emite un `Command` al `ParkingLotService` para `ApproveSpotListingCommand`.
  7. Si no, espera o maneja el flujo.
  8. Una vez que el `provider` y el primer `spot` están aprobados, el `Orchestrator` podría emitir un `ProviderFullyOnboardedEvent` que el `NotificationService` usa para enviar un email de bienvenida completo.
* **Diagrama `Mermaid`: `Event Orchestrator/Processor` Conceptual en ParkWise**

**Tecnologías para Implementar el `API Gateway` y el `Event Orchestrator/Processor` en ParkWise:**

* **`API Gateway`:**
  * **FastAPI mismo:** Para un `gateway` ligero o `BFF`, una `app` FastAPI puede actuar como `proxy` y orquestador de llamadas síncronas, usando `httpx` para comunicarse con los `microservices` `backend`.
  * **Soluciones Dedicadas:** Para necesidades más avanzadas (gestión de tráfico sofisticada, `plugins` de seguridad, `analytics` de API), se consideraría: Kong, Tyk, NGINX Plus, Envoy, o servicios `cloud` (AWS API Gateway, Azure API Management, Google Cloud API Gateway).
  * **Para ParkWise (enfoque didáctico):** Conceptualizaremos que una o más `FastAPI applications` pueden tomar el rol de `gateway` o `BFF`.
* **`Event Orchestrator/Processor`:**
  * Un **`microservice` Python/FastAPI** que consume de Kafka (usando `aiokafka`) es una opción viable y consistente con el `stack`.
  * Para procesamiento de `streams` más complejos o `stateful`: **Kafka Streams**, **Apache Flink**, o librerías Python como **Faust**.
  * Para `workflow orchestration` de larga duración con estado explícito, `retries` y `timeouts` complejos: **Temporal.io**, **Camunda Platform**.
  * **Para ParkWise (enfoque didáctico):** Se modelará como un consumidor Kafka (Python/`aiokafka`) con lógica de decisión que puede emitir nuevos `events` o `commands`.

**Conclusión: Puntos Centrales de Control para un Ecosistema Coherente y Escalable**

El `API Gateway` y un `Event Orchestrator/Processor` son componentes arquitectónicos que aportan orden y eficiencia a un sistema de `microservices` como ParkWise.

* El **`API Gateway`** actúa como el embajador y protector de tu sistema de cara al mundo exterior, simplificando la interacción del `client` y centralizando incumbencias transversales.
* El **`Event Orchestrator/Processor`** (o la lógica de orquestación distribuida en ciertos `consumers`) permite implementar `workflows` de negocio asíncronos más sofisticados que una simple cadena de `events`, asegurando que procesos complejos se completen correctamente.

Ambos ayudan a gestionar la complejidad inherente a las arquitecturas distribuidas, permitiendo que ParkWise escale, evolucione y mantenga una alta calidad de servicio.

***

### 16.6 Integración de DB relacional y no relacional

La idea es mostrar cómo diferentes `microservices` de ParkWise pueden elegir la tecnología de base de datos que mejor se adapte a sus necesidades (`polyglot persistence`), y cómo `Docker Compose` facilita tener estas bases de datos funcionando localmente para el desarrollo y las pruebas de integración.

***

En ParkWise, no todos los `microservices` tendrán las mismas necesidades de persistencia de datos. Algunos se beneficiarán de la estructura y las garantías ACID de una base de datos relacional (MariaDB será nuestra elección principal aquí), mientras que otros podrían requerir la flexibilidad o las capacidades de `query` específicas de una base de datos NoSQL (como MongoDB o Redis). Esto es la **persistencia políglota (`polyglot persistence`)**.

`Docker Compose` es nuestro aliado para simular este entorno diverso localmente desde el inicio del proyecto.

**1. Elección de `Datastores` por `Microservice` en ParkWise (Ejemplos):**

* **`Identity Service`:**
  * **Necesidades:** Almacenar `user profiles`, credenciales `hashed`, `roles`, `permissions`. Relaciones claras entre `users` y `roles`.
  * **Elección:** **MariaDB**. Ideal por su naturaleza transaccional y estructurada para estos datos críticos.
  * **Tecnología de Acceso:** SQLAlchemy ORM (como en 13.1).
* **`Parking Lot Service`:**
  * **Necesidades:**
    * Datos estructurales de `parking spots` (ID, `provider_id`, dirección, tipo, `amenities` base).
    * Información geoespacial (`GeoJSON` para ubicación) para búsquedas por proximidad.
    * Gestión de disponibilidad en tiempo real (podría ser muy intensivo en escrituras/lecturas).
  * **Elección:**
    * **MariaDB:** Para los datos estructurales y relacionales.
    * **(Opción A) MariaDB con `GIS extensions`:** Para `queries` geoespaciales.
    * **(Opción B) MongoDB:** Si las `queries` geoespaciales son muy complejas o si los atributos de los `spots` varían enormemente, MongoDB podría ser una mejor opción para la parte de "catálogo" de `spots`.
    * **(Opción C) Redis:** Para el `tracking` de disponibilidad en tiempo real de `spots` individuales (ej. contadores o `sets` para `available/occupied spots`). Esto reduce la carga en la base de datos principal para `updates` muy frecuentes.
  * **Tecnología de Acceso:** SQLAlchemy (MariaDB), `Motor` (MongoDB), `aioredis` (Redis).
  * _Para mantener la simplicidad inicial del proyecto con MariaDB como base, nos enfocaremos en MariaDB con sus capacidades GIS para `Parking Lot Service`, y mencionaremos Redis como un `cache/real-time store` opcional._
* **`Booking Service`:**
  * **Necesidades:** Almacenar reservas, sus `status`, `timeslots`, relación con `users` y `spots`. Transacciones importantes.
  * **Elección:** **MariaDB**. Ideal por las relaciones y la necesidad de consistencia transaccional.
  * **Tecnología de Acceso:** SQLAlchemy ORM.
* **`Payment Service`:**
  * **Necesidades:** Registrar `transactions`, `payment status`, `payouts`. Alta consistencia y auditoría.
  * **Elección:** **MariaDB**.
  * **Tecnología de Acceso:** SQLAlchemy ORM.
* **`Notification Service`:**
  * **Necesidades:** Almacenar `notification templates`, historial de notificaciones (opcional), `user notification preferences`.
  * **Elección:** **MariaDB** (para `templates` y preferencias). El historial podría ir a una NoSQL si el volumen es masivo, pero empezamos con MariaDB.
  * **Tecnología de Acceso:** SQLAlchemy ORM.
* **`Review Service`:**
  * **Necesidades:** Almacenar `reviews` (texto), `ratings` (numérico), relación con `users` y `spots`. Potencial para `nested comments` o estructuras flexibles.
  * **Elección:** **MongoDB** podría ser una buena opción aquí por la flexibilidad del contenido de las `reviews` y las estructuras anidadas. Alternativamente, MariaDB con `JSON fields` o una estructura relacional simple.
  * **Tecnología de Acceso:** `Motor` (MongoDB) o SQLAlchemy (MariaDB).
  * _Para el proyecto, decidimos usar **MongoDB** para el `Review Service` para ilustrar la persistencia políglota._

**2. Configuración en `docker-compose.yml` para Soportar Múltiples `Datastores`:**

Nuestro archivo `docker-compose.yml` (como el que empezamos a esbozar en 15.3) se expandiría para incluir estos `datastores`.

```yaml
# docker-compose.yml (extracto relevante para datastores)
version: '3.8' # O la versión que estés usando

services:
  # --- Microservice: Identity Service (Usa MariaDB) ---
  identity_service:
    build: ./identity_service # Asume Dockerfile en ese directorio
    ports:
      - "8001:8000"
    environment:
      - DATABASE_URL=mysql+aiomysql://parkwise_user:parkwise_pass@mariadb_identity:3306/identity_db 
      # (aiomysql es el driver async para MariaDB/MySQL con SQLAlchemy)
    depends_on:
      mariadb_identity:
        condition: service_healthy
    networks:
      - parkwise_network

  # --- Microservice: Parking Lot Service (Usa MariaDB, opcionalmente Redis) ---
  parking_lot_service:
    build: ./parking_lot_service
    ports:
      - "8002:8000"
    environment:
      - DATABASE_URL=mysql+aiomysql://parkwise_user:parkwise_pass@mariadb_parking:3306/parking_db
      - REDIS_URL=redis://redis_cache:6379/0
    depends_on:
      mariadb_parking:
        condition: service_healthy
      redis_cache: # Si se usa Redis
        condition: service_started # Redis es rápido en iniciar
    networks:
      - parkwise_network

  # --- Microservice: Booking Service (Usa MariaDB) ---
  booking_service:
    build: ./booking_service
    ports:
      - "8003:8000"
    environment:
      - DATABASE_URL=mysql+aiomysql://parkwise_user:parkwise_pass@mariadb_booking:3306/booking_db
    depends_on:
      mariadb_booking:
        condition: service_healthy
    networks:
      - parkwise_network

  # --- Microservice: Review Service (Usa MongoDB) ---
  review_service:
    build: ./review_service
    ports:
      - "8004:8000"
    environment:
      - MONGO_URL=mongodb://mongo_review:27017
      - MONGO_DATABASE_NAME=review_db
    depends_on:
      mongo_review:
        condition: service_started # O un healthcheck más robusto si es necesario
    networks:
      - parkwise_network

  # --- Databases ---
  mariadb_identity:
    image: mariadb:10.11 # Usar una versión específica
    environment:
      MYSQL_ROOT_PASSWORD: root_password_secure
      MYSQL_DATABASE: identity_db
      MYSQL_USER: parkwise_user
      MYSQL_PASSWORD: parkwise_pass
    volumes:
      - identity_db_data:/var/lib/mysql
    ports: # Exponer solo si necesitas acceso directo desde el host, y con puerto diferente
      - "3301:3306"
    healthcheck:
      test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost", "-u$$MYSQL_USER", "-p$$MYSQL_PASSWORD"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - parkwise_network

  mariadb_parking: # Similar a mariadb_identity, pero para parking_db
    image: mariadb:10.11
    environment:
      MYSQL_ROOT_PASSWORD: root_password_secure
      MYSQL_DATABASE: parking_db
      MYSQL_USER: parkwise_user
      MYSQL_PASSWORD: parkwise_pass
    volumes:
      - parking_db_data:/var/lib/mysql
    ports:
      - "3302:3306"
    healthcheck: # Idéntico healthcheck
      test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost", "-u$$MYSQL_USER", "-p$$MYSQL_PASSWORD"]
      interval: 10s; timeout: 5s; retries: 5
    networks:
      - parkwise_network

  mariadb_booking: # Similar, para booking_db
    image: mariadb:10.11
    environment:
      MYSQL_ROOT_PASSWORD: root_password_secure
      MYSQL_DATABASE: booking_db
      MYSQL_USER: parkwise_user
      MYSQL_PASSWORD: parkwise_pass
    volumes:
      - booking_db_data:/var/lib/mysql
    ports:
      - "3303:3306"
    healthcheck: # Idéntico healthcheck
      test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost", "-u$$MYSQL_USER", "-p$$MYSQL_PASSWORD"]
      interval: 10s; timeout: 5s; retries: 5
    networks:
      - parkwise_network
      
  mongo_review:
    image: mongo:6.0 # Usar una versión específica
    volumes:
      - review_db_data:/data/db
    ports: # Exponer solo si necesitas acceso directo desde el host
      - "27017:27017" 
      # MongoDB tiene un healthcheck más simple por defecto si no se especifica,
      # o se puede añadir uno con 'mongosh --eval "db.adminCommand("ping")"'
    networks:
      - parkwise_network

  redis_cache: # Si el Parking Lot Service lo usa
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - parkwise_network

# Definir volúmenes nombrados para persistencia de datos
volumes:
  identity_db_data:
  parking_db_data:
  booking_db_data:
  review_db_data:

# Definir la red compartida
networks:
  parkwise_network:
    driver: bridge
```

**Puntos Clave de esta Configuración `Docker Compose`:**

* **Bases de Datos Separadas:** Cada servicio que necesita su propia persistencia tiene una `service definition` para su base de datos (ej. `mariadb_identity`, `mariadb_parking`, `mongo_review`).
* **Nombres de Servicio para Conexión:** Los `microservices` se conectan a sus bases de datos usando el nombre del servicio definido en `docker-compose.yml` como el `hostname` (ej. `DATABASE_URL=...@mariadb_identity:3306/...`). Docker Compose maneja la resolución de DNS dentro de la red personalizada `parkwise_network`.
* **`Environment Variables` para Configuración:** Las `database URLs`, `usernames`, `passwords` se pasan a los `microservices` y a los `containers` de base de datos a través de `environment variables`. (En producción, esto se manejaría con `secrets managers`).
* **`Volumes` para Persistencia:** Se usan `named volumes` (ej. `identity_db_data`) para asegurar que los datos de la base de datos persistan incluso si los `containers` se detienen y se reinician.
* **`Healthchecks`:** Se definen `healthchecks` para las bases de datos MariaDB para que los servicios que dependen de ellas (`depends_on: condition: service_healthy`) esperen hasta que estén listas antes de arrancar. MongoDB y Redis también pueden tener `healthchecks`.
* **Red Personalizada (`parkwise_network`):** Todos los servicios están en la misma red `bridge`, lo que facilita la comunicación entre ellos por nombre de servicio.

**3. Integración en el Código del `Microservice` (Recordatorio):**

*   **`IdentityService` (SQLAlchemy):**

    ```python
    # app/core/config.py
    # DATABASE_URL = os.getenv("DATABASE_URL", "mysql+aiomysql://user:pass@localhost:3301/identity_db") 
    # ...
    # app/db/session.py -> usa esta DATABASE_URL para crear el async_engine
    ```
*   **`ReviewService` (`Motor`):**

    ```python
    # app/core/config.py
    # MONGO_URL = os.getenv("MONGO_URL", "mongodb://localhost:27017")
    # MONGO_DATABASE_NAME = os.getenv("MONGO_DATABASE_NAME", "review_db")
    # ...
    # app/db/mongo_db.py -> usa estas variables para crear el AsyncIOMotorClient
    ```

**4. Diagrama `Mermaid`: Persistencia Políglota en ParkWise con `Docker Compose`**

```mermaid
graph TD
    subgraph "Docker Host (Entorno Local)"
        DC["docker-compose.yml"]

        subgraph "Containers Gestionados por Compose"
            Net[(parkwise_network)]
            
            subgraph IdentityServiceContainer["Identity Service (FastAPI)"]
                IS_App[App Code]
            end
            subgraph ParkingLotServiceContainer["Parking Lot Service (FastAPI)"]
                PLS_App[App Code]
            end
            subgraph ReviewServiceContainer["Review Service (FastAPI)"]
                RS_App[App Code]
            end

            subgraph MariaDB_Identity_Container["mariadb_identity (MariaDB)"]
                MDB_I_Data[(identity_db_data)]
            end
            subgraph MariaDB_Parking_Container["mariadb_parking (MariaDB)"]
                MDB_P_Data[(parking_db_data)]
            end
            subgraph MongoDB_Review_Container["mongo_review (MongoDB)"]
                Mongo_R_Data[(review_db_data)]
            end
            subgraph RedisContainer["redis_cache (Redis)"]
                Redis_Icon[Redis]
            end
            
            IdentityServiceContainer -- "mysql+aiomysql://...@mariadb_identity" --> MariaDB_Identity_Container
            ParkingLotServiceContainer -- "mysql+aiomysql://...@mariadb_parking" --> MariaDB_Parking_Container
            ParkingLotServiceContainer -- "redis://redis_cache..." --> RedisContainer
            ReviewServiceContainer -- "mongodb://mongo_review..." --> MongoDB_Review_Container

            MDB_I_Data -. Montado en .-> MariaDB_Identity_Container
            MDB_P_Data -. Montado en .-> MariaDB_Parking_Container
            Mongo_R_Data -. Montado en .-> MongoDB_Review_Container
            
            Net --- IdentityServiceContainer
            Net --- ParkingLotServiceContainer
            Net --- ReviewServiceContainer
            Net --- MariaDB_Identity_Container
            Net --- MariaDB_Parking_Container
            Net --- MongoDB_Review_Container
            Net --- RedisContainer
        end
        DC -. Define y Orquesta .-> DockerHost
    end
    style DC fill:#FFF9C4,stroke:#F1C40F
```

**Conclusión: `Docker Compose` Como Cimiento para la Persistencia Políglota Local**

Para el proyecto ParkWise, la capacidad de definir y ejecutar localmente un entorno con **múltiples tipos de bases de datos** (`polyglot persistence`) y múltiples instancias de la misma base de datos (una por servicio) es crucial para simular de cerca un entorno de producción de `microservices` y para permitir el desarrollo y prueba de la integración de cada servicio con su `datastore` elegido.

`Docker Compose` nos da exactamente eso:

* **Un `stack` de desarrollo completo y reproducible** con un solo comando (`docker compose up`).
* La capacidad de usar **MariaDB para servicios que se benefician de SQL** y **MongoDB para servicios que necesitan flexibilidad de esquema** (como el `ReviewService`).
* La opción de añadir otros `backing services` como Redis fácilmente.

Este `setup` local es el primer paso para asegurar que la lógica de persistencia de cada `microservice` ParkWise (usando SQLAlchemy para MariaDB y `Motor` para MongoDB, como se vio en 13.1 y 13.7) funcione correctamente antes de pasar a `pipelines` CI/CD y despliegues en Kubernetes.



## 16.7 Seguridad, validación y pruebas completas



Para el proyecto **ParkWise**, este punto define las estrategias integrales para asegurar la robustez, fiabilidad y seguridad de la plataforma a través de sus `microservices`.

1. **Seguridad (`Security`):**
   * **`Authentication`:**
     * El **`Identity Service`** gestionará la autenticación de `users` (`drivers`, `providers`, `admins`) y `machine-to-machine (M2M)` `clients` (otros servicios).
     * Se utilizarán **`JSON Web Tokens (JWT)`** (estándar `RS256` o `ES256`) como `access tokens` y `refresh tokens`.
     * El **`API Gateway`** (16.5) realizará la validación inicial de `JWTs` para peticiones externas. Los `microservices` internos podrán revalidar el `token` o confiar en la `authentication` del `gateway` (propagando la identidad del `user` a través de `headers` internos seguros o un `context token` simplificado).
     * Se implementará `OAuth2 Password Grant` para el `login` de `users` y `Client Credentials Grant` para `M2M authentication`.
   * **`Authorization`:**
     * Basada en `roles` y `permissions` definidos en el `Identity Service` e incluidos como `claims` en el `JWT`.
     * Cada `microservice` será responsable de aplicar `authorization checks` para sus `endpoints` y `resources` basándose en estos `claims`.
     * Se usarán `FastAPI dependencies` para la lógica de `authorization`.
   * **Seguridad de `Transport`:** `HTTPS/TLS` obligatorio para toda comunicación externa (API Gateway) e interna (`service-to-service`, `WebSocket Secure - WSS`).
   * **Seguridad de `Secrets`:** Gestión de `secrets` (credenciales de BBDD, `API keys` de terceros, `JWT secrets`) mediante un `secrets manager` (ej. HashiCorp Vault, o los provistos por el `cloud provider` en despliegues K8s).
   * **`Input Validation`:** (Ver abajo).
   * **Protección contra Vulnerabilidades Comunes OWASP:** Se aplicarán las mejores prácticas (ej. `rate limiting` en el `gateway`, `headers` de seguridad como `X-Content-Type-Options`, `CSP`, `HSTS`).
2. **Validación (`Validation`):**
   * **Validación de `Payloads` de API (Nivel de `Endpoint`):**
     * **Pydantic `models`** se usarán exhaustivamente en todos los `endpoints` REST y `WebSocket message handlers` de FastAPI en cada `microservice` para validar la estructura, tipos y `constraints` de los datos de entrada (como en 11.3). Esto incluye `nested models` para `payloads` complejos.
     * FastAPI devolverá automáticamente errores HTTP `422 Unprocessable Entity` con detalles.
   * **Validación de Reglas de Negocio (`Business Rule Validation`):**
     * Se realizará dentro de los `Application Services` o `Domain Entities/Aggregates` (como en 16.4).
     * Lanzarán excepciones de dominio específicas que los `adapters` (ej. `API endpoints`) traducirán a `HTTP error responses` apropiados (ej. `400 Bad Request`, `409 Conflict`).
   * **Validación de `Events`:**
     * Los `payloads` de los `events` publicados en Kafka también se definirán y validarán con Pydantic `models` (o Avro/Protobuf si se usa un `Schema Registry`) antes de la publicación por el `producer` y al ser consumidos por el `consumer`.
   * **`Database Schema Validation` (MongoDB):**
     * Para el `Review Service` (que usa MongoDB), se definirán `JSON Schemas` a nivel de `collection` para los `fields` críticos, complementando la validación de Pydantic en la aplicación (como en 13.8).
3. **Pruebas Completas (`Comprehensive Testing`):**\
   Se implementará una estrategia de `testing` multi-capa para ParkWise, automatizada en `CI pipelines` (como en 14.10 y 15.5).
   * **`Unit Tests` (14.2):**
     * Para la lógica de dominio (`aggregates`, `domain services`), `application services` (`use cases`), y `utility functions` dentro de cada `microservice`.
     * Se usará `pytest` y `unittest.mock` (o `pytest-mock`) para aislar las unidades, `mockeando` dependencias externas (repositorios, otros servicios, `event publishers`).
   * **Pruebas de Integración (`Integration Tests` - 14.5):**
     * **Nivel de Servicio:** Probar la interacción entre los `application services`, `repositories`, y una `test database` temporal (ej. MariaDB/MongoDB en Docker, SQLite `:memory:` para SQLAlchemy en algunos casos).
     * **Nivel de `Endpoint` (Intra-Servicio):** Usar `TestClient` de FastAPI para probar los `endpoints` REST y `WebSocket` de cada `microservice` individualmente, interactuando con su `test database`. Las dependencias de _otros_ `microservices` se `mockearán`.
   * **Pruebas de Contrato (`Contract Tests`):**
     * (Opcional, pero recomendado para madurez) Usar herramientas como Pact para verificar que los contratos entre `microservices` (APIs REST, `events`) se cumplen, sin necesidad de un entorno `E2E` completo para cada cambio de contrato.
   * **Pruebas `End-to-End (E2E)` (14.6):**
     * Para `user journeys` críticos que abarcan múltiples `microservices` (ej. un `driver` busca un `spot`, reserva, paga, y recibe notificación).
     * Se ejecutarán contra un entorno de `staging` o `E2E` dedicado con todos los `microservices` de ParkWise desplegados.
     * Usarán clientes HTTP (ej. `httpx`) para interactuar con el `API Gateway` y verificar los resultados y efectos secundarios a través de las APIs de los servicios o `event listeners` de prueba.
   * **Validación de `Events` y `Queues` en `Tests` (14.7):**
     * Pruebas específicas para verificar que los `producers` publican los `events` correctos a Kafka y que los `consumers` los procesan correctamente, usando `test topics` y `utility consumers/producers` en los `tests`.
   * **Cobertura de Pruebas (`Code Coverage` - 14.8):**
     * Se medirá con `coverage.py` (vía `pytest-cov`) y se buscará un umbral razonable (ej. >80-85%) para la lógica crítica, con informes generados en CI.
   * **Estructura y `Fixtures` (14.9):** Se seguirá una estructura de `tests` organizada y se usarán `pytest fixtures` extensivamente para `setup/teardown` y datos de prueba.

Este enfoque integral de seguridad, validación y `testing` es fundamental para garantizar que ParkWise sea una plataforma fiable, segura y de alta calidad.



### 16.8 Colas con Kafka y WebSockets para real-time



En el proyecto ParkWise, la comunicación en tiempo real (`real-time`) y los flujos de `events` desacoplados son esenciales. Para esto, **Kafka** será nuestro `message broker` principal para la comunicación asíncrona entre `microservices` y para alimentar las actualizaciones `real-time` a los `clients` a través de **`WebSockets`**. El entorno de desarrollo local se gestionará con **`Docker Compose`**.

1. **Kafka como `Event Backbone` de ParkWise:**
   * **Rol:** Kafka manejará todos los `domain events` y `integration events` significativos publicados por los `microservices` de ParkWise (ej. `ParkingSpotListedEvent`, `BookingConfirmedEvent`, `PaymentProcessedEvent`, `RealtimeAvailabilityUpdate` para un `spot`).
   * **`Topics` Kafka:** Se definirán `topics` específicos para diferentes tipos de `events` o dominios de negocio, permitiendo a los `microservices` suscribirse solo a los `streams` de `events` que les conciernen.
     * Ejemplos de `Topics`: `parkwise.parking.availability`, `parkwise.booking.lifecycle`, `parkwise.payment.status`, `parkwise.user.notifications`.
   * **`Event Payloads`:** Los `events` publicados en Kafka tendrán `payloads` `JSON` bien definidos (validados con Pydantic antes de la publicación y al consumir), incluyendo `metadata` como `event_id`, `timestamp`, `source_service`, y `correlation_id` para trazabilidad.
2. **`WebSockets` para Interacción `Real-Time` con `Clients`:**
   * **Casos de Uso en ParkWise:**
     * **Disponibilidad de `Parking Spots`:** Los `clients` (mapas en la app web/móvil) se suscribirán (vía `WebSocket`) para recibir `updates` en tiempo real sobre la disponibilidad de `spots` en un área o para `spots` específicos.
     * **Notificaciones de Reserva:** Confirmaciones, recordatorios, `updates` de estado de una reserva.
     * **Alertas de `Provider`:** Notificaciones a los `providers` sobre nuevas reservas para sus `spots`.
     * **(Futuro) Chat `Provider-Driver`:** Para coordinar llegadas, etc.
   * **Gestión:** El `Notification Service` (o `FastAPI instances` con capacidad de `gateway WebSocket`) manejará las `WebSocket connections` y la lógica de `rooms/subscriptions` (como en 10.3 y 10.4).
3. **Integración Kafka -> `WebSockets` para `Push` de `Events` a `Clients`:**
   * El `Notification Service` (o un componente dedicado dentro de las `FastAPI gateway instances`) actuará como un **consumidor de Kafka** para `topics` relevantes.
   * Cuando el `Notification Service` consume un `event` de Kafka (ej. `ParkingSpotAvailabilityChangedEvent` con `spot_id: "S123", status: "OCCUPIED"`):
     1. Determina qué `WebSocket clients` (o `rooms`) están interesados en este `update` (ej. `clients` visualizando el mapa donde está S123, o `clients` con una reserva activa para S123).
     2. Formatea un `message` `WebSocket` apropiado (ej. `{"type": "SPOT_UPDATE", "spotId": "S123", "status": "OCCUPIED"}`).
     3. Publica este `message` `WebSocket` en el **`WebSocket Pub/Sub Backend`** (Redis, como vimos en 10.4, para `fan-out` a las `FastAPI instances` correctas).
     4. Las `FastAPI instances` con las `WebSocket connections` activas relevantes hacen `push` del `message` a los `clients`.
4. **`Docker Compose` para el Entorno Local de Desarrollo y Pruebas:**\
   El archivo `docker-compose.yml` de ParkWise incluirá:
   * **Servicio Kafka:**
     * Usará una imagen oficial de Kafka (ej. `confluentinc/cp-kafka` o `bitnami/kafka`).
     * Requerirá un servicio **ZooKeeper** (para versiones de Kafka que lo necesiten) o se configurará en modo **KRaft** (para versiones más nuevas de Kafka que eliminan la dependencia de ZooKeeper). Para simplicidad en un entorno local, una configuración Kafka + ZooKeeper es común, o una imagen "todo-en-uno" si existe y es estable.
     * Se definirán variables de entorno para configurar los `brokers`, `topics` por defecto (o se crearán mediante `scripts` de inicialización o por los `producers/consumers` al arrancar).
   * **`Microservices` FastAPI (`Producers` y `Consumers` de Kafka):**
     * Cada `microservice` (Identity, Parking Lot, Booking, Payment, Notification, Review) tendrá su propia `service definition` en `Docker Compose`.
     * Se les pasarán las URLs del `broker` Kafka y los nombres de los `topics` a través de `environment variables`.
     * Los `microservices` que actúen como `WebSocket gateways` (o el `Notification Service` si es separado) también tendrán la configuración para el `WebSocket Pub/Sub Backend` (ej. URL de Redis).
   * **Redis (para `WebSocket Pub/Sub Backend` y `Caching` opcional):**
     * Un servicio Redis.
   * **Bases de Datos (MariaDB, MongoDB):**
     * Como se definió en 13.9 y 16.6, cada `microservice` con persistencia propia tendrá su `container` de base de datos o se conectará a uno compartido para su `bounded context`.
   *   **Ejemplo de Sección Kafka y ZooKeeper en `docker-compose.yml` (Simplificado):**

       ```yaml
       # docker-compose.yml (extracto)
       services:
         zookeeper:
           image: confluentinc/cp-zookeeper:7.3.2 # Usar versión específica
           container_name: parkwise_zookeeper
           environment:
             ZOOKEEPER_CLIENT_PORT: 2181
             ZOOKEEPER_TICK_TIME: 2000
           networks:
             - parkwise_network

         kafka:
           image: confluentinc/cp-kafka:7.3.2 # Usar versión específica
           container_name: parkwise_kafka
           depends_on:
             - zookeeper
           ports: # Exponer puerto para acceso desde el host si es necesario para debug/tools
             - "9092:9092" # Puerto interno de Kafka en el container
           environment:
             KAFKA_BROKER_ID: 1
             KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
             KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
             KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092 
             # PLAINTEXT://kafka:29092 es para comunicación interna entre containers en la misma red Docker
             # PLAINTEXT_HOST://localhost:9092 es para que tu aplicación en el host pueda conectar
             KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
             KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
             KAFKA_CONFLUENT_LICENSE_TOPIC_REPLICATION_FACTOR: 1 # Para imágenes Confluent
             KAFKA_CONFLUENT_BALANCER_TOPIC_REPLICATION_FACTOR: 1 # Para imágenes Confluent
             # Podrías añadir KAFKA_CREATE_TOPICS para crear topics al inicio
             # KAFKA_CREATE_TOPICS: "parkwise.booking.events:1:1,parkwise.parking.events:1:1" (topic:partitions:replication)
           networks:
             - parkwise_network
         
         # ... tus servicios FastAPI (web, identity_service, booking_service, notification_service, etc.) ...
         # que se conectarán a 'kafka:29092' (o 'localhost:9092' si el productor/consumidor corre en el host)

         # ... tu servicio Redis ...

       # networks:
       #   parkwise_network:
       #     driver: bridge
       ```

       * **Nota sobre `KAFKA_ADVERTISED_LISTENERS`:** Es crucial para que los `clients` puedan conectarse a Kafka correctamente tanto desde dentro de la red Docker como (opcionalmente) desde el `host`. La configuración puede variar.
5. **Implementación en los `Microservices` ParkWise:**
   * **Productores (`Producers`):** Los `services` (ej. `BookingService`) usarán `aiokafka` (14.7, 9.9) para publicar `domain events` en los `topics` Kafka después de completar operaciones de negocio (idealmente usando el patrón `Transactional Outbox` - 13.5).
   * **Consumidores (`Consumers`):**
     * El `NotificationService` (o los `FastAPI WebSocket gateways`) consumirá `events` de Kafka con `aiokafka`.
     * Otros `microservices` también pueden consumir `events` para mantener sus propios `read models` o `trigger` lógica.
     * Los consumidores serán `async` y diseñados para ser idempotentes (14.7, 9.6).
   * **`WebSocket Handlers`:** La lógica en los `endpoints` `@app.websocket` gestionará las suscripciones de los `clients` (ej. a `updates` para `room_id` específicos) y usará el `ConnectionManager` y el `Pub/Sub backend` (Redis) para el `fan-out` de mensajes `real-time`.

**Diagrama `Mermaid` Simplificado del Flujo Kafka -> `WebSocket` en ParkWise (con `Docker Compose` implícito):**

```mermaid
graph TD
    Booking[Booking Service] -->|1. BookingConfirmedEvent| Kafka[Kafka Broker]
    Kafka -->|2. Evento recibido| Notif[Notification Service]
    Notif -->|3. Publica mensaje| Redis[Redis Pub/Sub]
    Redis -->|4. Push al cliente| WS[WebSocket del Usuario]

```

Este `setup` con `Docker Compose` permitirá a los desarrolladores de ParkWise:

* Levantar todo el `stack` (Kafka, ZooKeeper/KRaft, Redis, MariaDBs, MongoDB, y todos los `microservices` FastAPI) localmente con `docker compose up`.
* Desarrollar y probar los flujos `event-driven` y las interacciones `real-time` de manera integral.
* Asegurar que la configuración de conexión a Kafka y otros `backing services` sea correcta desde las primeras etapas.


### 16.9 Despliegue automatizado con CI/CD en Kubernetes

Habiendo definido cómo ParkWise utilizará Kafka y `WebSockets` dentro de un entorno `Docker Compose` para desarrollo (16.8), el siguiente paso es llevar esta complejidad a un entorno de producción o `staging` de manera automatizada y escalable. Esto nos lleva al despliegue en **Kubernetes** gestionado por `pipelines` de **CI/CD**.


Para el proyecto ParkWise, el objetivo es desplegar nuestros `microservices` (contenedores Docker) en un `cluster` de Kubernetes, y automatizar este proceso utilizando un `pipeline` de CI/CD. Esto sigue las prácticas modernas de DevOps para asegurar despliegues rápidos, fiables y consistentes.

1. **Kubernetes como Plataforma de Orquestación de ParkWise:**
   * **Elección:** Kubernetes (K8s) será la plataforma de orquestación para todos los `microservices` de ParkWise y sus `backing services` (Kafka, Redis, MariaDB, MongoDB – aunque las bases de datos en producción a menudo se gestionan como servicios externos al `cluster` K8s, como AWS RDS, Google Cloud SQL, o se despliegan con `StatefulSets` y operadores K8s dedicados).
   * **Beneficios para ParkWise:** Escalabilidad, `self-healing`, `rolling updates`, `service discovery` y `load balancing` para cada `microservice`.
2. **Artefactos de Despliegue para Kubernetes:**
   * Para cada `microservice` de ParkWise, se crearán `manifests` Kubernetes (o se gestionarán a través de Helm/Kustomize como vimos en 15.4):
     * **`Deployment`:** Para definir el estado deseado del `microservice` (ej. `image` a usar – versionada semánticamente como en 15.2, número de `replicas`, `update strategy`).
     * **`Service`:** Para exponer cada `microservice` dentro del `cluster` con un `DNS name` estable y `load balancing` (ej. `ClusterIP service`).
     * **`Ingress` (gestionado por un `API Gateway` o un `Ingress Controller`):** Para exponer ciertos servicios (como el `API Gateway` de ParkWise, o directamente algunos `endpoints` si no hay `gateway` para todo) al tráfico externo.
     * **`ConfigMap` y `Secret`:** Para gestionar la configuración y los `secrets` de cada `microservice` (URLs de base de datos, credenciales de Kafka, `API keys` de terceros).
     * **`PersistentVolumeClaim` (para `Stateful Services`):** Si las bases de datos o Kafka se despliegan dentro de K8s y necesitan almacenamiento persistente.
     * **`Health Probes` (`liveness` y `readiness`):** Definidas en los `Deployments` para que K8s sepa si los `Pods` de ParkWise están sanos y listos (15.10).
3. **`Pipeline` CI/CD para ParkWise (ej. con GitHub Actions, como en 15.5):**\
   El `pipeline` se encargará del ciclo completo desde el `commit` de código hasta el despliegue.
   * **`Trigger`:** `Push` a `main branch` (para despliegue a `staging`/`production`) o `merge` de `Pull Request`.
   * **Etapas del `Pipeline` CI:**
     1. **`Checkout Code`:** Obtener el código del `microservice` específico.
     2. **`Setup Environment`:** Configurar Python, instalar dependencias.
     3. **`Lint & Format Check`:** Ejecutar `flake8`, `black`, `isort`.
     4. **`Unit & Integration Tests`:** Ejecutar `pytest` (14.1-14.5, 14.7). Las pruebas de integración podrían necesitar `service containers` (Docker-in-Docker si el `runner` CI lo permite, o servicios proveídos por la plataforma CI) para bases de datos o `brokers` temporales.
     5. **`Code Coverage Check`:** Ejecutar `pytest --cov` y fallar si el `coverage` es inferior al umbral (14.8).
     6. **`Build Docker Image`:** Construir la `Docker image` del `microservice` usando el `Dockerfile` eficiente (15.1).
     7. **`Tag Docker Image`:** Etiquetar la imagen con la versión semántica (15.2) y, opcionalmente, con el `git commit SHA`.
     8. **`Push Docker Image`:** Subir la imagen al `container registry` (ej. GHCR, Docker Hub, ECR).
   * **Etapas del `Pipeline` CD (Despliegue Continuo/Entrega Continua):**\
     Esta parte se activa si la CI es exitosa. Para ParkWise, usaremos **GitOps con Argo CD** (15.6) como el mecanismo preferido para CD.
     1. **Actualizar el `Configuration GitOps Repository`:**
        * El `pipeline` CI, después de pushear la nueva `Docker image`, hará `checkout` del `Config Repo`.
        * Modificará el `manifest` Kubernetes (o `values.yaml` de Helm, o `kustomization.yaml`) del `microservice` correspondiente para apuntar a la nueva `image tag`.
        * Hará `commit` y `push` de este cambio al `Config Repo`.
     2. **Argo CD Detecta y Sincroniza:**
        * Argo CD (desplegado en el `cluster` K8s de `staging` o `production`) detecta el cambio en el `Config Repo`.
        * Compara el estado deseado (Git) con el estado actual del `cluster`.
        * Aplica los cambios para desplegar la nueva versión del `microservice` (ej. iniciando un `rolling update`).
     3. **(Opcional) `Smoke Tests` Post-Despliegue:** El `pipeline` podría ejecutar un conjunto simple de `tests` contra el `endpoint` recién desplegado para una verificación rápida.
     4. **(Opcional) Notificación:** Informar al equipo del estado del despliegue.
4. **Entornos (`Environments`):**
   * El `pipeline` CI/CD debe poder desplegar a diferentes entornos (ej. `development`, `staging`, `production`).
   * Esto se gestiona típicamente con:
     * **`Branches` Git Diferentes en el `Config Repo`:** Una `branch` por entorno.
     * **`Overlays` Kustomize Diferentes:** Un `overlay` por entorno en el `Config Repo`.
     * **Archivos `values` de Helm Diferentes:** `values-staging.yaml`, `values-prod.yaml` en el `Config Repo`.
     * Argo CD se configura para monitorizar la `branch` o `path` del `Config Repo` correspondiente a cada entorno K8s.
5. **`Rollbacks` en el `Pipeline` CI/CD:**
   * **Automático (vía Argo CD + Git):** Si un despliegue gestionado por Argo CD resulta en una aplicación `unhealthy`, y se tienen políticas de `auto-rollback` (o mediante la reversión del `commit` en el `Config Repo`), Argo CD puede revertir al estado estable anterior.
   * **Manual/Semi-Automático:** Si los `smoke tests` post-despliegue fallan en el `pipeline` CI/CD, el `pipeline` puede `triggerear` un `kubectl rollout undo` o revertir el `commit` en el `Config Repo`.

**Diagrama `Mermaid`: `Pipeline` CI/CD con GitOps para un `Microservice` ParkWise**

```mermaid
graph TD
    A[Dev push a App Repo (BookingService)] --> B{CI Pipeline (GitHub Actions)}
    B --> B1[1. Checkout + Tests + Coverage]
    B1 -->|OK| B2[2. Build Docker Image v1.1.0]
    B2 -->|OK| B3[3. Push al Container Registry]
    B3 -->|OK| B4[4. Update Config Repo a v1.1.0]
    B4 -->|Push| C[GitOps Config Repo]

    C -->|Cambio detectado| D[ArgoCD Controller]
    D -->|Git vs Estado real| K8s_API[Kubernetes API Server]
    K8s_API -->|Estado actual| D
    D -->|OutOfSync → Sync| K8s_API
    K8s_API -->|Aplica cambios| K8s_Cluster[Cluster Kubernetes]

    K8s_Cluster -->|Estado post-deploy| Monitor[Prometheus]

    Note right of K8s_Cluster: Rolling update a BookingService v1.1.0

    Monitor -->|Métricas o Alertas| Feedback{Feedback Loop}
    Feedback -->|Falla| RB1[Rollback: revertir commit en Config Repo]
    Feedback -->|Falla| RB2[Rollback: sync versión anterior]

    B1 -->|Falla| CI_Fail[Notificar fallo CI]
    B2 -->|Falla| CI_Fail
    B3 -->|Falla| CI_Fail
    B4 -->|Falla| CI_Fail
    RB2 -->|Falla rollback| CD_Fail[Notificar fallo despliegue]

    style A fill:#D5F5E3,stroke:#2ECC71
    style C fill:#FFF9C4,stroke:#F1C40F
    style D fill:#AED6F1,stroke:#3498DB
    style K8s_Cluster fill:#E8DAEF,stroke:#8E44AD
    style CI_Fail fill:#FFCDD2,stroke:#C62828
    style CD_Fail fill:#FFCDD2,stroke:#C62828





```

Este `pipeline` automatizado para ParkWise asegura que cada cambio de código en un `microservice` pase por un riguroso proceso de validación y que su despliegue a Kubernetes sea gestionado de forma declarativa, versionada y auditable a través de GitOps con Argo CD. Esto aumenta la velocidad de entrega y la fiabilidad del sistema.

***

### 16.10 Documentación y repositorio versionado
