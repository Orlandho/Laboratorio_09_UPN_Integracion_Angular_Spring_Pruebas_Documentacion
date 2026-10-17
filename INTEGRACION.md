# Integración Angular - Spring Boot: Validación, Errores y Búsqueda

[![Angular](https://img.shields.io/badge/Angular-19%2B%20%2F%2022-DD0031?style=flat-square&logo=angular&logoColor=white)](https://angular.dev/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.3.4-6DB33F?style=flat-square&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Status](https://img.shields.io/badge/Pruebas%20de%20Integraci%C3%B3n-100%25%20Aprobadas-success?style=flat-square)]()

**Universidad:** Universidad Privada del Norte (UPN)  
**Curso:** Soluciones Web y Aplicaciones Distribuidas (SIST1402A)  
**Periodo:** 2026-2  
**Laboratorio:** 09 - Integración Consolidada  
**Estudiante:** Orlando Dorival  

---

## 🌐 1. API y Arquitectura de Trabajo

- **Frontend Angular:** `http://localhost:4200`
- **Backend Spring Boot:** `http://localhost:8080` (o endpoint Codespaces `https://*-8080.app.github.dev/api/productos`)
- **Base de Datos:** H2 In-Memory (modo MySQL para pruebas inmediatas sin dependencias externas) y MySQL `localhost:3306`.

---

## 📋 2. Matriz de Casos de Prueba de Integración (Casos 1 al 8)

| Caso | Acción | HTTP Status | Resultado Esperado | Evidencia y Verificación |
|:---:|---|:---:|---|---|
| **1** | Listar productos | `200 OK` | Se muestra catálogo completo en la tabla con badges de stock | Petición `GET /api/productos` visible en Network con array JSON de 5 elementos. |
| **2** | Buscar por nombre | `200 OK` | Solo muestra coincidencias del criterio | Petición `GET /api/productos/buscar?nombre=teclado`. Actualiza URL con `?q=teclado`. |
| **3** | Registrar válido | `201 CREATED` | Nuevo ID generado y retorno automático al catálogo | Petición `POST /api/productos` con payload válido. Retorna HTTP 201 y redirecciona a `/productos`. |
| **4** | Formulario inválido | *Sin petición* | La aplicación bloquea el envío y muestra alertas en pantalla | Al pulsar Guardar con nombre corto o precio 0, se marcan campos en rojo y no se emite tráfico HTTP. |
| **5** | Editar existente | `200 OK` | Cambios visibles inmediatamente en base de datos y catálogo | Carga `/api/productos/{id}`, permite modificar campos y envía `PUT /api/productos/{id}` retornando 200 OK. |
| **6** | Eliminar propio | `204 NO CONTENT` | El registro desaparece de la base de datos y la tabla se refresca | Petición `DELETE /api/productos/{id}` con confirmación previa. Tabla recargada automáticamente. |
| **7** | Buscar ID inexistente | `404 NOT FOUND` | Mensaje controlado en interfaz ("El recurso solicitado no existe.") | `GET /api/productos/99999` mapeado por `manejarError()` en lugar de error en consola no controlado. |
| **8** | Backend apagado | `0 (Error de red)` | Mensaje controlado en pantalla ("No se pudo conectar con la API.") | Al detener el servidor Spring Boot, el frontend captura status 0 y muestra banner de advertencia. |

---

## 🛡️ 3. Validaciones Implementadas

- **Nombre:** Obligatorio (`Validators.required`), mínimo 3 caracteres (`Validators.minLength(3)`).
- **Precio:** Obligatorio (`Validators.required`), estrictamente mayor a 0 (`Validators.min(0.01)`). Mensaje: *"El precio debe ser mayor que 0."*
- **Stock:** Obligatorio (`Validators.required`), no negativo (`Validators.min(0)`). Mensaje: *"El stock no puede ser negativo."*
- **Botón Guardar:** Bloqueado (`disabled="guardando"`) durante la sincronización HTTP para evitar peticiones duplicadas.

---

## ⚡ 4. Manejo Centralizado de Errores HTTP

Implementado mediante operador RxJS `catchError` en `ProductoService`:
```typescript
private manejarError(error: HttpErrorResponse): Observable<never> {
  let mensaje = 'Ocurrió un error inesperado.';

  if (error.status === 0) {
    mensaje = 'No se pudo conectar con la API.';
  } else if (error.status === 400) {
    mensaje = 'Los datos enviados no son válidos.';
  } else if (error.status === 404) {
    mensaje = 'El recurso solicitado no existe.';
  } else if (error.status === 500) {
    mensaje = 'Error interno del servidor.';
  }

  return throwError(() => new Error(mensaje));
}
```

---

## 🔐 5. Extensión de Endpoints Protegidos (JWT Auth Interceptor)

Se implementó el interceptor funcional moderno `authInterceptor` en `src/app/interceptors/auth.interceptor.ts`:
```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = localStorage.getItem('token');
  if (!token) return next(req);

  return next(req.clone({
    setHeaders: {
      Authorization: `Bearer ${token}`
    }
  }));
};
```
Registrado globalmente en `app.config.ts` mediante `provideHttpClient(withInterceptors([authInterceptor]))`.
