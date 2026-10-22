# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Resttek: plataforma de gestión de restaurantes. Monorepo con **npm workspaces** (`packages/*`):

| Paquete | Tipo | Puerto |
| --- | --- | --- |
| `@resttek/api` | Express 5 + TypeScript (ESM) + SQLite | 3000 |
| `@resttek/web-admin` | Angular 21 | 4200 |
| `@resttek/web-empleados` | Angular 21 (cocina, barra, salón) | 4201 |
| `@resttek/web-clientes` | Angular 21 (pedidos de clientes) | 4202 |
| `@resttek/web-shared` | Librería Angular compartida (auth, interceptors, login/register) | — |

La documentación de referencia está en `docs/` (arquitectura, modelo de datos, glosario). `docs/revisiones/` recoge las inconsistencias conocidas entre la documentación y el código: consúltalas antes de fiarte de la documentación en un punto dudoso.

## Comandos

Todos desde la raíz:

```bash
npm install              # instala y enlaza todos los workspaces
npm run seed             # datos de prueba (idempotente, INSERT OR IGNORE)
npm run dev:api          # tsx watch src/server.ts
npm run dev:admin        # / dev:empleados / dev:clientes  (ng serve con proxy /api → :3000)
npm test                 # vitest run (solo la API tiene tests)
```

- Un solo test: `npm test -w @resttek/api -- src/services/order.service.test.ts` (o `-t "nombre del test"`).
- Modo watch: `cd packages/api && npm run test:watch`.
- Build de un frontend: `npm run build -w @resttek/web-admin`.
- No hay linter configurado. `web-clientes` tiene `.prettierrc`.
- Credenciales de seed: la contraseña de cada usuario es su propio email (p. ej. `admin@resttek.com` / `admin@resttek.com`). Lista completa en el README.

## Arquitectura de la API (`packages/api/src`)

Conviven **dos estilos**; identifica cuál aplica antes de tocar código:

1. **Hexagonal + DDD, solo en `contexts/employee/`** (empleados, auth y roles): `domain/` (entidad `Employee` con constructor privado + `create()`, value objects `Role`/`Email`, interfaces con prefijo `I`), `application/` (casos de uso con un único `execute()`, dobles en `application/mocks/`) e `infrastructure/` (repositorio SQLite, bcrypt y HTTP). El cableado de dependencias está en `infrastructure/http/dependencies.ts`. `contexts/shared/` contiene `Email`, los middlewares `authenticate`/`authorize(roles)` y el `errorHandler`.
2. **Por capas para `restaurant`, `dish`, `ingredient` y `order`**: `models/` (interfaces planas + funciones `normalizeX()` en lugar de value objects), `repositories/` (interfaz sin prefijo `I` + implementación SQLite en el mismo fichero), `services/` (lógica y validación; un servicio con varios métodos), `controllers/` y `routes/`. **La composición de dependencias se hace en el propio fichero de rutas**.

Detalles que no son obvios:
- Los imports usan alias de `tsconfig.json` (`@config/*`, `@errors/*`, `@shared/*` → `contexts/shared`, `@employee/*`, `@models/*`, `@repositories/*`, `@services/*`, `@controllers/*`, `@routes/*`, `@scripts/*`) y **siempre llevan extensión `.js`** (ESM con `nodenext`).
- Los routers montados bajo `/restaurants/:restaurantId/...` necesitan `Router({ mergeParams: true })`.
- Errores: `AppError` → errores en `errors/DomainErrors.ts`; el `errorHandler` asigna el código HTTP **por nombre de clase** (lista explícita de 404, `InvalidCredentialsError` → 401 y el resto de `AppError` → 400). Un error nuevo de tipo "no encontrado" debe añadirse a esa lista. `OrderController` es la excepción: hace su propio `try/catch` y no llama a `next(error)`.
- BD: instancia única `dbConfig` (`config/database.ts`); las tablas se crean al arrancar (`CREATE TABLE IF NOT EXISTS`). Con `NODE_ENV=test` usa `:memory:`; si no, `packages/api/resttek.db`. El mapeo `snake_case` → `camelCase` se hace con alias en el SQL.
- Los clientes son filas de `employees` con `role = 'cliente'`; el rol de gerente se almacena como `manager`.
- Al crear un pedido, una línea con `quantity: N` se expande a N `order_items` con `quantity: 1` (estado por unidad).
- Los tests son unitarios y están junto al código (`*.test.ts`); los dobles de la capa por capas están en `repositories/mocks/`. No hay tests HTTP (`supertest` está instalado, pero no se usa).

## Arquitectura de los frontends

- Angular 21 con componentes standalone, signals y sin zonas (ningún paquete instala `zone.js`). RxJS solo para HTTP.
- `web-admin` y `web-empleados`: `features/<feature>/{models,pages,services,store}`. Los componentes solo hablan con un **store** (`providedIn: 'root'`, signals privadas expuestas con `asReadonly()`, trío `loading`/`error`/datos, convierte el `Observable` del service con `firstValueFrom`). `OrderStore` de empleados hace polling cada 30 s.
- `web-clientes` es distinta: modelos y servicios en `core/`, features como componentes sueltos que hacen `.subscribe()` directamente sobre los services. El único store es `CartStore` (local). El polling se hace con `setInterval` en los componentes.
- URL de la API: token `API_URL` (`web-shared`) = `environment.apiUrl` = `'/api/v1'`. `web-clientes` inyecta el token; `web-admin` y `web-empleados` importan `environment` directamente.
- `web-shared` se consume desde el fuente (`main: src/index.ts`, sin compilar): exporta `AuthStore` (token y usuario en `localStorage`), `AuthService`, `authGuard`, `authInterceptor`, `errorInterceptor` (401 → logout + `/login`) y los componentes de login y registro. Hay que reiniciar `ng serve` para ver los cambios. Todo lo nuevo que se comparta debe exportarse desde `index.ts`.
- Estilos: `web-shared/src/lib/styles/base.css` **no se importa**; cada app tiene su propia copia en `src/styles.css`.
- En `web-empleados`, el filtrado por rol solo afecta a la navegación (`computed()` en `ShellComponent`); la autorización real la hace la API.

## Scripts

`scripts/seed-issues.sh` crea issues de GitHub a partir de `scripts/issues.json` usando `gh` y `jq` (`--dry-run` para simular). Ver `scripts/README.md`.
