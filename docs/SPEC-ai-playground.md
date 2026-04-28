# SPEC: AI Playground

**Author:** Qoder Admin quest
**Date:** 2026-04-20
**Target app(s):** New standalone project — "AIPlayground"
**Target quest(s):** New quest (to be created), or Qoder Admin for initial build

## Overview

A unified web interface for interacting with all open-source AI models running on the office 4090 GPU. One place to chat with LLMs, generate images (Flux), generate video (HunyuanVideo), and manage installed models. Designed as a personal AI workbench — not user-facing, just for KL.

This replaces/supersedes the existing OllamaChat single-file app with a more comprehensive, multi-modal tool. Unlike OllamaChat (which only does chat), this covers the full range of capabilities on the 4090.

## User Stories

- As KL, I want to test new LLMs immediately after pulling them, without switching between different UIs
- As KL, I want to generate test images with Flux to preview prompts before using them in story generation
- As KL, I want to generate test videos with HunyuanVideo to experiment with clip styles
- As KL, I want to see GPU status (VRAM usage, loaded models) at a glance
- As KL, I want to pull/delete Ollama models remotely without SSH-ing into the office PC
- As KL, I want to compare outputs from different LLMs side by side (same prompt, different models)
- As KL, I want my conversations and generated media to persist across sessions

## Current State

**OllamaChat** exists as a single HTML file (`OllamaChat/index.html`) with:
- Basic Ollama chat with streaming, markdown rendering, syntax highlighting
- Model picker dropdown (read-only, no install/delete)
- No image/video generation
- No conversation persistence
- Hardcoded to `localhost:11434` (requires being on the same machine or VBScript launcher)

**ComfyUI** is accessible at `http://100.113.43.60:8188` with:
- Flux Dev (text-to-image) — tested and working
- HunyuanVideo (text-to-video) — tested and working
- `comfyui_client.py` in VocabVista provides the workflow patterns

**Ollama** is accessible at `http://100.113.43.60:11434` with 11 models installed.

## Architecture

### Routing

The AI Playground talks to the 4090 office PC, which is on Tailscale (`100.113.43.60`). Two access patterns:

