# AGENTS.md

## Cursor Cloud specific instructions

OpenWA is a NestJS backend (`src/`) plus a React/Vite dashboard (`dashboard/`). It is a
self-hosted WhatsApp API Gateway. Standard commands live in `package.json` (root and
`dashboard/`), `README.md`, and `CONTRIBUTING.md` — reference those rather than duplicating.

### Services / how to run

- Dev (both services, hot reload): `npm run dev` — starts the NestJS API on `http://localhost:2785`
  (API under `/api`, Swagger at `/api/docs`) and the Vite dashboard on `http://localhost:2886`
  (the dashboard proxies `/api` + `/socket.io` to the API). Run it in a persistent/tmux terminal.
- API only: `npm run start:dev`. Dashboard only: `npm run dashboard:dev`.

### Non-obvious caveats

- **No `.env` is required.** On first run the app auto-creates `data/.env.generated` with
  SQLite + local-storage defaults, so it boots with zero external services (no Postgres/Redis/MinIO).
  Precedence is process env > `.env` > `data/.env.generated`. Redis/queue/Docker are optional and
  disabled by default; the `DockerService "Docker not available"` warning at startup is expected and harmless.
- **Admin API key is auto-generated on first boot** and printed to the API logs (a line like
  `owa_k1_...` under `[AuthService] 🔑 API Key (newly created)`). Grab it from the log to log into
  the dashboard or call the API. It persists in `data/.api-key`. Deleting the `data/` dir resets it.
- **The dashboard login field takes that API key** (the button is labeled "Connect").
- **WhatsApp engine (whatsapp-web.js) launches a headless Chromium** when a session is started.
  Puppeteer's bundled Chromium is downloaded during `npm ci` (postinstall) to `~/.cache/puppeteer`.
  Default `PUPPETEER_ARGS` already include `--no-sandbox`, so it launches fine in the container.
  Starting a session (`POST /api/sessions/:id/start`) moves it to `qr_ready` and exposes a QR at
  `GET /api/sessions/:id/qr` — scanning it with a real phone is the only step needing a human.
- **`npm ci` at the repo root also installs the dashboard and applies engine source patches** via
  the root `postinstall` hook (`scripts/postinstall.js` runs `dashboard/` `npm ci` + the
  `patch-wwebjs-*` / `patch-baileys-*` scripts). Do not run a separate dashboard install.

### Lint / test / build

- Backend: `npm run lint`, `npm test` (Jest), `npm run build`.
- Dashboard: `npm --prefix dashboard run lint`, `npm --prefix dashboard test`,
  `npm --prefix dashboard run build`. Build both with `npm run build:all`.
