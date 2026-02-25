# 🃏 Blackjack Game — Project Specification

## Overview

A classic/retro-themed casino blackjack game built as a single-page web application. The game features realistic blackjack rules, rich audio/visual feedback, and a unique "Possessions Mode" that activates when a player runs out of money.

---

## Visual Design

### Aesthetic
- **Theme:** Classic / Retro Casino (1950s–1970s Vegas style)
- **Color Palette:**
  - Deep green felt table: `#1a4a2e` / `#0f3320`
  - Gold/amber accents: `#c9a84c` / `#f0c040`
  - Cream card faces: `#fdf6e3`
  - Dark wood border/frame: `#3b1f0a`
  - Muted red: `#8b1a1a`
- **Typography:**
  - Headings: Serif font (e.g., *Playfair Display* or *Cinzel*) — evokes old-school casino signage
  - Body/UI labels: Monospaced or slab-serif (e.g., *Courier Prime* or *Roboto Slab*)
- **Card Style:** Classic white cards with traditional pip symbols (♠ ♥ ♦ ♣), red suits for hearts/diamonds, black for spades/clubs
- **Chip Style:** Circular casino chips with colored rings and denominations ($5, $25, $100, $500)
- **Background:** Dark wood-grain texture or dark green felt pattern; faint decorative border around the table

### Layout
```
┌─────────────────────────────────────────────┐
│  BLACKJACK   BALANCE: $500  COUNT: +2  ♪   │
│─────────────────────────────────────────────│
│              DEALER'S HAND                  │
│         [Card] [Card] [???]                 │
│          Dealer Score: ?? + ?               │
│─────────────────────────────────────────────│
│  Strategy Hint → Stand          (if ON)     │
│─────────────────────────────────────────────│
│           GAME STATUS MESSAGE               │
│─────────────────────────────────────────────│
│              PLAYER'S HAND                  │
│         [Card] [Card] [Card]                │
│          Player Score: 17                   │
│─────────────────────────────────────────────│
│  Current Bet: $50              [CLEAR]      │
│  [Chip $5] [Chip $25] [Chip $100] [Chip $500]│
│─────────────────────────────────────────────│
│   [HIT]  [STAND]  [DOUBLE]  [SPLIT]  [DEAL] │
└─────────────────────────────────────────────┘
```

---

## Instructions Screen (Pre-Game)

Displayed before the game begins. Player must click "Deal Me In" to proceed.

### Content to Display:
1. **Goal:** Get a hand value closer to 21 than the dealer, without going over ("busting").
2. **Card Values:**
   - Number cards (2–10) = face value
   - Face cards (Jack, Queen, King) = 10
   - Ace = 1 or 11 (whichever helps more)
3. **How to Play:**
   - You and the dealer each start with 2 cards. One dealer card is hidden.
   - Choose to **Hit** (take a card), **Stand** (keep your hand), **Double Down** (double your bet, take exactly one more card), **Split** (if you have a matching pair), or **Insurance** (side bet when dealer shows an Ace).
   - After you stand, the dealer reveals their hidden card and draws until they reach 17 or higher.
4. **Winning:**
   - Beat the dealer's total without busting = **you win** (1:1 payout).
   - Get exactly 21 with your first two cards (Ace + 10-value) = **Blackjack!** (3:2 payout).
   - Insurance wins = **2:1 payout**.
   - Dealer busts = **you win**.
   - Tie = **Push** (bet returned).
   - Go over 21 = **Bust** (you lose).
   - Hit $0 = **Possessions Mode** activates.

### Table Options (Pre-Game Settings)

Configurable before clicking "Deal Me In":

| Setting | Options | Default |
|---|---|---|
| Decks | 1 / 2 / 4 / 6 | 1 |
| Double Down | ON / OFF | ON |
| Splitting | ON / OFF | ON |
| Insurance | ON / OFF | ON |
| Card Count | ON / OFF | ON |
| Strategy Hints | ON / OFF | OFF |
| Dealer Tells | ON / OFF | OFF |