1. **Local access (KL's laptop on Tailscale):** Direct to Ollama/ComfyUI endpoints
2. **Remote access (from anywhere):** Through the LLM proxy at `api.kreativeland.com`

The app should support both — configurable base URLs with sensible defaults.

### Tech Stack

| Layer | Choice | Rationale |
|-------|--------|-----------|
| **Framework** | Pure HTML/CSS/JS | Matches portal pattern, no build step, instant deploy |
| **Styling** | CSS custom properties | Reuse portal's design tokens |
| **Font** | Inter (Google Fonts) | Consistent with KreativeLand ecosystem |
| **Markdown** | marked.js (CDN) | Same as OllamaChat, proven |
| **Syntax highlighting** | highlight.js (CDN) | Same as OllamaChat, proven |
| **Persistence** | localStorage | Conversations, settings, generation history |
| **Deployment** | Vercel (static) or local | No backend needed — all API calls go to Ollama/ComfyUI |

### File Structure

```
AIPlayground/
├── index.html          # App shell, tab navigation, settings
├── style.css           # Full design system + component styles
├── js/
│   ├── app.js          # Main app state, routing, tab management
│   ├── chat.js         # LLM chat module
│   ├── image.js        # Image generation module
│   ├── video.js        # Video generation module
│   ├── models.js       # Model management module
│   └── api.js          # Shared API client (Ollama + ComfyUI)
├── AGENTS.md           # Quest instructions
└── .git (file)         # -> D:/GitRepos/AIPlayground.git
```

## Proposed Design

### Navigation

Fixed sidebar (desktop) / bottom tabs (mobile) with 4 main sections + settings:

| Tab | Icon | Purpose |
|-----|------|---------|
| **Chat** | 💬 | LLM conversations with streaming |
| **Image** | 🎨 | Flux text-to-image generation |
| **Video** | 🎬 | HunyuanVideo text-to-video generation |
| **Models** | ⚙️ | GPU status, model management, pull/delete |

### Design System

Inherit from the KreativeLand portal with a new accent color for this project:

```css
/* AI Playground accent — cyan/electric blue */
--playground-accent: #22d3ee;
--playground-glow: rgba(34, 211, 238, 0.25);

/* Inherited from portal */
--bg-base: #09090b;
--bg-surface: #111114;
--bg-elevated: #18181c;
--text-primary: #f0f0f3;
--text-secondary: #8b8b99;
--border: rgba(255, 255, 255, 0.06);
--radius-xl: 20px;
--ease: cubic-bezier(0.4, 0, 0.2, 1);
```

### Tab 1: Chat

**Layout:** Two-column on desktop (conversation list left, active chat right). Single column on mobile.

**Conversation List (left sidebar):**
- List of saved conversations with title + model name + timestamp
- "New Chat" button at top
- Conversations persist in localStorage
- Delete/rename conversations

**Chat Area (main):**
- Model selector dropdown (populated from `GET /api/tags`)
- System prompt editor (collapsible, saved per conversation)
- Message thread with user/assistant bubbles
- Streaming response with progressive markdown rendering (pattern from OllamaChat)
- Stop generation button (AbortController)
- Copy code blocks button
- Token count display (from Ollama response metadata: `eval_count`, `prompt_eval_count`)

**Advanced Options (collapsible):**
- Temperature slider (0.0 - 2.0, default 0.7)
- Max tokens input
- `think` toggle (enable/disable reasoning mode for supported models)
- `format: json` toggle (force JSON output)

**API calls:**
```
GET  http://100.113.43.60:11434/api/tags          → model list
POST http://100.113.43.60:11434/api/chat           → streaming chat
     { model, messages, stream: true, think, options: { temperature, num_predict } }
```

### Tab 2: Image Generation

**Layout:** Prompt input area on left, generated images gallery on right.

**Controls:**
- Prompt textarea (multi-line)
- Negative prompt textarea (optional, Flux ignores this but future models may use it)
- Width/Height dropdowns or presets: 512x512, 768x768, 1024x768, 1024x1024, 1024x1536
- Steps slider (1-50, default 20)
- Seed input (blank = random, or enter specific seed for reproducibility)
- CFG slider (1.0-20.0, default 3.5 for Flux)
- "Generate" button

**Generation flow:**
1. Submit workflow to ComfyUI: `POST http://100.113.43.60:8188/prompt`
2. Poll progress: `GET http://100.113.43.60:8188/history/{prompt_id}` every 2s
3. Show progress indicator (queued → generating → complete)
4. Download output: `GET http://100.113.43.60:8188/view?filename=X&type=output`
5. Display in gallery with full-size lightbox on click

**Gallery:**
- Grid of generated images with timestamp + prompt snippet
- Click to expand full-size
- Download button
- Copy prompt button
- Re-use prompt button (sends back to prompt input)
- History persisted in localStorage (store prompt + settings + image as base64 data URL, with a reasonable limit like last 50 images)

**Workflow (from comfyui_client.py):**
```
UNETLoader(flux1-dev.safetensors, fp8_e4m3fn)
  → DualCLIPLoader(t5xxl_fp16.safetensors, clip_l.safetensors, type:flux)
    → CLIPTextEncode(positive prompt)
    → CLIPTextEncode(negative, empty)
  → EmptySD3LatentImage(width, height)
  → KSampler(cfg 3.5, euler, simple, steps, seed)
    → VAELoader(ae.safetensors)
      → VAEDecode
        → SaveImage
```

### Tab 3: Video Generation

**Layout:** Similar to Image tab but with video-specific controls.

**Controls:**
- Prompt textarea
- Resolution presets: 512x320 (fast), 640x480, 768x512
- Frame count: 25, 37, 49 (more frames = longer video, much slower)
- Steps slider (1-50, default 20)
- Seed input
- FPS display (fixed at 8 for HunyuanVideo output)
- Estimated time display (rough: frames x steps x ~0.5s)
- "Generate" button

**Generation flow:** Same as Image but uses HunyuanVideo workflow. Output is animated WEBP.

**Gallery:** Grid of generated videos with inline playback (WEBP animation or `<video>` tag). Download button.

**Workflow (from comfyui_client.py):**
```
UNETLoader(hunyuan_video_t2v_720p_bf16.safetensors, fp8_e4m3fn)
  → DualCLIPLoader(llava_llama3_fp8_scaled.safetensors, clip_l.safetensors, type:hunyuan_video)
    → CLIPTextEncode(positive)
    → CLIPTextEncode(negative, empty)
  → EmptyHunyuanLatentVideo(width, height, frames)
  → KSampler(cfg 6.0, euler, simple, steps, seed)
    → VAELoader(hunyuan_video_vae_bf16.safetensors)
      → VAEDecode
        → SaveAnimatedWEBP(fps:8, quality:80)
```

### Tab 4: Models & GPU

**Layout:** Dashboard-style cards.

**GPU Status Card:**
- GPU name (RTX 4090)
- VRAM usage bar (used/free/total)
- ComfyUI version
- Ollama version
- Auto-refresh every 10s

**Installed Models List:**
- Table: model name, size (GB), family, quantization, parameter count
- Source from `GET /api/tags` (includes `details.parameter_size`, `details.quantization_level`)
- Delete button per model → `DELETE /api/delete { name: "model:tag" }`
- Sort by name/size

**Pull New Model:**
- Text input for model name (e.g., `llama3:8b`, `deepseek-coder:6.7b`)
- "Pull" button → `POST /api/pull { name: "model:tag", stream: true }`
- Progress bar showing download status (stream response includes `completed`/`total` bytes)

**ComfyUI Models:**
- List diffusion models, VAEs, text encoders from `GET /models/{folder}`
- Read-only (ComfyUI models are managed via file system, not API)

**API calls:**
```
GET    http://100.113.43.60:11434/api/tags       → list models
DELETE http://100.113.43.60:11434/api/delete      → delete model
POST   http://100.113.43.60:11434/api/pull        → pull model (streaming progress)
GET    http://100.113.43.60:11434/api/ps          → currently loaded models
GET    http://100.113.43.60:8188/system_stats     → GPU/VRAM info
GET    http://100.113.43.60:8188/models/{folder}  → ComfyUI model inventory
```

### Settings (Modal or Sub-page)

- **Ollama URL:** Default `http://100.113.43.60:11434`, editable
- **ComfyUI URL:** Default `http://100.113.43.60:8188`, editable
- **Default chat model:** Dropdown, saved to localStorage
- **Default image size:** Preset selector
- **Theme:** Dark only (matches ecosystem)
- **Clear all data:** Nuke localStorage (conversations, history, settings)

### Status Bar (Global, bottom or header)

Always visible strip showing:
- Ollama: online/offline (green/red dot)
- ComfyUI: online/offline (green/red dot)
- VRAM: usage bar (compact)
- Currently loaded model (from `/api/ps`)

Polls every 15 seconds.

## Implementation Phases

### Phase 1: Chat + Models (Core)
1. Set up project structure (index.html, style.css, js/ modules)
2. Implement design system (CSS custom properties from portal)
3. Build tab navigation shell
4. Implement Chat tab — model picker, streaming chat, markdown rendering, conversation persistence
5. Implement Models tab — GPU status, model list, pull/delete
6. Implement global status bar
7. Implement Settings

### Phase 2: Image Generation
1. Build Image tab UI — prompt input, controls, gallery
2. Implement ComfyUI workflow submission (port Flux workflow from comfyui_client.py to JS)
3. Implement progress polling + output download
4. Gallery with lightbox + localStorage persistence

### Phase 3: Video Generation
1. Build Video tab UI — prompt input, video-specific controls
2. Implement HunyuanVideo workflow submission
3. Video playback (animated WEBP)
4. Gallery persistence

### Phase 4: Polish & Extras
1. Conversation export (JSON/Markdown)
2. Image/video batch generation
3. Prompt templates/presets library
4. Compare mode (same prompt across models)
5. Keyboard shortcuts
6. Add card to KreativeLand portal

## Deployment

**Option A (Recommended): Vercel static deploy**
- Same pattern as other KreativeLand apps
- URL: `playground.kreativeland.com`
- No build step, just deploy the HTML/CSS/JS files
- Note: Vercel cannot reach Tailscale network directly. The app runs in the user's browser, which CAN reach Tailscale (if KL's device is on Tailscale). This works because all API calls are client-side fetch, not server-side.

**Option B: Local only**
- Python HTTP server like OllamaChat's launch.vbs pattern
- Only accessible from KL's Tailscale-connected devices

**CORS consideration:** Ollama has `OLLAMA_ORIGINS=*` set (per office setup doc). ComfyUI may need CORS headers configured — check `--enable-cors-header '*'` startup flag. If CORS is an issue, the VocabVista backend can act as a proxy (it already has `comfyui_client.py`).

## Open Questions

- [ ] **Project name:** "AI Playground" or something else? (affects folder name, URL, portal card)
- [ ] **New quest or Qoder Admin builds it?** If new quest, needs AGENTS.md + Quest Registry entry + To Do list. If Qoder Admin, it's a sub-project of this quest.
- [ ] **CORS on ComfyUI:** Need to verify ComfyUI allows browser CORS requests. If not, may need to proxy through VocabVista backend or add `--enable-cors-header` to ComfyUI startup args (Scott task).
- [ ] **Image storage:** localStorage has a ~5-10MB limit. For heavy image/video history, may need to save to disk via a small backend helper, or just keep the last N items.
- [ ] **Portal card:** Should this appear on kreativeland.com? It's a personal tool, not a product. Maybe a "Tools" section or hidden admin link.
