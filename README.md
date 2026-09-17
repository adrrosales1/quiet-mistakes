# Quiet Mistakes

A page that pulls a player's recent games from the public Chess.com API, analyses them with
Stockfish in the browser, and shows where their errors actually cluster — including the games
they *won* that still contain a serious mistake nobody punished.

Everything runs client-side. Nothing is uploaded. The username and the two settings are kept in
`localStorage` so they survive a reload; no games and no analysis are ever written anywhere.

## Files

```
index.html              the whole app
engine/chess.esm.js     chess.js 1.4.0 — PGN parsing and move generation
engine/stockfish.js     Stockfish 10 compiled to WASM (single-threaded)
engine/stockfish.wasm   the engine binary (367 KB)
engine/COPYING-stockfish.txt  GPL v3, the licence Stockfish.js is distributed under
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

## Deploy — Cloudflare Pages (easiest, ~3 minutes)

1. Cloudflare dashboard → **Workers & Pages** → **Create** → **Pages** → **Upload assets**.
2. Name it `quiet-mistakes`.
3. Drag in the **contents** of this folder — `index.html` and the `engine/` folder.
   (Drag the files, not the enclosing folder, or everything lands one level too deep.)
4. Deploy. You get `https://quiet-mistakes.pages.dev`.

## Deploy — Wrangler CLI

```bash
npx wrangler pages deploy . --project-name quiet-mistakes
```

## Test it locally first

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

It must be served over http, not opened as a `file://` path — the Stockfish worker and the
ES module both need a real origin.

## Checks after deploying

- Enter `adrros07`, 10 games, depth 12. It should finish in well under a minute.
- The `.wasm` must be served as `application/wasm`. Cloudflare Pages does this by default;
  if the engine never starts, that is the first thing to look at in the network tab.
- The only external request should be to `api.chess.com`.
