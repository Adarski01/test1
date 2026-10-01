# The Ink Well: integration guide

**For the Claude session that has the master file open.**
Master: `the-unwritten-source.jsx` (the one under `the-unwritten 3`).

This guide puts the Ink Well mini-game into the game, step by step. The design is final and approved. The playable prototype is in `prototype/inkwell.html`: open it in a browser to see exactly how everything should look and feel. The art is in `assets/img/`.

---

## 0. Rules for this work (read first)

- **Another session may be editing the same master file.** Make small, anchored edits only. **Re-read the file just before each write**, and never write it back from an older copy.
- **Do not rebuild anything that exists.** In particular, the sewn cards come from the existing **Ligature**: use `ligPrize` and `mkLig` exactly as described in the master under "A SEWN CARD AS A PRIZE".
- **Saves must never break.** Every new field is additive and default-guarded (`|| {}`, `|| []`, `|| 0`). Nothing is renamed or removed.
- **Anchors below were taken from an older copy of the source.** If an anchor string is not found, search for the nearest equivalent (the constant name is always given). Never guess at a location; ask if unsure.
- **Work in the order of the steps.** After every step the game must still load and play normally. Test after each one.
- Use the locked lore names exactly: Lost Souls, Doubt, The Author, Echo of the Fallen, Scriptorium.

---

## 1. What is being added (summary)

- **A rare room on the map:** *The Ink Well*. At most one per volume. Also reachable as a Marginalia seal.
- **The Well screen:** a flooded hall with a small hand winch on a stone jetty. 3 casts per visit (4 with the tree). You lower an iron hook into the ink, wait for a bite, **Strike**, then **reel** (hold to keep the catch inside the light).
- **What comes up:** gold, a potion, a card, a rare card, Lost Souls, a lost page, Doubt, a *drowned card* (tree line IV), a *sewn card* (tree line V), or **a creature that bites back**.
- **Creatures:** 11 creatures that live **only** in the Well. A creature is a **normal fight to the death**, exactly like any other fight. Before it starts you may **Let it go** (the cast is spent, nothing else happens).
- **Pull levels:** each creature pulls *light*, *heavy* or *abyssal*. Heavier pulls are harder to reel and pay more. **Casting into the dark** makes every creature bigger and longer in the ink (+25% HP, drawn 15% larger), one pull level heavier, +1 Strength, and doubles the gold. **Every line written in the tree makes every reel calmer.**
- **The Rim:** a small plaque on the Well screen with 3 candles. Every creature you beat lights one (a black flame if it came from the dark). The candles stay lit from Well to Well until the tale ends. **Only losing a fight puts them out** (which, in this game, is death anyway). Three candles bring up **The Undertow**.
- **The Undertow:** a leviathan boss-tier fight. It holds shrouded bodies in its tentacles and hurls them at you. "Not yet" postpones it to the next Ink Well, once per tale.
- **The ninth soul tree:** *What The Ink Kept*.
- **The Catalogue of Drowned Words:** 21 entries, tracked separately from the Compendium.

---

## 2. Assets

Copy every file from `assets/img/` into the game's `assets/img/`, and add each key to `ART` (multi-file build: path; standalone build: base64 data URI, like every other entry).

