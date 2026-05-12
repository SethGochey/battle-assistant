> The code and below readme were AI generated.

# ☩ Battle Assistant — 40k Combat Calculator

A single-file, offline-capable web app for Warhammer 40,000 players. Build faction rosters, compose armies, track a live battle, and get probabilistic combat recommendations — all at the table, no internet required after the first load.

> **No install. No account. No server.** Open the HTML file in any modern browser and play.

---

## Features

### ⚔ Factions — Unit Template Library

Define reusable unit templates organized by faction. Everything you build here flows into your armies and battle calculations.

- Create any number of factions (Space Marines, Orks, Necrons, etc.)
- Each faction holds **units**, each unit holds one or more **datasheets**
- Datasheets capture the full stat line: Model Count, Wounds, Toughness, Armor Save, Invuln Save, Feel No Pain
- **Shooting profiles** per datasheet: Name, Attacks, Range, BS, Strength, AP, Damage
- **Melee profiles** per datasheet: Name, Attacks, WS, Strength, AP, Damage
- Attacks and Damage fields accept dice expressions: `D6`, `2D6`, `D3`, `2D6+1`, or plain integers
- **Faction-wide profile library** — save a shooting or melee profile once at the faction level and stamp independent copies into any datasheet with one tap (see below)

### 🛡 Armies — Compose Your Forces

Build named armies from faction unit templates.

- Select a faction, then add units to your roster
- The same unit type can be added multiple times (each gets a unique slot)
- Army cards show total unit count and a model summary

### ☩ Battle — Live Game Tracker + Analysis Engine

Run a battle between two armies with full model tracking and combat recommendations.

**Setup**

- Choose Army A and Army B (must be different armies)
- Optionally name the battle

**Active Battle**

- **Model & wound trackers** — large +/− buttons per datasheet for model count and partial wounds on the last model
- **Activated toggle** — dims the unit card to visually remove it from your mental field; auto-resets when the Round increments
- **Resource bar** — built-in Command Points and Round trackers; add any custom resource (Fate Points, Cabal Points, etc.)
- **Remove from Battle** — mark a unit as destroyed without deleting it from your army
- **Reset Activations** — one tap to clear all activated states at the start of a new round

**Recommendation Engines**

| Button | What it does |
|---|---|
| **⚔ Attack** | For the selected unit, ranks all enemy units by expected model kills. Shows per-target expected wounds, expected kills, and efficiency. |
| **☩ Destroy…** | For the selected unit, shows how many wounds it expects to deal to each enemy unit vs. that unit's total HP — highlights which targets it can likely destroy outright. |
| **⬡ Destroy Anything** | Scans all enemy units and finds the cheapest friendly activation sequence to destroy each one. Sorted by fewest activations needed. |
| **⬡ Simulate Battle** | Runs a deterministic 5-round simulation: each side picks its most efficient target each round, applies expected wounds, removes destroyed units, and produces a round-by-round narrative report with a final outcome. |

### ⬡ Data — Import / Export / Reference

- **Export** — downloads all factions and armies as a JSON file
- **Import & Merge** — loads a JSON file and adds any factions/armies not already present (deduplicates by ID)
- **Import & Replace** — replaces all local data with the imported file (with confirmation)
- **Clear All Data** — wipes localStorage (confirmation required)
- **Rules Reference** — wound chart and save modifier reminder for quick table lookup

---

## Data Entry — Faster Ways to Build Rosters

Entering a full faction manually is the most time-consuming part of getting started. Two features exist to speed this up significantly.

### ⊞ Paste from Spreadsheet

If you maintain your unit stats in Excel or Google Sheets, you can copy rows directly and paste them into the unit editor — no file upload required.

**How it works:**

1. In the unit editor, click **⊞ Paste from Spreadsheet** (next to + Add Datasheet)
2. In your spreadsheet, select the rows you want to import and copy them (Ctrl+C / Cmd+C)
3. Click inside the paste area and paste (Ctrl+V / Cmd+V)
4. The app parses your data and shows a preview immediately — check it before committing
5. Click **Import** to append the datasheets to the current unit

Parsed datasheets are appended to any datasheets you've already entered manually, so you can mix both approaches freely.

**Spreadsheet format:**

The first column of every row sets the row type. Subsequent columns follow a fixed order depending on the type.

| Row type | Columns |
|---|---|
| `DATASHEET` | Type · Name · Models · Wounds · T · Save · Invuln · FNP |
| `SHOOT` | Type · Name · Range · Attacks · BS · S · AP · Damage |
| `MELEE` | Type · Name · Attacks · WS · S · AP · Damage |

**Rules:**
- Profile rows (SHOOT, MELEE) belong to the closest preceding DATASHEET row above them
- Attacks and Damage accept dice notation (`D6`, `2D6+1`, `D3`) or plain numbers
- Save columns accept `3`, `3+`, `-`, or `7` — anything blank or `-` is treated as None (7)
- Header rows from your spreadsheet will be reported as errors and skipped; all valid rows still import

**Example (as it would appear when copied from a spreadsheet):**

