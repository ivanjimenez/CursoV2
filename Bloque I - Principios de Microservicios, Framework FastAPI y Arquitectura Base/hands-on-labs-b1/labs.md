# BLOQUE 1 – Hands-on Labs

> ⏱️ **Duración total estimada:** 7 horas\
> 🧱 **Proyecto base:** App de e-commerce (usuarios, productos, pedidos) en FastAPI + MariaDB

***

## 🔹 LAB 1 – Monolito Base: FastAPI + MariaDB

| Item            | Detalles                                                                          |
| --------------- | --------------------------------------------------------------------------------- |
| 🕒 Duración     | 1.5 h                                                                             |
| 🎯 Objetivo     | Partir de una app monolítica realista con routers separados                       |
| 🧠 Temas        | Tema 1 (1.1 a 1.7), Tema 2.1 a 2.4                                                |
| ⚙️ Tecnologías  | FastAPI, Docker Compose, Pydantic, SQL crudo                                      |
| 📁 Entregable   | Monolito completo: `/users`, `/products`, `/orders`                               |
| 🧪 Tareas clave | <p>- Clonar<br>- Entender capas<br>- Ejecutar API REST<br>- Analizar dominios</p> |
| 🧩 Repositorios | `lab01-monolito-ecommerce-inicial`                                                |

***

## 🔹 LAB 2 – Refactor: Microservicios + Gateway

| Item            | Detalles                                                                    |
| --------------- | --------------------------------------------------------------------------- |
| 🕒 Duración     | 1.5 h                                                                       |
| 🎯 Objetivo     | Separar dominios en microservicios y exponerlos a través de un API Gateway  |
| 🧠 Temas        | Tema 1.7 a 1.11, Tema 2.5, 2.10, Tema 3.1 a 3.3                             |
| ⚙️ Tecnologías  | httpx, routers FastAPI, Docker Compose avanzado                             |
| 📁 Entregable   | 3 servicios (`users`, `products`, `orders`) + 1 gateway (`api`)             |
| 🧪 Tareas clave | <p>- Crear microservicios<br>- Configurar red interna<br>- Gateway REST</p> |
| 🧩 Repositorios | `lab01-microservicios-ecommerce-final` (fase intermedia)                    |

***

## 🔹 LAB 3 – Gestión de Errores y Resiliencia

| Item            | Detalles                                                            |
| --------------- | ------------------------------------------------------------------- |
| 🕒 Duración     | 1.5 h                                                               |
| 🎯 Objetivo     | Añadir manejo de errores resiliente y Circuit Breaker básico        |
| 🧠 Temas        | Tema 4 completo                                                     |
| ⚙️ Tecnologías  | pybreaker, logging, fallback handlers                               |
| 📁 Entregable   | API Gateway con retry y tolerancia a fallos de servicios caídos     |
| 🧪 Tareas clave | <p>- Simular fallos<br>- Implementar retry<br>- Circuit breaker</p> |
| 🧩 Repositorios | `lab01-resilience-gateway`                                          |

***

## 🔹 LAB 4 – Seguridad Básica con JWT y CORS

| Item            | Detalles                                                                    |
| --------------- | --------------------------------------------------------------------------- |
| 🕒 Duración     | 1.5 h                                                                       |
| 🎯 Objetivo     | Añadir autenticación JWT y configuración segura de endpoints                |
| 🧠 Temas        | Tema 5 completo                                                             |
| ⚙️ Tecnologías  | FastAPI JWT Auth, CORS, validación con Pydantic                             |
| 📁 Entregable   | Sistema protegido con login, tokens y autorización por scope                |
| 🧪 Tareas clave | <p>- Generar y validar JWT<br>- Proteger endpoints<br>- Configurar CORS</p> |
| 🧩 Repositorios | `lab01-secure-microservices`                                                |

***

## 📦 Resultado final tras Bloque 1

Un sistema distribuido con:

* 🔐 Seguridad básica
* 🛡️ Resiliencia básica
* 🌐 Comunicación REST
* 📦 Despliegue profesional en Docker Compose

***

¿Quieres que empecemos ya con la **implementación paso a paso del LAB 1: Monolito base**, con su README, enunciado y código completo inicial?
