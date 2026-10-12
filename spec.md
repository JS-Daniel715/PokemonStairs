# Pokemon Stairs (포캣몬 계단) — Feature Spec

**English** | [한국어](spec_KR.md)

> This spec describes the game as currently implemented. The development log and decision record are at the end.
> For code structure see [`ARCHITECTURE.md`](ARCHITECTURE.md); for working rules see `CLAUDE.md`.
> Quoted on-screen text is the English-mode text shown in the game.

## 1. Overview
- **Name**: 포캣몬 계단 (English: Pokemon Stairs)
- **Version / developer**: v1.0, JS Daniel Han. The bottom of the menu card shows "Pokemon Stairs v1.0 developed by JS Daniel Han (2026.10.04)" in both languages.
- **Genre**: casual game in the style of *Infinite Stairs* (two buttons: Climb / Turn)
- **Platform**: mobile web first, with desktop keyboard support. A single `index.html` (no build tool, no external libraries)
- **Launch**: the `포캣몬 계단` shortcut on the desktop (a Finder alias), or open `game/index.html` in a browser
- **Theme**: Pokémon. Personal and learning use only; not distributed publicly.
- **Core loop**: start → climb → game over → (pick a character, hatch eggs, check the ranking) → play again

## 2. Core play

### 2.1 Controls
| Action | Description | Keyboard | Mobile |
|---|---|---|---|
| Climb | Step one stair diagonally up in the direction you face | `→` / `D` / `Space` | Bottom-right button |
| Turn | Flip direction, then step up one stair | `←` / `A` | Bottom-left button |

- You start facing right.
- If there is no stair where you step, you **fall and the game ends**.
- While the menu is open: `Space` / `Enter` starts, `←` / `→` switch between owned characters, and `Esc` closes an open window.

### 2.2 Stairs
- Each stair is shifted ±1 column from the previous one.
- The first 3 steps go straight to the right. After that the path keeps its direction 60% of the time and turns 40% of the time.
- 20 stairs are generated ahead of the player. Stair colors cycle through 6 colors.

### 2.3 Timer gauge
- Ranges from 0 to 100. At 0 the game ends with **time up**.
- It does not drain before the first input.
- Drain per second: `min(10 + score × 0.12, 40)`
- Each successful step refills +7 (up to 100).
- Below 20% it blinks red.

### 2.4 Camera and effects
- There is no auto-scroll. The camera follows the player smoothly (the player stays at 68% of the screen height).
- Every step plays particles and a sound effect.
- **Milestones**: "N stairs!" appears at 10, 25, 50, and every multiple of 100.
- If you collect something on a stair, that replaces the milestone and everything is announced together, e.g. "Got Charmander! · Got an egg!".
- Game-over effects: a fall spins the character down; time up shows the front view with closed eyes, shaking. The menu appears 0.8 s later.
- Sound effects are short WebAudio tones and can be muted with the 🔊 button on the HUD (shared by all players).

### 2.5 Background
- A bright Pokémon-world scene: blue sky gradient, sun, drifting clouds, distant mountains → far hills → near hills.
- Each layer follows the camera at its own rate, so the hills sink away as you climb, leaving sky and clouds. Clouds repeat endlessly.
- Text on the game view (milestones, NEW!, EGG!) is yellow with a blue outline.
- Other background maps can be bought in the shop (6.3).

### 2.6 Coins
- Each stair has a **50%** chance of holding one gold coin (at most one per stair).

| Coin | Look | Chance |
|---|---|---|
| 1 coin | Small gold Poké Ball | 30% |
| 5 coins | Small gold Pikachu face | 15% |
| 10 coins | Medium gold Poké Ball | 5% |

- Stairs where a Pokémon waits (every 50 from 50 to 350) and egg stairs (350, 700) get no coin, so the drawings don't overlap.
- Stepping on the stair picks the coin up with a floating "+5"-style number and a sound.
- Picked-up coins are added to **the player's total coins** right away and saved. The game-over screen shows "+N coins" for the run.

