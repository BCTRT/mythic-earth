# Mythic Earth

A web-based voice-driven exploration game built by a dad and his son.

Wander seven mythological regions of the world as a kid explorer, discover folklore creatures by getting close to them, and give each one its own call by recording your voice. Earn coins as your discovered creatures roam their regions, and spend them to unlock new portals back at the hub city.

No ads. No accounts. No in-app purchases. Just a kid, a microphone, and a globe full of monsters.

---

## About this project

This is a side project built by **Brian** and his son using **Claude** (for design and decisions) and **Claude Code** (for the actual build). The son drives the creative calls — animal choices, voice characters, art reactions — and Brian works with Claude Code on the implementation. The deliverable is a free web game that runs in any modern browser, including a Chromebook, with no install required.

The game is heavily inspired by **MyVoiceZoo** (the original idle-zoo voice-recording game), but reskinned around global folklore and built on an exploration-and-discovery loop instead of a buy-from-menu loop.

---

## The game in one paragraph

You start at a hub city with seven portals — one open, six locked. Step through the open portal and you're in the **Norse Fjord**. A controllable kid explorer wanders the region. Walk near a wild frost giant, a "Discovery!" moment fires, and a recording UI prompts you to give the giant its voice — three seconds, however silly or heroic you want. The giant now lives in the fjord, calling out in your recorded voice. Discovered creatures generate coins over time. Each region has a hidden **house with a ranch** — find it, settle creatures into it, and they earn double coins. Each region also has **5 quests + 1 main quest**, **trinkets** hidden in the world or earned from quests, and **one Legendary creature** that can only be acquired with coins from the region's catalog. Bring trinkets back to the hub to decorate it — the hub fills with mementos from every region you've explored. Save up enough coins to unlock the next portal. Find every creature, voice every one, complete every quest, and earn the title **Master Mythologist**.

---

## Quick facts

- **Title:** Mythic Earth
- **Genre:** Voice-driven exploration / casual idle
- **Platform:** Web browser (Chromebook-first)
- **Tech:** Phaser 3 + vanilla JavaScript
- **Hosting:** GitHub Pages (free)
- **Audio:** Web Audio API + MediaStream Recording API (browser-native, no plugins)
- **Save:** Browser localStorage (one slot per browser, no accounts)
- **Art style:** Top-down 32x32 pixel art
- **Launch regions:** 7 (Norse, Aztec, Celtic, Yokai, Outback, Andean, Sahara)
- **Launch creatures:** ~42 (35 wild discoverable + 7 buyable Legendaries)
- **Launch quests:** 42 (5 per region + 1 main quest per region)
- **Launch trinkets:** ~49 (~7 per region)
- **Per-region houses:** 1 hidden house + ranch each (3–5 creature slots, 2x coin boost)
- **Build budget:** $0 (free assets, free hosting, free tools)
- **Build pace:** open-ended. Worked on session-by-session as time allows.
- **Save system:** auto-save, "welcome back" screen on return, file-based backup/restore (OneDrive-friendly), and 3 save slots — designed so progress is never lost.

---

## Project documents

This project ships with three documents. Read in this order:

1. **README.md** *(this file)* — overview, setup, how to run and deploy.
2. **GAME_DESIGN.md** — the full game spec: mechanics, region/creature roster, art-pack mapping, cohesion plan, phased roadmap. Read this before opening Claude Code.
3. **CLAUDE.md** — sits in the repo root and tells Claude Code our conventions every session. Short and operational; you mostly won't touch it after setup.

---

## Tech stack and why

- **Phaser 3** — most mature 2D HTML5 game engine. WebGL/Canvas rendering, audio, scenes, asset loading, animations all built in. Runs in any browser. Excellent docs and Claude Code knows it well.
- **Vanilla JavaScript (ES modules)** — no TypeScript, no build step beyond a tiny dev server. Keeps the code readable and the project simple to debug on a Chromebook.
- **No backend** — the game is fully client-side. Voice clips and progress live in the player's browser only. Simpler, free, private by default.
- **GitHub** — code repo and version history. Brian commits, GitHub Pages auto-deploys.
- **GitHub Pages** — free static hosting at `https://<username>.github.io/<repo-name>/`. The deploy URL is what the son plays on his Chromebook.
- **Claude Code** — does the actual writing of the game code, guided by GAME_DESIGN.md and CLAUDE.md.

---

## Prerequisites

