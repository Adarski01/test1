# The Drowned: the 12th character

**For the session that already applied `GUIDE.md` and `UPDATES.md`.** It builds the playable class that the Drowned Cage (GUIDE 11b / UPDATES U8) unlocks. Apply it **after** U8 is in and working.

Same rules as before:
- Make small anchored edits.
- Re-read the master just before each write, and never write it back from an older copy.
- Keep saves backward-compatible: only add fields, and default-guard everything.

**Follow Tal, the 11th class, as the template.** Copy how Tal is declared and wired in everywhere:
- `CLASSES.tal`, `CHAR_STORY.tal`, the `cls:'tal'` cards and the relic `foldedPage`;
- the class select grid (560×750 plate) and the secret unlock with `unlockHint`;
- Tal's two earned looks (`TAL_TONE_NAMES`, `talLook`, `prog.talSeen`, `skinPerkOf`); see D6;
- rewards, the shop pool, Mastery, the Compendium and the Chronicle.

Wherever Tal appears in a list of classes, The Drowned goes after him. Do not invent a new pattern where Tal already has one.

**New art** (in `assets/img/`, add to `ART` the way Tal's art is keyed):
- Class plates, 560×750, keyed like Tal's (`cls_tal`, `cls_tal_light`, `cls_tal_dark`):
  - `cls_drowned`: the base look.
  - `cls_drowned_light`: the light-ending look.
  - `cls_drowned_dark`: the dark-ending look.
- `ch_drowned` (380×560, cut out) stays for the Well reveal.
- `rl_lastBreath`: the relic icon, 128×128.
- Card art, 240×240:
  - `cardart_holdUnder`
  - `cardart_comeUpForAir`
  - `cardart_undercurrent`
  - `cardart_deadWeight`
  - `cardart_whereDoesItGo`
  - `cardart_pressure`
  - `cardart_ironBars`
  - `cardart_everythingRises`
  - `cardart_chainedShut`
  - `cardart_stillAsking`

All art was made with Higgsfield, so it is covered by the existing AI content disclosure.

---

## D1. The class entry

```js
drowned:{name:'The Drowned',title:'The Sentence Held Under',
  hook:'The one character who would not stop asking the Author where he was going.',
  icon:'⛓️',color:'#2fa3b5',cls:'drown',relic:'lastBreath',secret:true,
  unlockHint:'Something is still breathing at the bottom of the Ink Well. Bring it up, and listen to all it has to say.',
  blurb:'Sink specialist. What goes under always comes back, and costs nothing when it does.',
  law:'What Goes Under',
  lawText:'You can Sink cards into the Depths (up to 3). At the start of each turn the oldest card down there Surfaces into your hand and costs 0 that turn. Nothing you Sink is ever lost, but the Depths empty when the fight ends.',
  deck:()=>[...Array(4).fill('strike'),...Array(4).fill('defend'),'holdUnder','comeUpForAir']}
```

Unlock: `(prog.well||{}).frag>=5`. That is the 5th fragment told at the Well, as in GUIDE 11b. Use the same place and the same check style as Tal's unlock. When it first unlocks, show the line *"The Drowned can now be written. A new character waits in the Scriptorium."* if U8 does not already show it.

## D2. The mechanic: Sink and Surface

**State:** `combat.depths = []`, an ordered list of card instances, oldest first, max 3. It is combat-only, never saved into the deck, and cleared at the end of every fight. Default-guard it as `combat.depths||[]`, so a combat saved before this change still loads.

**Sinking a card:**
- Move a card from the hand to the end of `depths`.
- Which card: use the existing hand-pick UI if the game has one (the one used by cards that choose a card in hand). Otherwise Sink the highest-cost card in hand other than the one being played; on a tie, take the leftmost.
- If the hand is empty, nothing happens.
- If `depths` already holds the max, the oldest card Surfaces first to make room.

**Surfacing:**
- Remove the oldest card from `depths` and put it into the hand, with cost 0 until end of turn (a temporary cost, the same way any "costs 0 this turn" effect already works).
- If the hand is at its size limit, the card stays down and tries again next turn.

**Start of each player turn:** after the draw, one card Surfaces, if any.

**What doesn't touch the Depths:**
- Sunk cards are not in the draw, discard or exhaust piles.
- Shuffles, discards, exhaust effects and enemy effects that take cards from hand or piles (`eatPage`, grip, etc.) leave them alone.
- A gripped card can't be sunk.

**UI:**
- A small "Depths" stack next to the draw pile: up to 3 face-down cards with a teal edge (`#2fa3b5`), plus a number.
- Tapping or hovering it lists the cards inside, oldest first.
- **Sink:** the card slides down and fades into the stack.
- **Surface:** it rises out of the stack into the hand with a few teal bubbles.
- Keep it reachable at phone width.

**Keyword tooltips** (add to the glossary the game uses for keywords):
- **Sink:** put a card from your hand into the Depths (up to 3). It is not discarded.
- **Surface:** the oldest card in the Depths returns to your hand and costs 0 this turn.

## D3. Starting relic

```js
lastBreath:{r:'r',name:'The Last Breath',icon:'🫧',desc:'The first card you Sink each combat Surfaces upgraded (for this combat).'}
```

Not in any shop or reward pool, the same as the other starting relics. The upgrade is on the combat copy only; the deck card is not changed.

## D4. The cards (`cls:'drown'`, 10, like Tal)

The `text` is what the card shows; write it in the same way the other cards' text is generated or written.

| id | Name | Type | Rarity | Cost | Effect | Upgrade |
|---|---|---|---|---|---|---|
| `holdUnder` | Hold Under | attack | c (starter) | 1 | Deal 7. Sink a card from your hand. | dmg 10 |
| `comeUpForAir` | Come Up For Air | defense | c (starter) | 1 | Gain 6 block. A card Surfaces now. | block 9 |
| `undercurrent` | Undercurrent | attack | c | 1 | Deal 6. If the Depths are not empty, deal 4 more. | dmg 9 |
| `deadWeight` | Dead Weight | defense | c | 1 | Gain 8 block. Sink a card from your hand. | block 11 |
| `whereDoesItGo` | Where Does It Go? | utility | c | 1 | Draw 2. Sink a card from your hand. | cost 0 |
| `pressure` | Pressure | attack | u | 2 | Deal 4, once, plus once more for each card in the Depths. | dmg 6 |
| `ironBars` | Iron Bars | defense | u | 2 | Gain 11 block and 2 Thorns. | block 14, Thorns 3 |
| `everythingRises` | Everything Rises | special | r | 2 | Every card in the Depths Surfaces now. Exhaust. | cost 1 |
| `chainedShut` | Chained Shut | special | r | 1 | For the rest of this combat, the Depths hold 4, and each card that Surfaces gives you 3 block. Exhaust. | cost 0 |
| `stillAsking` | Still Asking | attack | l | 2 | Deal 10 to ALL. Instead of going to the discard pile, this Sinks itself. | dmg 14 |

Suggested data, reusing existing fields where they exist (`dmg`, `block`, `draw`, `gain`, `aoe`, `exhaust`, `hits`, `u`). Only the new behaviours get new keys:
- `sink:1`: after the card's other effects, Sink one card from hand.
- `op:'surfaceNow'`: one card Surfaces.
- `op:'surfaceAll'`: every card in the Depths Surfaces.
- `op:'ifDeep', bonusDmg:4`: the bonus applies only when `depths.length>0`.
- `op:'perDepth'`: hits = 1 + `depths.length`, counted when played.
- `op:'chainedShut'`: sets `combat.depthMax=4` and `combat.surfaceBlock=3`.
- `sinkSelf:true`: when played, the card goes to the Depths instead of discard. If the Depths are full, the oldest Surfaces first.

These cards are in the reward, shop and Mastery pools only for The Drowned, the same as Tal's. No ally cards for now.

## D5. His story (`CHAR_STORY.drowned`)

Same shape as Tal's. Gate the fragments the same way.

```js
drowned:{q:'He asked the Author where the bridge went. Will anyone ever answer him?',
  frag:[
    {t:'The lantern he carried was not his. He was meant to hand it over at the far end of the bridge. The bridge never got a far end.',at:s=>s.runs>=1,hint:'walk one tale as him'},
    {t:'He counts the links of every chain he sees. It is the only number the Well ever taught him.',at:s=>s.runs>=3,hint:'walk three of his tales'},
    {t:'He does not hate the Author. He would only like, once, to be told where he is going, or to be allowed to find out.',at:s=>s.wins>=1,hint:'write him an ending'}],
  mem:{n:/* next free memory number, as the other classes do */,t:'The bridge, finished at last, plank by plank, in his own hand and not the Author’s. At the far end, someone takes the lantern from him.',at:s=>s.wins>=2,hint:'write him a second ending'},
  voice:{start:['I asked where this goes. Nobody answered. So I will go and look.','Up, then. Slowly.'],
    boss:['You are only another question.','I have been under heavier things than you.'],
    low:['Not under again. Not yet.'],
    noInk:['Breathe. Wait.'],
    win:['There. That is where it went.']},
  end:{light:'He reaches the end of the bridge and finds it was never finished. He finishes it himself, and hands the lantern to the one who was walking behind him all along.',
    dark:'He goes back down. Not into the cage: into the quiet, where the others stopped asking. He finds that he can stop too, and the Well closes over the last question in the book.'}}
```

Also show **the five fragments told at the Well** (GUIDE 11b) in his story page, under a heading *"Told at the Well"*, from `prog.well.frag`. They are already unlocked by the time he is playable.

If the game has a per-class line in the Author's final narration or an "ending" blurb list (Tal has one, for example `tal:'The name was always his…'`), add:
*drowned: "He was never locked away for being wrong. He was locked away for asking, and he never stopped, and in the end the question was the only thing in the Well still moving."*

## D6. His two looks: light and dark

He follows **Tal's two fates**, not the bought Remembered look:
- **No Remembered skin and no signature potion.** He was not a draft that lost itself. He was locked away for asking, so there is no face to hand back.
- A look is **opened by living it:** finish a run as him, and the tone of the ending that run earned (`storyEnding(fr).tone`) opens that look.
- Once a look is open, the player can wear it whenever they like.
- These looks are **not for sale**, and `whole` never gates them.

Build it the same way as Tal's:
- `prog.drownedSeen={}`: add it to the default `prog`, default-guarded, and set it in the same place that sets `talSeen`.
- `DROWNED_TONE_NAMES={light:'The One Who Finished The Bridge',dark:'The One Who Stayed Under'}`.
- `drownedLook(prog)`: the same logic as `talLook`, with the preference in `prog.skin.drowned`.
- In the class screen, give him the same three-look picker as Tal:
  - the ORIGINAL look (`ART.cls_drowned`);
  - the light look (`ART.cls_drowned_light`), once opened;
  - the dark look (`ART.cls_drowned_dark`), once opened.
- `heroPortrait` and every other place that draws Tal's chosen look should draw his chosen look too.

**Perks**, in `skinPerkOf`, next to Tal's:

| Look | Name | Perk shown | Rule |
|---|---|---|---|
| base | ORIGINAL | Remove 1 starting card | `{removeCard:1}` (same as every base look) |
| light | ✦ THE ONE WHO FINISHED THE BRIDGE | Cards that Surface come up upgraded | `{surfaceUpgrade:true}`: every card that Surfaces is upgraded for the rest of that combat (combat copy only). |
| dark | ✦ THE ONE WHO STAYED UNDER | Cards that Surface hit ALL for 3 · sinking one costs 1 HP | `{surfaceAoe:3,sinkHp:1}`: each Surface deals 3 to all enemies; each Sink costs 1 HP (not when the hand is empty and nothing sinks). |

With the light look, The Last Breath no longer adds anything. That is fine: the look is the reward for the ending.

## D7. Checks

- [ ] An old save loads, and the class list shows The Drowned locked, with its hint.
- [ ] With `prog.well.frag>=5` he unlocks and appears after Tal in the grid, with the plate and the teal accent.
- [ ] Starting deck: 4 Strike, 4 Defend, Hold Under, Come Up For Air; relic The Last Breath.
- [ ] Sink and Surface:
  - Sinking takes a card from hand into the Depths (max 3, or 4 with Chained Shut).
  - One card Surfaces each turn, costing 0.
  - The Last Breath upgrades the first sunk card.
  - The Depths are empty in the next fight.
- [ ] Pressure counts the Depths; Still Asking sinks itself and comes back next turn free.
- [ ] His 10 cards appear only in his rewards and shop, with art.
- [ ] Story page: question, 3 fragments, memory, both endings, and the 5 Well fragments.
- [ ] A run as him that earns the light ending opens *The One Who Finished The Bridge*; one that earns the dark ending opens *The One Who Stayed Under*. Both can be worn, and their perks apply. He has no Remembered look for sale.
- [ ] Phone width: the Depths stack and its tooltip are reachable; no sideways scroll.

When done, tell the user what was added and anything you had to decide differently.