### 2.7 Score and records
- The score is the number of stairs climbed in the run.
- Each player's best score and **the character used for it** are saved. A new record shows "NEW BEST".

## 3. Screens

### 3.1 Layout and theme
- Bright theme: sky-blue to light-green page background, white pill-shaped HUD, white menu cards.
- From top to bottom: HUD → game view (canvas) → two control buttons. Margins are kept small so the game view is as large as possible (control buttons 62 px tall; HUD and controls at most 460 px wide).
- The canvas scales its 400×700 logical resolution to the screen size and pixel ratio.

### 3.2 HUD
- Left: current player name and overall rank ("Rank #2 of 3", updated live from the current score)
- Center: score of this run
- Right: the player's best score and the 🔊 mute button
- Below: the timer gauge

### 3.3 Menu (title / game over)
- A white card over the whole screen. The game view behind it is blurred, and the card uses the full screen height regardless of the canvas size.
- Card contents, top to bottom:
  1. Header
     - Title: game name (yellow with a blue outline), description, control hints
     - Game over: "Whoops, you fell!" / "Time's up!", score · best · rank, NEW BEST, rank change ("Rank up! #2 → #1"), new friends from this run, eggs picked up, coins picked up
  2. Player bar (7.2) with the **coin box** (total coins so far) on its right
  3. Character select (4.5)
  4. Bottom buttons: **🏆 Ranking · 🎒 Bag · Start! (Play again!)**
  5. Version / developer line: "Pokemon Stairs v1.0 developed by JS Daniel Han (2026.10.04)"
- Items 1–3 scroll together, while **the bottom buttons and the developer line stay pinned** to the bottom of the card on any screen size.
- On short screens (640 px or less) the title description is hidden and the title and spacing shrink.

### 3.4 Windows (change player / ranking / bag / shop)
- White cards shown above the menu. They close with the Close button, by tapping the dimmed backdrop, or with `Esc`.
- While a window is open, keyboard input does not reach the game (so names can be typed).

### 3.5 Language (한국어 / English)
- Switch with the **English / 한국어** button in the top-right of the menu card. The default is Korean.
- All on-screen text, Pokémon names (Pichu, Charmander …), and text on the game view ("Got Charmander!", "EGG!", "50 stairs!") switch together.
- Switching while a menu or window is open redraws it in place (including game-over results).
- The setting is saved and shared by all players. Player names are shown exactly as entered.

### 3.6 Mobile
- Buttons respond on `pointerdown` for fast input.
- Double-tap zoom, page scrolling, and the long-press menu are blocked, and safe areas (notches) are respected.
- Only areas that need it, such as the menu and lists, allow vertical scrolling.

## 4. Characters

### 4.1 Full list and how to get them (80)
| Base form (how to get) | 1st evolution | 2nd evolution |
|---|---|---|
| **Pichu** (owned from the start) | Pikachu | Raichu |
| Charmander (stair 50) | Charmeleon | Charizard |
| Squirtle (stair 100) | Wartortle | Blastoise |
| Bulbasaur (stair 150) | Ivysaur | Venusaur |
| Cyndaquil (stair 200) | Quilava | Typhlosion |
| Chikorita (stair 250) | Bayleef | Meganium |
| Totodile (stair 300) | Croconaw | Feraligatr |
| Mew (stair 350) | — | — |
| Cleffa (green egg) | Clefairy | Clefable |
| Igglybuff (green egg) | Jigglypuff | Wigglytuff |
| Dunsparce (green egg) | Dudunsparce | — |
| Wimpod (green egg) | Golisopod ※ | — |
| Smoochum (green egg) | Jynx | — |
| Tadbulb (green egg) | Bellibolt | — |
| Larvesta (green egg) | Volcarona ※ | — |
| Fidough (yellow egg) | Dachsbun | — |
| Munchlax (yellow egg) | Snorlax | — |
| Sizzlipede (yellow egg) | Centiskorch | — |
| Indeedee (yellow egg) | — (does not evolve) | — |
| Snom (yellow egg) | Frosmoth | — |
| Ekans (yellow egg) | Arbok | — |
| Articuno · Zapdos · Moltres (shop, 30,000 coins each) | — | — |
| Mewtwo (shop, 100,000 coins) | — | — |
| Geodude (Desert map, 3,000 stairs) | Graveler | Golem |
| Swinub (Desert map, 5,000 stairs) | Piloswine | Mamoswine |
| Trapinch (Desert map, 10,000 stairs) | Vibrava | Flygon |
| Bellsprout (Jungle map, 3,000 stairs) | Weepinbell | Victreebel |
| Hoppip (Jungle map, 5,000 stairs) | Skiploom | Jumpluff |
| Treecko (Jungle map, 10,000 stairs) | Grovyle | Sceptile |
| Poliwag (Sea map, 3,000 stairs) | Poliwhirl | Poliwrath |
| Horsea (Sea map, 5,000 stairs) | Seadra | Kingdra |
| Spheal (Sea map, 10,000 stairs) | Sealeo | Walrein |

