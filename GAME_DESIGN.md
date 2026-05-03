# Mythic Earth — Game Design Document

> The full design spec. Read this before opening Claude Code.
> If you only read one section, read **Core Gameplay Loop**.
> Last updated: see commit history.

---

## Table of contents

1. Vision
2. Core gameplay loop
3. Player character
4. Voice recording mechanic
5. Discovery mechanic
6. Coin economy
7. Quest system
8. Creature catalog (and Legendaries)
9. Houses & ranches
10. Trinkets & hub decoration
11. World structure: hub + 7 regions
12. Region detail (creatures, Legendary, trinket list, theme)
13. UI / HUD
14. Visual cohesion plan
15. Asset map
16. Audio design
17. Save / load
18. Accessibility
19. Roadmap
20. Out of scope / post-launch backlog
21. Open questions
22. Asset reskin notes & custom-art shortlist

---

## 1. Vision

**Mythic Earth** is a top-down, voice-driven exploration game for kids. The player wanders seven mythological regions of the world as a kid explorer, discovers folklore creatures by getting close to them, and gives each one a unique call by recording their own voice. Discovered creatures generate idle coins. Coins unlock new regional portals at a central hub. Trinkets found across the regions decorate the hub. Quests give purpose. Hidden ranches let the player adopt favorites and boost their income.

The game has no fail state, no combat, no ads, no in-app purchases, and no accounts. It is a quiet, creative collection-and-personalization toy with enough structure to keep a 9-year-old engaged for weeks.

**What makes it different from MyVoiceZoo:**

- Exploration with a real avatar (not a tap-to-buy menu).
- Discovery is the unlock moment, not a purchase.
- Seven culturally distinct regions instead of one zoo.
- Quests, trinkets, and ranches add long-tail content beyond pure collection.
- Hub decoration creates a visible souvenir shelf of the player's adventure.

---

## 2. Core gameplay loop

```
Spawn at hub city (7 portals visible — 1 unlocked, 6 locked behind coin gates)
   │
   ▼
Walk to unlocked portal → enter region
   │
   ▼
Explore region as kid avatar
   │
   ├── Walk near a wild creature → "Discovery!" → record 3-second voice → creature is now bonded
   ├── Find hidden trinket → added to inventory
   ├── Find quest giver / quest objective → progress quests
   ├── Find region's hidden house → unlocks ranch (3-5 creature slots, 2x coin generation)
   │
   ▼
Open Field Journal / Catalog / Quest panel from HUD as desired
   │
   ▼
Discovered creatures generate idle coins (whether player is in region, hub, or game is closed)
   │
   ▼
Return to hub anytime via "Hub" button
   │
   ├── Spend coins to unlock next portal (gating progression)
   ├── Spend coins to buy region Legendaries (one per region)
   ├── Place trinkets in hub decoration grid (cosmetic + small bonus)
   │
   ▼
Repeat across 7 regions
   │
   ▼
Endgame: discover all ~42 creatures, complete all 42 quests, collect all ~49 trinkets
   → "Master Mythologist" certificate screen
```

**Session length target.** Pickable in 5 minutes (do a quest, find a creature, leave) or sittable for 60 minutes (explore a whole region). The idle coin layer keeps the game rewarding even when not actively playing.

---

## 3. Player character

**Name:** TBD by son. Default placeholder: "the Explorer."

**Sprite source:** PIPOYA Free RPG Character Sprites 32x32. Pick one of the provided character bases as the default; future iteration can recolor or re-sprite to look like the son's idea of an adventurer (backpack, hat, etc.).

**Movement:**
- Keyboard: arrow keys or WASD
- Mouse / trackpad: click-to-move (path-find toward click point)
- Touch (Chromebook tablet mode): tap-to-move

**Animations needed:** idle (single frame OK), walk in 4 directions (4 frames each minimum). PIPOYA pack provides these.

**Speed:** ~96 px/sec (3 tiles/sec) — fast enough that traversal is not tedious, slow enough that proximity-based discovery has time to trigger.

**Collisions:** the player cannot walk through trees, rocks, water, buildings, or ranch fences. Walking into a creature triggers Discovery (see §5), not collision.

**No combat. No health. No fail state.**

---

## 4. Voice recording mechanic

**Trigger:** Discovery moment (§5) auto-opens the Recording Modal.

**Modal UI:**
- Centered card with the creature's name, silhouette, and region.
- Big red "Record" button.
- 3-second countdown ("3… 2… 1…") with a visual ring filling.
- Live waveform visualization while recording.
- Auto-stops at 3 seconds.
- "Play back" button to preview.
- "Use this voice" / "Record again" buttons.
- A close (×) button — but if the player closes without recording, the creature stays undiscovered. Recording is the discovery commitment.

**Browser API:** `navigator.mediaDevices.getUserMedia({ audio: true })` + `MediaRecorder` with WebM/Opus encoding (smallest, well-supported).

**Permission flow:** first time the player tries to discover a creature, the browser asks for microphone permission. We pre-warn with a friendly modal: "Mythic Earth uses your microphone to give creatures their voice. We never send your recordings anywhere — they live only on this device."