- **Card Count:** Displays a running Hi-Lo count in the header.
- **Strategy Hints:** Shows the statistically optimal move (Hit / Stand / Double / Split) during the player's turn.
- **Dealer Tells:** Applies a subtle color tint to the hole card back — warm red for high-value cards (10+), cool blue for low-value cards (2–6).

---

## Game Modes

### Standard Mode
- Player starts with a configurable bankroll (e.g., $500).
- Place bets using chip buttons before each round.
- Play until bankroll is empty or player exits.

### Possessions Mode (Unlocked at $0)
- Triggered automatically when the player's balance hits $0.
- Both the player AND dealer wager valuable personal possessions instead of money.
- **Possessions pool** (alternating or random selection each round):
  - Wristwatch ⌚
  - Leather Wallet 👛
  - Sunglasses 🕶️
  - Gold Ring 💍
  - Car Keys 🔑
  - Silk Tie 🪡
  - Gold Chain 📿
  - Briefcase 💼
  - Overcoat 🧥
- UI shifts to show possession icons/names as "bets"
- Possessions accumulate in a "winnings tray" if the player wins
- Flavor text / narrative lines appear (e.g., *"The dealer slides his Rolex onto the table..."*)
- Game ends when all possessions are depleted or player chooses to walk away

---

## Game States

| State | Description | Active UI Elements |
|---|---|---|
| `idle` | No game in progress | Chip buttons, Deal button |
| `dealing` | Cards being dealt (animated) | None (locked) |
| `player_turn` | Player making decisions | Hit, Stand, Double, Split |
| `dealer_turn` | Dealer revealing/drawing | None (locked) |
| `round_complete` | Win/loss/push resolved | New Round button |
| `bust` | Player or dealer over 21 | New Round button |
| `blackjack` | Natural 21 on deal | New Round button |
| `possessions_mode` | Balance = $0, wagering possessions | Modified bet UI |
| `game_over` | All possessions lost | Restart button |

---

## Blackjack Rules

- **Deck:** Standard 52-card deck, shuffled before each round (or after ~75% penetration for realism)
- **Dealer Rules:** Dealer hits on soft 16 or less, stands on soft 17+
- **Blackjack Payout:** 3:2
- **Double Down:** Allowed on any initial two-card hand
- **Split:** Allowed when both initial cards have equal value; re-splitting allowed up to 3 times
- **Insurance:** Offered when dealer shows an Ace; costs half the current bet; pays 2:1 if dealer has Blackjack (toggleable in Table Options)
- **Surrender:** Not implemented

### Edge Cases
- **Natural Blackjack (player):** Pays 3:2; dealer also checks — if dealer has blackjack too, result is a Push
- **Natural Blackjack (dealer):** Dealer wins unless player also has blackjack (Push)
- **Bust:** Any hand over 21 immediately loses, regardless of the other party's hand
- **Push (Tie):** Exact same total; bet is returned to player, no gain/loss
- **Soft Ace:** Ace counted as 11 until hand would bust, then recounted as 1
- **Split Aces:** Each receives only one additional card
- **Double on Split:** Allowed on non-Ace splits

---

## Animations

- **Card Flip:** 3D CSS flip animation (~0.4s) when a card is revealed
- **Card Deal:** Cards slide in from a "shoe" in the corner (~0.3s per card, staggered)
- **Chip Stack:** Chips animate onto the betting circle when clicked
- **Win:** Cards subtly glow gold; chips animate back to player's stack
- **Bust:** Red flash / shake animation on busted hand
- **Blackjack:** Gold sparkle / confetti burst
- **Dealer Reveal:** Flip animation on the hidden card
- **Balance Update:** Counter ticks up/down smoothly

---

## Audio

