# Quiet Mistakes

A page that pulls a player's recent games from the public Chess.com API, analyses them with
Stockfish in the browser, and shows where their errors actually cluster — including the games
they *won* that still contain a serious mistake nobody punished.

Everything runs client-side. Nothing is uploaded. The username and the two settings are kept in
`localStorage` so they survive a reload; no games and no analysis are ever written anywhere.

## Files

```
public/index.html              the whole app
public/engine/chess.esm.js     chess.js 1.4.0 — PGN parsing and move generation
public/engine/stockfish.js     Stockfish 10 compiled to WASM (single-threaded)
public/engine/stockfish.wasm   the engine binary (367 KB)
public/engine/COPYING-stockfish.txt  GPL v3, the licence Stockfish.js is distributed under
```

No build step, no dependencies to install, no external CDN at runtime.

## Licences and terms

**Stockfish.js — GPL v3.** Shipped here unmodified, source included, licence text in
`engine/COPYING-stockfish.txt`, upstream at https://github.com/nmrugg/stockfish.js. The GPL
applies to Stockfish itself, not to this page, which only talks to it over UCI.

**chess.js — BSD 2-Clause.** https://github.com/jhlywa/chess.js

**Chess.com Published-Data API.** Free, public, read-only, no key. Their one rule that matters
here: requests go one at a time, never in parallel — see `fetchGames`, which awaits each response
before making the next call. Parallel requests can earn a `429` or an outright block. They also
ask that their board palettes, piece designs, sound effects and move-classification glyphs be left
alone; nothing here uses them. Board colours, Unicode piece glyphs, thresholds and wording are
this project's own.

Their docs ask for a user-agent carrying contact information. A page running in the browser cannot
set one — `User-Agent` is a forbidden header for `fetch` — so each request carries the visitor's
own browser and IP, exactly as if they had opened chess.com themselves.

## Deploy — Cloudflare Workers

The site is served as a Workers static-assets project: no Worker script, no build step.
`wrangler.jsonc` points at `public/`, and everything else in the repo stays unpublished.

From the dashboard: **Workers & Pages** → **Create** → **Continue with GitHub** → pick this
repo. Leave **Build command** empty. Cloudflare reads `wrangler.jsonc` for the rest.

Or from the CLI:

```bash
npx wrangler deploy
```

Every push to `main` redeploys.

## Test it locally first

```bash
cd public && python3 -m http.server 8080
# then open http://localhost:8080
```

It must be served over http, not opened as a `file://` path — the Stockfish worker and the
ES module both need a real origin.

## Checks after deploying

- Enter `adrros07`, 6 games, depth 10. It should finish in seconds.
- The `.wasm` must arrive as `application/wasm`. If the engine never starts, that is the first
  thing to look at in the network tab.
- The only external request should be to `api.chess.com`.
- The footer prints the file's own timestamp, which is the quickest way to tell a stale cache
  from a stale deploy.
