# LLM Switchboard — spec.md

A single-page web application (`index.html`) that lets users interact with large language models from OpenAI and Anthropic through their APIs. Built for a class assignment on AI API integration.

---

## Supported Providers & Models

### OpenAI
- GPT-4o
- GPT-4o Mini
- GPT-4 Turbo
- GPT-3.5 Turbo

### Anthropic
- Claude Opus 4.5
- Claude Sonnet 4.5
- Claude Haiku 4.5

> **Note on Anthropic & CORS:** Anthropic's API does not allow direct browser requests due to CORS (Cross-Origin Resource Sharing) security policy. The app handles this gracefully by showing a clear, educational explanation when Anthropic is selected. A real production app would route Anthropic requests through a backend proxy server. This limitation is intentional and serves as a teaching moment.

---

## API Key Handling

- **File upload** — accepts `.env` or `.csv` files. Parses `OPENAI_API_KEY` and `ANTHROPIC_API_KEY` variables automatically.
- **Paste/manual entry** — two password fields, one per provider.
- Keys are stored **in memory only** (JavaScript variables). They are never written to disk, localStorage, or transmitted anywhere except directly to the respective model API.
- A privacy notice is displayed to the user confirming this guarantee.
- Status indicators confirm which keys have been loaded successfully.

---

## Output Modes

### Unstructured (Free Text)
The model responds with natural language — paragraphs, explanations, stories. The response is displayed as readable text. This is the default mode.

### Structured (JSON Schema)
The model is prompted to respond only with valid JSON matching a user-defined schema. The app includes a schema editor with pre-built templates:
- 🌿 Species Profile
- 🌍 Ecosystem Summary
- ⚡ Natural Phenomenon
- 💬 Sentiment Analysis
- 📋 Structured Summary

JSON responses are displayed with syntax highlighting (keys, strings, numbers, booleans, and nulls each have distinct colors).

---

## Example Prompts

Pre-loaded prompts users can select to auto-fill the prompt box:

### Science & Nature (themed)
- Wolf reintroduction in Yellowstone
- Mycelium network communication
- Deep-sea bioluminescence
- Monarch butterfly navigation

### General AI Demos
- Explain ML to a 12-year-old
- Pros & cons of renewable energy
- History of the internet (3 sentences)
- Haiku about artificial intelligence

---

## Prompt Library (Design Feature)

Users can save any prompt to an in-memory library using the 💾 Save button. Saved prompts appear in a dropdown and can be reloaded at any time. The library persists for the duration of the session (cleared on page refresh). This turns the Switchboard into a personal prompt workbench.

---

## 🔥 Stretch Challenge 1: Prompt Library

A collapsible prompt library panel lives in the sidebar, letting users save, name, reload, and delete prompts during their session.

**How it works:**
- Type any prompt in the prompt box
- Optionally give it a custom name in the name field; if left blank, the first 48 characters of the prompt are used as the name
- Click **💾 Save** — the prompt is added to the library instantly
- The panel shows a live count badge of saved prompts
- Click any saved prompt name to load it back into the prompt box
- Click **×** to delete individual prompts
- The panel is collapsible (click the header to open/close)

All library data is stored in memory only — it clears when the page is refreshed. This turns the Switchboard into a personal prompt workbench for the session.

---

## 🔥 Stretch Challenge 2: Response Metrics Dashboard

A persistent metrics bar appears at the top of the page after the first successful response, showing:
- **Last Response Time** — how long the most recent request took (seconds)
- **Tokens Used** — total tokens consumed (input + output, from the API response)
- **Response Length** — character count of the response text
- **Total Requests** — cumulative number of prompts sent this session
- **Average Response Time** — rolling average across all requests

Token counts are pulled directly from the API response (`usage.total_tokens` for OpenAI, `usage.input_tokens + usage.output_tokens` for Anthropic). Displayed as "N/A" if the API does not return token data.

---

## Long Response Display (Design Feature)

Responses longer than ~320px are automatically collapsed with a fade overlay and a "Show full response" button. Clicking expands the card to show the full response. A "Collapse" button returns it to the compact view. This keeps the UI clean when responses are very long.

---

## Error Handling

All errors are caught and displayed as styled error cards (red background) with friendly human-readable messages:

| Error Type | Message Shown |
|---|---|
| CORS (Anthropic) | Explains the browser security restriction and what a proxy would do |
| Network failure | Suggests checking connection and API key |
| 401 Unauthorized | Tells user their key was rejected |
| 429 Rate Limited | Tells user to wait and retry |
| 500 / 503 Server Error | Tells user the API is having issues |

---

## UI Layout

- **Header** — sticky top bar with logo and subtitle
- **Metrics Bar** — appears below header after first request; shows session stats
- **Sidebar (left, 360px)** — all controls: keys, provider, model, comparison toggle, output mode, schema editor, prompt input, prompt library, send button
- **Response Panel (right, flex)** — scrollable area where response cards appear, newest on top

---

## Technical Details

- **Single file:** `index.html` — no build tools, no dependencies, no frameworks
- **Fonts:** DM Serif Display (headings), DM Sans (body), DM Mono (code/JSON) — loaded from Google Fonts
- **No localStorage** — all state is in JavaScript memory only
- **Responsive** — collapses to single-column on screens under 900px wide
- **Deployment:** GitHub Pages (static file hosting — just push `index.html` to your repo's `main` branch and enable Pages in Settings)

---

## Deployment Steps (GitHub Pages)

1. Open VS Code in your project folder
2. Initialize git if needed: `git init`
3. Add your file: `git add index.html`
4. Commit: `git commit -m "Add LLM Switchboard"`
5. Push to GitHub: `git push origin main`
6. Go to your repo on GitHub → Settings → Pages → Source: `main` branch → Save
7. Your live URL will be: `https://YOUR_USERNAME.github.io/YOUR_REPO_NAME/`
8. Submit that URL to Canvas
