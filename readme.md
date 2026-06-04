# Porthole PWA

An animated porthole (round ship window) rendered on an HTML5 canvas, packaged as a Progressive Web App. Built as a learning/demonstration project for PWA development, with a physics-based roll motion simulator.

Live at: **https://kneh.github.io/port-hole/**  
Repo: **https://github.com/kneh/port-hole**

---

## What it does

- Renders a brass-framed porthole with an animated night sea scene (waves, moon, stars, reflections)
- Simulates vessel roll motion: the horizon and waterline shift vertically and tilt based on the porthole's position on the vessel
- Settings panel (toggle with the ⚙ button) lets you configure vessel position and roll parameters
- Works fully offline via service worker caching (PWA)
- Installable on Android and iOS home screen

---

## File structure

```
port-hole/
├── index.html       # entire app — canvas rendering + settings UI + SW registration
├── manifest.json    # PWA manifest (name, icons, theme, start_url)
├── sw.js            # service worker — cache-first offline strategy
├── icons/
│   ├── icon-192.png # PWA icon (brass porthole motif, generated with PIL)
│   └── icon-512.png
└── README.md        # this file
```

No build step, no dependencies, no node_modules. Plain HTML/CSS/JS — deploy by copying files.

---

## Settings panel

| Setting | Description |
|---|---|
| Side | PS (port) or SB (starboard) — flips the roll phase |
| Dist from CoG (m) | Lateral distance from centre of gravity — drives lateral acceleration and apparent water surface tilt |
| Height from CoG (m) | Vertical distance from CoG — controls how much the horizon rises/falls per degree of roll. Negative = below CoG |
| Roll period (s) | Full cycle duration, 4–30 s |
| Roll amplitude (°) | Peak roll angle, 0–30° |

Roll readout (bottom left) shows live angle and rate in °/s.

---

## Roll motion physics

Roll is a pure sinusoid:

```
angle(t) = amplitude × sin(2π/period × t)
rate(t)  = amplitude × (2π/period) × cos(2π/period × t)
```

The porthole view responds to roll in two ways:

**1. Vertical horizon shift**  
The porthole is mounted at `height` metres above CoG and `dist` metres to one side. When the vessel rolls by angle φ (radians):

```
verticalTravel = height × sin(φ)       // metres
horizonShift   = verticalTravel × (IR / 12)   // pixels, IR = inner radius
```

A porthole high above CoG (e.g. +8 m) sees dramatic waterline movement. At CoG level (0 m) there is no vertical shift.

**2. Apparent water surface tilt**  
Lateral acceleration from rolling creates a centripetal force. The free water surface tilts perpendicular to the resultant of gravity + lateral acceleration:

```
lateralAcc   = -dist × ω² × φ          // ω = 2π/period
horizonSlope = atan2(lateralAcc × sideSign, 9.81)   // radians
```

This slope is applied to each wave row as `(x - cx) × horizonSlope`.

**Side sign convention:**  
- PS (port): `sideSign = +1` — rolling to port dips the porthole, horizon rises
- SB (starboard): `sideSign = -1` — rolling to port raises the porthole, horizon falls

---

## PWA notes

- HTTPS required for service worker (GitHub Pages provides this automatically)
- `start_url: "."` and relative paths in `sw.js` are intentional — GitHub Pages serves from a subdirectory (`/port-hole/`), not the root. Using absolute paths (`/`) would break caching.
- Cache name: `port-light-v1` — bump this string in `sw.js` when deploying breaking changes to force clients to re-cache
- Offline indicator badge shown bottom-right when `navigator.onLine === false`
- iOS: install via Safari → Share → Add to Home Screen (no automatic prompt on iOS)
- Android: Chrome shows an automatic "Add to Home Screen" banner

---

## Local development

No build tooling needed. Serve with any static file server:

```bash
# Python
python3 -m http.server 8000

# Node (npx, no install)
npx serve .
```

Then open `http://localhost:8000`. Localhost is the one exception where browsers allow service workers without HTTPS.

To inspect the service worker and cache: Chrome DevTools → Application tab → Service Workers / Cache Storage.

---

## Deploying updates

```bash
git add .
git commit -m "describe change"
git push
```

GitHub Pages redeploys within ~60 seconds. If users are cached on an old version, bumping the `CACHE` string in `sw.js` forces the new service worker to activate and re-cache on next visit.

---

## Roadmap / next steps

- [ ] Pitch motion (fore-aft tilt of horizon)
- [ ] Heave motion (vertical displacement)
- [ ] Daytime sky variant (sun, clouds)
- [ ] Persist settings to localStorage across sessions
- [ ] Accept live motion data over WebSocket instead of simulated sine wave


---

## Tech context

Built during a conversation exploring PWA architecture for industrial marine vessel tooling (stabilizers, HPU systems). The porthole is a standalone demo; the motion physics pattern (sinusoidal roll → viewport transform) is intended as a reusable component for future HMI screens.

The broader project context: Beckhoff TwinCAT 3 / EtherCAT systems, JMobile HMI, Modbus — this PWA layer sits on top as a web-based companion display, not a replacement for the PLC.

---

## License

MIT — see LICENSE file if present, or add one at github.com/kneh/port-hole.
