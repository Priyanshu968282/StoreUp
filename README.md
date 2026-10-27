# 🛒 Apna Bazaar — Your shop. Your voice. Online.


_Built with ❤️ for the people of India· Powered by Google Gemini Live API_

**Turn any local shop into a live online store just by walking through it and talking.**

A shopkeeper points their phone at the shelf and speaks in their own language.  
Apna Bazaar *sees* the products through the camera, *hears* the shopkeeper live,  
confirms each item conversationally, and publishes a valid **ONDC** catalog that  
buyers across India can shop — **no typing, no forms, no English required.**

---

## 🎯 The Problem

**India has 63 million kirana stores. Less than 1% sell online.**

Not because they don't want to — but because every platform forces them to:
- ❌ Type in English
- ❌ Navigate complex seller dashboards
- ❌ Photograph every product manually
- ❌ Fill 50+ product forms

**Apna Bazaar removes every barrier.** The shopkeeper just talks and shows their shelf. That's it.

---

## 🚀 Why This Isn't "a Chatbot with a Mic"

This is architecturally impossible as a text chatbot:

| Capability | How |
|---|---|
| **Sees + hears simultaneously** | Camera feed (1fps JPEG) + microphone (PCM 16kHz) stream to Gemini Live in parallel |
| **Proactive vision** | AI points out products you *didn't* mention — "I also see Parle-G behind those" |
| **Interruptible real-time** | Talk over it, correct mid-sentence — it adapts instantly |
| **Multilingual** | Hindi, Kannada, Tamil, Telugu, English — switches the moment you switch |
| **Agentic function calling** | AI decides *when* to call `add_product()` — shopkeeper never touches a button |
| **Market price intelligence** | Google Search grounding checks if stated prices seem off |

---

## ⚙️ Architecture

```
Phone (React PWA)                 FastAPI backend                Google AI
─────────────────                 ───────────────                ─────────
 mic (PCM 16k)  ───▶  WebSocket ───▶  Gemini Live bridge  ───▶  Gemini Live API
 camera (1 fps) ───▶               ◀── audio replies ◀───────   (audio+video in,
                                                                  audio out,
                                                                  function calling)
                       function calls: add_product / describe / publish
                                  │
                                  ├─▶  In-memory product store
                                  ├─▶  Beckn v1.2.0 catalog builder (ONDC RET10)
                                  └─▶  Nano Banana 2 Lite ── product images
                                  │
 Buyer app (mock ONDC)  ◀── /catalog ── published store snapshot
```

### End-to-End Flow

1. **Scan & speak** — shopkeeper walks the shelf; audio + 1fps video stream to Gemini Live over WebSocket.
2. **Converse** — the model confirms each product's name, price and quantity out loud, then calls `add_product()`.
3. **Publish** — on confirmation it names the store, generates a description, and builds a **valid ONDC Beckn `on_search` catalog** (RET10 / Grocery).
4. **Product images** — **Nano Banana 2 Lite** generates catalog images in the background (cached, with emoji fallback).
5. **Buyer app** — a mock ONDC buyer app reads the published catalog and shows the live storefront, searchable by product.

---

## 🧠 Google AI Stack

| Layer | Model / API | Purpose |
|---|---|---|
| **Live conversation** | `gemini-3.1-flash-live-preview` (Gemini Live API) | Simultaneous audio + video streaming, function calling, interruptible dialogue |
| **Product images** | `gemini-3.1-flash-lite-image` (Nano Banana 2 Lite) | Background e-commerce product photo generation, cached |
| **Market price checks** | Google Search grounding (in-session) | Real-time price intelligence during shelf scan |
| **Catalog generation** | Managed Agent → `gemini-3.5-flash` → deterministic | Three-tier fallback Beckn v1.2.0 catalog builder |
| **Context management** | `ContextWindowCompressionConfig` (50K tokens) | Long scanning sessions don't overflow |

> **Key differentiator:** Function calling, Google Search grounding, and audio+video streaming all run **inside a single Gemini Live session**. Not separate API calls. Not a pipeline. One continuous multimodal conversation.

---

## 🛡️ Failure Recovery (Rubric Requirement)

| Scenario | How Apna Bazaar Handles It |
|---|---|
| **AI miscounts products** | Asks "How many do you have?" instead of guessing — never invents a count |
| **AI misidentifies a product** | Accepts correction instantly — "Daily Milk" → "Dairy Milk", no arguing |
| **Duplicate function calls** | 3-second debounce with call-signature deduplication |
| **Product added without consent** | 4-step confirm: Recognize → Price → Quantity → Verbal "Yes" before `add_product()` |
| **Beckn catalog fails** | Three-level fallback: Managed Agent → model self-correct (2x) → deterministic builder |
| **Invalid catalog fields** | 315-line schema validator checks every ONDC-required field before publish |
| **Image generation fails** | Emoji fallback per category — store is still fully functional |
| **Media during tool calls** | `_tool_call_active` flag pauses audio/video during function execution |
| **Manual override** | Review screen lets shopkeeper edit price/quantity or remove products before publish |

