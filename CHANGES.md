# Implementation Plan - Tainted Cain Recipe Calculator (TBOI Repentance+)

Create a web application to calculate and display *Bag of Crafting* (Sacabolsas) recipes for Tainted Cain in **The Binding of Isaac: Repentance+**, replicating the exact logic, UI structure, and modifiers specified in the source reference image.

---

## User Review Required

> [!IMPORTANT]
> - **WASM Accelerator / Generator Core:** Determining exact Repentance+ seed-based recipe outputs client-side requires compiled WebAssembly logic (e.g., from Isaac crafting calculator modules) or a fallback deterministic hash calculation matching the game's internal recipe algorithm.
> - **Pickup Weights & Quality Score:** Recipes must account for the sum of component values (e.g., Golden Hearts, Bone Hearts, Gigabombs, Mega Batteries having higher weight values) to output corresponding item quality tiers (Quality 0 to Quality 4).

---

## Proposed Changes

### 1. Project Architecture & Setup
- Single-page application structure (HTML5, Tailwind CSS via CDN or bundle, Vanilla JS / TypeScript or React).
- Dark theme inspired by The Binding of Isaac UI style (`#1e1e1e` background, slate/zinc containers, custom pixel/serif fonts).

---

### 2. Header & Seed Input Section
- **Title Banner:** "Sintetiza la Excavadora de Bolsas del Tesoro" / "Bag of Crafting Calculator".
- **Seed Input Box:**
  - Styled with blue banner header (`Semillas`).
  - Monospaced, uppercase, 8-character game seed input (e.g., `ABCD EFG1`).
  - Event listener for seed validation and real-time recalculation trigger.

---

### 3. Pickups Selection Grid (Component Inventory)
- Grid container rendering 29 pickup types in 2 main rows/sections:
  - **Hearts:** Red, Soul, Black, Eternal, Gold, Bone, Rotten.
  - **Coins:** Penny, Nickel, Dime, Lucky Penny, Golden Penny.
  - **Keys:** Key, Golden Key, Charged Key.
  - **Bombs:** Bomb, Golden Bomb, Giga Bomb.
  - **Batteries:** Micro Battery, Battery, Mega Battery, Golden Battery.
  - **Special / Consumables:** Cards, Pills, Runes, Dice Shards, Soul Stones, Golden Pills, Poop.
- Each pickup card contains:
  - Icon / Sprite representation.
  - `<select>` dropdown or counter input (values `0` to `29+`) to declare current inventory quantity.

---

### 4. Control Toolbar & Filtering Logic
- **Recipe Limit Selector:** Radio/Button group for `1. Receta`, `10. Receta`, `20 recetas`, `50 recetas`, `100 recetas` (Active state highlighted in green).
- **Execution Toggles:**
  - `Habilitar la actualización en tiempo real` / `Desactiva la actualización en tiempo real`.
  - `Activar la aceleración WASM` / `Desactivar la aceleración WASM`.
- **Game State Modifiers (Checkboxes/Toggles):**
  - `Modo avaricia` (Greed Mode recipe pools).
  - `Retos diarios (incluyendo Después del Capítulo 4)`.
  - `Transformado en un alma errante` (Tainted Forgotten / Lost state modifiers).
  - `(Corrompido) dueño de la tienda` (Tainted Keeper state).
  - Item pools & Trinket locks (`Objeto: Orbe Sagrado de la Cruz`, `(Oro/varios) Accesorios: ¡No!`).
- **Action Buttons:**
  - `Comienza la síntesis` (Triggers manual recipe calculation batch).
  - `Vacía los componentes` (Resets all pickup inputs to 0).

---

### 5. Crafting Engine & Calculation Logic
- **Slot Selection Algorithm:**
  - Select 8 pickups from the declared inventory.
  - Calculate total pickup value sum (Component Quality Weight).
  - Derive item ID based on Seed hash + Pickup Combination + Unlocked Pools.
- **WASM Mock/Integration Placeholder:**
  - Prepare interface `calculateRecipes(seed, availablePickups, modifiers, limit)`.

---

### 6. Results Display Section
- List of valid 8-pickup recipes found.
- Display each result card with:
  - Output Item Icon & Name.
  - Item Quality Rating (0 to 4 stars).
  - 8-slot ingredient badge list showing exact pickup icons used.

---

## Verification Plan

### Automated Tests
- Test ingredient combination generator (ensure no recipe contains more pickups than available in inventory).
- Test reset function (`Vacía los componentes`).

### Manual Verification
1. Input seed `ABCD EFG1`.
2. Add `8 Red Hearts` and `8 Soul Hearts`.
3. Click `Comienza la síntesis`.
4. Verify recipe cards render showing valid 8-component combinations and resulting item outputs.
5. Toggle `Modo avaricia` and verify output pool shifts accordingly.