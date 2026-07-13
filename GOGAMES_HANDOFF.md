# GOGAMES — Project Handoff (bootstrap doc for any Claude session)

Read this first. It replaces re-deriving context. Owner: Brecken (GitHub: GOGAMINGINC).

## What this is
GOGAMES = party games where each player's **phone is the controller** (motion sensors)
and the **laptop is the shared screen**. Two shipped proof-of-concepts:

- **GOGOLF** — phone is a golf club. Live: https://gogaminginc.github.io/gogolf/
- **GOKART** — phone held sideways is a steering wheel. Live: https://gogaminginc.github.io/gokart/

Both are **single-file `index.html`** games (repo root), deployed via **GitHub Pages**
(repos `GOGAMINGINC/gogolf` and `GOGAMINGINC/gokart`, branch main, root).

## Stack
- PeerJS (host peer id `'gogolf-'+CODE` / `'gokart-'+CODE`), QRCode.js, Three.js **r128**, all from cdnjs.
- iPhone Safari: DeviceMotion needs **HTTPS + user-gesture permission** — that's why GitHub Pages, never file://.
- Fonts: Fredoka (display) + Nunito (UI) from Google Fonts.

## Design system (Claude Design "Putt Party" house style — do not drift)
- Navy `#10233F` · Coral `#FF5A5F` (deep `#c9353a`) · Sun `#FFC93C` (deep `#e0a916`) · Grass `#2FBF71` · White.
- Kart-only environment: dark asphalt `radial-gradient(#3a3f4a → #2a2e37 → #1c1f26)`, checkered strips
  `conic-gradient(#10233F 90deg,#f2f4f7 0 180deg,#10233F 0 270deg,#f2f4f7 0)`.
- Chunky rounded corners (14–26px), 3–4px navy borders, **hard offset shadows no blur** (`0 6px 0`).
- Wordmark: `Go` white + second word sun-yellow, Fredoka 700, navy hard text-shadow.
- Characters: **Desk League cast** (8: pencil/fork/crayon/highlighter/mug/eraser/sticky/glue),
  shared makeFace/mitts/shoes builders, localStorage `gogames_character`.
- Player colors: `#FF5A5F #FFC93C #2FBF71 #8a53d6` (karts), full 8 incl `#6fd3ff #23A6D5 #ff9ec4 #FDD64B`.

## Key mechanics (don't rediscover these)
GOGOLF: gyro integrated on locked swing-plane axis (theta/thetaPeak, `swungThrough = |θ|<max(6,peak*.55)`);
power = ∫|accel|dt, `p=clamp((power-18)/40,.02,1)`; CLUBS driver 26–66 / iron 14–46 / wedge 6.5–30 speed,
putts `p=(power-5)/40`, sand: wedge×.6 other×.25; cup capture `dc<.5 && sp<2.2`; water = OB;
`onGreenNow()` excludes bunkers. Canned swing animation synced to real swing timing (user chose this over live mimicry).
GOKART: gravity-vector wheel angle with nearest-90° cardinal latch (`baseCard`) + trim, sent ±60;
`stepKart` physics, STATS {sp,hd,bo} per character; boost fills .08/s on-road, phone fires it by
**double-tap GAS** (`{t:'boost'}`); phone stays minimal: wheel + GAS/BRAKE/CENTER only, meters on laptop HUD;
lap-valid needs halfFlag in band N*.4–.6; Worker clock + substepping (≤.033) for hidden-tab physics.

## Test hooks (headless playtesting from console)
- Golf `window._gg`: state/onData/setMode/render — arm, then `swingstart`/`swing` messages simulate a swing.
- Kart `window._gk`: state()/ctl(s,g,b)/arm()/auto(bool)/bot()/render — e.g. `auto(true);bot();bot();arm()`
  runs a full bot race; poll `state()` for laps/finish. Note: `_gk` add paths don't call updateLobby (fine).