※ Wimpod and Larvesta evolve only after **1,000 stairs** with the base form.

### 4.2 Stair encounters
- Base-form Pokémon you don't have yet wait every 50 stairs (`UNLOCK_EVERY = 50`), hopping on their stair with a "NEW!" label.
- Stepping on that stair collects and saves them immediately. They are kept even if you die in that run.
- Pokémon you already own no longer appear on the stairs.

### 4.3 Stairs per character
- Every successful step adds +1 to the count of **the character you are playing**.
- The count keeps growing across runs; steps made before a death still count.

### 4.4 Evolution (= unlocking a new character)
- Reaching **500 stairs** with a base form unlocks its 1st evolution **as a new character**. The base form stays available.
  - Exceptions: Wimpod → Golisopod and Larvesta → Volcarona need **1,000 stairs**.
- Reaching **1,000 stairs with the 1st evolution** unlocks the 2nd evolution. The 1st evolution's count starts from 0 when it is unlocked.
- The moment a goal is reached during play, "Got X!" appears and the unlock is saved.

### 4.5 Character select
- **Detail card** (top): large picture, name, status badge, description
  - Owned: "Selected" badge, "N stairs · M more to Pikachu", and an evolution progress bar. If the next evolution is already owned: "X unlocked". Final stage: "Final form". No evolution: "Doesn't evolve".
  - Not owned: silhouette, "???", "Locked" badge, and how to get it:
    - Evolution: "Climb 500 stairs as Pichu to get it"
    - Stair Pokémon: "Waiting on stair 50"
    - Egg Pokémon: "Can hatch from a green-spotted egg (5%)", "Can hatch from a yellow-spotted egg (20%)"
    - Shop Pokémon: "Buy it in the shop for 30,000 coins"
    - Map Pokémon: "Climb 3,000 stairs on the Desert map to get it (now 2,995)"
- **Tabs** (between the detail card and the icon list): **Stairs · Eggs · Shop · Maps · All** filter the list. Each tab shows how many you own (e.g. "Stairs 5/22"). The list opens on **Stairs**.
  - Stairs: the Pichu line and the stair Pokémon lines (Charmander … Mew)
  - Eggs: lines hatched from green and yellow eggs (Cleffa … Larvesta, Fidough … Ekans)
  - Shop: Pokémon bought in the shop (Articuno, Zapdos, Moltres, Mewtwo)
  - Maps: map Pokémon lines (Geodude … Spheal)
- **Icon list** (bottom): the selected tab's evolution lines, grouped in 3s or 2s (Mew alone), in two columns.
  - Tapping an owned icon selects it (yellow border). Tapping a locked icon only shows its requirement in the detail card (blue dashed border).
  - Owned characters still waiting to evolve have a small progress bar under the icon.
- The selected character is saved per player.

### 4.6 Art and poses
- All 80 characters have a code-drawn front view and side view (later evolution stages are drawn larger).
- **Standing / waiting**: front view, blinking now and then.
- **Running**: side view. It lasts 0.22 s after each step, so pressing repeatedly looks like continuous running.