---

## 🏪 Agentic Commerce — Track 01

Apna Bazaar doesn't just digitize a store — it **creates a new revenue stream**.

```
Before Apna Bazaar:  Shopkeeper → Physical store only → Walk-in customers only
After Apna Bazaar:   Shopkeeper → Talks 2 min → ONDC catalog → Discoverable by:
                     → Every ONDC buyer app (Paytm, Magicpin, Mystore)
                     → Every AI shopping agent that speaks Beckn protocol
                     → Agent-to-agent commerce (machine-readable catalog)
```

**63M kirana stores × ₹2,000 avg monthly GMV = the TAM that's currently zero on digital.**

---

## 🏃 Run Locally

**Prerequisites:** Python 3.11+, Node 18+, a Gemini API key with billing enabled.

### 1. Configure

```bash
cp .env.example .env
# Edit .env and set GEMINI_API_KEY=your_key_here
```

### 2. Backend (FastAPI, port 8000)

```bash
cd backend
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS/Linux
python -m pip install -r requirements.txt
python run.py
```

### 3. Frontend (Vite + React, port 5173)

```bash
cd frontend
npm install
npm run dev
```

### 4. Open in browser

| URL | What |
|---|---|
| `http://localhost:5173/` | Shopkeeper onboarding (camera + mic) |
| `http://localhost:5173/buyer` | Mock ONDC buyer app |
| `http://localhost:5173/?debug=1` | Debug panel for judges |

> **📱 Phone access:** Use the Network URL printed by Vite (same WiFi).  
> Camera + mic need HTTPS — whitelist the LAN IP in `chrome://flags/#unsafely-treat-insecure-origin-as-secure`.

---

## 📊 What's Real vs. Mocked (Honest Scope)

| Component | Status |
|---|---|
| Live audio + video streaming to Gemini Live | ✅ Real |
| Multilingual conversation + product recognition | ✅ Real |
| Function calling (add/describe/publish) | ✅ Real |
| Google Search price grounding | ✅ Real |
| Nano Banana 2 Lite product images | ✅ Real |
| ONDC Beckn v1.2.0 catalog + 315-line validator | ✅ Real |
| Buyer app | ⚡ Mock (our storefront, not live ONDC gateway) |
| ONDC network registration | ⚡ Mock (exact payload generated, not registered) |
| Store persistence | ⚡ Mock (in-memory, single session) |
| FSSAI / EAN / GPS | ⚡ Mock (placeholder values, labeled in Debug Panel) |

---

## ⚙️ Configuration

| Variable | Purpose | Default |
|---|---|---|
| `GEMINI_API_KEY` | Gemini API key (never commit real key) | — |
| `LIVE_MODEL` | Live audio+video model | `gemini-3.1-flash-live-preview` |
| `IMAGE_MODEL` | Nano Banana 2 Lite | `gemini-3.1-flash-lite-image` |
| `ENABLE_PRODUCT_IMAGES` | Toggle image generation | `true` |
| `BECKN_MODE` | `deterministic` (demo-safe) or `auto` (agent path) | `deterministic` |
| `AGENT_ID` / `AGENT_MODEL` | Managed Agent + fallback for Beckn JSON | see `.env.example` |

---

## 📁 Project Structure

```
backend/
  app/
    config.py              # Models, ONDC constants, system prompt
    live_session.py        # Gemini Live ↔ browser WebSocket bridge
    function_handlers.py   # add_product / generate_store_description / publish_store
    product_store.py       # In-memory inventory + published snapshot
    beckn_builder.py       # Deterministic ONDC Beckn v1.2.0 catalog + 315-line validator
    agent_trigger.py       # Managed agent / self-correct / deterministic fallback
    image_gen.py           # Nano Banana 2 Lite product images
    main.py                # FastAPI app + endpoints
  run.py                   # Dev entrypoint
frontend/
  src/
    components/            # Home, Scanning, Review, Success, BuyerAppMock, DebugPanel
    hooks/                 # useLiveSession, useAudio, useCamera
    styles.css             # Warm clay/terracotta theme
  public/fonts/            # Self-hosted Montserrat + Plus Jakarta Sans
```

---

## 📜 License

MIT

---

_Built with ❤️ for the people of India· Powered by Google Gemini Live API_
