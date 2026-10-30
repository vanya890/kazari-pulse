# AGENTS.md

Pulse Drift is a rhythm gate runner for Telegram Mini App. The front-end is Vite +
TypeScript + Canvas (no game engine); the backend is Telegram Serverless
(`tgcloud/`). Read [docs/DEPLOY.md](docs/DEPLOY.md) before deploying and
[docs/tgcloud-sdk.md](docs/tgcloud-sdk.md) for the Serverless SDK reference.

## Layout

| Path | What it is |
| --- | --- |
| `src/entry.ts` | Entry point: launches Pulse Drift |
| `src/pulse/main.ts` | Main game loop, menus, UI, shop, themes |
| `src/pulse/game.ts` | Canvas 2D engine: physics, gates, collisions, rendering |
| `src/pulse/style.css` | Cyberpunk style, themes, animations |
| `src/pulse/catalog.ts` | Drones and color themes catalog |
| `tgcloud/schema.js` | DB tables (named exports). |
| `tgcloud/lib/` | Shared server code: catalog, score checks, profiles, payments, bot i18n. |
| `tgcloud/handlers/` | Telegram updates (`message`, `pre_checkout_query`). |
| `tgcloud/endpoints/` | Functions called by the Mini App via `Telegram.WebApp.Serverless.call`. |
| `docs/` | Deployment and SDK notes. |
| `dist/` | Build output uploaded by `tgcloud push` (gitignored). |

## Commands

```sh
npm run dev       # Vite dev server; local fallback when not inside Telegram
npm run build     # tsc --noEmit && vite build
npm run preview   # vite preview
npm run deploy    # build + tgcloud push
npx tgcloud status / diff / push / migrate / run
```

## Rules that bite

- Import Serverless modules with a relative path **and `.js`**: `../schema.js`,
  `./score.js`. The SDK is imported by name: `sdk`, `sdk/db`.
- Every DB call is async. No foreign keys. Drops only via `.deprecated()`.
- Modules under `tgcloud/` cannot import from `src/` or `node_modules/` at runtime.
- Never commit `bot token.txt`, `.tgcloud/`, `.env*`, `dist/`, or
  `.playwright-mcp/`.

## Product invariants

- Score equals number of gates cleared. Record stored in `localStorage`
  under key `kazari.pulse.best.v2`.
- Active drone stored under `kazari.pulse.drone.v1`.
- Active theme stored under `kazari.pulse.theme.v1`.
- Drones and themes are described in `src/pulse/catalog.ts`: `DRONES` and `THEMES`.
- Theme colors are set via CSS variables `--pulse-bg1`, `--pulse-bg2`,
  `--pulse-cyan`, `--pulse-coral`, `--pulse-yellow`.
- When changing `catalog.ts`, verify `main.ts` doesn't reference removed drone/theme IDs.

## Roadmap

Full list of planned improvements is in `ROADMAP.md`.

## Testing notes

- Headless Chromium throttles `requestAnimationFrame` to ~1 fps. In dev,
  call `window.__kazari.game.update(1 / 60)` in a loop for deterministic gameplay.
- `npm run build` is the TypeScript gate (`noUnusedLocals`, `noUnusedParameters`).
