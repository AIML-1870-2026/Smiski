# Readability Explorer — Project Specification

## Overview
A single-page web application that allows users to explore the readability of various text and background color combinations on digital screens. The tool helps content creators, designers, and developers make accessible color choices that meet WCAG (Web Content Accessibility Guidelines) standards.

---

## Application Structure

### Tabs
The application is divided into two primary tabs:

1. **Readability** — Main customization and preview tab
2. **Vision Type Simulation** — Color blindness simulation tab

---

## Tab 1: Readability

### Layout
- **Left Panel**: WCAG Compliance Indicator (fixed sidebar)
- **Center/Right Area**: Text preview, color controls, text size controls, and preset color schemes

---

### Left Panel — WCAG Compliance Indicator

**Title**: "WCAG Compliance"

**Displays**:
- Current contrast ratio formatted as `X.XX:1`
- Pass/Fail status for **Normal Text** (threshold: 4.5:1)
- Pass/Fail status for **Large Text** (threshold: 3:1)

**Color Coding**:
- **Green** (`#2E7D32` or equivalent accessible green) = PASS
- **Red** (`#C62828` or equivalent accessible red) = FAIL

**Labels**: Both visual color cues and text labels (e.g., "✓ Pass" / "✗ Fail") for accessibility.

**Recalculation**: Updates automatically whenever background color or text color changes.

---

### Text Display Area

- A styled text box in which users can type or paste any article or excerpt
- **Default/Sample Text**: A short paragraph explaining that the user can paste their own text here, and describing the purpose of the tool (e.g., "This is sample text. Paste your own article or excerpt here to explore how different color and size combinations affect readability on screen.")
- The text area reflects the currently selected background color, text color, and text size in real time

---

### Background Color Controls

Three sliders (one per channel) + corresponding integer input fields:
- **Red** (0–255)
- **Green** (0–255)
- **Blue** (0–255)

**Behavior**:
- Moving a slider updates the integer field immediately
- Editing the integer field updates the slider immediately
- Background color of the text display area updates in real time
- Contrast ratio recalculates automatically

**Luminosity Display**: Shows the computed relative luminance value of the background color (0.000–1.000, per WCAG formula), updated in real time.

---

### Text Color Controls

Three sliders (one per channel) + corresponding integer input fields:
- **Red** (0–255)
- **Green** (0–255)
- **Blue** (0–255)

**Behavior**: Same bidirectional sync as background color controls.

**Luminosity Display**: Shows the computed relative luminance value of the text color, updated in real time.

---

### Text Size Controls

- A single slider controlling font size
- Range: **8px – 72px**
- Corresponding integer input field (bidirectional sync)
- A note clarifying that text ≥ 18pt (24px) or bold text ≥ 14pt (approximately 18.67px) qualifies as "large text" under WCAG

---

### Contrast Ratio Display

- Shown prominently in or near the WCAG Compliance panel
- Format: `X.XX:1`
- **Calculation (per WCAG 2.1)**:
  1. Convert each RGB component to a linear value: `c / 255`; if `c_lin ≤ 0.04045` then `c_lin / 12.92`, else `((c_lin + 0.055) / 1.055) ^ 2.4`
  2. Relative luminance: `L = 0.2126 * R + 0.7152 * G + 0.0722 * B`
  3. Contrast ratio: `(L1 + 0.05) / (L2 + 0.05)` where `L1` ≥ `L2`
- Updates automatically on any color change

---

### Preset Color Schemes Box

A panel featuring clickable preset buttons organized into categories:

| Category | Examples |
|---|---|
| **High Contrast** | Black on White (#000 / #FFF), White on Black (#FFF / #000) |
| **Low Contrast** | Light gray on white, yellow on white |
| **Common Website Schemes** | Dark navy on white, charcoal on light gray, teal on off-white |
| **Problematic (Fail WCAG)** | Red on green, blue on purple, yellow on white |

Each preset button:
- Updates background color, text color sliders/fields immediately
- Triggers contrast ratio recalculation
- Is labeled clearly with the scheme name

---

## Tab 2: Vision Type Simulation

### Purpose
Simulates how the text preview area and the overall page color scheme would appear to individuals with different types of color vision deficiencies.

### Vision Mode Radio Buttons
Five mutually exclusive options:
1. **Normal** (default)
2. **Protanopia** — Red-blind
3. **Deuteranopia** — Green-blind
4. **Tritanopia** — Blue-blind
5. **Monochromacy** — Complete color blindness (grayscale)

### Behavior
- Selecting a non-Normal vision mode applies a CSS filter or color transformation matrix to the text preview (and ideally the full page UI) to simulate the selected condition
- **Color adjustment controls (sliders and integer fields) are disabled** unless "Normal" vision is selected, to prevent confusing interactions under simulated vision modes
- When a non-Normal mode is active, the UI displays a brief description of the condition

### Color Vision Simulation Matrices (SVG feColorMatrix)
- **Protanopia**: Reduces red channel sensitivity
- **Deuteranopia**: Reduces green channel sensitivity
- **Tritanopia**: Reduces blue channel sensitivity
- **Monochromacy**: Converts all colors to luminance-based grayscale

---

## Accessibility & Theme

### Overall Design Principles
- The application's own color scheme must pass WCAG AA contrast standards
- UI chrome (buttons, labels, panels) must be readable under all five simulated vision modes
- Neutral, high-contrast base palette (e.g., white/light gray UI with dark text) that remains distinguishable across all color blindness types
- No color-only information encoding — always paired with text labels or icons

### Typography
- UI labels: clean, readable sans-serif
- Text preview area: inherits user-selected font size; default body-style font

---

## Technical Notes

- **Framework**: Vanilla HTML/CSS/JavaScript (single `.html` file)
- **No external dependencies** required (optional: Google Fonts for UI typography)
- All calculations performed client-side in real time
- Bidirectional slider ↔ integer field sync via `input` event listeners
- Color vision simulation via SVG `feColorMatrix` filter applied as a CSS `filter` or injected SVG element

---

## File Deliverables

| File | Description |
|---|---|
| `readability-explorer.html` | Complete single-file application |
| `spec.md` | This specification document |
