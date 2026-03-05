# skills-portal

Portal de skills con Astro 5 en modo servidor: lee README y SKILL.md directamente de GitHub en runtime (sin clonar) usando un token de solo lectura. Incluye caché en memoria y concurrencia limitada para responder rápido.

## Requisitos rápidos

- Node 20 o 22.
- Archivo `.env` con `GITHUB_TOKEN` (o `GITHUB_PAT`). Fine-grained: Contents: Read y Metadata: Read sobre los repos necesarios; clásico: scope `repo`.
- Repos configurados en `config/repos.yaml` (campos: id, name, repoUrl, defaultBranch opcional, readmePath opcional, skillsPath opcional).

## Uso

- `npm install --legacy-peer-deps` (resolverá los peer actuales).
- `npm run dev` — servidor local en <http://localhost:4321>.
- `npm run build` — salida server (adapter node) lista para deploy.
- `npm run preview` — prueba del build.

## Comportamiento de datos

- Runtime: páginas `/skills`, `/skills/:slug`, `/repos`, `/repos/:id` consumen la GitHub API/RAW con `Authorization: Bearer <token>`.
- Cache en memoria por repo (TTL 5 min) para SKILL.md y concurrencia limitada (5 fetches simultáneos) para evitar latencias altas cuando hay muchos repos.
- Errores de red/GitHub se loguean con `[github] fetchText/fetchJson ... -> <status>`.

## Configuración de entorno

- `.env` en la raíz:
   - `GITHUB_TOKEN=github_pat_...` (no se imprime ni se expone al cliente).
- El proceso lee `.env` al arrancar; si cambias el valor, reinicia `npm run dev`.

## Estructura relevante

- `config/repos.yaml` — registro de repos y paths.
- `src/lib/github.ts` — fetch autenticado de README/skills.
- `src/lib/skills.ts` — listado, caché y parseo de SKILL.md.
- `src/pages/skills/*` y `src/pages/repos/*` — rutas SSR.

## Depuración

- Verifica el token: `curl -I -H "Authorization: Bearer $GITHUB_TOKEN" https://api.github.com/user` debe dar 200.
- Si hay 401/404 en server logs, revisa permisos del token y paths `readmePath/skillsPath` en `config/repos.yaml`.

## Seguridad

- El token solo se usa en el backend; no se envía al cliente.
- Usa tokens de lectura, con alcance mínimo a los repos necesarios.

## Licencia

Uso interno.
