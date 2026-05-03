# CLAUDE.md — operational rules for Mythic Earth

> This file lives in the repo root. Claude Code reads it automatically on every session.
> If you're a human reading this: skim it once, then move on to GAME_DESIGN.md for the design.
> If you're Claude Code: follow these rules. They are not suggestions.

---

## What this project is

**Mythic Earth** is a top-down, voice-driven exploration game for kids, built in Phaser 3 + vanilla JavaScript and deployed to GitHub Pages. Inspired by MyVoiceZoo, reskinned around global folklore, with seven explorable regions, ~42 creatures, hidden ranches, quests, trinkets, and hub decoration.

**Source of truth for design decisions:** `GAME_DESIGN.md`. If a question can be answered there, that's the answer.

**Audience:** built collaboratively by a parent and his ~9-year-old son. The son does not write code, but may glance at it. Code should be readable, comments should explain *why* not *what*.

---

## Hard rules (do not violate)

1. **Phaser 3 + vanilla ES modules.** No TypeScript. No Webpack. No Rollup. Vite for the dev server and production build.
2. **Top-down 32x32 pixel art.** Not isometric. Not 16x16. Not 64x64.
3. **No backend.** The game is fully client-side. No accounts, no cloud save, no analytics, no telemetry.
4. **No ads. No in-app purchases. No paywalls.** This is a kid's project; commercial mechanics are out of scope forever.
5. **No external runtime fetches.** All assets and scripts ship with the build. The game must work offline once loaded.
6. **Voice clips never leave the device.** They live only in IndexedDB on the player's browser. Never upload, log, or transmit recordings.
7. **No npm packages added without asking.** If you need a library, propose it first with the smallest viable alternative for comparison. Brian approves dependencies.
8. **Free assets only.** No paid art. No commissioned art unless Brian explicitly requests it. The asset shortlist is in `README.md`. New packs require a README + CREDITS.md update.
9. **CC-BY assets must have a CREDITS.md entry.** No exceptions. CC0 packs are listed there too for transparency.
10. **Save schema changes require a version bump and a migration function.** The current save key is `mythic_earth_save_v1`. Bump to `v2` and write a migrator if the shape changes. Players must not lose progress.

---

## File and folder layout

```
mythic-earth/
├── README.md
├── GAME_DESIGN.md
├── CLAUDE.md
├── CREDITS.md
├── package.json
├── vite.config.js
├── index.html
├── src/
│   ├── main.js                  # entry point; initializes Phaser
│   ├── config.js                # SINGLE source of all tunable numbers
│   ├── scenes/
│   │   ├── BootScene.js
│   │   ├── PreloadScene.js
│   │   ├── HubScene.js
│   │   ├── RegionScene.js       # parameterized by region id
│   │   └── UIScene.js           # persistent HUD overlay
│   ├── systems/
│   │   ├── SaveSystem.js        # localStorage + IndexedDB
│   │   ├── VoiceSystem.js       # MediaRecorder wrapper + playback
│   │   ├── CoinSystem.js        # idle accrual + offline catch-up
│   │   ├── DiscoverySystem.js   # proximity detection
│   │   ├── QuestSystem.js
│   │   ├── TrinketSystem.js
│   │   └── HubDecorSystem.js
│   ├── data/                    # data-driven content; see "Data files" below
│   │   ├── creatures.js
│   │   ├── regions.js
│   │   ├── trinkets.js
│   │   ├── quests.js
│   │   └── legendaries.js
│   └── ui/
│       ├── FieldJournal.js
│       ├── Catalog.js
│       ├── QuestPanel.js
│       ├── Inventory.js
│       └── RecordModal.js
├── assets/
│   ├── characters/              # kid avatar sprite sheets
│   ├── creatures/               # shared creature source sprites (tinted at runtime)
│   ├── tiles/
│   │   └── <region>/            # one folder per region
│   ├── icons/                   # trinket icons
│   ├── audio/
│   │   ├── music/
│   │   ├── sfx/
│   │   └── ui/
│   ├── ui/                      # UI sprite atlas
│   └── custom/                  # Piskel-drawn art added by Brian's son
└── public/                      # served as-is (favicons, etc.)
```

When in doubt where something belongs, match the existing pattern. Don't invent new top-level folders without proposing it.

---

## Data files (the data-driven rule)

Game content (creatures, regions, trinkets, quests, legendaries) lives in `src/data/*.js` as plain JS modules exporting arrays of objects. **Never hard-code creature data inside a scene file.** Adding a creature should be a one-line append to `data/creatures.js`, no engine changes required.

Example shape (`data/creatures.js`):

```javascript
export const creatures = [
  {
    id: 'geri_wolf',
    name: 'Geri the Wolf',
    region: 'norse',
    baseSprite: 'wolf',          // shared sprite key in assets/creatures/
    tint: 0xCFE4F2,              // applied via setTint() at runtime
    scale: 1.0,
    coinRate: 0.1,               // coins per second when wild
    ranchedRate: 0.2,            // coins per second when in a ranch
    description: 'One of Odin\'s wolves. Quiet and watchful.',
    spawnHint: 'Found near the frozen lakes.',
  },
  // ...
];
```