| Motion | Characters |
|---|---|
| Two-legged run | Raichu, the Charmander, Squirtle and Totodile lines, Typhlosion, Golisopod, Smoochum, Bellibolt (waddle), Munchlax, Snorlax (waddle), Indeedee, the Treecko line, Graveler, Poliwhirl, Poliwrath, Walrein (waddle) |
| Four-legged run | Pichu, Pikachu, the Bulbasaur and Chikorita lines, Cyndaquil, Quilava, Fidough, Dachsbun, the Swinub line, Trapinch |
| Hopping | The Cleffa and Igglybuff lines, Geodude (vaulting on its arms), the Bellsprout line, Poliwag, Sealeo |
| Rolling | Spheal, Golem |
| Crawling | Dunsparce, Dudunsparce, Wimpod, Larvesta, Sizzlipede, Centiskorch (reared up), Snom |
| Flying (floats even when standing) | Mew, Volcarona, Tadbulb, Frosmoth, Articuno, Zapdos, Moltres, the Hoppip line, the Horsea line, Vibrava, Flygon |
| Slithering | Ekans, Arbok |
| Gliding | Jynx, Mewtwo |

- Charizard flaps its wings while running, and Pokémon with flames, flowers, or spikes on their backs keep those features in the running pose.

## 5. Eggs and the item bag

### 5.1 Egg kinds and pick-up
| Egg | Stair | Hatch result |
|---|---|---|
| 🥚 **Green-spotted egg** | Stair **350** every game | 65% chance of nothing |
| 🥚 **Yellow-spotted egg** | Stair **700** every game | Always a Pokémon (100%) |

- Each egg waits once per run on its stair, wobbling with an "EGG!" label. Stepping on it puts it in the **item bag**, where it stays after the run (saved per player and per egg kind).
- If a Pokémon you don't own is waiting on the same stair (stair 350 = Mew), the egg sits to its left.
- Picking one up shows "Got an egg!" / "Got a yellow egg!", and the game-over screen shows "🥚 Found an egg! Hatch it in your bag".
- For testing, lower the `stair` values in `EGG_KINDS` (e.g. 30 and 50).

### 5.2 Item bag (🎒 Bag)
- Opened with the **🎒 Bag** button at the bottom of the menu. When you have eggs, the button shows a red count badge (all kinds combined).
- All eggs are shown in their kind's color. Tapping an egg hatches it (it wobbles for 1 s, then shows the result).

### 5.3 Hatch odds
**Green-spotted egg**
| Result | Chance |
|---|---|
| Nothing ("Oh no, nothing hatched…", cracked egg) | 65% |
| Cleffa, Igglybuff, Dunsparce, Wimpod, Smoochum, Tadbulb | 5% each (30% total) |
| Larvesta | 5% |

**Yellow-spotted egg**
| Result | Chance |
|---|---|
| Fidough, Munchlax, Sizzlipede | 20% each |
| Indeedee, Snom | 15% each |
| Ekans | 10% |

- A hatched Pokémon joins right away and can be selected ("X hatched! A new friend! Pick it in character select").
- If you already own it, the egg is used up and the game says "You already have this Pokémon".

## 6. Shop and maps

### 6.1 Opening the shop
- Tap the **coin box** in the menu (total coins, "🛒 Shop"). The shop shows your coins at the top.
- It has **Pokémon** and **Maps** tabs.
- **Buying**: the first tap turns the button into a red "Sure?"; a second tap within 3 s buys it. Without enough coins the button reads "Need coins" and is disabled.

### 6.2 Pokémon (no evolution)
| Pokémon | Price | Motion |
|---|---|---|
| Articuno | 30,000 coins | Flying (long tail, blue crest) |
| Zapdos | 30,000 coins | Flying (spiky black feathers) |
| Moltres | 30,000 coins | Flying (flame crest and tail) |
| Mewtwo | 100,000 coins | Gliding (long purple tail) |

