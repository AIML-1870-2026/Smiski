# Blackjack AI Agent — Project Specification

## Overview

A fully static single-page application (HTML + CSS + JS, no build step, no backend) that plays Blackjack and integrates an LLM as an AI agent advisor. The user uploads a `.env` file containing their Anthropic API key; the key is parsed in-memory only and never stored or sent anywhere except the API call itself.

---

## Reference Implementation

A `temp/` folder lives in the project repository. It contains the existing **LLM Switchboard** webpage — a working static page that already handles `.env` upload, Anthropic API calls via `fetch()`, and error handling. Claude Code must use it as a pattern reference for:

- `.env` file parsing (FileReader → in-memory string split, never written to disk/localStorage)
- `fetch()` call structure to `https://api.anthropic.com/v1/messages`
- Required headers: `Content-Type`, `x-api-key`, `anthropic-version`
- Response extraction: `data.content[0].text`
- Error handling: HTTP status codes, CORS notes, user-friendly messages

> ⚠️ The `temp/` folder must **not** be included in the final build or deployment.

---

## Screen Flow

Three screens in sequence:
1. **Splash** — Full-screen felt background with giant "BLACKJACK" title (pulsing gold), suit row (♠ ♥ ♦ ♣), subtitle "with AI Agent", and a gold "Let's Begin" button
2. **Settings/Instructions** — Rules grid, AI key upload, table options, "DEAL ME IN" button
3. **Game** — Full game layout with AI panel

---

## Visual Design

The new page must **copy the visual design of the existing Blackjack game** (`index.html`) in its entirety, including:

- CSS variables: `--felt`, `--felt-dark`, `--felt-light`, `--wood`, `--gold`, `--gold-light`, `--cream`, etc.
- Fonts: `Courier Prime` (monospace body) + `Playfair Display` (serif headings/titles)
- Card rendering (PNG-based with flip animations, backface, `.show` class)
- Hand zones with radial felt gradient backgrounds
- Chip UI and bet panel
- Status bar with `.s-win`, `.s-lose`, `.s-push`, `.s-bj` state classes
- Particle canvas overlay for win celebrations
- Splash title screen → settings/instructions screen → game screen (three-screen flow)
- All action buttons (HIT, STAND, DOUBLE, SPLIT, INSURANCE) with existing styling
- Music button, card-count display, hint bar
- Game-over overlay
- All existing animations (`dealIn`, `flipReveal`, `shake`, `glow-gold`, `pulse`)
- Wood-grain body texture via `repeating-linear-gradient`

---

## New: AI Agent Panel

Placed to the **right of the game area** (side panel, mirroring the professor's example layout). This panel contains:

### Header
- Title: `🤖 AI ANALYSIS`

### API Key Upload
- Drag-and-drop zone for `.env` file (same UX pattern as Switchboard)
- Status indicator: key loaded ✓ / not loaded

### AI Analysis Display
- After each deal, the AI panel shows the full LLM reasoning text
- Rendered clearly with appropriate typography

### Recommendation Bar (bottom of game area)
- `AI recommends: [ACTION]` label
- `EXECUTE RECOMMENDATION` button — applies the AI's action automatically

### Thinking Indicator
- Spinner or animated "Consulting AI…" state while the API call is in flight

---

## AI Agent Behavior

### Prompt Engineering
The system prompt instructs the LLM to respond **only** with a structured JSON object — never plain text. This avoids keyword-search ambiguity (e.g., a response mentioning both "hit" and "stand" in context).

**Required JSON response schema:**
```json
{
  "action": "hit" | "stand" | "double" | "split" | "insurance",
  "confidence": 0.0–1.0,
  "reasoning": "string — full explanation",
  "basic_strategy": "string — what basic strategy says",
  "risk_assessment": "low" | "medium" | "high"
}
```

### User prompt sent to LLM includes:
- Player hand (cards + score)
- Dealer up card
- Current bet and balance
- Available actions
- Risk tolerance setting (stretch feature)
- Explanation detail level (stretch feature)

### Console logging
Key interactions are logged:
- `[AI] Sending game state to LLM...`
- `[AI] Raw response: ...`
- `[AI] Parsed action: hit/stand/...`
- `[AI] Error: ...`

---

## Core Game Requirements

All game logic from the existing `index.html` is preserved:

| Feature | Status |
|---|---|
| Multi-deck shoe (1 / 2 / 4 / 6 decks) | ✅ existing |
| Dealing, scoring (soft aces) | ✅ existing |
| Hit / Stand / Double / Split / Insurance | ✅ existing |
| Balance tracking across hands | ✅ existing |
| Blackjack 3:2 payout | ✅ existing |
| Insurance 2:1 | ✅ existing |
| Card counting display | ✅ existing |
| Strategy hints | ✅ existing |
| Dealer tells | ✅ existing |
| Particle win effect | ✅ existing |
| Game over / possessions mode | ✅ existing |

---

## Stretch Challenges (All 3 Required)

### 1. Strategy Visualization
A visual **decision matrix** displayed in the AI panel alongside the recommendation. It shows:
- A color-coded grid of player hand total (8–21) vs. dealer up card (2–A)
- Current hand highlighted
- Color coding: green = stand, red = hit, yellow = double, blue = split
- Updates each hand to show the "correct" basic strategy move for the current situation

### 2. Performance Analytics
A collapsible **stats panel** tracking across all hands in the session:
- Win / Loss / Push counts and percentages
- Bankroll chart (sparkline of balance over time)
- AI decision quality: % of times player followed AI, % of those that won
- Running totals updated after each hand resolution

### 3. Explainability Controls
Three toggle modes for AI explanation depth, selectable before or during play:
- **Basic** — one-sentence recommendation only
- **Statistical** — recommendation + probability breakdown and card counting context
- **In-Depth** — full reasoning chain, why other options were rejected, risk narrative

Selecting the mode changes the system prompt sent to the LLM and the AI panel display style.

---

## Bonus Stretch (if time permits)

### 4. Risk Tolerance Settings
A slider or toggle in the settings panel:
- **Conservative** — the AI prefers standing and minimizing risk
- **Balanced** — follows basic strategy strictly
- **Aggressive** — the AI recommends higher-risk plays (double, split more often)

Affects the system prompt tone and is surfaced in the LLM reasoning.

---

## File Structure

```
/
├── index.html          ← Main deliverable (all HTML/CSS/JS in one file)
├── temp/               ← Reference implementation (Switchboard) — NOT deployed
│   └── index.html
└── spec.md             ← This file
```

---

## Technical Constraints

- **Static only** — no Node, no server, no build tools
- **Single file** — all CSS and JS inlined in `index.html`
- **In-memory API key only** — never `localStorage`, never sent to any server other than `api.anthropic.com`
- **Structured JSON responses** — never keyword-search the LLM reply
- **Console logging** — all AI interactions logged for debugging
- **No `temp/` in production** — reference folder excluded from deployment

---

## Testing Checklist

- [ ] `.env` upload parses key correctly
- [ ] API call fires after each deal
- [ ] JSON response is parsed (not keyword-searched)
- [ ] Execute Recommendation takes the correct action
- [ ] Balance updates correctly every hand
- [ ] All 3 stretch features work
- [ ] Console logs show AI interactions
- [ ] Game plays correctly without AI (key not uploaded)
- [ ] Error states handled gracefully (bad key, network fail, malformed JSON)
