  ---
name: vite-dockerfile
description: Creates a production-ready multi-stage Dockerfile for a Vite SPA served by nginx. Auto-detects the package manager (npm/yarn/pnpm), Node version, build output directory, and VITE_-prefixed environment variables from the project. Produces a Dockerfile, a minimal nginx.conf with SPA routing fallback, and a .dockerignore. Use when the user asks to containerize, dockerize, or create a Dockerfile for a Vite, React, Vue, Svelte, or similar static SPA project.
---

# Vite Dockerfile

Generate a reusable, production-minded Dockerfile for a Vite SPA that is served statically by nginx.

## Preconditions

- Confirm the target directory is a Vite project: a `vite.config.ts` / `vite.config.js` exists, or the `package.json` has `vite` in `devDependencies`.
- If a `Dockerfile` already exists in the target directory, show it to the user and ask whether to overwrite before proceeding.

## Phase 1: Auto-detect project facts

Before asking the user anything, read the project and gather:

1. **Package manager**: Presence of `package-lock.json` → npm, `yarn.lock` → yarn, `pnpm-lock.yaml` → pnpm. Pick the matching install command.
2. **Node version**: From `package.json` `engines.node`, `.nvmrc`, or `.node-version`. Default to the current LTS (currently 22) if missing.
3. **Build output directory**: Default is `dist/`. Check `vite.config.*` for `build.outDir` override.
4. **Build script**: Usually `npm run build`. Check `package.json` `scripts.build` — if it's unusual (e.g., a separate `build:prod`), surface it to the user.
5. **VITE_-prefixed env vars**: Grep `.env`, `.env.example`, or source files for `import.meta.env.VITE_*`. These need to be baked into the build at build time (via `ARG` and `ENV`).
6. **Framework**: React, Vue, Svelte, Solid, etc. — rarely matters for the Dockerfile, but useful context for the user.
7. **Router**: Check for `react-router`, `vue-router`, etc. If present, the SPA routing fallback in `nginx.conf` is required, not optional.

Report what you found in a compact summary so the user can correct wrong assumptions.

## Phase 2: Decisions (ask only if ambiguous)

| Decision | Default | When to ask |
|---|---|---|
| Runtime server | `nginx:alpine` | User prefers Caddy, a Node static server, etc. |
| Build stage image | `node:<LTS>-alpine` matching detected Node | User requests a specific version |
| Multi-stage | Always | Never skip |
| Install command | `npm ci` / `yarn install --frozen-lockfile` / `pnpm install --frozen-lockfile` | Detected from lockfile |
| Dependency caching | Lockfile-first layer cache | Never skip |
| Port | `80` | User requests unprivileged port or full non-root |
| Non-root | Keep nginx default (root master, `nginx` workers) | User wants full non-root (switch to port 8080 + config change) |
| SPA routing fallback | `try_files $uri $uri/ /index.html;` | Never skip — Vite SPAs need it |
| Health check | `wget --spider` on `/` (busybox, no install needed) | Skip if user declines |
| `VITE_*` env vars | Expose each detected var as an `ARG` with the `.env` value as default | Confirm list with user if many are found |

## Phase 3: Write the files

Write three files in the project root:

### Dockerfile

```dockerfile
# ─── Stage 1: Build ───────────────────────────────────────────────────────────
FROM node:<NODE_VERSION>-alpine AS build
WORKDIR /app

# VITE_* env vars are baked at build time — override with --build-arg
ARG VITE_API_BASE_URL=<DEFAULT_FROM_DOT_ENV>
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL
# (repeat ARG/ENV pair for each detected VITE_ variable)

# Cache dependencies — only re-runs when lockfile changes
COPY package.json <LOCKFILE> ./
RUN <INSTALL_CMD>

# Build the static assets
COPY . .
RUN <BUILD_CMD>

# ─── Stage 2: Runtime ─────────────────────────────────────────────────────────
FROM nginx:alpine

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/<OUT_DIR> /usr/share/nginx/html

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --spider -q http://localhost:80/ || exit 1
```

Substitutions:
- `<NODE_VERSION>`: detected or `22`
- `<LOCKFILE>`: `package-lock.json` / `yarn.lock` / `pnpm-lock.yaml`
- `<INSTALL_CMD>`: `npm ci` / `yarn install --frozen-lockfile` / `pnpm install --frozen-lockfile` (add `RUN npm install -g pnpm` above it for pnpm)
- `<BUILD_CMD>`: `npm run build` / `yarn build` / `pnpm build`
- `<OUT_DIR>`: `dist` (or the override from `vite.config.*`)
- `<DEFAULT_FROM_DOT_ENV>`: the value in `.env` for each `VITE_*` variable

### nginx.conf

```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

If the user wants to proxy API calls through nginx (e.g., frontend calls `/api/...`), add a `location /api/ { proxy_pass http://<backend>:<port>/; }` block — but only if they ask. By default, the SPA talks to the backend directly via `VITE_API_BASE_URL`.

### .dockerignore

```
node_modules
<OUT_DIR>
.git
.idea
.vscode
*.iml
npm-debug.log*
yarn-error.log*
.pnpm-debug.log*
.env.local
.env.*.local
```

## Phase 4: Report

Tell the user:
1. Where each file was written.
2. The build + run commands, e.g.:
   ```bash
   docker build -t <project-name> .
   docker run -p 80:80 <project-name>
   ```
3. How to override a `VITE_*` variable per environment:
   ```bash
   docker build --build-arg VITE_API_BASE_URL=https://api.prod.com -t <project-name> .
   ```
4. A reminder that Vite bakes env vars at build time — one image per environment, unless they adopt a runtime-substitution strategy (offer to help with that if relevant).

Do not build the image automatically unless the user asks.

## Notes & caveats

- **Runtime env var substitution**: If the user wants a single image that talks to different backends in different environments, the build-arg approach won't work. Offer to implement an `envsubst`-based entrypoint that rewrites placeholders in the built JS at container start — but only if they ask; it's more complex.
- **Proxying the backend through nginx**: Removes the `VITE_API_BASE_URL` coupling entirely (frontend uses relative URLs). Good for monolithic deployments, bad when frontend and backend are deployed separately. Mention it once, don't push.
- **Full non-root nginx**: Requires switching to port 8080, modifying `nginx.conf` to `listen 8080;`, and running `USER nginx`. The default `nginx:alpine` image already runs workers as `nginx`, which is the standard hardening level.
- **SSR / Next.js / Remix / Nuxt**: This skill is for **static SPAs only**. If the project is an SSR framework, warn the user — the Dockerfile would need a Node runtime stage instead of nginx, and the build/start commands are different.
