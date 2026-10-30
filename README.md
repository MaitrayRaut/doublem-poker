# Double Down Hold'em

A heads-up poker game where **Raise is replaced by Double Down**: every double down doubles the stake *and* makes the dealer slip one more face-up card into a random slot of the community cards. Five suits, an endless shoe, and the Balatro hands (Five of a Kind, Flush House, Flush Five) are all in play.

Two versions of the same game live in this repo:

| Version | Where | Run it |
|---|---|---|
| **Browser (HTML5 / JavaScript)** | `web/` (`index.html` + `game.js`) | open `holdem.html`, or upload `holdem-itch.zip` to itch.io |
| **LÖVE2D (Lua)** | repo root (`main.lua`, `game.lua`, ...) | `love .` (LÖVE 11.x) |

No libraries on either side, and nothing is drawn with shapes: every visual is a PNG sprite from `assets/`.

---

## How to play

You face a series of Rivals, one at a time. Win all their chips to move on to the next, tougher one.

**Controls**

| Action | Keyboard | Mouse / touch |
|---|---|---|
| Stand (match the current bet, or stay put if there is nothing to match) | `S` | Stand button |
| Double Down | `D` | Double Down button |
| Next hand / next Rival / new game | `Space` or `Enter` | the blue button |

There is no Fold button.

### Rules

