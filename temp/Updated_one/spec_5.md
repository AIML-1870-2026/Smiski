# Product Review Generator — spec.md

## Project Overview

A single-page web application that enables users to generate product reviews using OpenAI language models. The app accepts product details and user-controlled parameters (category, style, length, sentiment) and returns a formatted, human-sounding review rendered as HTML.

---

## Reference Implementation

The `temp/` folder contains my complete LLM Switchboard project (HTML, CSS, and JS files). This is NOT part of the current project — do not include it in the final build or deployment.

Use it as a reference for:
- How to parse a .env file for API keys (in-memory only)
- The fetch() call structure for OpenAI's chat completions API
- Error handling patterns for failed API requests
- How the code is organized across separate files
- The general approach to building a single-page LLM tool

Ignore these Switchboard features (not needed here):
- Anthropic integration (this project is OpenAI-only)
- The model selection dropdown / provider switching
- Structured output mode and JSON schema handling

This project uses unstructured (free-form) responses only.
Render the model's markdown output as formatted HTML.

---

## Key Design Constraints

- **OpenAI models only** — no Anthropic integration, no provider switching. Anthropic's API blocks direct browser requests (CORS), so OpenAI is the appropriate choice for a single-file, GitHub Pages deployment.
- **Unstructured responses** — the model returns free-form markdown text, not JSON. No schema templates or JSON parsing required.
- **Markdown rendering** — model responses include markdown formatting (bold, lists, headings). The app renders this as properly formatted HTML using the `marked.js` library loaded from a CDN.
- **API keys loaded from .env** — same in-memory-only pattern as the Switchboard. The user drops or pastes their key at runtime; it is never written to a file, never stored in localStorage, and never committed to the repository.
- **Single-file deployment** — one `index.html` file containing all HTML, CSS, and JavaScript, ready for GitHub Pages.

---

## Features

### Core Features
- Product name, category (dropdown), and optional comments/features input
- LLM Family selector (GPT-4 / GPT-3.5) and specific model selector within that family
- Overall sentiment slider (0–100) controlling the general positivity/negativity of the review
- Length selection: Short / Medium / Detailed
- Style selection: Conversational / Technical / Humorous
- Star rating selector (1–5 stars)
- Markdown-rendered review output
- Copy and Download buttons for the generated review
- About modal explaining the FTC rule and how the sentiment slider works
- Error handling with user-friendly messages for invalid keys, rate limits, and network errors

### Stretch Challenge 1 — Multiple Sentiment Layers
Per-aspect sentiment sliders allowing independent control of:
- **Price / Value** sentiment (Very Negative → Very Positive)
- **Features** sentiment (Very Negative → Very Positive)
- **Usability** sentiment (Very Negative → Very Positive)

Each aspect's sentiment value is passed into the prompt so the model reflects each one distinctly in the generated review, enabling nuanced, mixed reviews that feel realistic.

### Stretch Challenge 2 — Rich UI Components
Replaced dropdown-style controls with interactive sliders:
- **Tone / Style slider** across 6 positions: Formal → Critical → Balanced → Casual → Enthusiastic → Humorous
- **Review Length slider** across 3 positions: Short → Medium → Detailed

Both sliders display a live badge label that updates as the user drags, providing immediate visual feedback.

---

## API Key Handling

The app supports two methods for loading an OpenAI API key at runtime:

1. **Drop a .env file** — drag and drop (or click to select) a `.env` file. The app parses it in-memory using a `FileReader`, extracting `OPENAI_API_KEY=sk-...`. The file is never uploaded anywhere.
2. **Paste directly** — type or paste the key into a password input field.

In both cases the key is stored only in a JavaScript variable for the duration of the browser session. It is never written to disk, never sent to any server other than `api.openai.com`, and is lost when the tab is closed.

This approach satisfies the in-memory-only requirement from the Switchboard and avoids the GitHub secret-scanning issue that invalidates keys committed to public repositories.

---

## Tech Stack

| Component | Choice | Reason |
|-----------|--------|--------|
| Language | Vanilla HTML/CSS/JS | Single-file constraint; no build step needed |
| LLM Provider | OpenAI | CORS-compatible with direct browser requests |
| Markdown rendering | marked.js (CDN) | Lightweight, no install required |
| Fonts | DM Sans, DM Serif Display, DM Mono (Google Fonts) | Matches Switchboard design system |
| Deployment | GitHub Pages | Single `index.html`, no backend required |

---

## File Structure

```
/
├── index.html       ← entire application (HTML + CSS + JS)
├── spec.md          ← this file
└── temp/            ← reference only, NOT deployed
    └── (LLM Switchboard files)
```

---

## Prompt Design

The prompt sent to the OpenAI API incorporates all user-selected parameters:

- Product name, category, and optional details
- Overall sentiment score (0–100) with a human-readable label
- Per-aspect sentiment labels (Price, Features, Usability)
- Star rating, writing style, tone, and length
- Instruction to write in a natural, customer-like voice
- Instruction to start with a bold markdown title
- Instruction to output only the review — no preamble

This approach ensures the model's output is directly shaped by every control the user interacts with.
