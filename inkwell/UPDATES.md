# The Ink Well: updates after the first guide

**For the session that already applied `GUIDE.md` (steps 1–8).** Apply only what is below. Do not run the guide again.

Before starting, check what is already in the game. Some of these may already be in, if the guide you used was a later copy. Skip any item that is already there.

Same rules as the guide: small anchored edits, re-read the master just before each write, never write it back from an older copy, keep saves backward-compatible.

**New art** (copy from `assets/img/` into the game and add to `ART`):
`cut_wellRustedCage`, `cut_wellDrownedCage`, `ch_drowned`, `cut_wellWatermarkCrest`, `cut_wellBleed`, `cut_wellUndertowEmpty`, `sp_wellBody`, `cardart_heldBreath`, `cardart_chainRunsOut`, `cardart_undertowsPull`, `cardart_inkInTheLungs`, `cardart_salvage`. For U9: `cut_wellUndertowMaw`, `cut_wellCageSpat`.

The playable reference for everything below is `prototype/inkwell.html`. Section 3 has buttons that open each changed fight directly.

---

## U1. Creatures from the dark are bigger
A creature caught with *Cast into the dark* has **+25% HP** (`mkEnemy(id, 1.25*actHpMul, 1)`), is drawn **15% larger** in the arena, and its shown attack already includes the +1 Strength. Its status chip reads *From the dark · +25% HP · +1 Str · gold ×2*.

## U2. The Blots really merge, on screen
The fight starts with three small `wellBlot` enemies, each with its own HP bar. If two or more are alive at the start of turn 2, they **visibly flow together** (short merge animation) into one `wellGreatBlot`: HP = their remaining HP × 1.25, +1 Strength per extra blot. Area damage that kills them first stops it.

## U3. The Watermark really tears in two
At half HP it is replaced by **two different enemies** with their own art, each with half the remaining HP and a short tearing animation:
- `wellWatermarkCrest` (art `cut_wellWatermarkCrest`): holds the gripped card; intent 6, *Pulls under*.
- `wellBleed` (art `cut_wellBleed`): stings for 9–11.

## U4. The Crust lasts longer in later volumes
Its dissolve countdown is `2 + act + (dark ? 1 : 0)`: 4 turns in Volume II, 5 in III, 6 in IV, one more from the dark. It still gains +2 Strength each turn.

## U5. The Undertow shows every action
- Draw it as `cut_wellUndertowEmpty` (empty tentacles) with **body sprites** `sp_wellBody` hung from the tentacle tips. Positions (fractions of the art box, anchor = top of the body): `[.215,.27]`, `[.35,.12]`, `[.645,.12]`, `[.795,.27]` from the start, and `[.08,.56]`, `[.925,.56]` added in *What Sank*. Body width about 6.5% of the art width.
- **Hurl:** a body flies at the player (scale up, fade, red flash), then the tentacle drags it back up.
- **Sever** (15% of max HP in one player turn, 20% in Chapter II): that body falls into the ink and is gone. A `Bodies N` chip counts down.
- **What Sank** (half HP): two more bodies rise out of the ink into two more tentacles; it now hurls two a turn. With no bodies left, it only blocks.
- Intent shows `12 · Hurls a body` / `12×2 · Hurls two bodies` / `10 · Blocks`.

## U6. Drowned cards are random from real history; no waterline
- A drowned card is **any card actually played in an earlier tale** (all characters, with the upgrade it had), never from `legacy`. Provenance line from that history: *"Played by The Duelist · Tale 17 · broke off against The Last Editor"*, *"… · wrote an ending"*, *"Lost to the Well by … · taken by The Bookworm"*, or *"From a tale the Well no longer remembers"*.
- **Remove the horizontal teal line** across the art of drowned cards. Keep only a soft dark fade at the bottom and the teal *Drowned* badge.

## U7. Five cards that exist only in the Well
Add to `CARD_DEFS` with `well:true` (badge *Ink Well*, colour `#2fa3b5`; never in shops, rewards or class pools). A *card* catch is one of them 20% of the time, a *rare* catch 25%.

| id | Name | Rarity | Cost | Text |
|---|---|---|---|---|
| `heldBreath` | Held Breath | u | 1 | Gain 8 block. Next turn, gain 1 energy. |
| `chainRunsOut` | The Chain Runs Out | r | 2 | Deal 14 damage. If this kills, gain 15 gold. |
| `undertowsPull` | The Undertow's Pull | r | 1 | Pull a random card from your discard pile into your hand. It costs 0 this turn. Exhaust. |
| `inkInTheLungs` | Ink in the Lungs | u | 1 | Apply 3 poison and 1 weak to ALL. |
| `salvage` | Salvage | c | 0 | Draw 2. Gain 5 gold. Exhaust. |

## U8. The cages, and the 12th character
Everything in **section 11b of `GUIDE.md`** ("The cages, and the 12th character"): the rusted cage with a skeleton (choose one of three things, or pry one loose at a price), the Drowned Cage (from the dark, with Deep Water, 1.5%), freeing **The Drowned**, his five fragments over later visits, and the two new Catalogue entries (23 in total). The playable 12th class itself is a separate task, to design with the user first.

## U9. The Undertow swallowed him (changes how the Drowned Cage appears)
**Apply only after U8 works.** This changes U8 in one place: where the Drowned Cage comes from. Everything after the cage opens stays as U8 / GUIDE 11b describe:
- *Open it*;
- `prog.well.freed`;
- the five fragments;
- the unlock.

