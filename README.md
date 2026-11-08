# J.A.R.V.I.S. — Personal AI Voice Assistant

A futuristic personal AI voice assistant inspired by Tony Stark's legendary system, built with **React**, **TypeScript**, and a **Node.js** backend powered by the **Google Gemini API** and **Web Speech API**.

---

## ⚡ Features

1. **Voice Input via Web Speech API**:
   - High-performance Push-to-Talk interface supporting both **Hold-to-Talk** (hold down while speaking) and **Click-to-Toggle**.
   - Real-time speech transcription stream and silence detection.
   - Fallback text directive input whenever silence or typing is preferred.

2. **Google Gemini Integration**:
   - Backend-only API key security (kept strictly inside `server/.env`).
   - Powered by Gemini (`gemini-2.0-flash` / `gemini-1.5-flash` / `gemini-2.5-flash`).
   - Multi-turn conversation awareness with tool calling feedback loop.

3. **Spoken Replies with Speech Synthesis**:
   - Vocal response generation using the browser's `SpeechSynthesis` engine.
   - Automatically selects sophisticated UK/British English voices (e.g. Daniel, Oliver, George) with fine-tuned pitch and tempo for the classic JARVIS cadence.
   - Procedural Web Audio API sound effects for HUD activation, transmission, vocalization, and alarms.

4. **Polite, Intelligent Persona ("Sir")**:
   - Imbued with JARVIS's signature personality: courteous, intellectually composed, subtly witty, and addressing the user as **"Sir"**.

5. **Persistent Conversation Memory**:
   - Local SQLite database (`better-sqlite3` in WAL mode) storing complete historical exchanges and system telemetry.

6. **Extensible Function Calling Tools**:
   - 🕒 **`tell_time_date`**: Reports current time, date, day of the week, and timezone.
   - 🌐 **`open_website`**: Opens approved or requested destinations (e.g. YouTube, GitHub, Wikipedia, Reddit, or custom URLs) in a new browser tab.
   - 🔍 **`search_web`**: Performs Google or DuckDuckGo web queries and links.
   - ⏱️ **`set_timer`**: Triggers interactive countdown timers with visual HUD progress bars and audible completion alarms.
   - 🔌 **Extensible Architecture**: Easy to plug in new tools in `server/src/tools/` using the `JARVISTool` interface.

7. **Security Confirmation Safeguard**:
   - Destructive actions (such as purging conversation history or cancelling active protocols) require explicit user authorization. A holographic confirmation modal issues a secure token before any deletion occurs.

8. **Stark Industries Dark & Gold HUD**:
   - Sleek dark charcoal background (`#07090e`) with radiant gold/amber accents (`#D4AF37`, `#FFD700`).
   - Concentric animated **Canvas Arc Reactor Orb** pulsing dynamically according to vocal states: `IDLE`, `LISTENING`, `THINKING`, `SPEAKING`.
   - Live telemetry status gauges, active countdown timer bar, and scrolling chat stream.

---

## 🚀 Quickstart Guide

### 1. Prerequisites
- **Node.js** v18+ (tested on v20.18.0)
- **npm** v9+
- A modern browser with Web Speech API support (Google Chrome, Microsoft Edge, Brave, or Safari)

### 2. Configure the Gemini API Key
Navigate to `server/.env` (or copy from `server/.env.example`):

```bash
# In server/.env:
PORT=3001
GEMINI_API_KEY=your_actual_gemini_api_key_here
GEMINI_MODEL=gemini-2.0-flash
DATABASE_PATH=./jarvis.sqlite
```

> **Note**: You can obtain a free API key at [Google AI Studio](https://aistudio.google.com/app/apikey).
> If no API key is provided, JARVIS will operate in local fallback mode, allowing you to test local tools, HUD animations, and speech synthesis immediately.

### 3. Install Dependencies & Launch
From the root directory:

```bash
# Install all dependencies (root, server, and client)
npm run install:all

# Run both backend server and frontend client concurrently
npm run dev
```

- **Frontend Client**: [http://localhost:5173](http://localhost:5173)
- **Backend API**: [http://localhost:3001](http://localhost:3001)

---

## 🗣️ Voice Commands to Try

- *"Good morning, JARVIS."*
- *"What time is it right now?"*
- *"Open YouTube"*
- *"Search for James Webb Space Telescope discoveries"*
- *"Set a timer for 10 seconds"*
- *"Purge my conversation logs"* *(Triggers the holographic confirmation protocol)*

---

## 🛠️ Adding New Tools

JARVIS uses an extensible Tool Registry. To add a new capability:

1. Create a file in `server/src/tools/myTool.ts`:
```ts
import { JARVISTool, ToolContext, ToolExecutionResult } from './types.js';

export const myTool: JARVISTool = {
  name: 'get_weather',
  description: 'Fetches local weather forecasts for Sir.',
  parameters: {
    type: 'OBJECT',
    properties: {
      location: { type: 'string', description: 'City name' }
    },
    required: ['location']
  },
  async execute(args, context): Promise<ToolExecutionResult> {
    return {
      success: true,
      message: `The current forecast for ${args.location} is 21°C and clear skies, Sir.`
    };
  }
};
```

2. Register it in `server/src/tools/registry.ts`:
```ts
import { myTool } from './myTool.js';

// Inside constructor:
this.register(myTool);
```

Gemini will automatically receive the tool's schema and decide when to invoke it!

---

## 📂 Project Architecture

```
jarvis/
├── package.json              # Monorepo concurrent runner scripts
├── server/                   # Node.js backend
│   ├── .env                  # Backend API key & configuration
│   ├── src/
│   │   ├── index.ts          # Express API server
│   │   ├── db/
│   │   │   └── database.ts   # SQLite memory & persistence layer
│   │   ├── services/
│   │   │   └── gemini.ts     # Gemini API integration & persona engine
│   │   └── tools/
│   │       ├── types.ts      # Tool interface definitions
│   │       ├── registry.ts   # Extensible tool registry
│   │       ├── timeDate.ts   # Chronometer tool
│   │       ├── openWebsite.ts# Browser navigation tool
│   │       ├── searchWeb.ts  # Web search query tool
│   │       ├── timerTool.ts  # Countdown timer tool
│   │       └── clearMemory.ts# Destructive action with confirmation
└── client/                   # React + TypeScript frontend
    ├── src/
    │   ├── components/
    │   │   ├── ArcReactorOrb.tsx   # Canvas animated Stark Reactor Orb
    │   │   ├── ChatLog.tsx         # Scrolling chat log with badges
    │   │   ├── VoiceController.tsx # Push-to-Talk & text dock
    │   │   ├── HeaderHUD.tsx       # System telemetry & digital clock
    │   │   ├── TimersHUD.tsx       # Live countdown timers widget
    │   │   ├── ConfirmationModal.tsx # Holographic security confirmation
    │   │   └── ToolDrawer.tsx      # Registered capability viewer
    │   ├── services/
    │   │   ├── speechSynthesis.ts  # Vocal output & voice profile
    │   │   └── soundEffects.ts     # Web Audio API procedural sound FX
    │   └── hooks/
    │       └── useSpeechRecognition.ts # Push-to-talk speech recognition
```

