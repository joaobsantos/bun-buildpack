# bun-buildpack

Custom Heroku buildpack for Bun monorepos (Nx-friendly).

It lets each Heroku app choose which workspace project to build and run via env vars, minimizes slug size by cleaning root `node_modules`, and preserves only the runtime dependencies required to boot your server or serve your app.

## What it does

- Detects a Bun/Node project by `package.json` or `bun.lock`
- Optionally installs Node (for compatibility) and always installs Bun
- Runs install and build inside your selected `APP_DIR`
- Generates a minimal `package.json` inside your runtime output (`RUNTIME_DIR`) excluding `workspace:*` deps
- Installs production-only dependencies inside the runtime output directory
- Cleans root/app `node_modules` to shrink the slug, but keeps runtime deps
- Defines the `web` process only in the Release phase (no auto-start in `.profile.d`)

## Detection

Project is detected if either `package.json` or `bun.lock` exists in the build dir.

## Compile (build) phase

- Optional Node install if `NODE=true` (version via `NODE_VERSION`)
- Bun is installed and exported to PATH via `.profile.d/bun.sh`
- Execution directory resolution:
  - If `APP_DIR` is provided and exists, use it; else use repo root
- Installation and build run in the execution directory:
  - Install via `INSTALL_COMMAND` (default: `bun install`)
  - Build via `BUILD_COMMAND` (default: `bun run build`)
- Runtime packaging (inside `RUNTIME_DIR`, default: `dist`):
  - If `GENERATE_RUNTIME_PKG=true`, generates a minimal `package.json` with only non-`workspace:*` dependencies
  - If `PROD_INSTALL=true`, runs `bun install --production` in the runtime directory
- Cleanup if `CLEAN=true`:
  - Removes root `node_modules` and `APP_DIR/node_modules`
  - Preserves `RUNTIME_DIR/node_modules`
  - Removes caches (`tmp`, `.cache`, internal install caches) to reduce slug size
- No start command is executed or appended in this phase

## Release phase

The web process is decided by env vars (priority order):

1. `WEB_START_COMMAND`
2. `WEB_start_COMMAND` (fallback)
3. `START_COMMAND` (final fallback)

Working directory selection:

- If `START_IN_RUNTIME=true`, the release step will `cd` into `APP_DIR/RUNTIME_DIR` before executing the start command (if one is provided)
- Otherwise, it will `cd` only into `APP_DIR` (if provided)

No auto-run is emitted in `.profile.d`; only Release defines the `web` process.

## Supported environment variables

- APP_DIR (compile, release)
  - Path to the workspace app to build/run (e.g., `apis/servers/api-gateway-server`). If empty, repo root is used.

- RUNTIME_DIR (compile, release) – default: `dist`
  - Directory within `APP_DIR` containing the build output to run from (`dist`, `build`, or `.`).

- INSTALL_COMMAND (compile) – default: `bun install`
  - Command used to install dependencies inside `APP_DIR`.

- BUILD_COMMAND (compile) – default: `bun run build`
  - Build command executed in `APP_DIR`.

- START_COMMAND (release)
  - Fallback start command used if no `WEB_START_COMMAND` is provided (e.g., `bun ./index.js`).

- WEB_START_COMMAND (release)
  - Preferred explicit `web` process command.

- WEB_start_COMMAND (release)
  - Case-variant fallback for legacy setups.

- START_IN_RUNTIME (release) – default: `false`
  - If `true`, the release step will `cd` into `APP_DIR/RUNTIME_DIR` before executing the start command.

- CLEAN (compile) – default: `true`
  - If `true`, removes root and `APP_DIR` `node_modules` and caches after building. Runtime `node_modules` are preserved.

- NODE (compile) – default: `true`
  - If `true`, installs Node into `.heroku/node` for compatibility.

- NODE_VERSION (compile) – default: `20.9.0`
  - Version of Node to install when `NODE=true`.

- GENERATE_RUNTIME_PKG (compile) – default: `true`
  - If `true`, generates `RUNTIME_DIR/package.json` by copying only non-`workspace:*` dependencies from the app’s `package.json`.

- PROD_INSTALL (compile) – default: `true`
  - If `true`, runs `bun install --production` inside `RUNTIME_DIR` so runtime deps are present at boot.

- runtime.bun.txt / runtime.txt (compile)
  - If present, pins the Bun version via `bun-<version>` during install.

## Examples

### GraphQL API (gateway)

- APP_DIR: `apis/servers/api-gateway-server`
- BUILD_COMMAND: `bun run build:prod`
- RUNTIME_DIR: `dist`
- START_IN_RUNTIME: `true`
- START_COMMAND: `bun ./index.js`

### Another API (newsoftds)

- APP_DIR: `apis/servers/api-newsoftds-server`
- BUILD_COMMAND: `bun run build`
- RUNTIME_DIR: `build`
- START_IN_RUNTIME: `true`
- START_COMMAND: `bun ./index.js`

### React/Vite SPA (served by a static server)

- APP_DIR: `<web/app>`
- BUILD_COMMAND: `bun run build`
- RUNTIME_DIR: `dist`
- START_IN_RUNTIME: `true`
- START_COMMAND: `bunx serve -s . -l $PORT`