**The lore.** The Undertow is a collector: it gathers whatever the Author drops into the Well.
- **Bodies.** Characters thrown in without a cage drowned and sank. It does not eat them. It wraps each one in the sodden pages of the story it was cut from (the shrouds) and keeps them in its arms, which is why it has bodies to hurl.
- **Cages.** It cannot open iron with its arms, so it **swallows a cage whole** to crack it in its gut. His cage held, and he is still alive inside.
- **The rusted cages.** These are the ones it never found. They sank to the bottom on their own.

**New art:**
- **The Undertow is redrawn.** Replace `cut_wellUndertowEmpty` and `cut_wellUndertow` with the new files. It is now a clean standalone cutout:
  - built like a real octopus: a dome head with the face on the front, and all eight arms growing from one ring under the face;
  - every arm sweeps up and out around the head like a raised crown, and nothing hangs below the body, so it clearly floats;
  - no ink pool under it, so it can be animated later as its own character. Draw the ink at its feet from the arena, as for other Well creatures, if needed.

  The open-mouth art (`cut_wellUndertowMaw`) is identical except for the mouth. The mouth opens in proportion to the face, with the cage deep inside.
- **The body points move to the new tentacle tips.** If U5 is already in, update its points to `[[.215,.27],[.35,.12],[.645,.12],[.795,.27],[.08,.56],[.925,.56]]`:
  - the first four are the raised tentacles;
  - the last two are the side tentacles, used in *What Sank*.
- `cut_wellUndertowMaw`: The Undertow with its maw open and the chained cage visible in its throat. It has the same pose and framing as `cut_wellUndertowEmpty`, so it can be swapped in place.
- `cut_wellCageSpat`: the cage, just heaved up, lying in a pool of ink and slime.

**Changes:**
1. **Remove the random catch.** Take `dcage` out of the dark catch table. It is no longer reeled up at 1.5%. If U8 added it, remove that line only.
2. **The hint in the fight.** When The Undertow enters *What Sank* (half HP), and on every hurl after that, swap its art to `cut_wellUndertowMaw` for about 0.6 s, then back.
   - Only do this while `!prog.well.freed`.
   - The first time, add a log line: *"Something in its throat is glowing. Something in there is still breathing."*
3. **The cage comes up.** On the **first victory** over The Undertow while `!prog.well.freed`, show a new scene **before** its normal reward:
   - Image: `cut_wellCageSpat` on the Well background.
   - Caption: *"The Undertow heaves, and something comes up with the ink: an iron cage, chained shut, scraped by teeth. Something inside is still breathing."*
   - Button: **Open it**. It goes straight into the U8 Drowned Cage reveal: the character, his first line, and saving `prog.well.freed=true`, `frag=0`, `freedVisit`.
   - Then The Undertow's normal reward follows as before.
   - If The Undertow was already beaten before this update (an old save with `!prog.well.freed`), the cage comes up the next time it is beaten.
4. **Fragment 3** ("Why there are cages") becomes: *"He does not cross out the ones who ask. Crossing out leaves a mark on the page. He builds a cage around them instead, and lowers it into the Well, so the story cannot hear them. He did not know what lives down there. Or he did, and lowered us anyway."*
5. **The Catalogue entry** `dcage`: change its hint to *"In the belly of The Undertow"*. Leave the key and the count unchanged.
6. **The cage art** `cut_wellDrownedCage` is no longer used by a catch. Keep it in `ART` anyway: old saves may reference it in the Catalogue.

## U10. *Next visit* is centred
When the casts for this visit are used up, only the main button is left (*Next visit*). Hide the empty cells for *Cast into the dark* and *Leave the Well*, and centre that button in the control row, at most about 440px wide. On a phone it keeps its full width.

## U11. The reel is the centrepiece
While reeling (phase `reel`), the gauge is the main thing on screen, not a small panel on the right:
- **Position:** centred in the Well scene (`left:50%`, about 54% from the top), about 70% of the scene's height (62% on a phone).
- **Size:** the track is about 104px wide (84px on a phone), the progress meter about 18px. The hook glyph is about 54px, and the light zone and corner diamonds are scaled to match.
- **Background:** the scene behind it dims and blurs slightly (brightness about .42, blur about 1.5px). The chain and hook layer fades to about 55%. Both return when the reel ends.

The controls and the reel logic are unchanged. See `prototype/inkwell.html`: cast, wait for the bite and strike.

## U12. Round icon controls, the cast in the middle
The three text buttons under the Well become round icon buttons, each with a short caption under it:
- **Layout:** a three-column grid (`1fr auto 1fr`). The main button sits in the middle and is always centred. Leave the Well / Let go is on the left, Cast into the dark on the right.
- **Main button:**
  - Look: a large gold disc, about 88px (78px on a phone).
  - Icon and caption change with the moment: a hook and *Cast the line*, an arrow and *Next visit*, a lightning bolt and *Strike!* (pulsing gently), a winch and *Hold to reel* (or *Tap: rise / fall*).
- **Side buttons:**
  - Look: about 56px dark discs with a gold rim.
  - Icons: a door with an arrow for *Leave the Well*, a broken chain for *Let go*.
  - *Into the dark* is violet: a hook with a crescent moon.
- **Hidden buttons:** they keep their cell, so nothing moves.
- **Captions:** on a phone they wrap to two lines.
- **Labels:** every button keeps its full label as `title` and `aria-label`.

Icons are inline SVG with `stroke: currentColor`. Copy them from the prototype (`CTL_ICO`). Keep the existing handlers: pointer down/up on the main button for hold-to-reel, and Enter.

**The monster glyph** in the reel gauge (for a creature on the line) is now a dark predatory fish silhouette: teeth, a pale eye and a glowing lure, with a red outline. It replaces the pink toothed mouth. Copy it from the prototype (`GLYPH.maw`).

---

When done, tell the user which items you applied and which were already in.

**Next, as its own step:** `DROWNED.md` builds the 12th character (The Drowned). Start it only once U8 works.
