# Arquitectura y seguridad

[← Volver al README](../README.md)

## Arquitectura

<p align="center">
  <img src="diagramas/arquitectura.png" width="425" alt="El navegador y Cloud Scheduler llegan a Next.js en Cloud Run, que se conecta a PostgreSQL 17 en Cloud SQL y a la API de Steam"/>
</p>

<p align="center">
  <img src="diagramas/despliegue.png" width="210" alt="Un push a main pasa por Cloud Build, luego por las migraciones y termina en Cloud Run"/>
</p>

La base de datos no tiene redes autorizadas por ningún motivo.

## Seguridad

### Contraseñas

<p align="center">
  <img src="diagramas/contrasenas.png" width="234" alt="La contraseña pasa por NFKC, HMAC-SHA256 con pepper y scrypt con N=2^17, r=8, p=1 y salt de 16 bytes antes de guardarse como hash"/>
</p>

El pepper está fuera de la base, además los hashes se recalculan solos al iniciar sesión si cambian los parámetros.

### Gestión de Contraseñas filtradas

<p align="center">
  <img src="diagramas/contrasenas-filtradas.png" width="606" alt="GameNest envía a Have I Been Pwned los 5 primeros caracteres del SHA-1, recibe los sufijos posibles y los compara localmente"/>
</p>

### Métodos de seguridad aplicados

<p align="center">
  <img src="diagramas/capas-de-seguridad.png" width="229" alt="Cada acción pasa por sesión, límite de uso, validación Zod y autorización antes de llegar a la base con rol mínimo y RLS"/>
</p>

|  |  |
|---|---|
| Base de datos | Rol de la app sin permisos de estructura, RLS en todas las tablas y restricciones `CHECK` |
| Sesiones | Cookies `HttpOnly` y `Secure`, revocadas al cambiar la contraseña |
| Tokens | Enlaces de un solo uso hasheados y tokens OAuth cifrados |
| Direcciones IP | Solo como HMAC-SHA256 |
| Cabeceras | CSP con nonce, y HSTS |
| Secretos | Secret Manager, con acceso por servicio |
| Dependencias | Antigüedad mínima de 24 h y scripts de instalación bloqueados |
