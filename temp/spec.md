# spec.md — Product Review Generator

## Project Overview

A single-page web application that lets users generate AI-powered product reviews using OpenAI's API. Users configure product details, review parameters, and sentiment, then receive a formatted review rendered as HTML.

## Tech Stack

- **Frontend**: Single `index.html` file (HTML + CSS + vanilla JS)
- **Deployment**: GitHub Pages (no server required)
- **API**: OpenAI only (direct browser-to-API calls)
- **API Key Handling**: Loaded from a `.env` pattern — user pastes key into an in-memory input field at runtime; nothing is persisted or stored

## Key Design Constraints

- **OpenAI models only** — no Anthropic/Claude dropdown, no provider switching (CORS limitation)
- **Unstructured responses** — model returns free-form text, rendered as Markdown → HTML
- **Markdown rendering** — use `marked.js` (CDN) to parse and render the model's response
- **Single-file deployment** — everything lives in `index.html`
- **No backend** — all logic runs in the browser

## UI Sections

### 1. Header
- App title: "Product Review Generator"
- Minimal top bar

### 2. API Key Input
- Text input for OpenAI API key (type="password")
- In-memory only — never stored in localStorage or cookies
- Small label clarifying "Key is never saved"

### 3. Basic Product Information
- **Product Name** — text input (placeholder: "e.g., Wireless Headphones")
- **Category** — dropdown: Electronics, Clothing, Food & Beverage, Home & Garden, Sports & Outdoors, Beauty & Personal Care, Books, Toys & Games, Automotive, Other
- **Comments** — textarea for key features, strengths, weaknesses, specific experiences, or advanced instructions

### 4. LLM and Sentiment Selection
- **Model** — dropdown of OpenAI models (fetched dynamically from OpenAI `/v1/models` endpoint, filtered to GPT chat models, sorted with newest first)
- **Sentiment slider** — range 0–100
  - 0–33: Negative 😤
  - 34–66: Neutral 😐
  - 67–100: Positive 😊
  - Color gradient: red → yellow → green

### 5. Stretch Challenge Features

#### A. Multiple Sentiment Layers
- Three aspect-specific sentiment sliders:
  - **Price** 💰
  - **Features** ⚙️
  - **Usability** 🖱️
- Each slider is 0–100 with emoji + color feedback
- These replace/supplement the overall sentiment slider
- All three aspects are passed into the prompt

#### B. Rich UI Controls for Tone & Length
- **Length** — slider (not dropdown): Short / Medium / Long (3 stops, labeled)
- **Style/Tone** — slider (not dropdown): Casual / Balanced / Formal (3 stops, labeled)

### 6. Generated Review Output
- Renders markdown response as formatted HTML
- Shows "No review generated yet." placeholder until first generation
- **Download Review** button — exports review as a `.txt` file
- Loading state while waiting for API response

## Prompt Construction

Build the system + user prompt dynamically:

```
System: You are a helpful assistant that writes realistic product reviews.

User:
Write a [LENGTH] product review for "[PRODUCT NAME]" in the [CATEGORY] category.
Tone/Style: [STYLE]
Overall Sentiment: [SENTIMENT LABEL] ([SCORE]/100)
Price Sentiment: [PRICE_SCORE]/100
Features Sentiment: [FEATURES_SCORE]/100
Usability Sentiment: [USABILITY_SCORE]/100

Additional context: [COMMENTS]

Return only the review text, formatted with markdown (bold key points, use paragraph breaks). Do not include a title or preamble.
```

## Aesthetic Direction

**Tech Minimalism** — the most boring, competent, no-nonsense design possible. Think: internal tooling at a company that doesn't care about design but cares deeply about function.

- **Colors**: Near-white background (`#f8f8f8`), dark gray text (`#1a1a1a`), single blue accent (`#2563eb`) for interactive elements only
- **Font**: `JetBrains Mono` for everything — monospaced, zero personality, maximally "engineer brain"
- **No shadows**, no gradients, no rounded corners (or minimal: 2px max)
- **Borders**: 1px solid `#d1d1d1` — visible, utilitarian
- **Spacing**: Consistent 8px grid, generous padding — not tight, just functional
- **No animations** except a subtle spinner on load
- **Sliders**: Styled minimally — track + thumb, no flourishes
- Labels are uppercase, small, letter-spaced — like a config file

