# slides.nesher.co — deployment notes (v3, 2026-09-25)

## Architecture
- Repo: `C:\Users\claude\bolt-slides` (stackblitz/bolt-slides, Vite+React).
- `server.cjs` (v3): multi-deck server, port 8088. `/` → landing; `/<slug>/` → deck dir with index.html fallback. Per-deck access: public (open) or private (basic auth) from `deck.json`. Creds in Vaultwarden ("slides.nesher.co (basic auth)").
- Boot: Scheduled Task "Bolt Slides Node" (SYSTEM, at boot, direct node.exe action).
- Firewall: TCP 8088 (LAN + Tailscale). Cloudflare Tunnel → http://192.168.1.133:8088 (config backup on hpg6: ~/.hermes/data/cf-backups/20260925-slides/).
- Log: `C:\Users\claude\bolt-slides\slides.log`

## Decks (per-presentation URLs + access)
- `decks\<slug>\` = published decks (gitignored; survive git resets).
- `deck.json` = {"title","description","date","access":"public"|"private"}. Missing access ⇒ private. Access is read per request → flips are instant, NO restart.
- Landing: anonymous = public decks only; authed = all decks with chips.
- Publish: `npm run build` → `robocopy dist decks\<slug> /MIR` → write deck.json AFTER mirror (MIR deletes it). Helper: `publish.cmd <slug> "<title>" "<desc>" [public|private]`.
- Current: `/chatlytics-investor/` — Chatlytics investor briefing (15 slides; private).
- `/ralbad-copilot/` — רלב״ד — הצעת Microsoft 365 Copilot (12 slides, Hebrew RTL; private).

## Ops helpers
- `C:\Users\claude\slides-restart.cmd` — kill :8088 + rerun task + probes (needed only when server.cjs changes).
- `C:\Users\claude\bolt-slides\publish.cmd` — build + mirror + deck.json (access-aware).

## Engine upgrades (automatic)
- Weekly cron on hpg6 ("bolt-slides release monitor", Mon 09:37) → `~/.hermes/scripts/bolt-slides-upgrade.py`.
- Patch-preserving: local diff → `C:\Users\claude\bolt-slides-local.patch` (backups: hpg6 `~/.hermes/data/bolt-slides/`), `git reset --hard origin/main`, `git apply --3way`, `npm ci` + build. Untracked files (fonts/, server.cjs, publish.cmd, decks/) survive.
- To pick up engine fixes in a published deck: rebuild + republish it.

## Notes (learned)
- Fonts self-hosted: `src/styles/fonts/*.woff2` + @font-face in `src/styles/base.css`. Never external font CSS.
- `vite.config.ts` `base: './'` required for subpath serving.
- robocopy exit codes 0–7 = OK, ≥8 fail; cmd chain prefer `&`; findstr exits 1 on no-match.
- Verify in real Chromium (Playwright on hpg6) — Camoufox falsely reports font errors.
- SPA deck reveals (Build/Chat messages) need clicks; hash `#N` jumps to slide N on load.
