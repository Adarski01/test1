# The Ink Well: updates after the first guide

**For the session that already applied `GUIDE.md` (steps 1–8).** Apply only what is below. Do not run the guide again.

Before starting, check what is already in the game. Some of these may already be in, if the guide you used was a later copy. Skip any item that is already there.

Same rules as the guide: small anchored edits, re-read the master just before each write, never write it back from an older copy, keep saves backward-compatible.

**New art** (copy from `assets/img/` into the game and add to `ART`):
`cut_wellRustedCage`, `cut_wellDrownedCage`, `ch_drowned`, `cut_wellWatermarkCrest`, `cut_wellBleed`, `cut_wellUndertowEmpty`, `sp_wellBody`, `cardart_heldBreath`, `cardart_chainRunsOut`, `cardart_undertowsPull`, `cardart_inkInTheLungs`, `cardart_salvage`.

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
- Draw it as `cut_wellUndertowEmpty` (empty tentacles) with **body sprites** `sp_wellBody` hung from the tentacle tips. Positions (fractions of the art box, anchor = top of the body): `[.19,.26]`, `[.40,.11]`, `[.72,.06]`, `[.86,.15]` from the start, and `[.37,.38]`, `[.65,.77]` added in *What Sank*. Body width about 6.5% of the art width.
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

---

When done, tell the user which items you applied and which were already in.
