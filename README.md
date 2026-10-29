# Alumbrado GDL

PWA ciudadana para documentar fallas de alumbrado, agrupar reportes y dar seguimiento verificable a solicitudes oficiales. Un reporte en la app complementa los canales oficiales: no equivale a una solicitud municipal recibida.

## Requisitos

- Node.js 20 o posterior
- pnpm 9 (`corepack enable && corepack prepare pnpm@9.15.4 --activate`)
- PostgreSQL 16 con PostGIS 3.4 habilitado, instalado localmente

## Inicio local

1. Instala PostgreSQL 16 y PostGIS 3.4 con el gestor de paquetes de tu sistema e inicia PostgreSQL como servicio local.
2. Crea la base usando tu usuario local: `createdb alumbrado_gdl`.
3. Copia `.env.example` a `.env`. Si tu usuario PostgreSQL no coincide con el usuario del sistema, ajusta `DATABASE_URL` a `postgres://USUARIO@localhost:5432/alumbrado_gdl`. Configura un `SESSION_SECRET` aleatorio de al menos 32 caracteres.
4. Aplica las migraciones desde la raíz del proyecto, en orden:

   ```sh
   psql "$DATABASE_URL" -f apps/api/migrations/001_initial.sql
   psql "$DATABASE_URL" -f apps/api/migrations/002_sessions.sql
   ```

5. Ejecuta `pnpm install` y `pnpm dev`. Web: <http://localhost:5173>; API: <http://localhost:4000/health>.

El usuario PostgreSQL debe tener permisos para crear `pgcrypto` y `postgis` al aplicar la primera migración. El API lee `.env` desde el directorio desde el que se inicia pnpm.

## Estructura

- `apps/web`: React, Vite, TypeScript, React Router, TanStack Query y Leaflet.
- `apps/api`: Express, sesiones PostgreSQL, validación Zod y consultas parametrizadas.
- `packages/shared`: validaciones compartidas.

## Funciones incluidas

Registro e inicio/cierre de sesión con cookie `httpOnly`, hash bcrypt, rate limiting de autenticación, creación de reportes pendientes, moderación, mapa/lista pública, confirmación única, expedientes, folios oficiales con evidencia de acuse, actualizaciones públicas y descarga JSON de ficha. Los reportes no publicados y la identidad del reportante se excluyen de las respuestas públicas. La API es el único componente que accede a PostgreSQL.

Para habilitar la moderación local, asigna el rol mediante una operación administrativa directa en desarrollo: `UPDATE users SET role='moderator' WHERE email='correo@example.com';`. En producción, la asignación de roles debe hacerse desde un proceso administrativo controlado; no existe endpoint público para elevar permisos.

## Alcance operativo

Las coordenadas se envían como `[longitud, latitud]`. El mapa muestra únicamente reportes publicados. Las fotos y su almacenamiento de objetos, retiro de EXIF, PDF, backups, panel de métricas completo y recorrido E2E requieren integración/configuración adicional antes de un piloto; no se guardan archivos en el servidor en esta primera base ejecutable. Los enlaces de acuse y evidencia deben apuntar a almacenamiento privado controlado. El proveedor de teselas actual es OpenStreetMap y debe conservarse su atribución y cumplirse sus políticas de uso.