## Error Handling

- Invalid/missing API key → clear inline error message
- Network failure → display error in the review output area
- Empty product name → prevent submission, show inline validation

## File Structure

```
/
├── index.html       ← entire app (HTML + CSS + JS)
└── spec.md          ← this file
```

## Notes for Claude Code

- Do not use any CSS frameworks (no Tailwind, no Bootstrap)
- Do not use React or any JS framework — vanilla JS only
- Use `marked.js` from CDN for markdown rendering
- Fetch OpenAI models dynamically using the key provided by the user
- Filter model list to only include models whose ID starts with `gpt` and supports chat completions
- The `temp/` folder (if present) contains reference Switchboard code — use it to understand the fetch() patterns, .env key loading approach, and error handling style, then adapt for OpenAI

---

## Reference Implementation

The `temp/` folder contains the LLM Switchboard project. **Do not include it in the build or deploy.** Use it as a pattern reference only.

### API Key Loading

Two input methods — file upload and paste — both store the key in memory only, never in localStorage or cookies.

**File upload (`.env` / `.csv` / `.txt`):**
```js
function parseKeyFile(file) {
  const reader = new FileReader();
  reader.onload = (e) => {
    const text = e.target.result;
    const envOAI = text.match(/OPENAI_API_KEY\s*=\s*([^\s\n]+)/i);
    if (envOAI) { state.keys.openai = envOAI[1].trim(); }
    // Fallback: scan line-by-line for bare sk- tokens
    if (!envOAI) {
      text.split('\n').forEach(line => line.split(',').forEach(p => {
        const val = p.trim().replace(/['"]/g, '');
        if (val.startsWith('sk-') && !val.startsWith('sk-ant-')) state.keys.openai = val;
      }));
    }
  };
  reader.readAsText(file);
}
```

**Paste input:**
```js
function handlePasteKey() {
  const val = document.getElementById('api-key-input').value.trim();
  if (val) state.keys.openai = val;
}
```

### OpenAI fetch() Call

```js
async function callOpenAI(key, model, systemPrompt, userPrompt) {
  const res = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${key}`
    },
    body: JSON.stringify({
      model,
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user',   content: userPrompt }
      ],
      max_tokens: 1024
    })
  });
  if (!res.ok) {
    const e = await res.json().catch(() => ({}));
    throw new Error(`${res.status}: ${e?.error?.message || res.statusText}`);
  }
  const data = await res.json();
  return {
    text:   data.choices[0].message.content,
    tokens: data.usage?.total_tokens || null
  };
}
```

### Error Handling

Map raw errors to friendly inline messages (display in the review output area, not as `alert()`):

```js
function friendlyError(err) {
  const msg = err.message || '';
  if (msg.includes('Failed to fetch') || msg.includes('NetworkError'))
    return '🚫 Network Error: Could not reach the API. Check your connection and API key.';
  if (msg.includes('401') || msg.includes('Unauthorized'))
    return '🔑 Invalid API Key: The key was rejected. Double-check it and try again.';
  if (msg.includes('429'))
    return '⏱️ Rate Limited: Too many requests. Wait a moment and try again.';
  if (msg.includes('500') || msg.includes('503'))
    return '🔧 Server Error: The API is having issues. Try again in a moment.';
  return `❌ Error: ${msg}`;
}
```

### Loading / Button State Pattern

```js
async function generateReview() {
  const btn = document.getElementById('generate-btn');
  btn.disabled = true;
  btn.classList.add('loading');
  try {
    const { text } = await callOpenAI(/* ... */);
    renderReview(text);
  } catch (err) {
    renderReview(friendlyError(err), /* isError= */ true);
  } finally {
    btn.disabled = false;
    btn.classList.remove('loading');
  }
}
```

### Markdown Rendering

Use `marked.js` from CDN. Render the model's response as HTML inside the output area:

```html
<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
```

```js
// Unstructured (free-form) responses only — no JSON parsing
outputEl.innerHTML = marked.parse(text);
```

### Ignored Switchboard Features

These exist in the reference but are **not needed** for this project:
- Anthropic integration / provider switching
- Model selection dropdown (this project fetches and filters models dynamically)
- Structured output mode / JSON schema handling
- Prompt Library / metrics bar / side-by-side layout