**Storage:** voice clips stored as base64-encoded WebM blobs in IndexedDB (NOT localStorage — voice clips are too big for localStorage's 5MB cap; IndexedDB supports tens of MB easily). Key: `voice_<region>_<creature_id>`.

**Re-recording:** any time, via the creature's entry in the Field Journal or Catalog. Re-recording overwrites the prior clip.

**Playback in-world:** when a creature is on-screen, it plays its voice clip on a loop with a randomized 8-30 second interval, mixed at low volume. Multiple creatures call independently. A "Mute creature voices" toggle in settings dampens this for parents who need quiet.

**Optional pitch / filter pass (post-launch):** MyVoiceZoo runs each recording through a pitch shifter to match the animal. We can do the same with `AudioContext.createBiquadFilter()` and `playbackRate` manipulation. Not in MVP — keep recordings raw at first.

---

## 5. Discovery mechanic

**The moment that defines the game.** This is the core hook and it has to feel magical.

**Detection:**
- Each wild creature is placed at a fixed position in its region (set in region map data).
- A trigger zone of radius **40px** (~1.25 tiles) surrounds each undiscovered creature.
- When the player avatar enters the zone, fire the Discovery event.

**Discovery beat (the choreography):**
1. Game pauses (no movement, all idle animations freeze except the creature itself).
2. Camera zooms in slightly on the creature (1.0x → 1.2x over 0.5s).
3. Sparkly particle burst around the creature.
4. Discovery sting plays (short ~1.2s ascending chime — Kenney audio pack).
5. Banner text fades in: **"You discovered: [Creature Name]!"**
6. After 1.2s, banner fades out and Recording Modal opens automatically.

**State after recording:**
- Creature flagged as discovered.
- Voice clip saved.
- Field Journal entry added.
- Catalog entry transitions from silhouette to color.
- Coin generation starts ticking for this creature.
- The creature now wanders its region freely (small idle pacing animation), calling out in the recorded voice on the 8-30 second interval.

**If player closes the modal without recording:** creature stays undiscovered. The proximity trigger goes on a 30-second cooldown so the player can choose to try again later without retriggering immediately.

**Visual hint that an undiscovered creature is nearby:** subtle sparkle particle attached to the creature even before discovery, visible only when within ~80px (so the player knows "something is there" before the trigger fires).

---

## 6. Coin economy

**Single currency: Coins.**

**Generation:**
- Each discovered wild creature: **1 coin / 10 sec** (= 360/hour).
- Each ranched creature: **2 coins / 10 sec** (= 720/hour) — the ranch's whole reason to exist.
- Each Legendary creature (once purchased): **5 coins / 10 sec** (= 1,800/hour) — sets a high passive earn-rate.
- Idle income accrues whether the game is open or not (calculated on next load from a `lastSeen` timestamp).

**Sinks (places to spend coins):**
- **Portal unlocks** — escalating cost (see §11): 200 / 500 / 1,200 / 2,500 / 5,000 / 10,000.
- **Legendary creatures** — one per region, escalating: 300 / 800 / 1,800 / 4,000 / 8,000 / 15,000 / 30,000.
- **Hub decorations** — every trinket placed gives a small flat-rate bonus (+0.05/sec). Effectively free since trinkets are "found," not bought, but placement may have a small coin cost (e.g., 50 each) to give the coin economy reach.
- **(Optional, post-launch) Cosmetic kid avatar customizations** — hats, capes, etc.

**Anti-grind safeguards:**
- After each region is fully discovered + ranched, that region produces a small completion-bonus passive (+50/hr).
- "Sleep cap" on offline accrual: max 24 hours of offline coin gain per session, so coming back after a vacation isn't trivially overpowered.
- All coin numbers are tunable in `config.js` — Claude Code should expose them as a single object so balancing is easy.

---

## 7. Quest system

**Total quests at launch: 42** (5 side quests + 1 main quest per region × 7 regions).

**Quest types:**
- **Discovery quests** — "Discover 5 creatures in the Norse Fjord."
- **Voice quests** — "Record the kappa with a sound that goes 'glug glug.'" (No verification — honor-system; success is on Submit.)
- **Trinket quests** — "Recover the Lost Mead Cask." Trinket is hidden in the world; the quest just points the player toward the area.
- **Ranch quests** — "Settle 3 creatures into your Norse ranch."
- **Find quests** — "Find the hidden house in the Celtic Glen."
- **Story quests (main quest only)** — multi-step folkloric mini-arc per region. Example: in Norse, "Help Skoll the Wolf Pup Find His Pack" — a 3-step quest involving finding 3 wolf creatures, leading to the ranch unlock.

**Quest panel UI:**
- Opens via Quest button in HUD or hotkey `Q`.
- Tabs: **Active** | **Completed** | **All**.
- Each quest entry: name, region, type, progress bar / counter, reward preview.
- Clicking an active quest highlights its objective on the region map (if applicable) and adds a waypoint marker in-world.

**Quest rewards:**
- Side quests: 50-200 coins + occasional trinkets.
- Main quests: 500-2,000 coins + always a trinket + sometimes unlocks a Legendary discount.

**Quest givers:**
- Most quests come from a "Quest Board" sign in each region (visible from arrival).
- Some are auto-acquired by entering an area or discovering a creature.
- The hub city has a Master Quest Board listing every region's main quest.

**Authoring note for Claude Code:** quests live in `data/quests.js` as a flat array of quest objects (id, region, type, title, description, requirement, reward, prereq). New quests should be addable by appending to the array — no engine changes required.

---

## 8. Creature catalog (and Legendaries)

**Catalog UI:**
- Opens via Catalog button in HUD or hotkey `C`.
- Top of panel: region selector (tabs for each region; current region pre-selected if player is in one).
- Body: grid of creature cards. Each card:
  - Discovered: full color sprite, name, "Voice: ▶ Play" button to replay the recording, "Re-record" button, "In ranch / In wild" indicator.
  - Undiscovered (wild): silhouette + "???" name + a faint hint ("Found in the deep forest").
  - Legendary (locked): silhouette + name + cost in coins + "Buy" button. Greyed if player can't afford.
  - Legendary (unlocked): same as discovered, plus a small star icon.
- Bottom: per-region completion percentage and total-game completion percentage.

**Legendary creatures (one per region):**

- **Norse Fjord:** Jörmungandr (world-serpent hatchling) — 300 coins
- **Aztec Jungle:** Quetzalcoatl — 800 coins
- **Celtic Glen:** Cù-sìth (giant green hound) — 1,800 coins
- **Yokai Mountain:** Ryū (eastern dragon) — 4,000 coins
- **Outback Red Desert:** Rainbow Serpent — 8,000 coins
- **Andean Peaks:** Amaru (winged serpent) — 15,000 coins
- **Sahara Oasis:** Bennu (phoenix) — 30,000 coins

Costs scale roughly to portal-unlock progression so Legendaries are aspirational late-game goals, not impulse buys.

**Legendaries spawn rules:**
- Never appear wild — only obtainable via Catalog purchase.
- Once purchased, the Legendary appears in the region as a special, larger sprite, glowing softly.
- Recording the Legendary's voice is the same flow as a wild creature, fired automatically right after purchase.
- Legendaries earn 5 coins/10 sec — the highest rate in the game.

---

## 9. Houses & ranches

**One per region. Hidden. Discoverable.**

**Discovery:**
- Each region has a hidden house at a designed location.
- The house is invisible/inaccessible until a small condition is met:
  - Region 1 (Norse): wander into a specific glade.
  - Region 2 (Aztec): solve a tile-puzzle on a temple floor.
  - Region 3 (Celtic): follow fairy lights through a forest path.
  - Region 4 (Yokai): light 3 lanterns in a sequence.
  - Region 5 (Outback): follow dingo footprints to a billabong.
  - Region 6 (Andean): find a hidden trail behind a waterfall.
  - Region 7 (Sahara): dig at a marked X in the dunes.
- These conditions can also be a region's Main Quest — "Find your home in the Norse Fjord."

**House interior (optional micro-feature, can defer to post-launch):**
- A small interior scene (single screen) the player can enter.
- Hosts the region's trinket display (a shelf with placeable trinket slots).
- Provides a "Sleep" interaction that fast-forwards coin generation by 1 hour (limited use).

**Ranch:**
- Fenced area attached to the house exterior.
- 3 slots at first; expandable to 5 by completing the region's main quest.
- Drag-and-drop discovered creatures from the Field Journal into ranch slots.
- Ranched creatures earn 2x coins, are visible in the ranch (animated), and call their voice 2x more often.
- Creatures can be moved between wild and ranched state freely (no cost).

**Asset needs:**
- 7 region-themed house exteriors (32x32-ish single sprite or small multi-tile build).
  - Norse: longhouse with sod roof
  - Aztec: stepped pyramid hut
  - Celtic: stone roundhouse with thatched roof
  - Yokai: traditional Japanese minka with paper walls
  - Outback: shaded shack with corrugated roof
  - Andean: stone-and-thatch chukllu
  - Sahara: domed adobe with palms
- 7 region-themed fence styles (or one universal fence with palette swaps).

The CraftPix / itch.io tilesets per region include suitable building sprites; we'll reskin or filter for cohesion.

---

## 10. Trinkets & hub decoration

**Total trinkets: ~49 (7 per region × 7 regions).**

**How they're found (mix per §quest answer):**
- **~50% hidden in the world** — sparkle particle on the ground, in a chest, behind a pushable rock, on a tree branch. Player walks up and presses Interact to collect.
- **~50% quest rewards** — given on quest completion, side or main.

**Trinket inventory:**
- Accessible via Inventory button in HUD or hotkey `I`.
- Shows all collected trinkets, grouped by region.
- Each trinket: name, source ("Found in Norse Fjord glade" or "Reward from quest 'Help Skoll'"), small flavor text ("A horn carved from the antler of an old elk; its rim is dark with old mead.").

**Hub decoration:**
- The hub city has a placement grid overlay (toggled with a "Decorate" mode button).
- In Decorate mode: trinkets in inventory appear as draggable items at the bottom of the screen.
- Player drags trinkets onto floor tiles in the hub.
- Trinket sprites stay placed and visible.
- Each placed trinket gives a +0.05/sec coin bonus (cumulative).
- Placement costs 50 coins per trinket — small sink to give coins reach.
- Trinkets can be picked back up and re-placed at no cost.
- A "Reset hub" button clears all placed trinkets back to inventory.

**Why this matters for the game:** the hub becomes a visual scrapbook of the player's journey. After all regions are explored, the hub looks completely different than at game start — it's the visible reward of the whole experience.

---

## 11. World structure: hub + 7 regions

### Hub city

- A small isometric-ish village (top-down 32x32) at the world's "center."
- 7 portal arches arranged in a circle around a central plaza.
- 1 portal is open (Norse Fjord, free).
- 6 portals are closed: a stone door over each, with a coin-cost sign ("Unlocks for 500 coins").
- A Master Quest Board near the plaza.
- A "Decorate" toggle button (see §10).
- A "Field Journal" book on a pedestal (alt access to journal).
- A music box icon in the corner = settings (mute, voice settings, etc.).

**Portal unlock costs (escalating):**
- Norse Fjord — free (starting region)
- Aztec Jungle — 200
- Celtic Glen — 500
- Yokai Mountain — 1,200
- Outback Red Desert — 2,500
- Andean Peaks — 5,000
- Sahara Oasis — 10,000

**The order is suggested, not enforced** — the player can save up and unlock any region in any order after Norse. (We may wire it as fully open, OR enforce sequential — call this in implementation.)

### Region structure

Each region is a single scene (one screen, scrollable) approximately **1024x1024px** (32x32 tiles laid 32 wide × 32 tall). Edges are bounded — no infinite worlds.

Each region contains:
- A return-to-hub portal (visible at edge or center).
- 5 wild creatures at fixed locations.
- ~3-4 trinkets hidden in the world.
- 1 Quest Board sign.
- 1 hidden house + ranch (locked until discovered).
- Region-themed environment (terrain, props, ambient creatures that aren't part of the roster).
- Optional ambient music track (region-specific, looped).

---

## 12. Region detail

> **Note on this section:** every creature and trinket below is anchored to a confirmed free art pack. Where folkloric naming is used, the underlying sprite is a real-animal or generic-monster sprite that's been recolored or contextually renamed. The few items that truly need custom pixel art (kangaroo, llama, condor — the iconically region-specific creatures we couldn't source) are flagged and listed in §22 as a Piskel side-project for the son.

### 12.1 Norse Fjord

- **Theme:** snowy fjord with wooden longhouses, frozen lakes, mountain peaks.
- **Tileset:** Viking fantasy tileset (MariaParraGames) + PIPOYA snow base.
- **Music vibe:** light orchestral with a hardanger fiddle.
- **Wild creatures (5):**
  1. **Geri the Wolf** — CraftPix Hunt Animals (wolf, white/grey tint)
  2. **Snow Hare** — CraftPix Hunt Animals (hare, white tint)
  3. **Huginn the Raven** — OpenGameArt Animal Pack (bird, dark recolor)
  4. **Gullinbursti the Boar** — CraftPix Hunt Animals (boar, golden-brown tint)
  5. **Forest Fox** — CraftPix Hunt Animals (fox, frosted tint)
- **Legendary:** **Jörmungandr** (sea-serpent hatchling, 300 coins) — Magicae water dragon, recolored serpentine
- **Trinkets (7):** Rune Stone, Drinking Horn, Raven Feather, Bone Necklace, Mead Bottle, Iron Helm, Mjölnir Pendant — all sourceable from the 496 CC0 RPG Icons pack and Caz Free RPG Icons
- **Hidden house location:** behind a frozen waterfall in the NE quadrant.
- **Main quest:** "Help Geri Find His Pack" — discover all 5 wild Norse creatures.

### 12.2 Aztec Jungle

- **Theme:** stepped pyramids, jungle canopy, cenotes, marigolds.
- **Tileset:** Aztec OGA tileset + Sevarihk Aztec Asset Pack.
- **Music vibe:** clay flutes and rattles over warm percussion.
- **Wild creatures (5):**
  1. **Jaguar Cub** — CraftPix Hunt Animals (large cat sprite, recolored)
  2. **Capuchin Monkey** — OpenGameArt Animal Pack (monkey)
  3. **Xolo Dog** — CraftPix Hunt Animals (dog, dark coat)
  4. **Coati** — CraftPix Hunt Animals (raccoon, recolored auburn)
  5. **Macaw** — OpenGameArt Animal Pack (parrot, vibrant red/blue)
- **Legendary:** **Quetzalcoatl** (feathered serpent, 800 coins) — Magicae air dragon, recolored green-red feathered
- **Trinkets (7):** Obsidian Dagger, Jade Idol, Gold Sun Disc, Codex Scroll, Feather Headdress, Jaguar Tooth, Jungle Flower — all sourceable from generic RPG icon packs
- **Hidden house location:** atop a small temple, accessible after solving a 4-tile pressure-plate puzzle.
- **Main quest:** "Decode the Sun Stone" — find 3 sun-stone fragment trinkets.

### 12.3 Celtic Glen

- **Theme:** mossy forest, standing stones, mushroom rings, gentle streams.
- **Tileset:** Mana Seed Gentle Forest + PIPOYA grass base.
- **Music vibe:** harp and tin whistle, soft and melodic.
- **Wild creatures (5):**
  1. **Forest Stag** — CraftPix Hunt Animals (deer)
  2. **Wise Owl** — OpenGameArt Animal Pack (owl)
  3. **Glen Fox** — CraftPix Hunt Animals (fox, russet tint)
  4. **Forest Hare** — CraftPix Hunt Animals (hare)
  5. **Forest Boar** — CraftPix Hunt Animals (boar, mossy-brown tint)
- **Legendary:** **Cù-sìth** (giant green hound, 1,800 coins) — CraftPix Hunt Animals wolf, scaled up + bright green tint
- **Trinkets (7):** Lucky Clover, Druid's Staff, Faerie Bell, Brigid's Cross, Celtic Knot Pendant, Mossy Rune Stone, Honey Mead Pot — all sourceable from generic RPG icon packs
- **Hidden house location:** at the end of a path of fairy lights that appears only after the player enters a specific mossy clearing.
- **Main quest:** "Follow the Fairy Lights" — discover all 5 wild Celtic creatures, which lights the path.

### 12.4 Yokai Mountain

- **Theme:** torii gates, cherry blossoms, paper lanterns, koi pond, bamboo grove.
- **Tileset:** **GuttyKreum's Japan Collection: Japanese City (Free)** — 379 free 32x32 tiles of authentic Japanese architecture.
- **Music vibe:** koto, shakuhachi flute, soft taiko.
- **Wild creatures (5):**
  1. **Kitsune** (fox spirit) — CraftPix Free Yokai Pixel Art Character Sprites
  2. **Tanuki** (raccoon-dog) — CraftPix Free Yokai Pixel Art Character Sprites
  3. **Kappa** (river imp) — CraftPix Free Yokai Pixel Art Character Sprites
  4. **Tengu** (mountain spirit) — CraftPix RPG Monster Sprites (winged humanoid, recolored)
  5. **Sacred Shika Deer** — CraftPix Hunt Animals (deer, lighter tint)
- **Legendary:** **Ryū** (eastern dragon, 4,000 coins) — Magicae fire dragon, recolored with gold/red palette
- **Trinkets (7):** Tea Cup, Bamboo Flute, Paper Lantern, Kitsune Mask, Hairpin, Stone Lantern, Wishing Tag — sourceable from generic RPG icon packs
- **Hidden house location:** unlocked by lighting 3 paper lanterns in the correct order (clue is in the trinket flavor text).
- **Main quest:** "Light the Lanterns" — find 3 lantern trinkets and light them in correct order.

### 12.5 Outback Red Desert

- **Theme:** red rock, billabongs, eucalyptus, ancient dot-painting motifs.
- **Tileset:** CraftPix Desert + Free Swamp tileset for the billabong.
- **Music vibe:** mid-tempo percussion with low drone (didgeridoo audio sample, not sprite).
- **Wild creatures (5):**
  1. **Outback Dingo** — CraftPix Hunt Animals (canine, tan tint)
  2. **Outback Brumby** — OpenGameArt Animal Pack (horse, wild reddish coat)
  3. **Frilled Snake** — CraftPix Hunt Animals (snake, recolored)
  4. **Sand Hare** — CraftPix Hunt Animals (hare, sand tint)
  5. **Cockatoo** — OpenGameArt Animal Pack (parrot, white with yellow crest tint)
- **Legendary:** **Rainbow Serpent** (8,000 coins) — Magicae any dragon, rainbow palette
- **Trinkets (7):** Painted Stone, Ochre Pigment, Eucalyptus Sprig, Opal Gem, Animal Tooth, Wooden Charm, Carved Wood Disc — all sourceable from generic RPG icon packs
- **Hidden house location:** follow dingo paw prints across the desert to a billabong.
- **Main quest:** "Track the Dingo" — discover the dingo, then follow paw-print trail.
- **Optional custom art:** **Red Kangaroo** can be added later as a Piskel-drawn 6th creature (see §22). Keeps the Outback iconography alive without blocking launch.

### 12.6 Andean Peaks

- **Theme:** stepped mountain terraces, stone pathways, snowy peaks, soaring birds.
- **Tileset:** PIPOYA mountain + Mana Seed snow accents for high peaks.
- **Music vibe:** flutes and strings (panpipe audio sample, not sprite).
- **Wild creatures (5):**
  1. **Mountain Stag** — CraftPix Hunt Animals (deer, sturdy gray-brown tint)
  2. **Andean Fox** — CraftPix Hunt Animals (fox, gray tint)
  3. **Mountain Hare** — CraftPix Hunt Animals (hare, slate tint)
  4. **Mountain Owl** — OpenGameArt Animal Pack (owl)
  5. **Mountain Eagle** — OpenGameArt Animal Pack (large bird, dark plumage tint)
- **Legendary:** **Amaru** (winged serpent, 15,000 coins) — Magicae air dragon, recolored as winged serpent
- **Trinkets (7):** Mountain Crystal, Carved Stone Disc, Woolen Scarf, Wooden Cup, Pan Pipes, Sun Idol, Knotted Rope — all sourceable from generic RPG icon packs
- **Hidden house location:** behind a waterfall on a hidden mountain trail.
- **Main quest:** "Climb to the Sky Temple" — multi-step ascent unlocks at 3 creatures discovered.
- **Optional custom art:** **Llama** and **Andean Condor** can be added later as Piskel-drawn 6th and 7th creatures (see §22). Iconic but truly unsourceable in free packs.

### 12.7 Sahara Oasis

- **Theme:** dunes, palms, oasis pool, pyramid silhouette in the distance.
- **Tileset:** lordlouboo 2D Egyptian Tileset + Ventilatore Desert Oasis.
- **Music vibe:** oud and frame drum.
- **Wild creatures (5):**
  1. **Anubis-Jackal Pup** — Elthen 2D Pixel Art Anubis Sprites (already animated: float, swing, damage, death)
  2. **Sacred Scarab** — dedicated 2D Pixel Art Scarab Sprites pack
  3. **Sand Viper** — CraftPix Hunt Animals (snake, sand tint)
  4. **Fennec Fox** — CraftPix Hunt Animals (fox, light tan with oversized ears via tint)
  5. **Desert Mummy** — CraftPix Free Desert Enemy Sprite Sheets (mummy or sphinx variant)
- **Legendary:** **Bennu Phoenix** (30,000 coins) — Magicae fire dragon, recolored gold-and-red plumage
- **Trinkets (7):** Scarab Amulet, Papyrus Scroll, Ankh Pendant, Glass Jewel, Ostrich Feather, Palm Date, Pyramid Charm — all sourceable from generic RPG icon packs (Scarab Amulet is in the dedicated Scarab pack)
- **Hidden house location:** dig at an X marked on a buried map fragment.
- **Main quest:** "Decode the Buried Map" — collect 4 map fragment trinkets to reveal the dig site.

---

## 13. UI / HUD

### Persistent on-screen HUD

A minimal frame so the world stays readable. All elements anchored to corners; semi-transparent backgrounds.

**Top-left:**
- Coin counter with animated chime on increment.
- Current region name.

**Top-right:**
- Settings cog (mute, voice volume, creature voice volume, music volume, save/reset).

**Bottom-left:**
- Field Journal button (`J`)
- Catalog button (`C`)
- Quest button (`Q`)
- Inventory button (`I`)

**Bottom-right:**
- Hub button (instant-teleport to hub from any region — small fade transition).
- Map / mini-map (optional post-launch).

### Modals

All modals dim the background and pause the world.

- **Recording modal** (§4)
- **Field Journal** — list of all discovered creatures with their voices replayable.
- **Catalog** — region-tabbed grid with cards (§8).
- **Quest panel** — tabbed Active / Completed / All (§7).
- **Inventory** — region-tabbed trinket grid (§10).
- **Settings** — sliders for audio, mic test, save management, "About" page.

### Decorate mode (hub only)

- Toggleable. Hides the regular HUD.
- Shows a draggable trinket palette at the bottom.
- A grid overlay on placeable floor tiles.
- Drag trinkets onto tiles. Right-click (or long-press) a placed trinket to pick it back up.
- An "Exit Decorate" button at the top.

### Visual style for UI

- Soft cream parchment look with a thin dark border (matches a folklore-book aesthetic).
- Pixel-art font (e.g., m5x7 or similar) for in-world labels.
- Cleaner sans-serif fallback for body text in modals (better readability for kids).

---

## 14. Visual cohesion plan

The seven regions come from different artists. Without intentional effort they read as "stitched-together free packs." With intentional effort they read as one game with regional flavor.

### Tactics

1. **Unified post-processing shader.** Every region scene runs through a Phaser pipeline that applies:
   - Slight palette nudge (toward a shared muted-warm overall mood).
   - Subtle vignette darkening the screen edges.
   - 1-2% film grain.
   - Region-specific tint can layer on top (Norse cooler, Sahara warmer).
2. **Outline rule.** All sprites must have a 1-pixel dark outline. Packs without one get processed (Phaser shader pass) to add one. Packs that don't conform after that are dropped.
3. **Saturation cap.** No sprite can exceed 80% saturation. Excessively vivid packs get desaturated.
4. **Kid avatar at full saturation.** Always reads as foreground.
5. **Persistent HUD frame.** Consistent across every region.
6. **Consistent transition beats.** All region entries fade to black 0.4s, splash region name 0.6s, fade in 0.4s. Same on exit.
7. **Audio glue.** All regional music shares one composer's instrumentation grammar (Kevin MacLeod / incompetech.com or a similar single source). Different folk instruments, similar production.
8. **Shared particle, font, and UI sprite library.** No matter the region, sparkles look the same, fonts are the same, buttons are the same.

### Style audit milestone

End of Phase 3 / start of Phase 4: place 2 regions side-by-side and run an eye-test with the son. If a region or pack fights the others, swap or filter before scaling to all 7.

---

## 15. Asset map

This section maps each requirement to a specific source pack. **The README has the full link list** — this section is the per-region resolution.

### Player
| Need | Pack | Source |
|---|---|---|
| Kid avatar (idle + walk × 4 directions) | PIPOYA Free RPG Character Sprites 32x32 | itch.io |

### Norse Fjord
| Need | Source pack |
|---|---|
| Snowy fjord tiles | Viking fantasy tileset (MariaParraGames) + PIPOYA snow base |
| Geri the Wolf | CraftPix Hunt Animals — wolf, white/grey tint |
| Snow Hare | CraftPix Hunt Animals — hare, white tint |
| Huginn the Raven | OpenGameArt Animal Pack — bird, dark recolor |
| Gullinbursti the Boar | CraftPix Hunt Animals — boar, golden-brown tint |
| Forest Fox | CraftPix Hunt Animals — fox, frosted tint |
| Jörmungandr (Legendary) | Magicae Dragons — water dragon, serpentine recolor |
| Longhouse exterior | MariaParraGames Viking buildings |

### Aztec Jungle
| Need | Source pack |
|---|---|
| Jungle + temple tiles | Aztec OGA tileset + Sevarihk Aztec Asset Pack |
| Jaguar Cub | CraftPix Hunt Animals — large cat, recolored spots |
| Capuchin Monkey | OpenGameArt Animal Pack — monkey |
| Xolo Dog | CraftPix Hunt Animals — dog, dark coat |
| Coati | CraftPix Hunt Animals — raccoon, auburn tint |
| Macaw | OpenGameArt Animal Pack — parrot, vibrant red/blue |
| Quetzalcoatl (Legendary) | Magicae Dragons — air dragon, green-red feathered |
| Stepped pyramid | Sevarihk Aztec Asset Pack |

### Celtic Glen
| Need | Source pack |
|---|---|
| Forest tiles | Mana Seed Gentle Forest + PIPOYA grass base |
| Forest Stag | CraftPix Hunt Animals — deer |
| Wise Owl | OpenGameArt Animal Pack — owl |
| Glen Fox | CraftPix Hunt Animals — fox, russet tint |
| Forest Hare | CraftPix Hunt Animals — hare |
| Forest Boar | CraftPix Hunt Animals — boar, mossy-brown tint |
| Cù-sìth (Legendary) | CraftPix Hunt Animals — wolf, scaled up + bright green tint |
| Stone roundhouse | Mana Seed buildings (or generic medieval pack) |

### Yokai Mountain
| Need | Source pack |
|---|---|
| Asian-themed tiles | **GuttyKreum Japan Collection: Japanese City (Free)** — 379 free 32x32 tiles |
| Kitsune | CraftPix Free Yokai Pixel Art Character Sprites |
| Tanuki | CraftPix Free Yokai Pixel Art Character Sprites |
| Kappa | CraftPix Free Yokai Pixel Art Character Sprites |
| Tengu | CraftPix RPG Monster Sprites — winged humanoid, recolored |
| Sacred Shika Deer | CraftPix Hunt Animals — deer, lighter tint |
| Ryū (Legendary) | Magicae Dragons — fire dragon, gold/red palette |
| Minka house | GuttyKreum Japan Collection — Japanese buildings |

### Outback Red Desert
| Need | Source pack |
|---|---|
| Red rock + billabong tiles | CraftPix Desert + Free Swamp tileset |
| Outback Dingo | CraftPix Hunt Animals — canine, tan tint |
| Outback Brumby | OpenGameArt Animal Pack — horse, reddish coat |
| Frilled Snake | CraftPix Hunt Animals — snake, recolored |
| Sand Hare | CraftPix Hunt Animals — hare, sand tint |
| Cockatoo | OpenGameArt Animal Pack — parrot, white/yellow tint |
| Rainbow Serpent (Legendary) | Magicae Dragons — any base, rainbow palette |
| Outback shack | Generic medieval/farm building pack |
| **Red Kangaroo** *(optional, custom)* | **Piskel-drawn by son** — see §22 |

### Andean Peaks
| Need | Source pack |
|---|---|
| Mountain tiles | PIPOYA mountain + Mana Seed snow accents |
| Mountain Stag | CraftPix Hunt Animals — deer, sturdy gray-brown tint |
| Andean Fox | CraftPix Hunt Animals — fox, gray tint |
| Mountain Hare | CraftPix Hunt Animals — hare, slate tint |
| Mountain Owl | OpenGameArt Animal Pack — owl |
| Mountain Eagle | OpenGameArt Animal Pack — large bird, dark plumage tint |
| Amaru (Legendary) | Magicae Dragons — air dragon, winged-serpent recolor |
| Stone chukllu | PIPOYA mountain building (or generic) |
| **Llama** *(optional, custom)* | **Piskel-drawn by son** — see §22 |
| **Andean Condor** *(optional, custom)* | **Piskel-drawn by son** — see §22 |

### Sahara Oasis
| Need | Source pack |
|---|---|
| Desert tiles | lordlouboo 2D Egyptian Tileset + Ventilatore Desert Oasis |
| Anubis-Jackal Pup | **Elthen 2D Pixel Art Anubis Sprites** (animated: float, swing, damage, death) |
| Sacred Scarab | Dedicated **2D Pixel Art Scarab Sprites** pack |
| Sand Viper | CraftPix Hunt Animals — snake, sand tint |
| Fennec Fox | CraftPix Hunt Animals — fox, light tan tint |
| Desert Mummy | CraftPix Free Desert Enemy Sprite Sheets |
| Bennu Phoenix (Legendary) | Magicae Dragons — fire dragon, gold-and-red plumage |
| Adobe house | lordlouboo Egyptian buildings |

### Trinkets

49 trinket sprites at launch (~7 per region). Sourcing strategy:

**Primary source (~85% of trinkets):** **496 Pixel Art Icons for Medieval/Fantasy RPG (CC0)** on OpenGameArt + **Caz Pixel Fantasy RPG Icons Free Collection** (CC-BY) on itch.io. Together these provide thousands of generic fantasy items: weapons, gems, scrolls, jewelry, food, tools, runes, masks, charms, idols, statues, bones, feathers, flowers, drinks, books, lanterns, helms, and more. Most named trinkets in §12 map directly — e.g., "Druid's Staff" is a generic staff icon, "Mead Bottle" is a generic potion bottle, "Iron Helm" is a helmet icon, "Brigid's Cross" is a cross pendant. The trinket's flavor text in the Inventory UI is what gives it regional identity, not the sprite's exact form.

**Secondary source:** Kenney's UI/icon packs (CC0) for any gaps.

**Tertiary (custom art, ~6 trinkets, deferrable to post-launch):** A handful of culturally-specific trinkets do not exist in any free pack. They're listed in §22 with a Piskel side-project plan for the son to draw himself. The game ships fine without them; they can be added at any time without rework.

### Audio

| Need | Source |
|---|---|
| UI clicks, coin chime, discovery sting | Kenney audio packs (CC0) |
| Region ambient music (×7) | Kevin MacLeod / incompetech.com (CC-BY) |
| Footstep, ambient critter chirps | freesound.org (CC0 only — filter strict) |

---

## 16. Audio design

- **Master volume slider**, plus three sub-sliders: Music, SFX, Creature Voices.
- **Mute creature voices toggle** — for parents who need quiet.
- **Music ducks** when modals open (-12 dB).
- **Creature voices** play at randomized 8-30s intervals, max 3 simultaneous to prevent cacophony.
- **Discovery sting** plays at full volume even with music ducked.
- **Region transitions** include a brief silence (0.4s) before the new region's music starts.

---

## 17. Save / load

The save system is built so the player's progress is **never lost**. A 9-year-old should be able to close the game mid-quest, come back next week, switch to a different device, or share the Chromebook with a sibling — and his world is exactly where he left it.

### Storage

- **`localStorage`** for game state (coins, discovered creatures, trinkets, quest progress, ranch assignments, hub decorations, settings, last-seen timestamp). One key per save slot.
- **`IndexedDB`** for voice clips (too large for localStorage). Keyed by slot + creature ID.

### Save schema (per slot, in `localStorage` under key `mythic_earth_save_v1_slot<N>`)

```json
{
  "version": 1,
  "slotName": "Explorer",
  "createdAt": <timestamp>,
  "lastSeen": <timestamp>,
  "coins": <number>,
  "playerPosition": { "region": "norse", "x": 480, "y": 320 },
  "discovered": [ { "id": "geri_wolf", "voiceKey": "voice_norse_geri_wolf", "ranched": false } ],
  "legendariesOwned": [ "jormungandr" ],
  "regionsUnlocked": [ "norse", "aztec" ],
  "trinkets": [ { "id": "rune_stone", "placedAt": null } ],
  "hubDecorations": [ { "trinketId": "rune_stone", "x": 100, "y": 200 } ],
  "quests": { "active": [ ... ], "completed": [ ... ] },
  "ranches": { "norse": ["geri_wolf", "snow_hare"], ... },
  "settings": { "musicVolume": 0.6, "creatureVoiceVolume": 0.7, ... }
}
```

### Auto-save triggers

- On every state change (debounced to once per 2 seconds — avoids thrashing).
- On window blur / before-unload event (catches "closed the tab" cleanly).
- On scene transition (entering a region, returning to hub).
- Voice recordings are persisted to IndexedDB the moment recording finishes.

### Feature 1: Visible auto-save indicator

A tiny **"Saved ✓"** toast in the bottom-right corner of the HUD that fades in for ~1.5 seconds whenever a save fires. Subtle but visible — reassures the player that progress is real and persistent. Opacity capped low enough that it never blocks gameplay.

### Feature 2: Welcome-back screen

The first thing the player sees when they open the game (after the title screen). A friendly card showing what they already have:

```
Welcome back, Explorer!
Last played: yesterday at 4:32 PM
You've discovered 8 creatures.
1,240 coins.
3 quests in progress.
Your llama and condor say hi.

While you were exploring elsewhere,
your creatures earned you 230 coins!

[Continue]    [Switch slot]
```

The "while you were gone" idle accrual (capped at 24 hours per §6 anti-grind) is shown here as a celebratory line, not buried in the game.

### Feature 3: Save export / import (OneDrive backup pattern)

Two buttons in Settings:

- **Backup my game** — generates a `.json` file containing the active slot's full state plus all voice clips encoded as base64. Triggers a browser download with a default filename like `mythic-earth-explorer-2026-05-01.json`.
- **Restore from backup** — opens a file picker, reads the `.json`, validates the schema version (and runs migrations if needed), then overwrites the active slot.

**Why this matters:** Brian's family is on OneDrive (his project workspace lives there). The expected pattern: drop the backup file into OneDrive, where it auto-syncs across devices and survives Chromebook resets, browser-cache clears, or a new device. It's also shareable — siblings can hand off a save, or a kid can take their save to a friend's house.

The export includes everything: state + voice clips. The kid never loses his recordings.

### Feature 4: Multiple save slots

**3 save slots** at launch. A "Slot picker" screen sits between the title and the welcome-back card.

- **First-ever launch:** Slot 1 is auto-created and named "Explorer." Game proceeds straight to the welcome flow.
- **Returning player with one slot:** Slot picker is a single-tap "Continue Explorer" screen.
- **Multiple slots:** Slot picker shows each slot with name, creatures discovered, coins, and last-played date. Tap a slot to load. Buttons: New, Rename, Delete (with confirm), Export, Import.

**Use cases:** siblings sharing a Chromebook ("Mine" / "Brother's"), a "fresh playthrough" without losing the main game, or a parent trying things out without overwriting the kid's save.

### Reset

A "Reset this slot" button in Settings, with double confirmation. Clears the active slot's localStorage + IndexedDB. Other slots are untouched. (To reset everything, the player would reset each slot individually — intentional friction so a misclick doesn't nuke a sibling's progress.)

### Versioning

The `version` field on each slot lets us migrate save data later if the schema changes. **Rule:** when the schema changes, bump the version + ship a migration function in `SaveSystem.js`. Never silently lose data. The `mythic_earth_save_v1_slot<N>` key pattern means we can transition to `_v2_` keys and run migrations on first load after an update.

### No cloud save service at launch

Each device's localStorage is the canonical store. The export/import file is the bridge between devices, and OneDrive handles the syncing for free. We may revisit a true cloud save service post-launch if multi-device convenience becomes a real ask — but with backup files, it's almost certainly unnecessary.

---

## 18. Accessibility

- **Keyboard-only play:** every action reachable via keyboard. Recording can use `R` to start.
- **Visual feedback on audio events:** the discovery sting flashes a small visual on the screen so a hearing-impaired player sees the moment.
- **High-contrast mode toggle** in settings: doubles outline thickness on sprites and increases UI font size by 25%.
- **No flashing or strobing effects.** Particle bursts are gentle.
- **Microphone alternatives** *(post-launch)*: a "type a word, we'll synthesize a voice" fallback for kids who can't or don't want to record.
- **Reduced motion** option that disables camera zoom and particle bursts on discovery (still plays the sting).

---

## 19. Roadmap

> **A note on pacing.** This is a side project. There is no deadline. Phases below are ordered by dependency, not duration — work through them session-by-session as time allows. Each phase has clear "done" criteria so it's obvious when to move on.

**Phase 0 — Concept & flair.** ✓ DONE. Theme, mechanics, regions, art rules locked.

**Phase 1 — Design docs.** ✓ DONE. README, GAME_DESIGN, CLAUDE.md.

**Phase 2 — Repo & deploy plumbing.**
- Create GitHub repo `mythic-earth`.
- Scaffold Phaser 3 + Vite project.
- Hello-world scene with the kid avatar and a single tile background.
- Wire up GitHub Pages deploy via `gh-pages` package.
- Ship to live URL.
- **Done when:** the son can open the URL on his Chromebook and see his avatar. Big morale win — something real, on the web, that he had a hand in.

**Phase 3 — MVP loop.**
- Norse Fjord region only.
- 1-2 wild creatures (e.g., Geri the Wolf) at fixed locations.
- Discovery + recording flow end-to-end.
- Idle coin generation.
- Save / load to localStorage + IndexedDB.
- All 4 save features wired (auto-save toast, welcome-back, export/import, slots).
- Hub city skeleton with one open portal (Norse) — others greyed.
- **Style audit milestone:** verify Norse looks coherent on the kid's Chromebook.
- **Done when:** the player can walk into Norse, discover a creature, record its voice, leave, return tomorrow, and find everything intact.

**Phase 4 — Content expansion.**
- Add the remaining 6 regions, one at a time (suggested order: Aztec → Celtic → Yokai → Outback → Andean → Sahara — escalating coin costs match this).
- Wire all 35 wild creatures with their reskin tints.
- All 7 hidden houses + ranches.
- Quest system + 42 quests.
- Catalog UI + 7 Legendaries (purchase flow).
- Trinket inventory + collection points + 49 trinkets.
- Hub decoration grid.
- Field Journal with re-record.
- **Done when:** all content is reachable and all systems are playable. May still look rough — that's Phase 5.

**Phase 5 — Polish.**
- Unified shader pass across regions for visual cohesion.
- Ambient music per region.
- UI sound effects (clicks, coin chime, discovery sting).
- Transition beats (fade-out, region card, fade-in).
- Region completion celebrations (golden border + small fireworks when all creatures discovered).
- Master Mythologist endgame screen when everything is done.
- Settings panel (audio, save management, accessibility).
- Bug bash with the son as QA — he'll find things you won't.
- **Done when:** the game *feels* like one cohesive experience, not a stitched-together demo.

**Phase 6 — Share & post-launch.**
- Send URL to family.
- Optional PWA manifest so it pins to Chromebook home screen like an app.
- Add post-launch backlog items as the son requests them (concerts, costumes, festivals, more regions, more quests, more trinkets, custom Piskel sprites for kangaroo / llama / condor).
- **Done when:** the project shifts from "build" mode to "live" mode — adding things over time as ideas hit.

---

## 20. Out of scope / post-launch backlog

These are deferred until after launch — they go on the "what's next" list:

- **Concerts.** Compose voice sequences using discovered creatures.
- **Costumes.** Make all creatures use one creature's voice.
- **Festivals.** Weekly regional bonus events.
- **Weather + day/night** per region.
- **Pitch / filter pass** on voice recordings for animal-y effects.
- **More regions:** Slavic Forest, Polynesian Islands, Indian Subcontinent, North American Indigenous folklore, Subsaharan African folklore.
- **More creatures per region.**
- **Cosmetic kid avatar customization** (hats, capes, recolors).
- **Cloud save** (would require a backend).
- **Multiplayer / sharing voice clips with friends.**
- **House interior decoration** (a per-region inside scene).
- **Mini-map** in regions.
- **Achievement system** ("Recorded 10 creatures with a song-voice," etc.).

---

## 21. Open questions

**Things to decide as we build, not blockers:**

1. **Portal unlock order:** strictly sequential (Norse → Aztec → Celtic → ...) or fully open after Norse (player can save up for any of the 6)?
2. **House interior:** ship at launch or defer to post-launch?
3. **"Mute creature voices" default:** on or off? (Recommend: off, but with prominent toggle so parents can disable if needed.)
4. **First-time mic permission flow:** soft pre-prompt before the browser's prompt, or just let the browser prompt fire on first discovery?
5. **Quest waypoint markers:** subtle floor arrow, big floating "!", or only on the Quest Panel map view?
6. **Coin generation visualization:** silent (number ticks up), gentle floating "+1" particles every increment, or both?
7. **Re-recording penalty:** none (recommend) or small coin cost to discourage spam?

**Any of these can shift during the build without rewriting major systems.** They are tuning-stage calls.

---

## 22. Asset reskin notes & custom-art shortlist

The free-asset path means many creatures will share a base sprite from CraftPix Hunt Animals or OpenGameArt Animal Pack with a Phaser tint applied at runtime. Here's how that works in practice and what's truly needed in custom art.

### Phaser tint pattern (applies to most reskins)

Phaser supports per-sprite color tinting via `sprite.setTint(color)`. This multiplies the sprite's existing pixels by the tint color, meaning a single source sprite can become dozens of variants without editing the image itself. Example:

```javascript
// One wolf source sprite → multiple region variants
const wolfNorse = this.add.sprite(x, y, 'wolf').setTint(0xCFE4F2); // pale icy blue
const wolfCeltic = this.add.sprite(x, y, 'wolf').setTint(0x6B8E5A); // mossy green (Cù-sìth Legendary)
const wolfOutback = this.add.sprite(x, y, 'wolf').setTint(0xD2A878); // tan (dingo)
```

**Rule of thumb:** if a creature only differs from a base sprite in color or saturation, use `setTint()`. If it needs a different shape (e.g., wings on an otherwise wingless body), it needs custom art.

### Reskin shortlist (no custom art needed — all tint-only)

The following creatures will use a shared source sprite with a tint applied. Claude Code should set up a `creatures.js` data file with `{ baseSprite, tintColor, scale }` for each.

- All wolves and dogs (Geri, Glen Fox, Andean Fox, Outback Dingo, Cù-sìth) → CraftPix Hunt Animals wolf/fox sprites + tint.
- All hares (Snow Hare, Forest Hare, Sand Hare, Mountain Hare) → CraftPix Hunt Animals hare + tint.
- All deer (Forest Stag, Mountain Stag, Sacred Shika) → CraftPix Hunt Animals deer + tint.
- All boars (Gullinbursti, Forest Boar) → CraftPix Hunt Animals boar + tint.
- All birds-of-prey and ravens (Huginn, Wise Owl, Mountain Owl, Mountain Eagle) → OpenGameArt Animal Pack bird sprites + tint.
- All snakes (Frilled Snake, Sand Viper) → CraftPix Hunt Animals snake + tint.
- All Legendaries (Jörmungandr, Quetzalcoatl, Cù-sìth, Ryū, Rainbow Serpent, Amaru, Bennu) → Magicae Dragons (4 base types: Air, Earth, Fire, Water) + tint and slight scale-up to mark them as "epic."

### Cohesion safeguards for reskins

- Reskinned sprites must still respect the §14 cohesion rules: 1px outline, ≤80% saturation.
- If the tint pushes a sprite over the saturation cap, dial it back and accept a softer color rather than violating the rule.
- The unified shader pass (§14) runs *after* tints are applied, so the per-region atmosphere will harmonize all reskins automatically.

### Custom-art shortlist (Piskel side-project for the son)

These are creatures and trinkets that genuinely cannot be sourced from free packs and would need to be drawn from scratch. **None of these are launch-blocking** — the game ships without them and they can be added post-launch at any time. Suggested as a fun father/son afternoon in **[Piskel](https://www.piskelapp.com/)** (free, browser-based, runs on Chromebook).

**Creatures (3, all "iconic but unsourceable" 6th-creature-of-region additions):**

| Creature | Region | Why custom | Approx. complexity |
|---|---|---|---|
| Red Kangaroo | Outback | No marsupial sprites in free packs | Medium — leaping pose, pouch, distinctive silhouette |
| Llama | Andean | No camelid sprites in free packs (no llama, alpaca, or vicuña) | Medium — long neck and woolly body are distinctive |
| Andean Condor | Andean | Generic large-bird sprites lack the condor's massive wingspan and bald head | Medium — wingspan-spread pose works best |

**Trinkets (3-6, region-iconic objects without generic icon analogues):**

| Trinket | Region | Notes |
|---|---|---|
| Boomerang | Outback | Curved blade icons exist but are too sword-like; a true boomerang is a 5-min draw |
| Didgeridoo | Outback | Wooden flute icons exist as a placeholder; a tubular instrument with painted patterns reads better |
| Quipu (knotted strings) | Andean | A specific Inca record-keeping object — the rope-and-knots silhouette is unique |
| Mate Gourd | Andean | Drinking vessel with a metal bombilla straw — distinctive |
| Kohl Jar | Sahara | Generic jar icon works as placeholder; small Egyptian-style jar with stick is more authentic |
| Marigold Wreath | Aztec | Bright orange flower wreath; a generic flower icon mostly works as placeholder |

**Piskel project plan if/when you want to do it:**

1. Open piskelapp.com on the Chromebook.
2. New sprite, 32x32 grid.
3. Pick a creature or trinket from the list above.
4. Show the son a reference image (a real-world photo or famous illustration).
5. Draw together — the son makes the calls, parent helps with shading.
6. Export as PNG, drop into `assets/custom/` in the repo.
7. Tell Claude Code: "Add the kangaroo from `assets/custom/kangaroo.png` to the Outback region as a 6th creature." Claude Code wires it into the data and you've added content to the live game.

This turns "asset gap" into "kid-engagement gold" — the son drew the kangaroo in his own game.

### Asset risk summary

- **~70% of creature sprites:** direct use of source pack, no edit needed.
- **~25% of creature sprites:** Phaser tint at runtime (`setTint()`) — zero art-editing work.
- **~5% (3 creatures):** would need custom art if we want them at launch. Currently all marked as optional / post-launch additions.

- **~85% of trinket icons:** direct from 496 CC0 RPG Icons or Caz Pixel Fantasy RPG Icons.
- **~10% of trinket icons:** generic icon used with regionally-themed flavor text (player brain fills the gap).
- **~5% (3-6 trinkets):** optional custom art, deferred to Piskel sessions.

Bottom line: **the game can ship complete with $0 art budget and no custom drawing required.** Custom art is bonus content that can be added incrementally as the son enjoys drawing.

---

*End of GAME_DESIGN.md*