### Background Music
- **Style:** Smooth jazz / lounge piano (1950s-60s Vegas feel)
- **Ambience:** Muffled crowd chatter layered underneath
- **Toggle:** Music on/off button in bottom-left corner (persists preference)
- **Implementation:** Loop via Web Audio API or `<audio>` element

### Sound Effects
| Trigger | Sound |
|---|---|
| Hover over any button | Soft click / tick sound |
| Click any button | Distinct click / tap sound |
| Card dealt | Card slide/slap sound |
| Card flipped | Card flip/whoosh sound |
| Chips placed | Chip clink/rattle |
| Player wins | Upbeat chime / coin jingle |
| Player loses | Low thud / sad trombone sting |
| Blackjack | Celebratory fanfare |
| Bust | Dramatic low tone |
| Push | Neutral "ping" |

### Audio Implementation Notes
- All SFX generated via the **Web Audio API** directly (oscillators + noise buffers — no third-party library)
- Card flip sound uses a pre-loaded `.wav` file (`240776__f4ngy__card-flip.wav`) via an `<audio>` element
- Background music uses an `<audio loop>` element; music and SFX run on separate gain nodes so the toggle only affects music
- Hover sounds are subtle and not intrusive (low volume, ~50ms)

---

## Technical Specification

### Stack
- **Framework:** Single HTML file with vanilla JS
- **Styling:** Inline `<style>` block with CSS custom properties; no external CSS framework
- **Audio:** Web Audio API (oscillators + noise) for SFX; `<audio>` elements for card flip WAV and background music
- **State Management:** Plain JS object (`S`) mutated directly; render functions called explicitly

### File Structure
```
blackjack/
├── index.html (or App.jsx)
├── spec.md
└── README.md (optional)
```

### Key Functions / Modules
- `Audio` (IIFE) — Web Audio API engine; exposes `click`, `hover`, `cardDeal`, `cardFlip`, `chip`, `win`, `lose`, `bust`, `blackjack`, `push`, `toggleMusic`
- `renderCards` / `renderBalance` / `renderButtons` / `renderPoss` — DOM update functions
- `deal` / `dealPoss` — dealing sequence with staggered animation
- `hit` / `stand` / `doubleDown` / `split` — player action handlers
- `runDealer` — dealer draw loop (hits until ≥ 17)
- `resolveRound` / `bjResolve` / `resolvePoss` — win/loss/push evaluation
- `getStrategyHint` — basic strategy calculator (pairs → soft hands → hard hands)
- `spawnParticles` — canvas-based confetti for wins / blackjack
- `checkPossMode` — transitions game to Possessions Mode at $0

---

## Testing Scenarios

| Scenario | Expected Result |
|---|---|
| Player hits to 21 | Win (if dealer < 21) or Push |
| Player busts | Immediate loss, no dealer draw needed |
| Dealer busts | Player wins (if player hasn't busted) |
| Player gets Blackjack | 3:2 payout unless dealer also has BJ |
| Both get Blackjack | Push |
| Player doubles, hits 21 | Win with doubled payout |
| Player splits aces | One card each, no further hits |
| Soft ace keeps hand valid | Ace correctly flips from 11 to 1 when needed |
| Balance hits $0 | Possessions Mode activates |
| All possessions lost | Game over screen shown |
| Music toggle | Music stops/starts without affecting SFX |
| Hover over button | Subtle hover sound plays |
| Click button | Click sound plays |
| Dealer shows Ace (insurance ON) | Insurance popup appears |
| Player takes insurance, dealer has BJ | Insurance pays 2:1; hand resolves as dealer BJ |
| Player declines insurance, dealer has BJ | Dealer wins normally |
| Strategy hints ON | Optimal move shown during player turn |
| Card count ON | Running Hi-Lo count displayed in header |
| Dealer tells ON | Hole card back tinted red (high) or blue (low) |

---

## Out of Scope (v1)

- Multiplayer / online features
- Side bets (Perfect Pairs, 21+3, etc.)
- Persistent leaderboard
- Mobile-specific layout adjustments (though should be reasonably responsive)