```
DATASHEET	Intercessor Sergeant	1	3	4	3	-	-
SHOOT	Bolt Rifle	24	2	3	4	-1	1
MELEE	Close Combat Weapon	3	3	4	0	1
DATASHEET	Intercessor	4	2	4	3	-	-
SHOOT	Bolt Rifle	24	2	3	4	-1	1
MELEE	Close Combat Weapon	3	3	4	0	1
```

The preview panel shows every datasheet and its profiles before import, along with any warnings (unnamed profiles, out-of-range values) and hard errors (wrong column count, unknown row type). Warnings don't block import; errors are skipped row by row so the rest of your data still comes through.

### ⊙ Faction-Wide Profile Library

Many units in the same faction share identical weapon profiles — Close Combat Weapon, Bolt Pistol, Krak Grenade, and so on. Rather than re-entering these every time, you can save profiles to a faction-level library and stamp copies into datasheets on demand.

**How it works:**

- Inside the **faction editor**, scroll down to the **Profile Library** section
- Add shooting or melee profiles to the library directly, or use the **⊙** button on any existing profile row in the datasheet editor to push a copy up to the library
- Inside the datasheet editor, click **⊕ From Library** next to any profile table to open a picker showing all saved profiles of that type
- Selecting a profile stamps an independent copy into the datasheet — it is fully disconnected from the library original, so editing one doesn't affect the other

The library shows how many profiles it contains on the faction card in the factions list.

---

## Combat Math

All recommendations use **expected values** — no randomness. The engine calculates the statistically optimal play across all surviving models.

### Wound roll

| Condition | Roll needed |
|---|---|
| S ≥ T×2 | 2+ |
| S > T | 3+ |
| S = T | 4+ |
| S < T | 5+ |
| S×2 ≤ T | 6+ |

### Save

Modified save = Armor Save − AP. Invuln save ignores AP. The better (lower) of the two applies. A roll of 1 always fails.

### Expected wounds formula

```
attacks × P(hit) × P(wound) × P(fail save) × P(fail FNP) × min(damage, target wounds)
```

Shooting uses all surviving models firing. Melee uses each model's single best profile.

---

## Known Limitations (v1)

These are deliberate scope decisions, not bugs:

- **Mortal wounds** are not a separate profile type — they bypass saves entirely and add complexity. Planned for v2.
- **Re-rolls** are not modeled — too army-specific for v1. Recommendations will be slightly conservative for armies with re-roll auras.
- **Simulation is deterministic** — it uses expected values, not Monte Carlo. It will not capture high-variance outcomes (multi-damage weapons against multi-wound models, for example).
- **Movement and charge distance** are not modeled — if a unit has melee profiles it is assumed eligible to fight. Positioning is the player's responsibility.

---

## Data Persistence

All faction and army data is stored in your browser's **localStorage** under the keys `w40k_factions` and `w40k_armies`. Battle state is ephemeral (in-memory only — it resets when you close or refresh the tab). Use the **Export** feature to back up your data.

---

## Getting Started — Step by Step

### 1. Build your factions

1. Go to **⚔ Factions** → click **+ Add Faction**
2. Enter a faction name and click **Save Faction**
3. Click **+ Add Unit**, name the unit (e.g. "Intercessor Squad"), click **Save**
4. The unit editor opens — click **+ Add Datasheet** for each model type in the unit, or use **⊞ Paste from Spreadsheet** to import multiple datasheets at once from a spreadsheet
5. Fill in the stat line. Add shooting and/or melee profiles using the inline tables, or stamp them from the faction profile library with **⊕ From Library**
6. Attacks/Damage cells accept dice notation: type `D6`, `2D6+1`, `D3`, or a plain number

### 2. Build your armies

1. Go to **🛡 Armies** → click **+ Add Army**
2. Name the army and select its faction
3. Click **+ Add** next to each unit you want to include (add the same unit multiple times for duplicates)
4. Click **Save Army**

### 3. Run a battle

1. Go to **☩ Battle**
2. Select Army A and Army B, optionally name the battle, click **Begin Battle**
3. Track models and wounds using the **−/+** buttons on each unit card
4. Mark units as **Activated** after they act; click **Reset Activations** each new round
5. Use the resource bar to track Command Points, round number, and any custom resources
6. Hit **⚔ Attack** on a unit to see its best targets ranked by expected kills
7. Hit **⬡ Destroy Anything** to find the most killable enemy and the activation sequence to eliminate it
8. Hit **⬡ Simulate Battle** for a full 5-round projected outcome

### 4. Back up your data

Go to **⬡ Data** → **Export JSON** to download a backup. Import it on another device or browser with **Import & Replace**.

---

## Browser Support

| Browser | Support |
|---|---|
| Chrome / Edge 90+ | ✅ Full |
| Firefox 88+ | ✅ Full |
| Safari 14+ | ✅ Full |
| Mobile Chrome / Safari | ✅ Full (touch-optimised) |

---

## Contributing

Issues and pull requests welcome. The entire app lives in a single HTML file — no build step, no dependencies, no framework. Just open it and edit.

---

*Not affiliated with Games Workshop. Warhammer 40,000 is a trademark of Games Workshop Ltd. This tool is a fan-made utility for personal use.*
