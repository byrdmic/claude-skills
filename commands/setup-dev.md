Analyze this project and set up a unified `dev` script that starts both the frontend and backend concurrently using a single command. Follow these steps carefully:

## Step 1: Detect Project Structure

- Check if this is a monorepo (look for `pnpm-workspace.yaml`, `lerna.json`, `turbo.json`, or a root `workspaces` field in `package.json`)
- If monorepo: identify each workspace package by reading their `package.json` files
- If single repo: inspect the root `package.json` and directory structure

## Step 2: Identify Frontend

Look for these frontend frameworks and their dev commands:

| Framework | Indicators | Typical Dev Command |
|-----------|-----------|-------------------|
| **Next.js** | `next` in dependencies, `next.config.*` | `next dev` |
| **Expo** | `expo` in dependencies, `app.json`/`app.config.*` | `expo start` |
| **Vite** | `vite` in dependencies, `vite.config.*` | `vite dev` |
| **Create React App** | `react-scripts` in dependencies | `react-scripts start` |
| **Remix** | `@remix-run/dev` in dependencies | `remix dev` |
| **Astro** | `astro` in dependencies | `astro dev` |
| **Angular** | `@angular/cli` in dependencies | `ng serve` |
| **Nuxt** | `nuxt` in dependencies | `nuxt dev` |
| **SvelteKit** | `@sveltejs/kit` in dependencies | `svelte-kit dev` |

## Step 3: Identify Backend

Look for these backend frameworks:

| Framework | Indicators | Typical Dev Command |
|-----------|-----------|-------------------|
| **Hono** | `hono` in dependencies | Check existing scripts |
| **Fastify** | `fastify` in dependencies | Check existing scripts |
| **Express** | `express` in dependencies | Check existing scripts |
| **NestJS** | `@nestjs/core` in dependencies | `nest start --watch` |
| **Koa** | `koa` in dependencies | Check existing scripts |
| **Elysia/Bun** | `elysia` in dependencies | Check existing scripts |

Also look for non-Node backends:
- **Python** (FastAPI, Django, Flask): look for `requirements.txt`, `pyproject.toml`, `manage.py`
- **Go**: look for `go.mod`
- **Rust**: look for `Cargo.toml`

## Step 4: Confirm with the User

Before making any changes, present your findings:
- What you detected as the frontend (framework, package name, dev command)
- What you detected as the backend (framework, package name, dev command)
- The proposed `dev` script

Ask the user to confirm or correct your detection. If anything is ambiguous (e.g., multiple possible backends, unclear which package is which), ask for clarification.

## Step 5: Install and Configure

Once confirmed:

1. Install `concurrently` as a workspace/root dev dependency if not already present
2. Set up the root `package.json` scripts:
   - `"dev"` — runs both frontend and backend via `concurrently -k -n web,api -c blue,green "<frontend_cmd>" "<backend_cmd>"`
   - `"dev:web"` — runs only the frontend
   - `"dev:api"` — runs only the backend
3. Preserve any existing `dev` script by moving it to `dev:web` (or the appropriate sub-script)
4. Use the `-k` flag so Ctrl+C kills both processes

For monorepos, use `pnpm --filter <package> dev` style commands.
For single repos, use the direct commands from the package scripts.

## Important Notes

- Do NOT delete or overwrite existing scripts other than `dev`
- If a `dev` script already uses `concurrently` and looks correct, tell the user it's already set up
- Use `pnpm` if there's a `pnpm-lock.yaml`, `npm` if there's a `package-lock.json`, `yarn` if there's a `yarn.lock`, `bun` if there's a `bun.lockb`
- The naming convention `dev` / `dev:web` / `dev:api` should be consistent across all projects