- Bought Pokémon join right away ("🎉 X joined your team!"). They are grouped in the **Shop** tab of the character list; while locked, the hint reads "Buy it in the shop for 30,000 coins".

### 6.3 Maps (game background)
| Map | Price | Scenery |
|---|---|---|
| Sky map | Free (owned by default) | Blue sky, green hills |
| Jungle map | 10,000 coins | Teal sky, layered forest, round bushes and palm trees |
| Desert map | 10,000 coins | Orange sky, sand dunes, pyramids and cacti |
| Sea map | 10,000 coins | Blue sky, islands, rolling waves |

- A bought map is applied immediately. Owned maps can be switched any time with "Use"; the current one shows "In use".
- Maps are saved per player and apply both behind the menu and during play.
- Each owned map's shop row shows progress, e.g. "5,001 stairs · map Pokémon 2/3".

### 6.4 Map Pokémon
- Every map except the Sky map has Pokémon that can only be earned there.
- **Stairs climbed while the map is in use** are counted separately per player and per map (any character counts). When a goal is passed, "Got X!" appears during play.

| Map | 3,000 stairs | 5,000 stairs | 10,000 stairs |
|---|---|---|---|
| Desert | Geodude | Swinub | Trapinch |
| Jungle | Bellsprout | Hoppip | Treecko |
| Sea | Poliwag | Horsea | Spheal |

- Map Pokémon evolve like the others (500 stairs with them → 1st evolution, 1,000 stairs with the 1st evolution → 2nd). See the table in 4.1.

## 7. Players and ranking

### 7.1 Players
- Several players can be registered on one device (browser). There is no server or login, and records are not shared with other devices.
- Saved separately for each player: best score, the character used for it, owned characters, stairs per character, selected character, eggs in the bag (per kind), total coins, owned maps and the map in use, and stairs per map.
- **On first launch** (no saved data), a "Welcome! 👋" name prompt appears first. It cannot be closed and the game cannot start until a name is entered. The HUD reads "Enter your name".
- Only if a single-player save from before players existed is found is it moved to a player named **"한준서"**.
- The last player used becomes the current player on the next launch.

### 7.2 Player bar and changing players
- The menu has a player bar: name, "Best N (character) · Rank #N · Pokémon n/80", and a **Change player** button.
- To the right of the player bar is the **coin box**: a gold Poké Ball icon, total coins so far (e.g. "1,234"), and "🛒 Shop". Tapping it opens the shop.
- Change player window:
  - Name input (trimmed, 1–10 characters). A new name creates a fresh record with only Pichu; an existing name switches to that player.
  - List of registered players (sorted by name, with best scores, current player highlighted). Tapping one switches immediately.
  - An empty or too-long name shows a hint ("Please enter a name." / "Names can be up to 10 characters.").
- Switching players returns to the title screen and refreshes the HUD, player bar, character select, and bag for that player.

### 7.3 Ranking
- Opened with the **🏆 Ranking** button in the menu.
- Sorted by best score, high to low (ties by name). Equal scores share a rank.
- Each row: rank (🥇🥈🥉 for 1–3), a picture of **the character used for the best score**, name (the current player gets a "Me" badge and yellow highlight), "with Charmander · Pokémon n/80", and the best score.
- Records set before the best-score character was saved show Pichu.

## 8. Saved data (`localStorage`, prefix `pikachuStairs.`)
| Key | Contents |
|---|---|
| `users` | Per-player records `{ name: { best, bestChar, owned: [id], stats: { id: count }, selected, eggs: { green, yellow }, coins, maps: [id], map, mapStats: { map: count } } }` (JSON). An older numeric `eggs` value is read as green eggs |
| `currentUser` | Name of the current player |
| `muted` | Mute (0/1), shared by all players |
| `lang` | Display language `ko` / `en`, shared by all players |
| `collectionVersion` | Changing it resets every player's `owned` / `stats` / `selected` (best scores kept). Currently 3 |
| `dataFixes` | Number of one-time data fixes applied. 1 = remove egg Pokémon earned during egg testing, 2 = correct the Tadbulb/Bellibolt ids (`shroodle` → `tadbulb`, `tadbulb` → `bellibolt`), 3 = reset Indeedee, Munchlax, Fidough, and Tadbulb (and their evolutions) |
| (old) `best`, `owned`, `stats`, `selected` | Single-player keys from before players existed. Moved into the "한준서" record and deleted on first launch |