| Key | Used for |
|---|---|
| `bg_inkWell` | The Well screen (hall with the winch). |
| `bg_inkWell_arena` | Fight background for Well creatures (same hall, no winch). |
| `bg_inkWell_deep` | The dark cast, The Barbed Unsaid, The Undertow. |
| `loc_inkWell_cut` | Map building for the new room. |
| `cut_wellRedacted` … `cut_wellUndertow` (13 files), plus `cut_wellWatermarkCrest`, `cut_wellBleed` (the Watermark's two halves) | Creature cutouts, read by `ART['cut_'+e.id]`. |
| `orn_seal_well` | Wax seal on a Well creature's intent pill. |
| `rl_authorsHook` | Relic icon. |
| `cardart_almostWord` | Card art for *The Word You Almost Had*. |
| `pt_healing_hd`, `pt_frostInk_hd`, `pt_dryInk_hd` | Optional: sharper potion icons (360 px) for the three potions. |

All creature cutouts are transparent WebP, 700 px on the long side (The Undertow 800 px).

---

## 3. Step 1: the room on the map

**3a. `NODE_META`** (anchor: `const NODE_META={`). Add an entry:

```js
inkWell:{icon:Anchor,color:'#5aa9e6',label:'The Ink Well'},
```

If `Anchor` is not imported from lucide, use `Droplets` or any icon already imported. The colour `#5aa9e6` was chosen to avoid Rest, the Cartographer's Table and the Exemplar. It is close to The Summoner's accent (`#5fb0e8`), so check it on screen.

**3b. `DOOR_COST`** (anchor: `const DOOR_COST={`). Add `inkWell:2,`.

**3c. `rollPageType`** (anchor: `return pick(['deep','shrine','blackMarket','challenge'])`). Add `'inkWell'` to that array. Then enforce **at most one Ink Well per volume**: where the page nodes are generated, if an `inkWell` was already placed on this page, re-roll it as `'shrine'`. (Use the same place that already prevents duplicates of other rare rooms, if there is one.)

**3d. `MARGINALIA`** (anchor: `const MARGINALIA={`). Add a seal so it can be bound in the Outline:

```js
inkWell:{name:'The Ink Well',short:'Ink Well',node:'inkWell',act:1,need:0,ink:1,art:'loc_inkWell_cut',
  desc:'Where the Author held things under until the candle went out. Some of the candles did not.',hint:''},
```

**3e. `enterNode`** (anchor: `const enterNode=node=>{`, then the long `else if(type===...)` chain). Add a branch:

```js
else if(type==='inkWell'){setRun(p=>({...p,well:wellVisitStart(p.well,bonuses)}));setCtx(null);setScreen('well');}
```

**3f. `IN_RUN_SCREENS`** (anchor: `IN_RUN_SCREENS=[`). Add `'well'` so a save made at the Well resumes at the Well.

**3g. The map legend / room list**, if the game shows one, gets the same entry. The room's description on the map:
> *Where the Author held things under until the candle went out. Some of the candles did not.*
> Everything that goes under in the Atlas ends up here: what the Author drowned, what he doubted, and the pages of every tale of yours once it is over.

**Test:** start a run, force a page with an Ink Well (temporarily raise the rare-room chance), enter it. Until Step 4 the screen is empty; that is expected.

---

## 4. Step 2: saves

**Run state** (per tale). Default-guard everywhere with `run.well||WELL_RUN0`:

```js
const WELL_RUN0={candles:[],undertowWaiting:false,undertowVisit:0,notYetUsed:false,undertowBeaten:false,visits:0,casts:0,picks:0,caughtThisVisit:false};
// candles: [{dark:bool}] up to 3
const wellVisitStart=(w,bonuses)=>{const W={...WELL_RUN0,...(w||{})};return{...W,visits:W.visits+1,casts:3+(bonuses.wellCasts?1:0)+(bonuses.wellHook?1:0),picks:0,caughtThisVisit:false};};
```

**Progress state** (across tales), inside `prog`:

```js
prog.well = prog.well || { taken:0, undertowDown:0, cat:{} };
// taken: things pulled out of the Well (all tales). cat: Catalogue entry -> true.
```

The soul tree uses the existing `prog.upgrades.well` / `prog.forks.well` like every other tree, so no new field is needed for it.

**The relic** *The Author's Hook* lives in the run's normal relic list (`run.relics`), so it saves like any relic.

---

## 5. Step 3: the ninth soul tree

**`SOUL_TREES`** (anchor: `satchel:{name:'The Satchel'`; add after the last tree):

```js
well:{name:'What The Ink Kept',sub:'everything the Author drowned to keep writing',tint:'#2fa3b5',
  icon:React.createElement(Anchor,{size:16,className:"ico"}),lines:[
  {t:'A Line In The Ink',d:'+1 cast at every Ink Well.'},
  {t:'A Steady Line',d:'Bites wait longer to be struck, and the light on the reel is a third wider.'},
  {fork:[{k:'shallows',t:'Shallows',d:'Doubt never takes the hook. Nothing bites back unless you cast into the dark.'},
         {k:'deep',t:'Deep Water',d:'Rare catches come twice as often, and so do the things that bite back. The Barbed Unsaid rises.'}]},
  {t:'The Drowned Draft',d:'You may catch a card from an earlier tale, upgrade and all.',seal:'well25'},
  {t:'Two Pages, One Thread',d:'Sewn cards surface: the Ligature’s thread, caught in the ink.',seal:'wellUndertow'}]},
```

Costs are the shared `VERSE_COST` (50 / 120 / 300 / 850 / 1,800). The tint `#2fa3b5` was chosen so it does not clash with The Satchel.

**`SEALS`** (anchor: `drunk50:{`; add after it):

```js
well25:{t:'Take twenty-five things out of the Ink Well.',at:S=>S.wellTaken>=25,of:S=>[S.wellTaken,25]},
wellUndertow:{t:'Pull back against The Undertow, and win.',at:S=>S.wellUndertow>=1,of:S=>[S.wellUndertow,1]},
```

**`sealState`** (anchor: `const sealState=(prog,maxAsc,soulPeak)=>{`). In the returned object add:

```js
wellTaken:((prog.well||{}).taken)||0, wellUndertow:((prog.well||{}).undertowDown)||0,
```

**`verseBonuses`** (anchor: `const verseBonuses=prog=>{`). In the returned object add:

```js
wellCasts:at('well',1), wellSteady:at('well',2), wellShallows:road('well','shallows'), wellDeep:road('well','deep'),
wellDrowned:at('well',4), wellSewn:at('well',5), wellEase:L('well'),
```

(`wellHook` is not a tree bonus: set `bonuses.wellHook = run.relics.includes('authorsHook')` where the Well reads it, or check the relic directly.)

**Test:** the Scriptorium shows 9 trees; buying lines works; the seals on IV and V hold until met.

---

## 6. Step 4: the Well screen

This is the biggest step. Port the prototype's engine into a React screen. The file `prototype/inkwell.html` contains the whole engine in one readable `<script>` (search for `THE INK WELL — prototype engine`). The parts to port, and what to keep exactly:

**Props of the new screen** `InkWellScreen({run,setRun,prog,setProg,bonuses,onFight,onLeave})`. Render it from the main switch next to the others (anchor: `screen==='shop'&&run&&React.createElement(ShopScreen,`):

```js
screen==='well'&&run&&React.createElement(InkWellScreen,{run,setRun,prog,setProg,bonuses,
  onFight:(enc,meta)=>{setCtx({encounter:enc,nodeType:'well',well:meta,returnTo:'well'});setScreen('combat');},
  onLeave:()=>{setCtx(null);setScreen('map');}}),
```

**Layout** (see the prototype):
- The stage: `bg_inkWell` as a cover background, a `<canvas>` over it for the chain, the hook, the ripples, the shadows and the splash.
- Top-left: the **casts** chip (hook icons). Top-right: the **Catalogue** chip (book icon, `n / 21`, tapping it opens the Catalogue).
- Bottom-left: **The Rim** plaque with 3 candles (SVG in the prototype, function `hud()`).
- The reel gauge on the right during a reel (track, light, catch glyph, fill meter, tension bar for creatures).
- Controls under the stage: **Cast the line** (becomes *Strike!* / *Hold to reel*), **Cast into the dark**, **Leave the Well** (becomes *Next visit* only in the prototype; in the game, leaving returns to the map).
- Options: *Tap instead of hold*, *Relaxed reel* (accessibility; store them in `prog.settings` if the game has one).

**The winch and chain** (canvas, from `frame()`):
- Chain origin: art point `[.558,.452]` (the bottom of the painted pulley). Idle hook rests at `[.558,.545]`. Landing point `[.558,.635]`.
- Art point to screen pixel: `artPt(fx,fy,W,H)` with the background drawn `cover` and `background-position: center 62%`.
- Links are sized like the painted chain: `step = 8.6*sc`, ellipse `5.2*sc × 2.9*sc`, where `sc = max(W/1600, H/893)`.
- Casting lowers the hook **straight down**. Once in the ink the hook is hidden; only the chain going in, a ripple ring, and the hook's cyan glow under the surface show.
- The hook glows sea-glass cyan `#5ee6c6`, like the candles on the jetty.
- On a creature bite the chain turns red and goes taut. **In the dark scene** (`bg_inkWell_deep`) no chain is drawn: the painted chain is the line, and a bite is a red pulse where it enters the ink.

**Flow** (from `cast`, `strike`, `startReel`, `reelStep`, `endReel`, `reveal`):
- `cast(dark)`: spend a cast, roll the catch from `table(dark)`, pick the shadow, schedule nibbles and the bite (1.2–3.4 s).
- A shadow shows what is coming: small quill-shadow = loot, a page = drowned/sewn, a circling hesitant one = Doubt, and a shape per creature (`shadow` field below).
- `strike()`: too soon = the cast is lost; within the window = **Good**; within 0.3 s (0.2 s for a creature) = **Perfect**. The window is 850 ms (+400 ms with *A Steady Line*) plus 80 ms grace. A Perfect on loot steps it up once (gold→potion→card→rare).
- `startReel()`: take the profile from `WELL_REEL` (below). For a creature, the profile is its **pull level** (light/heavy/abyss); in the dark, one level heavier. Then apply the tree ease: `speed*=1-.045*e; swing*=1-.03*e; drain*=1-.06*e; fill*=1+.045*e` with `e = number of tree lines written`. Zone height: `.26` (`.34` with *A Steady Line*) times the profile's `zone`.
- `reelStep()`: port it exactly (it was tuned by simulation). Holding pushes the light up (+2.6/s², −2.2/s² when released, ×0.6 while the catch is inside). Thrash (creatures only): 650 ms warning, 1 s thrash; holding through it builds tension (1.1/s), letting the line go slack freezes progress; tension 1.0 parts the line. *Heavy/abyss* motions thrash twice in a row. *Relaxed reel* never fails.
- **Losing a reel** loses the cast, nothing else. **Winning a reel** on loot shows the reveal; on a creature it opens the encounter card (below).

**The catch table** (`table(dark)`, weights):

```js
// normal cast
{gold:30, potion:12, card:16, rare:5*rm, souls:6*rm, lore:2*rm, doubt:shallows?0:8,
 monster:shallows?0:(deep?24:12), drowned:wellDrowned?12:0, sewn:wellSewn?4*rm:0}
// into the dark
{monster:84, rare:6*rm, souls:5*rm, drowned:wellDrowned?3:0, sewn:wellSewn?2:0}
// rm = 2 with Deep Water, else 1. With The Author's Hook, the first catch of each visit is always 'rare'.
```

**Loot rewards** (use the game's own grants):
- gold: 15–40 (×2 from the dark). potion: a random potion via the normal potion grant (respect the satchel cap). card: one common from the class pool. rare: one rare (upgraded on a Perfect). Lost Souls: 10–25 (×2 dark, ×1.5 Perfect), counted like every other soul source. lore: an unfound lore page (all six found: 40 gold). Doubt: `mkCard('doubt')` into the deck.
- **drowned card** (line IV): a card from `prog` history, shown with a provenance line under it: *"Played by The Alchemist · Tale 14 · broke off against The Storykeeper"*, *"… · wrote an ending"*, or *"From a tale the Well no longer remembers"* if unknown. Never from `legacy` (that belongs to *The Pages You Keep*).
- **sewn card** (line V): `ligPrize(run,{maxRarity:'u'})`, upgraded halves on a Perfect. Show it with `<CardView card={card}/>`. Caption: *"Two pages, one thread, caught in the ink. Bound into your deck."* ("sewn into your deck" is reserved for Doubt.)
- Every thing taken out: `prog.well.taken++`, and mark it in `prog.well.cat`.

**The reveal screen** (prototype `reveal()`): the item rises out of the ink (dark silhouette → colour, a ripple ring behind it), a **New** stamp on its corner the first time, a one-line caption, and *Back to the Well*. Potions and lost pages show on a small framed plaque with name, rarity and what they do.

---

## 7. Step 5: the creatures

**Encounter card** (prototype `renderFight`): when a creature is reeled in, show it in the arena (`bg_inkWell_arena`, or `bg_inkWell_deep` from the dark) with its name, *Only in the Ink Well*, HP, trait line, rules, and two buttons: **Fight** and **Let it go**. *Let it go* costs nothing but the cast. **Fight** calls:

```js
onFight([mkEnemy(id, (dark?1.25:1)*actHpMul, dark?1:0)], {id, dark, pull, perfect})
```

For The Blots the encounter is three of them: `[mkEnemy('wellBlot'),mkEnemy('wellBlot'),mkEnemy('wellBlot')]`. Pass the act's `hpMul` like the map's fights do.

A Perfect strike gives the creature **Weak 1** at the start of the fight.

**Add to `ENEMY_DEFS`** (anchor: `const ENEMY_DEFS={`). All ids start with `well` to avoid collisions (for example, `redacted` already exists). Every entry has `wellOnly:true`. Built from actions the game already has (`atk`, `atk2`, `atkN`, `blk`, `buf`, `grip`, `eatPage`, `drinkInk`, `splitAt`, `thorns`, `hands`, `countdown`, `summon`, `phases`, `reacts`). **Pull Under** is the existing `grip`: the creature holds one of your cards until you pry it loose or kill it.

```js
wellRedacted:{name:'The Redacted',hp:[26,32],icon:'〰️',art:'cut_wellRedacted',wellOnly:true,pull:'light',shadow:'eel',hands:1,
  acts:[{t:'grip'},{t:'atk',v:[5,8]},{t:'blk',v:[5,7]},{t:'atk',v:[5,8]}],
  note:'A line the Author struck through so hard the ink went on without him. It swims the way a censor’s pen moves: long, flat, and certain.'},
wellWatermark:{name:'The Watermark',hp:[52,62],icon:'🪼',art:'cut_wellWatermark',wellOnly:true,pull:'heavy',shadow:'blob',hands:1,splitAt:0.5,
  acts:[{t:'grip'},{t:'atk',v:[7,9]},{t:'blk',v:[6,8]},{t:'atk',v:[7,9]}],
  note:'A page the Author left in the ink too long. The paper went soft and clear, and the ones who drowned against it stayed pressed into it, like watermarks. They are still trying to get out.'},
wellBlotter:{name:'The Blotter',hp:[50,58],icon:'🩸',art:'cut_wellBlotter',wellOnly:true,pull:'heavy',shadow:'blob',hands:1,drinkMult:1,
  acts:[{t:'grip'},{t:'drinkInk'},{t:'atk',v:[6,8]},{t:'drinkInk'}],
  note:'Blotting paper, pressed onto every wet page the Author regretted. It has drunk so much ink it reads backwards now, and it is still thirsty.'},
wellQuillUrchin:{name:'The Quill Urchin',hp:[80,90],icon:'✒️',art:'cut_wellQuillUrchin',wellOnly:true,pull:'heavy',shadow:'spiky',hands:1,thorns:3,
  acts:[{t:'grip'},{t:'blk',v:[10,13]},{t:'atk',v:[10,12]},{t:'atk2',v:[5,6]}],
  note:'Every pen the Author broke in a temper went into the Well. They found each other at the bottom and grew a body to hold them.'},
wellBookworm:{name:'The Bookworm',hp:[64,74],icon:'🐛',art:'cut_wellBookworm',wellOnly:true,pull:'heavy',shadow:'worm',hands:1,
  acts:[{t:'grip'},{t:'eatPage'},{t:'atk',v:[10,12]},{t:'eatPage'}],
  note:'It ate through a whole shelf of books the Author never finished, and sank under the weight. Down there it learned to keep what it eats.'},
wellBlot:{name:'A Blot',hp:[10,12],icon:'💧',art:'cut_wellBlots',wellOnly:true,pull:'light',shadow:'dots3',
  acts:[{t:'atk',v:[3,4]},{t:'atk',v:[3,5]}],note:'Where a pen rests too long, ink pools. In the Well the pools find each other.'},
wellGreatBlot:{name:'The Great Blot',hp:[34,38],icon:'⚫',art:'cut_wellGreatBlot',wellOnly:true,
  acts:[{t:'atk',v:[9,11]},{t:'atk2',v:[5,6]}],note:'Three drops, one opinion.'},
wellUnsentLetter:{name:'The Unsent Letter',hp:[32,38],icon:'✉️',art:'cut_wellUnsentLetter',wellOnly:true,pull:'light',shadow:'diamond',hands:1,
  acts:[{t:'grip'},{t:'atk',v:[6,8]},{t:'blk',v:[5,7]}],
  note:'A letter the Author wrote to one of his characters and never sent. It has circled the bottom of the Well ever since, looking for the address.'},
wellTipOfTheTongue:{name:'The Tip of the Tongue',hp:[48,56],icon:'🎣',art:'cut_wellTipOfTheTongue',wellOnly:true,pull:'light',shadow:'lure',
  acts:[{t:'lure'},{t:'atk',v:[7,9]},{t:'lure'},{t:'atk',v:[7,9]}],
  note:'The Author has leaned toward it. So have you: the word you almost had. It hangs in the dark on a thread of its own flesh, glowing. Behind it, the teeth.'},
wellCrust:{name:'The Crust',hp:[62,70],icon:'🦀',art:'cut_wellCrust',wellOnly:true,pull:'light',shadow:'spiky',countdown:4,crust:true,
  acts:[{t:'blk',v:[12,12]},{t:'atk',v:[8,8]},{t:'atk',v:[8,8]},{t:'atk',v:[8,8]}],
  note:'Ink that dried on a page nobody turned for a hundred years, and crawled off it. Back in the Well, the wet ink is taking it apart, and it is angry about that.'},
wellBottomFeeder:{name:'The Bottom-Feeder',hp:[38,46],icon:'🐟',art:'cut_wellBottomFeeder',wellOnly:true,pull:'light',shadow:'wide',
  acts:[{t:'feedDoubt'},{t:'atk',v:[6,8]},{t:'atk',v:[6,8]}],
  note:'The Author’s second thoughts do not dissolve. They sink, and settle, and something at the bottom has grown fat on them.'},
wellBarbedUnsaid:{name:'The Barbed Unsaid',hp:[70,80],icon:'🪝',art:'cut_wellBarbedUnsaid',wellOnly:true,pull:'abyss',shadow:'huge',deepOnly:true,hands:3,
  acts:[{t:'grip'},{t:'atkN',v:[4,4]},{t:'grip'},{t:'atkN',v:[4,4]}],
  note:'Every line ever dropped into this Well, the Author’s first and then every draft of you, lost a hook in something. Most of them lost it in this. It keeps them the way other things keep teeth.'},
wellUndertow:{name:'The Undertow',hp:[100,100],icon:'🐙',art:'cut_wellUndertow',wellOnly:true,miniboss:true,bodies:4,
  acts:[{t:'hurl',v:[12,12]},{t:'hurl',v:[12,12]},{t:'blk',v:[10,12]}],
  phases:[{at:.5,chapter:'WHAT SANK',msg:'Every tentacle comes up at once. It has dragged two more of them out of the ink.',
    gain:{bodies:2},acts:[{t:'hurl2',v:[12,12]},{t:'hurl2',v:[12,12]}]}],
  note:'Everything the Author drowned to keep the story moving, and everyone. It still holds some of them. It remembers every one, and it remembers you.'},
```

**New enemy actions** (small; add them where the combat resolver handles `t:` values, next to `eatPage`/`drinkInk`). Each one has a safe fallback so the fight works even before it is written:

| Action / flag | Behaviour | Fallback |
|---|---|---|
| `t:'lure'` | Adds *The Word You Almost Had* to the player's hand. | Treat as `atk` 6. |
| `t:'feedDoubt'` | Removes one Doubt from hand/draw/discard: heals 6, +1 Strength, remembers it (max 2). On death, those Doubts are gone for good. If the player lets it go before the fight, nothing changes. | Treat as `buf` 1. |
| `crust:true` + `countdown` | **Turns scale with the volume:** 4 in Volume II, 5 in III, 6 in IV, +1 from the dark (set `countdown` when the enemy is made: `2+act+(dark?1:0)`). Each of its turns: Block → 0, +2 Strength (a Perfect strike skips the first +2). At 0 it dissolves: the fight ends as a win, **gold only**, but it still lights a candle. | Remove `countdown` until this is written (make sure the existing countdown never makes it explode). |
| `t:'hurl'` / `'hurl2'` | Hurls a shrouded body: damage `v`, once / twice. `bodies` counts down per hurl. | Treat as `atk` / `atk2`. |
| Sever (The Undertow) | Dealing 15% of its max HP in one player turn (20% in *What Sank*) severs a tentacle: `bodies−1` and it cannot hurl this turn. At 0 bodies it only blocks. | Leave out. |
| Merge (The Blots) | An encounter of three `wellBlot`. If two or more are alive at the start of turn 2, they merge into one `wellGreatBlot` (HP = their HP ×1.25, +1 Strength per extra blot). Area damage stops the merge. The arena must really change: the three small blots flow into one `cut_wellGreatBlot` with a short merge animation. | Three separate blots. |
| Watermark split | At half HP it becomes two **different** enemies: `wellWatermarkCrest` (art `cut_wellWatermarkCrest`: a torn half-bell with the crest and one trapped face; holds the gripped card; intent 6 *Pulls under*) and `wellBleed` (art `cut_wellBleed`: a ragged shred bleeding red-black ink; stings 9–11). Each gets half of the remaining HP. Use the game's `splitAt` with these two ids as the halves. Play a short tear animation. | Plain `splitAt:0.5`. |
| Quill Urchin | Each of its turns: Thorns −1, Strength +1. | Plain `thorns:3`. |
| Bookworm belly | +3 damage per card it ate (`eatPage`), max 2. All eaten cards come back on death. | Plain `eatPage`. |

**The card for The Tip of the Tongue** (`CARD_DEFS`, anchor `const CARD_DEFS={`):

```js
almostWord:{name:'The Word You Almost Had',type:'utility',r:'c',cost:0,energy:1,draw:1,exhaust:true,fleeting:true,noRecord:true,
  art:'cardart_almostWord',desc:'Gain 1 energy. Draw 1. The Tip of the Tongue’s next attack deals double. Exhaust. Fleeting.'},
```

(If the game has no `fleeting`, make it exhaust at end of turn by the same mechanism the game already uses for temporary cards.)

**Keep Well creatures out of everything else:**
- Not in `ACT_POOLS`, `ACT_ELITES`, variants, Echo, or events.
- **`compendiumStats`** (anchor: `const compendiumStats=prog=>{`): in the `enemies` line, add `&&!ENEMY_DEFS[id].wellOnly` to both `got` and `total`, so existing players' Compendium percentage does not change. The Well has its own Catalogue.
- **Bestiary:** skip `wellOnly` in its lists (same filter), or give them their own "The Ink Well" row. Your call.
- The Undertow gives **no keys, no contracts, no boss relic** and never counts as a boss for anything outside the Well.

**Shadows and motions** (the reel reads the creature's shape and movement): `eel`, `blob`, `spiky`, `worm`, `dots3`, `diamond`, `lure`, `wide`, `huge`, drawn in `drawShadow()` in the prototype. Motions: Redacted `floater`, Watermark `mixed`, Blotter `dart`, Quill Urchin `sinker`, Bookworm `mixed`, Blots `jitter`, Unsent Letter `glide`, Tip `lure`, Crust `still`, Bottom-Feeder `bottom`, Barbed Unsaid `heavy`.

**Reel profiles** (`WELL_REEL`, port exactly):

```js
const WELL_REEL={
  gold:{motion:'smooth',speed:.8,swing:.25,zone:1,fill:.28,drain:.14}, potion:{motion:'floater',speed:.9,swing:.3,zone:1,fill:.26,drain:.15},
  card:{motion:'mixed',speed:1,swing:.35,zone:1,fill:.24,drain:.16}, lore:{motion:'sinker',speed:1.1,swing:.4,zone:1,fill:.22,drain:.18},
  souls:{motion:'floater',speed:1.2,swing:.4,zone:1,fill:.22,drain:.18}, rare:{motion:'dart',speed:1.3,swing:.45,zone:.92,fill:.2,drain:.2},
  drowned:{motion:'mixed',speed:1.4,swing:.45,zone:.9,fill:.19,drain:.21}, sewn:{motion:'dart',speed:1.5,swing:.5,zone:.88,fill:.18,drain:.22},
  doubt:{motion:'clingy',speed:.8,swing:.2,zone:1,fill:.3,drain:.1},
  light:{motion:'mixed',speed:1.05,swing:.4,zone:1.05,fill:.22,drain:.16,thrash:true},
  heavy:{motion:'mixed',speed:1.3,swing:.45,zone:.88,fill:.18,drain:.21,thrash:true},
  abyss:{motion:'heavy',speed:1.45,swing:.5,zone:.8,fill:.16,drain:.24,thrash:true}};
```

Simulated results (good player / 20 tries): empty tree, a heavy creature from the dark lands 4 in 20; 3 tree lines, 20 in 20 in ~10 s; full tree, everything in 4–7 s.

---

## 8. Step 6: after a Well fight

**`onCombatWin`** (anchor: `const onCombatWin=(`):
- Where `isElite` is computed, add `(ctx.nodeType==='well'&&ctx.well&&ctx.well.dark)` so a creature from the dark gets the **elite rarity boost**.
- After the normal bookkeeping, if `ctx.nodeType==='well'`:
  - Gold: 25–45, ×1.5 heavy, ×2 abyss, ×2 from the dark (multiply). The Crust dissolving: 20–30 gold only.
  - One card pick via `rewardCards(isElite, …)`, with **one extra choice** for a heavy pull or the dark, and every slot one rarity step higher for an abyssal pull. At most **2 card rewards per Well visit** (`run.well.picks`); after that, gold only.
  - Any card the creature held (grip / belly) returns to the deck.
  - **Light a candle:** unless The Undertow is waiting, push `{dark}` to `run.well.candles`. If this makes 3, the reward screen's *continue* goes to The Undertow's rising instead of the Well.
  - `prog.well.taken++`, mark the creature in `prog.well.cat`.
  - Set `ctx.returnTo='well'` on the reward context.
- **Winning The Undertow** (`ctx.well.id==='wellUndertow'`):
  - First time in this tale: **120 Lost Souls**; later in the same tale: **40**.
  - The relic **The Author's Hook** the first time ever (`+1 cast at every Ink Well; the first catch of each visit is always rare`). Add it to `RELIC_DEFS` with `r:'r'`, `noShop:true`, never in the relic pool.
  - `prog.well.undertowDown++` (this breaks the seal on tree line V).
  - Choose one of three sewn cards: `ligPrize(run,{sig:true})` for the hero's signature, and two from `ligPrize(run)`. If a call returns `null`, use a normal rare card instead. Title: *"Bound into your deck"*.
  - The candles are spent: `run.well.candles=[]`, `undertowWaiting=false`, `undertowBeaten=true`.

**The reward screen's exit** (anchor: `onDone:()=>setScreen('map')` inside `screen==='reward'`): change to

```js
onDone:()=>setScreen(ctx&&ctx.returnTo||'map')
```

**Losing** a Well fight is a normal death. Nothing special is needed. (The candles belong to the run, so they are gone with it.)

---

## 9. Step 7: The Undertow rising and The Rim

- When the third candle lights, the next screen is the rising (prototype `undertowRises()`): the stage goes silent and dark, then the leviathan rises. Title card *Chapter I, The Surface*, the two chapter panels, and two buttons: **Face it** (starts the fight: `onFight([mkEnemy('wellUndertow')],{id:'wellUndertow'})`) and **Not yet**.
- **Not yet** (once per tale): `undertowWaiting=true`, `undertowVisit=run.well.visits`. It rises on the **first cast of the next Ink Well**, with the kicker *"It waited for you."* While it waits, winning another fight adds no candle.
- **The Rim plaque:** 3 candles in the Well HUD (bottom-left); unlit = dark stub, lit = cream candle with a cyan flame, from the dark = black flame. A newly lit one plays a short ignition. It glows while The Undertow waits.
- Chapter texts:
  - *The Surface:* "Each great tentacle holds a shrouded body: the drowned, characters the Author gave up on. Every turn it hurls one at you for 12. Deal 15% of its HP in one turn to sever a tentacle: the body it held sinks, and it has one fewer to throw."
  - *What Sank:* "At half HP it drags two more bodies up out of the ink and hurls two a turn. Severing now takes 20%."

---

## 10. Step 8: the Catalogue of Drowned Words

A panel reachable from the Well's book chip (and, if you like, from the Compendium). 23 entries, each unlocked by `prog.well.cat[key]`. Locked entries show `? ? ?` and a hint.

| Key | Name | Hint when locked |
|---|---|---|
| gold | Gold | What Surfaces |
| potion | A potion | What Surfaces |
| card | A card | What Surfaces |
| rare | A rare card | What Surfaces |
| souls | Lost Souls | What Surfaces |
| lore | A lost page | What Surfaces |
| doubt | Doubt | Circles, hesitates, then bites |
| drowned | A drowned card | Line IV of the tree |
| sewn | A sewn card | Line V of the tree |
| wellRedacted | The Redacted | Well-born · a long black bar under the ink |
| wellWatermark | The Watermark | Well-born · Volume II · faces in a pale bell |
| wellBlotter | The Blotter | Well-born · a stain that spreads |
| wellQuillUrchin | The Quill Urchin | Well-born · bristling, slow to rise |
| wellBookworm | The Bookworm | Well-born · a long pale shape, feeding |
| wellBlot | The Blots | Well-born · three dots on the water |
| wellUnsentLetter | The Unsent Letter | Well-born · glides, trailing ribbon |
| wellTipOfTheTongue | The Tip of the Tongue | Well-born · a light ahead of the dark |
| wellCrust | The Crust | Well-born · hard-shelled, and crumbling |
| wellBottomFeeder | The Bottom-Feeder | Well-born · it smells Doubt |
| wellBarbedUnsaid | The Barbed Unsaid | Only in Deep Water, or in the dark |
| wellUndertow | The Undertow | Seen only after three candles |
| rcage | A rusted cage | Something heavy that does not fight |
| dcage | The Drowned Cage | Deep Water, in the dark, chained shut |

Never-caught entries are weighted ×2 when a creature is picked, so the Catalogue fills over time.

---

## 11b. The cages, and the 12th character

**Why there are people in cages (lore).** When a character stopped following the page (asked the Author a question he could not answer, or walked somewhere he had not written), he did not cross them out: crossing out leaves a mark. He locked them in an iron cage and lowered it into the Well, so the rest of the story could not hear them asking. The skeletons are the ones who stopped asking. The one still alive never did.

**Assets:** `cut_wellRustedCage`, `cut_wellDrownedCage`, `ch_drowned` (the character, full body).

**A rusted cage** (catch key `rcage`)
- Weight: 3 in the normal table, 5 in the dark table. No tree needed.
- Reel: `{motion:'sinker',speed:1.1,swing:.4,zone:.9,fill:.17,drain:.2}`, no thrash. Shadow: a small cage shape. Reel glyph: a cage.
- Reveal: the cage, the caption *"Someone the Author locked away and lowered into the Well. He stopped asking long ago. He is still holding on to what he carried."*, and **three** random choices out of: a common/uncommon relic, a satchel (gold + 2 potions), a card from another class's pool, an unfound lore page (or 40 gold), a purse of 60 gold. Take one.
- **Pry it loose** (45% of cages): a fourth, red-bordered choice: a rare relic (costs 5 HP) or an upgraded rare card (sews a Doubt into the deck).
- Counts as taken (`prog.well.taken++`) and as a Catalogue entry.

**The Drowned Cage** (catch key `dcage`)
- Weight: 1.5, **dark table only**, only with *Deep Water*, and only while `!prog.well.freed`. Once freed it never appears again.
- Reel: the heaviest pull in the game: `{motion:'heavy',speed:1.6,swing:.55,zone:.72,fill:.14,drain:.26,thrash:true}` (then the normal tree ease).
- Reveal: the chained cage, the lore line above, *"This one is chained shut, and something inside is still breathing."*, button **Open it** → the character appears: **The Drowned**, *The Sentence Held Under*, saying *"He did not cross me out. He locked me in, and let the chain run until the page stopped moving. Then he forgot which page it was."*
- Saves: `prog.well.freed=true`, `prog.well.frag=0`, `prog.well.freedVisit=<visit count>`.

**The fragments.** From the next Ink Well on, each visit (once per visit, at arrival) shows one fragment, in order, until all 5 are told: then the class unlocks. Texts:
1. *The first thing he remembers:* "There was a lantern. I was carrying it for someone. I do not remember who, only that they were walking behind me, and then they were not."
2. *The question:* "He wrote me halfway across a bridge. I stopped and asked him where the bridge went. He did not have an answer. I think that was the first time anyone had asked him."
3. *Why there are cages:* "He does not cross out the ones who ask. Crossing out leaves a mark on the page. He builds a cage around them instead, and lowers it into the Well, so the story cannot hear them."
4. *The others:* "There were others down there, in cages like mine. They kept asking, for a while. Then one by one they went quiet, and the quiet ones stopped needing anything at all."
5. *What he wants now:* "Not an ending. He owes me one, but I have stopped waiting for it. Write me a beginning instead. I will find the rest myself."

On the 5th: *"The Drowned can now be written. A new character waits in the Scriptorium."* Store the fragments in `prog.well.frag` and show them also in his `CHAR_STORY` once unlocked.

**The 12th class: The Drowned** (`CLASSES.drowned`; do this as its own task, after the Well works)
- Title: *The Sentence Held Under*. Hook: *The one character who would not stop asking the Author where he was going.* Accent: `#2fa3b5` (the Well's teal).
- Mechanic **Sink and Surface:** *Sink* a card: it goes into the Depths (max 3). At the start of each turn the oldest sunk card *Surfaces* into the hand and costs 0 that turn. Different from The Voidwalker's exhaust: the card always comes back.
- Starting relic **The Last Breath:** the first card you Sink each combat Surfaces upgraded.
- Starting deck: 4 Strike, 4 Brace, *Hold Under* (1: deal 7, Sink a card from your hand), *Come Up For Air* (1: gain 6 block, a card Surfaces now).
- Still to design with the user before building: his card pool, the CHAR_STORY question, unlockable fragments, memory, voice lines and light/dark endings.
- Unlock: `prog.well.frag>=5`.

---

## 11. Which creatures appear where

- Volume I: The Redacted, The Blots, The Unsent Letter, The Bottom-Feeder.
- Volume II adds: The Watermark (Volume II only), The Blotter, The Tip of the Tongue, The Crust.
- Volumes III–IV add: The Quill Urchin, The Bookworm. The Blotter, Tip, Crust and Bottom-Feeder continue.
- The Barbed Unsaid: Volumes II–IV, **only** from the dark or with *Deep Water*.
- HP scales with the act the same way the game already scales normal enemies (use `hpMul` from the act, like the map's fights).

---

## 12. Final checks

- [ ] An old save (from before the Well) loads, and its Compendium percentage is unchanged.
- [ ] A save made at the Well resumes at the Well with the same casts and candles.
- [ ] The map shows at most one Ink Well per volume, and the Marginalia seal works.
- [ ] Loot, a Perfect, a failed reel, *Let it go*, a fight won, a fight lost (death) all behave as in the prototype.
- [ ] Three candles bring up The Undertow; *Not yet* works once per tale and waits for the next Well.
- [ ] Beating The Undertow gives 120 / 40 Lost Souls, The Author's Hook once, breaks the line V seal, and offers the sewn cards through `ligPrize`.
- [ ] The tree lines change the Well immediately (extra cast, wider light, Shallows / Deep Water, drowned and sewn catches).
- [ ] Well creatures never appear anywhere else.
- [ ] Phone width: the reel, the plaque and the fight buttons are all reachable; no sideways scroll.
- [ ] Steam note: all new art was made with Higgsfield (AI), covered by the existing AI content disclosure.