Same pattern for regions, trinkets, quests. Schemas defined inline at top of each file with a short comment.

---

## The config.js rule

**All tunable numbers live in `src/config.js`.** Coin generation rates, portal costs, voice clip duration, discovery proximity radius, music volumes, save-debounce intervals, animation speeds — anything a designer might want to tweak without touching scene logic.

Bad:
```javascript
// in DiscoverySystem.js
if (distance < 40) { triggerDiscovery(); }
```

Good:
```javascript
// in DiscoverySystem.js
import { config } from '../config.js';
if (distance < config.discovery.proximityRadius) { triggerDiscovery(); }
```

When in doubt, externalize the number.

---

## Reskin and tint rules

Most creatures share a small set of base sprites (wolf, deer, hare, bird, etc.) with a Phaser tint applied at runtime. Defined in `data/creatures.js`. Do not edit source PNGs to recolor them — use `sprite.setTint(color)`.

The only sprites that get their own dedicated PNG file are:
- The kid avatar (PIPOYA base).
- Region-specific creatures with no shared base (Anubis-Jackal, Scarab, Yokai pack creatures, Magicae Dragons).
- Custom Piskel-drawn sprites in `assets/custom/`.

See **GAME_DESIGN.md §22** for the full reskin shortlist and the visual cohesion rules every tint must respect (1px outline, ≤80% saturation).

---

## Coding conventions

- **ES modules.** `import` / `export`. No CommonJS.
- **No globals.** State lives in systems or in the Phaser scene's data registry. Don't attach things to `window`.
- **Naming:** PascalCase for classes/scenes, camelCase for everything else, kebab-case for filenames inside `assets/`.
- **One responsibility per file.** A 500-line file is a smell. Split it.
- **Comments explain *why*, not *what*.** Good: `// Cap offline accrual at 24h to avoid trivializing return-from-vacation.` Bad: `// Set max to 24 hours.`
- **Async:** `async/await` over `.then()` chains. Top-level errors caught and surfaced via the `errorHandler` system.
- **Readability over cleverness.** Imagine a 9-year-old asking "what does this do?" If you can't explain in one sentence, simplify.

---

## Workflow conventions

**When you start a session:**
1. Read this file (you're doing it now).
2. Check `git status` and the most recent commit log to see where things are.
3. If the user references a design point, check `GAME_DESIGN.md` first before asking.

**When you make changes:**
- Small, focused commits. Imperative messages: "Add Norse Fjord region scaffolding," not "added stuff."
- One feature per commit when possible.
- After commit, mention the specific files changed in your reply so Brian can review.

**When you're unsure:**
- If the answer is in `GAME_DESIGN.md`, use it.
- If it's a §21 "Open question" from `GAME_DESIGN.md`, ask Brian explicitly.
- If it's a coding-style call, default to the simplest readable solution.
- If it's a dependency question, propose alternatives with their trade-offs and let Brian pick.

**When you finish:**
- Run `npm run build` before declaring done; broken builds aren't done.
- If applicable, deploy to GitHub Pages and confirm the live URL works.

---

## What NOT to do

- Don't change the visual perspective. Top-down 32x32 is locked.
- Don't migrate to TypeScript, Webpack, React, or any other framework / language.
- Don't add a backend, accounts, ads, IAP, telemetry, or analytics.
- Don't fetch assets from a CDN at runtime — bundle everything.
- Don't store voice clips in localStorage. IndexedDB only.
- Don't commit any actual voice recordings to the repo.
- Don't break the save format without a version bump + migration.
- Don't add npm packages without proposing them first.
- Don't add features that aren't in `GAME_DESIGN.md` without asking. Scope creep is a kid-frustration risk; this game ships with what's spec'd.
- Don't write a 1,000-line file when three 300-line files would do.
- Don't optimize prematurely. 60fps on Chromebook is the target; meet it, then move on.

---

## Performance budget

- **Frame rate target:** 60 fps on a mid-range Chromebook (the son's device).
- **Bundle size cap:** ~50 MB total. Bigger and the game loads slowly on school WiFi.
- **Sprites on screen:** soft cap of 50 active sprites per scene. Use object pooling for trinkets / particles.
- **Voice playback:** hard cap of 3 simultaneous creature voice clips. Queue or drop the rest.

---

## Quick lookups

| Where do I... | Answer |
|---|---|
| ...add a new creature? | Append to `src/data/creatures.js`. |
| ...tune a coin rate? | Edit `src/config.js`. |
| ...change a region's music? | Drop file in `assets/audio/music/`, update `data/regions.js`. |
| ...add a quest? | Append to `src/data/quests.js`. |
| ...wire a custom Piskel sprite? | Drop PNG into `assets/custom/`, reference it in the appropriate `data/*.js` entry. |
| ...add a new HUD button? | Edit `scenes/UIScene.js` and the relevant `ui/*.js` modal. |
| ...look up the design? | `GAME_DESIGN.md`. |
| ...check what assets are sourced from where? | `README.md` asset list, or `GAME_DESIGN.md §15`. |

---

## When in doubt

Ask. Brian would rather answer a quick clarifying question than untangle a wrong assumption later. He likes options with a recommendation; lead with one.

---

*End of CLAUDE.md*