- Automatic fixes on load: a missing `bestChar` becomes `pichu`, and the old ids `rellor` / `rabsca` / `grafaiai` become `wimpod` / `golisopod` / `bellibolt`.

## 9. Out of scope
Obstacles, items other than eggs, online (cross-device) ranking, server accounts and login, deleting or renaming players, background music, difficulty settings, PWA / app store release

## 10. Development log
In the order the work was done.

| # | Change |
|---|---|
| 1 | **MVP spec and build**: rebuilt a left/right-choice auto-scroll prototype as Infinite-Stairs-style Climb/Turn with a timer gauge. Mobile support, best-score saving, sound effects |
| 2 | **Pikachu redrawn**: closer to the real Pikachu, with a front pose when standing and a side pose (four-legged run) when running |
| 3 | **Character collecting**: 7 Pokémon (Charmander, Squirtle, Bulbasaur, Cyndaquil, Chikorita, Totodile, Mew) wait on the stairs; character select screen. Test spacing of 10 stairs → release spacing of 50, with a collection reset |
| 4 | **Wrote CLAUDE.md / spec.md** |
| 5 | **Evolution**: stairs per character, evolutions unlocked as new characters at 500 / 1,000 stairs, starter changed to Pichu, evolution art added (22 total), collection reset (`collectionVersion` 3) |
| 6 | **Bright theme**: sky, cloud and hill background, larger game view, redesigned character select (detail card + icon list), fixed the Play-again button position |
| 7 | **Named the game "포캣몬 계단"**, desktop shortcut |
| 8 | **Players and ranking**: local multi-player profiles, existing record moved to "한준서", name and rank on the HUD, ranking screen |
| 9 | **Menu layout rebuilt**: fixed the hidden Start button by making the menu a full-screen card with pinned buttons |
| 10 | **Best-score character**: the character used for the best score is saved and shown in the ranking (older records show Pichu) |
| 11 | **Eggs and item bag**: egg on stair 350, bag, hatch odds, 7 egg Pokémon and 9 evolutions (38 total) |
| 12 | **Wimpod fix**: redrew the wrongly drawn Pokémon (Rellor) as the real 꼬시레 (Wimpod), 1st evolution changed to Golisopod. Wimpod and Larvesta evolve at 1,000 stairs |
| 13 | **Egg testing**: egg moved from stair 100 to every 30 stairs with higher odds for testing |
| 14 | **Back to release settings**: egg on stair 350 at 5% each; egg Pokémon earned during testing removed from all players (eggs in bags kept) |
| 15 | **Name prompt on first launch**: a fresh install asks for a name instead of creating "한준서" |
| 16 | **Korean/English toggle**: English / 한국어 button switching all text and Pokémon names |
| 17 | **English name and developer credit**: English name changed to "Pocketmon Stairs", "개발자 Daniel.JS.Han" shown in the menu |
| 18 | **English name changed again**: "Pokemon Stairs" |
| 19 | **Developer credit changed**: "developed by Daniel JS Han" (both languages) |
| 20 | **Version line**: "Pokemon Stairs v1.0 developed by JS Daniel Han (2026.10.04)" (both languages) |
| 21 | **빈나두's evolution changed**: Grafaiai → 찌리배리. The tadpole (Tadbulb) was mistakenly drawn as 찌리배리 at this point |
| 22 | **빈나두 / 찌리배리 corrected**: 빈나두 = electric tadpole (Tadbulb), 찌리배리 = green electric frog (Bellibolt), redrawn. The original mouse drawing (Shroodle) removed. Old records moved to the new ids once |
| 23 | **Tadbulb redrawn**: round cream face, yellow head with a wavy edge, tadpole tail hanging below the face. Bulb on the tail tip removed |
| 24 | **Character list tabs**: Stairs / Eggs / All tabs with owned counts |
| 25 | **Default tab changed**: the character list opens on the Stairs tab |
| 26 | **Yellow-spotted egg**: yellow egg on stair 700 (always hatches), Fidough, Munchlax, Sizzlipede, Indeedee, Snom, Ekans and 5 evolutions (Dachsbun, Snorlax, Centiskorch, Frosmoth, Arbok) added (49 total), eggs stored per kind |
| 27 | **Egg test settings**: green egg on stair 30, yellow egg on stair 50 (release: 350 / 700) |
| 28 | **Egg release settings restored**: green egg on stair 350, yellow egg on stair 700 |
| 29 | **Coins**: 50% chance per stair of 1 / 5 / 10 coins (30 / 15 / 5%), total coins saved per player, player bar shortened with a total-coins box on its right |
| 30 | **Collection reset**: Indeedee, Munchlax, Fidough, and Tadbulb earned during testing removed from all players once (Wimpod already evolved at 1,000 stairs, so no change there) |
| 31 | **Shop**: opened from the coin box; Pokémon (Articuno · Zapdos · Moltres 30,000 / Mewtwo 100,000) and maps (Jungle · Desert · Sea 10,000 each); applying maps; Shop tab in the character list |
| 32 | **Map Pokémon**: stairs counted per map on the Desert, Jungle, and Sea maps; 9 map Pokémon at 3,000 / 5,000 / 10,000 stairs; Maps tab; map progress in the shop (62 total) |
| 33 | **Map Pokémon evolutions**: 18 evolutions for the 9 map Pokémon (500 / 1,000 stairs); Maps tab grouped by evolution line (80 total) |
| 34 | **GitHub docs**: `README.md` (intro, setup, controls, progression, notice), `ARCHITECTURE.md` (structure, loop, data model, extending), screenshots in `docs/screenshots/`, `.gitignore` |
| 35 | **Docs in English**: `README.md` and `ARCHITECTURE.md` rewritten in English |
| 36 | **Korean docs kept**: `README_KR.md` and `ARCHITECTURE_KR.md` (Korean), linked with the English versions; README screenshots retaken in English mode |
| 37 | **Spec in English**: this `spec.md` written in English; the Korean version moved to `spec_KR.md` |