- **Ante ("the blind").** Both players ante every hand. It starts at about 2% of the smaller stack (20 against Rival #1, 40 against #2, 80 against #3 ...) and **doubles every hand**. A new Rival resets it.
- **No flop, turn or river.** All 5 bench cards are dealt **face-up at the start**, and there is a **single betting round**.
- **Stand** matches the current bet (or does nothing if there is nothing to match).
- **Double Down** doubles the stake (20 → 40 → 80 ...) and the dealer adds **one more face-up card at a random position** of the bench. The other player must **Stand** (match) or **Double Down again** (4×, 8× ...). There is no limit except the chips in play.
- **Your stand is final.** After you stand, the Rival can no longer double down in that hand, so you are never asked twice and the game never decides for you. If the Rival stands first, you may still double down and it has to answer.
- **All-in.** If you are out of chips (or cannot cover a bet) you still click Stand once; the Rival just stands by itself.
- **Showdown.** Your best 5 cards out of your 2 hole cards plus the **whole** bench (5 to 16+ cards). Both 5-card hands are shown, then a big **YOU WIN / YOU LOSE** appears.
- **Cards.** An endless random shoe with **five suits** (Spades, Hearts, Diamonds, Clubs, Stars). Cards are drawn with replacement, and each bench card has a 20% chance to copy a card already in play, so duplicates (and the Balatro hands) really show up.

**Hand ranking** (low to high): High Card, Pair, Two Pair, Three of a Kind, Straight, Flush, Full House, Four of a Kind, Straight Flush, **Five of a Kind, Flush House, Flush Five**.

### Progression

- Beat a Rival (their chips reach 0) and **you keep your chips**. The next Rival sits down with **double the chips of the previous one** (1,000, 2,000, 4,000 ...).
- Lose all your chips and everything **restarts** from Rival #1.
- Big popups mark both moments: **RIVAL DEFEATED!** and **BANKRUPT!**
- Each Rival has its own face sprite with three expressions (neutral, smug, defeated).
- After every action there is a short moment where input is ignored, so you can see what just happened.

---

## The Rivals (bots)

Every Rival owns an **expectation table**: for each number of cards on the bench (5 to 16) it lists a **low end** and a **high end**.

```lua
{
  name = "Gambler", note = "doubles down on almost anything",
  --        5   6   7   8   9   10  11  12  13  14  15  16   (bench cards)
  expect  = { P,  P,  P,  TP, T,  S,  F,  F,  FH, FH, FH, Q },    -- low end
  ceiling = { Q,  Q,  Q,  Q,  SF, SF, K5, K5, K5, FHS,FHS,F5 },   -- high end
}
```

On its turn the Rival evaluates its current best hand (hole cards + the whole bench) and compares it with the table for the current bench size:

| Its hand | It does |
|---|---|
| weaker than the low end | **Stands** |
| between low end and high end | **Doubles down** (stands if it cannot) |
| at or above the high end | **Stands** (already winning: it lets you walk into it) |

An entry is a hand name, or `{ "Pair", 11 }` to also ask for a rank (here: Jacks or better). The first 8 Rivals are hand-written (Gambler, Rookie, Regular, Moody, Careful, Skeptic, Mastermind, Shark); every Rival after that is generated deterministically, so Rival #20 is always the same Rival.

To add or tune a bot, edit `bots.lua` (Lua) / the `Bots` block in `web/game.js` (JS). Both versions are kept identical: a cross-check of 40 Rival tables and 6,000 random hands gave 0 mismatches.

---

## Running it

### Browser version

- **Just play:** open `holdem.html` (a single file with the code and all sprites embedded).
- **Publish on itch.io:** upload `holdem-itch.zip` as an **HTML** project, tick "played in the browser" and set the viewport to **820 × 720**. `index.html` is at the root of the zip, as itch.io requires. The canvas scales to the window and works with touch.
- **Develop locally:** `cd web && python3 -m http.server`, then open <http://localhost:8000>.
- **Rebuild the packages** after editing `web/`: `python3 tools/build_web.py` (writes `holdem.html` and `holdem-itch.zip`).

### LÖVE version

1. Install [LÖVE 11.x](https://love2d.org/).
2. Run `love .` in the repo root, or zip `main.lua game.lua poker.lua bots.lua sprites.lua conf.lua assets/` into `holdem.love` and double-click it.

---

## Project layout

```
main.lua       LÖVE front-end: layout, animation, popups, input
game.lua       rules + state machine (no drawing)
poker.lua      cards, shoe, hand evaluator (any number of cards, duplicates allowed), best-5 picker
bots.lua       the Rivals and their expectation tables
sprites.lua    loads assets/ and offers the draw helpers
conf.lua       window: 820 x 720
assets/        all sprites (PNG) + assets/README.txt describing every layout
web/
  index.html   page with the <canvas>
  game.js      the whole browser game (poker, bots, rules, renderer, input) in one file
  assets/      copy of the sprites
tools/
  make_sprites.py   generates every sprite (Pillow + numpy)
  make_faces.py     generates the 16 Rival faces (called by make_sprites.py)
  build_web.py      builds holdem.html and holdem-itch.zip
```

The rules live in `game.lua` and in the matching `Game` section of `game.js`; they contain no drawing code, so they are easy to test headless.

---

## Art

All sprites are generated, so there are no licensing worries. To use your own art, overwrite the PNGs in `assets/` (and `web/assets/`) keeping the **same file names and layouts**; the layout of each sheet is described in `assets/README.txt` and at the top of `sprites.lua`. Highlights:

- `cards.png`: 13 columns (2 to A) × 5 rows (suits), 160 × 224 cells
- `faces.png`: 16 faces × 3 expressions (normal, win, lose), 128 × 128 cells
- `rays.png`, `confetti.png`, `sparkle.png`, `cracks.png`, `vignette.png`, `pixel.png`: **white** popup effects that the game tints at draw time
- `panel.png`, `outline.png`, `button.png`: 9-slice images

To regenerate the placeholder art:

```bash
pip install pillow numpy
python3 tools/make_sprites.py          # writes assets/
cp assets/*.png web/assets/            # keep the browser copy in sync
python3 tools/build_web.py
```

---

## Tweaking the rules

`Game.RULES` in `game.lua` (`RULES` in `game.js`):

| Setting | Default | Meaning |
|---|---|---|
| `startChips` | 1000 | your chips, and Rival #1's; each next Rival has double |
| `ante` | 20 | smallest starting blind |
| `anteShare` | 0.02 | starting blind ≈ this share of the smaller stack, rounded to 20 × 2^k |
| `echoChance` | 0.20 | chance a bench card copies a card already in play (more Balatro hands) |
| `botFoldChance` | 0 | chance a weak Rival folds instead of matching (0 = never) |

---

## Testing

The game logic was checked headless (the Lua logic under Lua 5.3, and the browser build in headless Chromium via Playwright), including: chip conservation over hundreds of thousands of actions, bot decisions against their tables, the best-5 picker against brute force, the evaluator in Lua vs JS (identical on 6,000 random hands), "never asked again after standing", and a stubbed-LÖVE run of the whole UI that fails on any shape-drawing call or out-of-bounds sprite quad.

---

## Credits

Poker hands beyond Straight Flush (Five of a Kind, Flush House, Flush Five) are borrowed from *Balatro*; this project is an unofficial fan experiment and is not affiliated with it.
