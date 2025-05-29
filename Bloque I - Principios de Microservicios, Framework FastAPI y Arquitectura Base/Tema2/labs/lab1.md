# LABORATORIO 1

## 1. Objetivos 

 
- **Diseñar**  dos microservicios REST con FastAPI.
 
- **Probar**  localmente cada servicio.
 
- **Dockerizar**  ambos servicios.
 
- **Orquestar**  con Docker Compose para simular un pequeño ecosistema.

- **Añadir**: un API Gateway (NGINX).
 
- **Realizar**  peticiones entre servicios y pruebas de integración.


## 2. Prerrequisitos 

 
- Python ≥ 3.11 instalado.
 
- Docker y Docker Compose instalados.
 
- Editor de código (VSCode).
 
- HTTPie o Postman para probar las APIs.
## 3. Estructura del proyecto 



```bash
lab1/
├── microservice-users/
│   ├── app/
│   │   ├── main.py
│   │   └── models.py
│   ├── requirements.txt
│   └── Dockerfile
├── microservice-items/
│   ├── app/
│   │   ├── main.py
│   │   └── models.py
│   ├── requirements.txt
│   └── Dockerfile
└── docker-compose.yml
```