## 11. Decision record
- Controls follow the Infinite Stairs style (Climb / Turn); time pressure comes from a timer gauge.
- Unlocking an evolution does not remove the base form; it stays selectable.
- A 2nd evolution requires 1,000 stairs **with the 1st evolution** (separate from the base form's count).
- The collection was reset when evolution was introduced; best scores were kept.
- Players are local profiles on the device. The owner of the original record is "한준서".
- A fresh install does not create a default player; it asks for a name first.
- Egg Pokémon evolve too (1st at 500, 2nd at 1,000 stairs; Wimpod and Larvesta at 1,000).
- If an egg hatches a Pokémon you already own, only the egg is used up.
- 꼬시레 is the Gen 7 Wimpod, and its 1st evolution is Golisopod.
- After egg testing, the egg Pokémon and evolutions earned during the test were removed once (including their stair counts; a selected or best-score character fell back to Pichu). Eggs left in bags were kept.
- 빈나두 is the Gen 9 electric tadpole Tadbulb, which evolves into 찌리배리 (Bellibolt). At first 빈나두 was mistaken for Shroodle (a mouse) with Grafaiai as its evolution.
- Among the yellow-egg Pokémon only Indeedee does not evolve; the others evolve at 500 stairs.
- The shop request "파이어, 썬더, 파이어" was interpreted as the three legendary birds (Articuno, Zapdos, Moltres).
- Map Pokémon evolve too (500 / 1,000 stairs). Poliwhirl evolves into Poliwrath (not Politoed).