## Deploy ritual (GitHub web upload — no git, no tokens)
1. Edit canonical file on Windows side: `outputs/index.html` (golf) or `outputs/kart3/index.html` (kart).
2. Syntax check in sandbox. **Pitfall:** the /mnt mount clips files to first-seen size — if `wc -c` looks
   stale/truncated, reconstruct via head + append-tail heredoc, or use a fresh filename.
3. Chrome tab → `github.com/GOGAMINGINC/<repo>/upload/main` → `file_upload` tool (reads Windows side correctly).
4. Fill commit message, click Commit. **The first click almost always silently fails** — screenshot,
   re-click the green button (~x390,y639 at 1568-wide), then verify via
   `api.github.com/repos/GOGAMINGINC/<repo>/contents/index.html` (check `size` + `atob(content)` for a
   marker string). raw.githubusercontent caches ~5 min — use the contents API.
5. Wait ~40s for Pages, then fetch the live URL with a cache-bust param and string-check.
6. Playtest via _gg/_gk if logic changed; skip full races for pure-CSS changes.

## Other known pitfalls
- THREE r128: no CapsuleGeometry; Object3D.position not assignable (use .set / partAt helper).
- Road/ribbon meshes need `side:THREE.DoubleSide` (winding).
- Screenshots of never-focused tabs come back black (canvas not composited) — trust HUD/state instead.
- github.com pages CSP-block fetches to github.io — navigate to the game origin before live checks.
- CDP evaluate ~45s limit — fire-then-poll for long waits.
- Records: localStorage `gogolf_best_round`, `gokart_best`.

## Backlog — ALL SHIPPED (2026-07-12/13)
Everything below is built, verified, and live. Landing page now serves at https://gogaminginc.github.io/.
1. **GOGOLF turn-based multiplayer** ✅ — per-player balls (colored course markers), "away plays farthest"
   turn rotation, phone "⛳ YOUR TURN!" / "⏳ waiting" alerts, live laptop leaderboard, end-of-round
   scorecard (per-hole grid + winner), lobby with join list + START ROUND / FREE PLAY. Late joiners sit
   out the current hole. Phone sends {t:'join',name,char}; host broadcasts {t:'turn',your,who,mode}.
2. **GOKART design package** ✅ — In-Race HUD restyle (#posCard/#lapCard/#timePill/#standings/#boostBar,
   navy "CIRCUIT 1" #mmWrap minimap), Pick Your Racer screen (#racerSelect: live 3D kart preview via a
   second WebGLRenderer, stat card from STATS, 8-helmet rail, LOCK IN), DRIFT (GAS+BRAKE in a corner →
   charge → boost on release), ITEMS (🍄 boost / 🍌 banana spin-out / ⭐ star; item boxes + phone USE
   button {t:'item'}), racing helmets on the cast (makeHelmet, bbox-fitted per driver).
3. Nice-to-haves ✅ — ghost kart (best-lap translucent replay), golf hole flyover, sand particles,
   putting-green mode (range→flags→course→putt), GOGAMES landing page.

## Next ideas (fresh backlog — nothing started)
- GOGOLF: per-player character/color pick on the phone; hole flyover skip button; wind indicator polish.
- GOKART: phone-side DRIFT/ITEM buttons (design shows them; currently drift = GAS+BRAKE, item = USE btn);
  more tracks/circuits; blue-shell-style catch-up item; online-gap tuning for standings.
- Shared: real join flow on the landing page; sound/music pass; mobile-landscape host support.

## Token-efficiency rules for future sessions
- Start fresh chats per work session; point Claude at this doc (it's committed in the gokart repo root:
  fetch `https://gogaminginc.github.io/gokart/GOGAMES_HANDOFF.md` — one call, full context).
- Batch several changes per deploy; one commit ritual per batch.
- Grep/targeted edits only — never read the whole 60–75KB game files.
- Verify with API string checks, not screenshots, unless it's a visual change.
- Full bot-race tests only when physics/logic changed.
