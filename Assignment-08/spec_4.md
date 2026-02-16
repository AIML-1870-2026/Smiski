# spec.md — "Should I Treat Myself?" Decision Neuron

## Decision Domain

**Question:** "Should I treat myself today or not?"

**"Yes" means:** Go ahead — get that coffee, snack, new shirt, or whatever treat you've been eyeing. You've earned it.

**"No" means:** Hold off for now. Save your energy, money, or calories for a better moment.

---

## Inputs (4 Factors)

| # | Input Name | Emoji | Range | Weight | Sign | Rationale |
|---|-----------|-------|-------|--------|------|-----------|
| 1 | How Bad the Week Was | 😤 | 0–1 | +0.9 | Positive | Rougher week → more deserving of a treat |
| 2 | Accomplishments This Week | 🏆 | 0–1 | +1.2 | Positive | More accomplishments → earned a reward (strongest positive factor) |
| 3 | Point in the Week | 📅 | 0–1 | +0.6 | Positive | 0 = Monday, 1 = Weekend. Closer to the weekend → more likely to treat |
| 4 | Current Mood | 😊 | 0–1 | -0.7 | Negative | Already feeling great → less *need* for a treat to boost mood |

---

## Bias / Default Tendency

**Name:** Treat-Yourself Energy

**Range:** -3 to +3

**Interpretation:** Represents your natural baseline inclination. A positive bias means you're the kind of person who naturally gravitates toward self-care and treats. A negative bias means you're naturally restrained and need stronger reasons to indulge.

---

## Activation Function

Primary: **Sigmoid** (σ), which squashes the weighted sum into a 0–100% probability.

Additional functions available for comparison (Stretch Challenge 3):
- **Step Function** (1958 Perceptron) — hard binary, no uncertainty
- **ReLU** (Modern deep learning) — linear above zero, clipped below

---

## Decision Boundary Panel

**X Axis (default):** 😤 How Bad the Week Was

**Y Axis (default):** 🏆 Accomplishments This Week

These two were chosen because they're the most interesting to visualize together — they represent the "deserve it because it was hard" vs. "deserve it because I crushed it" tension. The boundary line shows exactly where the neuron flips from "save it" to "treat yourself."

**Visual elements:**
- 2D heatmap colored with the scenario's accent theme
- Gold dashed contour line at the 50% decision threshold
- Gold crosshair dot tracking current slider position
- Dropdown selectors to swap any two inputs onto the axes

---

## Training Mode

**What points represent:** Past decisions or hypothetical scenarios. Each point is a combination of two input values (plotted on X/Y) labeled as either "Yes, I treated myself" or "No, I didn't."

**What makes it interesting:** You can place points that reflect your *actual* past behavior, then watch the neuron learn *your* personal decision pattern. The boundary line visibly rotates and shifts with each training step, showing gradient descent in action.

**Controls:**
- Click to add labeled points (toggle Yes/No)
- **Step** button: one training iteration with visible boundary update
- **Auto** button: continuous training at adjustable speed
- **Reset** button: clear all points, randomize weights
- **Load Preset** button: populate with a sample dataset
- Learning rate slider (0.01–2.0)
- Speed slider for auto-training (20–500ms)
- Live display of: step count, accuracy %, point count, learned weights and bias

---

## Stretch Challenge Features

### Feature 1: Multi-Scenario Neuron
Five preset scenarios, each with unique inputs, weights, emoji, labels, and color theme:
- 🧁 Treat Myself? — pink and purple
- 🎓 Choose This College? — blue and sky blue
- 🐕 Adopt a Pet? — yellow and orange
- 🚗 Road Trip? — green and teal
- 💻 Buy This Tech? — grey and silver

Switching scenarios updates the entire UI: title, sliders, weights, math display, neuron output, and accent colors.

### Feature 2: Decision Boundary Visualizer
Full 2D heatmap rendering with theme-aware gradient colors, gold decision boundary contour, crosshair position tracker, and axis selection dropdowns.

### Feature 3: Activation Function Showdown
Three selectable activation functions (Sigmoid, Step, ReLU) with:
- Live output comparison
- Function curve visualization with current z-value marker
- Compare toggle to overlay all three curves simultaneously
- Math breakdown for each function

### Feature 4: Two-Neuron Chain
Neuron 1 (the treat neuron) feeds its output into Neuron 2 which adds:
- 💰 Budget Available (weight: 0.8)
- 👫 Friend Available (weight: 0.6)
- Adjustable connection weight between neurons
- Neuron 2 bias
- Full chain math display showing z₁ → a₁ → z₂ → final output

### Feature 5: Sensitivity Analysis
- Line chart sweeping each input 0→1 while holding others at current values
- Color-coded curves per input using scenario theme colors
- Dot markers showing current slider positions
- Ranked influence bar chart showing which input swings the output the most

---

## UI/UX Requirements

- **Static dark background** (#111115) that stays consistent across all scenarios
- **Themed accent colors** change per scenario — affecting card top borders, title ombre gradient, slider thumbs, value badges, neuron glow, buttons, tab highlights, and chart colors
- **Title ombre effect** using CSS gradient text fill from accent → accent2
- **Responsive layout** with 2-column grid collapsing to single column on mobile
- **Smooth transitions** (0.3–0.6s) on theme changes, neuron state updates, and tab switches
- **Canvas-based visualizations** for sigmoid curve, boundary heatmap, activation curves, and sensitivity charts
- Fonts: Bricolage Grotesque (display), DM Sans (body), DM Mono (code/math)

---

## Tech Stack

Single HTML file with embedded CSS and JavaScript. No build step, no dependencies, no frameworks — just vanilla HTML/CSS/JS with Google Fonts CDN. Designed for direct deployment to GitHub Pages.
