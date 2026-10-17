# Laboratorio 09 - Integración Frontend - Backend: Angular + Spring Boot

[![Angular](https://img.shields.io/badge/Angular-19%2B%20%2F%2022-DD0031?style=for-the-badge&logo=angular&logoColor=white)](https://angular.dev/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.3.4-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Java](https://img.shields.io/badge/Java-17%2B%20%2F%2025-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://openjdk.org/)
[![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)]()
[![UPN](https://img.shields.io/badge/UPN-2026--2-blue?style=for-the-badge)]()

> **Curso:** Soluciones Web y Aplicaciones Distribuidas (SIST1402A)  
> **Estudiante:** Orlando Dorival  
> **Universidad:** Universidad Privada del Norte (UPN)  
> **Semana:** 9  

---

## 🎯 1. Logro y Alcance del Laboratorio

Consolidar la integración completa entre **Angular** y **Spring Boot**, fortaleciendo la calidad de la aplicación mediante:
- **Validaciones reactivas avanzadas** en formularios y feedback inmediato al usuario.
- **Manejo centralizado y tipado de errores HTTP** (`status 0, 400, 404, 500`) con traducción a mensajes comprensibles.
- **Búsqueda en servidor sincronizada con Query Params** en la URL (`/productos?q=...`) y botón de limpieza.
- **Extensión de seguridad para endpoints protegidos** mediante interceptor HTTP funcional (`authInterceptor`) con tokens Bearer JWT.
- **Pruebas integrales de flujo completo:** Navegador -> Angular -> API -> Base de datos -> Angular.
- **Documentación exhaustiva de consumo de servicios** en [`INTEGRACION.md`](INTEGRACION.md).

---

## 🏗️ 2. Arquitectura de la Solución

El diagrama en formato **Draw.io** se encuentra en [`diagramas/arquitectura-lab09.drawio`](diagramas/arquitectura-lab09.drawio).

```
Navegador Web (SPA :4200)
    │  (Rutas /productos?q=...)
    ▼
authInterceptor (Adjunta Bearer Token si existe)
    │
    ▼
ProductoService (manejarError + catchError)
    │  (HTTP REST / JSON)
    ▼
CorsConfig.java (Filtro CORS)
    │
    ▼
Spring Boot REST API (:8080)
    │  (GET, POST 201, PUT 200, DELETE 204, /buscar)
    ▼
Spring Data JPA -> H2 In-Memory / MySQL Database
```

---

## 📁 3. Estructura del Proyecto

```
Laboratorio_09_UPN_Integracion_Angular_Spring_Pruebas_Documentacion/
├── backend/                                  # Servidor Spring Boot REST API
│   ├── src/main/java/com/example/laboratorio4/
│   │   ├── config/CorsConfig.java            # Habilitación CORS
│   │   ├── controller/ProductoController.java# Endpoints CRUD + /buscar
│   │   ├── model/Producto.java               # Entidad Producto
│   │   ├── repository/ProductoRepository.java# Repositorio JPA
│   │   ├── service/ProductoService.java      # Servicio de persistencia
│   │   └── Laboratorio4Application.java      # Entrypoint Spring Boot
│   ├── src/main/resources/
│   │   ├── application.properties            # Configuración H2 / MySQL
│   │   └── data.sql                          # Catálogo precargado
│   ├── src/test/java/com/example/laboratorio4/
│   │   └── ProductoControllerTest.java       # Pruebas MockMvc
│   └── pom.xml
├── frontend/                                 # Aplicación Cliente Angular
│   ├── src/app/
│   │   ├── components/catalogo/              # Catálogo con búsqueda por servidor y QueryParams
│   │   ├── components/encabezado/            # Header de la aplicación
│   │   ├── interceptors/auth.interceptor.ts  # Interceptor funcional Bearer JWT
│   │   ├── models/producto.ts                # Interfaz TypeScript
│   │   ├── pages/producto-form/              # Formulario con validaciones visibles y estados
│   │   ├── services/producto.service.ts      # Cliente HTTP con catchError y traducción
│   │   ├── services/producto.service.spec.ts # Pruebas unitarias de mapeo de errores
│   │   ├── app.config.ts                     # Registro de interceptores y router
│   │   └── app.routes.ts                     # Definición de rutas
│   ├── angular.json
│   └── package.json
├── diagramas/
│   └── arquitectura-lab09.drawio             # Diagrama editable Draw.io
├── INTEGRACION.md                            # Registro de los 8 casos de prueba
└── README.md
```

---

## ⚡ 4. Aspectos Técnicos Destacados

1. **Gestión de Errores con `catchError`:**  
   En `ProductoService`, cada petición pasa por `manejarError()` traduciendo códigos técnicos en mensajes claros:
   - `Status 0`: "No se pudo conectar con la API."
   - `Status 400`: "Los datos enviados no son válidos."
   - `Status 404`: "El recurso solicitado no existe."
   - `Status 500`: "Error interno del servidor."
2. **Búsqueda Integrada con Backend y Query Params:**  
   El criterio de búsqueda se envía tanto al endpoint `/api/productos/buscar?nombre=...` como a la URL `/productos?q=...`, permitiendo recargar la página o compartir el enlace sin perder la búsqueda.
3. **Validaciones en Formulario Reactivo:**  
   - Mensajes específicos para precio mayor a 0 y stock no negativo.
   - Bloqueo de botón durante el envío asíncrono para evitar duplicidad de transacciones.
4. **Auth Interceptor Funcional:**  
   Listo para autenticación JWT inyectando automáticamente la cabecera `Authorization: Bearer <token>`.

---

## 🧪 5. Pruebas y Verificación

- **Backend (MockMvc):** 4/4 pruebas de integración ejecutadas y aprobadas (`mvn clean test`).
- **Frontend (Vitest / Angular Testing):** 5/5 pruebas unitarias aprobadas, incluyendo validación del mapeo de errores 404 y status 0 (`npm test -- --watch=false`).
- **Build Frontend:** Generación exitosa sin errores (`npm run build`).

---

## 🚀 6. Instrucciones de Ejecución

### 1. Iniciar Backend
```bash
cd backend
mvn spring-boot:run
```
Disponible en `http://localhost:8080`.

### 2. Iniciar Frontend
```bash
cd frontend
npm install
ng serve --host 0.0.0.0
```
Disponible en `http://localhost:4200`.

---

## 📄 7. Evidencias de Integración
Revise el documento [`INTEGRACION.md`](INTEGRACION.md) con la matriz completa de los 8 casos de prueba evaluados.