- A computer that can run Claude Code (the build machine — doesn't need to be the Chromebook).
- Brian's existing **GitHub account**.
- **Node.js 18+** installed on the build machine (Phaser dev server needs it).
- A **microphone** on whatever device the game is played on. Chromebooks have one built in.
- A modern browser (Chrome, Edge, Firefox, Safari) on the play device.

The son's Chromebook only needs the browser — nothing installed there.

---

## First-time setup

### 1. Create the GitHub repository

1. Sign in to GitHub.
2. Click **New repository**.
3. Name it `mythic-earth` (this becomes the URL).
4. Make it **Public** (required for free GitHub Pages hosting).
5. Check "Add a README file."
6. Create.

### 2. Clone the repo onto the build machine

In a terminal:

```bash
git clone https://github.com/<your-username>/mythic-earth.git
cd mythic-earth
```

### 3. Drop the design docs into the repo

Copy these three files from this folder into the repo root:

- `README.md` *(replaces the auto-generated one)*
- `GAME_DESIGN.md`
- `CLAUDE.md`

Commit them:

```bash
git add README.md GAME_DESIGN.md CLAUDE.md
git commit -m "Add design docs"
git push
```

### 4. Open the repo in Claude Code

```bash
claude code
```

Claude Code will automatically read `CLAUDE.md` and follow the conventions in it. Your first prompt to Claude Code should be along the lines of:

> Read GAME_DESIGN.md and scaffold the Phaser 3 project per Phase 2 in the roadmap. Use vanilla ES modules, no TypeScript, no Webpack. Keep it simple enough for a kid to read.

### 5. Install dependencies

Claude Code will likely create a `package.json` and run `npm install`. If you need to do it manually:

```bash
npm install phaser
npm install --save-dev vite
```

### 6. Acquire art assets

All art is free, from four sources (mix of CC0 and CC-BY licenses):

- [Kenney.nl](https://kenney.nl/) — CC0, no attribution required
- [itch.io game assets (free filter)](https://itch.io/game-assets/free) — mixed licenses, check each pack
- [OpenGameArt.org](https://opengameart.org/) — mixed licenses, check each pack
- [CraftPix.net free section](https://craftpix.net/freebies/) — free with attribution

Below is the pack shortlist identified during research. Download each, drop into `assets/` per the structure in CLAUDE.md, and commit to the repo so the game ships self-contained. **GAME_DESIGN.md → Asset map** has the per-region and per-creature breakdown of which pack supplies what.

#### Player character

- [PIPOYA Free RPG Character Sprites 32x32](https://pipoya.itch.io/pipoya-free-rpg-character-sprites-32x32) — kid explorer base, 4-direction walk animations

#### Tilesets (environments)

- [PIPOYA Free RPG World Tileset 32x32](https://pipoya.itch.io/pipoya-free-rpg-world-tileset-32x32-40x40-48x48) — base biome tiles, used as the foundation across multiple regions
- [Mana Seed Gentle Forest by Seliel the Shaper](https://seliel-the-shaper.itch.io/gentle-forest) — Celtic Glen, Andean meadows
- [Viking fantasy tileset by MariaParraGames](https://mariaparragames.itch.io/viking-fantasy-tileset-pixel-art-game-asset) — Norse Fjord
- [Aztec Tileset (OpenGameArt)](https://opengameart.org/content/aztec-tileset) — Aztec Jungle base
- [Aztec Asset Pack by Sevarihk](https://sevarihk.itch.io/aztec-asset-pack) — Aztec Jungle decorations and architecture
- [2D PixelArt Egyptian TileSet by lordlouboo](https://lordlouboo.itch.io/2d-pixelart-egyptian-tileset) — Sahara Oasis primary
- [Fantasy Tileset Desert Oasis by Ventilatore](https://ventilatore.itch.io/the-fantasy-tileset-desert-oasis) — Sahara Oasis supplementary
- [Free Swamp 2D Tileset Pixel Art](https://free-game-assets.itch.io/free-swamp-2d-tileset-pixel-art) — Outback waterholes, Celtic bog edges

#### Creatures

- [CraftPix Free Yokai Pixel Art Character Sprites](https://craftpix.net/freebies/free-yokai-pixel-art-character-sprites/) — Yokai Mountain (kitsune, tanuki, etc.) — animated
- [Elthen 2D Pixel Art Anubis Sprites](https://elthen.itch.io/2d-pixel-art-anubis-sprites) — Sahara (anubis-jackal) — float, hand raised, swing, damage, death animations
- [CraftPix Free Desert Enemy Sprite Sheets](https://craftpix.net/freebies/free-desert-enemy-sprite-sheets-pixel-art/) — Sahara creatures — 6 animated enemies
- [Magicae Free Pixel Art Dragons](https://magicae-games.itch.io/free-pixel-art-dragons) — Yokai ryū dragons, Norse frost wyrms — Air/Earth/Fire/Water types with walk, run, fire-breath, swim, fly
- [CraftPix Free Top-Down Hunt Animals Pixel Sprite Pack](https://craftpix.net/freebies/free-top-down-hunt-animals-pixel-sprite-pack/) — Outback (dingoes, kangaroos), Andean (llamas, condors) — idle, walk, hurt, run, attack, death, flight
- [CraftPix Free RPG Monster Sprites Pixel Art](https://craftpix.net/freebies/free-rpg-monster-sprites-pixel-art/) — generic monster pack for reskinning (trolls, fae, mountain spirits, jaguar gods)
- [Mythological Beasts by Mork Smith](https://mork-smith.itch.io/mythological-beasts) — 40 folklore creature sprites for variety
- [OpenGameArt — Animal Pack](https://opengameart.org/content/animal-pack) — real-animal coverage (giraffe, panda, parrot, monkey, hippo, elephant) for biome flavor
- [Free 16x16 Philippine Mythological Creature Sprites by Shade](https://merchant-shade.itch.io/ph-myth-creatures) — bonus folklore (note: 16x16, will need scaling)
- [Tiny Monsters Pixel Art Pack](https://itch.io/game-assets/tag-32x32/tag-monsters) — 32x32 monster sprites with walking, attack, death, damage animations

#### Audio

- [Kenney audio packs](https://kenney.nl/assets/category:Audio) — UI sounds (clicks, coin chime, discovery sting). CC0.
- [freesound.org](https://freesound.org/) — supplementary sound effects. Mixed licenses, check each clip.
- [incompetech.com (Kevin MacLeod)](https://incompetech.com/music/) — ambient music per region. CC-BY (attribution required in CREDITS.md).

> **License hygiene:** every CC-BY pack we use must be credited in `CREDITS.md` in the repo root. Claude Code will maintain that file as we add packs. CC0 packs do not require attribution but we'll list them anyway for transparency.

---

## Running locally

From the repo root:

```bash
npm run dev
```

Vite (or whatever dev server Claude Code wires up) will print a URL like `http://localhost:5173`. Open that in any browser to play. Saving a code change auto-reloads the page.

To test microphone recording locally, the browser needs a secure context. `localhost` counts as secure, so this just works — no certificate setup needed.

---

## Deploying to GitHub Pages

Once `npm run build` produces a `dist/` folder, deploy it to the `gh-pages` branch:

```bash
npm run deploy
```

(Claude Code will set this script up; it usually wraps the `gh-pages` package.)

Then in the GitHub web UI:

1. Go to repo **Settings → Pages**.
2. Source: **Deploy from a branch**.
3. Branch: **gh-pages**, folder: **/ (root)**.
4. Save.

Wait ~60 seconds. The game is now live at `https://<your-username>.github.io/mythic-earth/`.

Send that URL to the son's Chromebook. Bookmark it. Done.

**One Chromebook note:** the first time he opens the game, the browser will ask permission to use the microphone. He needs to click **Allow**. After that, it remembers.

---

## Build roadmap

Full detail is in **GAME_DESIGN.md → Roadmap**. Short version:

- **Phase 0 — Concept & flair.** Done. Theme, regions, mechanics, and art rules are locked.
- **Phase 1 — Design docs.** Done (you're reading them).
- **Phase 2 — Repo & deploy plumbing.** Create repo, scaffold Phaser project, get a "Hello, Mythic Earth" page deployed to GitHub Pages. Big morale win — the son sees something he had a hand in, live on the web.
- **Phase 3 — MVP loop.** One region (Norse), three creatures, voice recording, idle coins, save/load. Boring but proves the whole pipeline works.
- **Phase 4 — Content expansion.** Add the other six regions and creatures. Wire up the hub, portals, coin gating, field journal, **creature catalog with Legendary purchases, quest panel, regional ranches with the 2x-coin boost, trinket inventory, and hub decoration grid**.
- **Phase 5 — Polish.** Unified shader pass for visual cohesion, ambient music per region, UI sounds, transition effects, region-completion celebrations, quest reward animations.
- **Phase 6 — Share & post-launch.** Send the URL to family. Add concerts, costumes, festivals, more regions, more quests, more trinkets over time as the son requests them.

---

## Working with the son

- **He drives the creatives.** When Claude Code asks "what color should the kid explorer's hat be?" — that's a him question, not a Brian question.
- **Show, don't tell, between phases.** Don't try to explain the code. Show him what it looks like in the browser and let him react.
- **Surprise + reveal cadence.** During the long Phase 4 stretch, share mockups every few days so he stays invested.
- **First voice recording is sacred.** When the recording UI works for the first time, stop everything and record a voice together. That's the moment that sells him on the project.
- **Avoid scope creep that he didn't ask for.** If he wants a "rainbow snake that breathes glitter," that goes on the post-launch list — don't let it derail the launch.

---

## Credits and licenses

- **Code license:** MIT (recommended — permissive, lets anyone learn from it).
- **Art assets:** mix of CC0 and CC-BY. The CC-BY packs require attribution. **GAME_DESIGN.md → Asset map** lists every pack, its source URL, and its license. A `CREDITS.md` file in the repo will collect the attributions.
- **Audio:** mix of CC0 (Kenney audio) and CC-BY (freesound.org clips, incompetech.com music). Same attribution treatment.
- **Voice recordings:** the son's. Stay in his browser, never uploaded anywhere.

---

## Useful links

- [Phaser 3 documentation](https://phaser.io/phaser3)
- [Phaser 3 examples](https://phaser.io/examples/v3)
- [MDN — Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [MDN — MediaStream Recording API](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream_Recording_API)
- [GitHub Pages docs](https://docs.github.com/en/pages)
- [Claude Code docs](https://code.claude.com/docs)
- [Kenney.nl — free game assets](https://kenney.nl/)
- [itch.io — free game assets](https://itch.io/game-assets/free)
- [OpenGameArt.org](https://opengameart.org/)
- [CraftPix.net free section](https://craftpix.net/freebies/)

---

*Built with care, on a budget of zero dollars and one weekend at a time.*
