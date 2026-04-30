# Office PC (hp-pc1) Skill

## When to Use
- Accessing Ollama models on the office PC via Tailscale
- Using ComfyUI for image/video generation
- Assigning tasks to Scott for the office PC
- Checking office PC hardware specs for resource planning
- SSH/remote access to the office PC

## Hardware Specs
- **Machine name:** hp-pc1
- **Tailscale IP:** 100.113.43.60
- **GPU:** Nvidia RTX 4090
- **RAM:** 64GB (upgraded from 32GB on 2026-04-30)
- **User:** `Damas` (Windows account)

## Ollama (Office PC)

### Endpoint
```
http://100.113.43.60:11434
```

### Available Models
- `qwen3.6:35b-a3b` -- **RECOMMENDED for batch tasks** (MoE, 35B params but only 3B active, fits in 24GB VRAM, fast + bilingual EN/CN)
- `qwen3.6:35b` -- dense 35B, does NOT fit in VRAM, crashes with 502
- `qwen3.6:27b` -- dense 27B, does NOT fit in VRAM, crashes with 502
- `gemma4:26b` -- dense 26B, does NOT fit in VRAM, crashes with 502
- `gemma4:e2b` -- small model (2B, works as CPU fallback)
- `glm-4.7-flash` -- 29.9B params (MoE, 3B active), 19GB, by Zhipu AI
- `nomic-embed-text` -- text embeddings
- `qwen3-coder:30b` -- code generation (may also crash, untested)

### IMPORTANT: Dense Models >7GB Crash with 502
As of 2026-04-30, any dense model larger than ~7GB fails with HTTP 502 Bad Gateway when Ollama tries to load it. This includes `gemma4:26b`, `qwen3.6:27b`, and `qwen3.6:35b`. Only MoE models (which use less active VRAM) and small models work. The 4090 has 24GB VRAM and 64GB system RAM, but Ollama's runner crashes instead of offloading to RAM.

**Workaround:** Use `qwen3.6:35b-a3b` (MoE variant) — it has 35B total params but only activates 3B at a time, fitting comfortably in VRAM.

### API Usage
```bash
# List models
curl http://100.113.43.60:11434/api/tags

# Generate text
curl http://100.113.43.60:11434/api/generate -d '{
  "model": "qwen3.6:35b-a3b",
  "prompt": "Hello",
  "stream": false
}'

# Pull a new model
curl -X POST http://100.113.43.60:11434/api/pull -d '{"name": "modelname"}'

# Delete a model
curl -X DELETE http://100.113.43.60:11434/api/delete -d '{"name": "modelname"}'
```

### CRITICAL: Python urllib Proxy Bypass
Python's `urllib` sends requests through the system proxy (GFW), which breaks Tailscale connections with 502 errors. You MUST bypass the proxy when calling the office Ollama from Python:

```python
# WRONG -- will route through GFW proxy, get 502
urllib.request.urlopen(req, timeout=60)

# CORRECT -- bypass proxy for Tailscale IP
opener = urllib.request.build_opener(urllib.request.ProxyHandler({}))
opener.open(req, timeout=60)
```

`curl` does NOT have this problem -- it skips the proxy for Tailscale IPs.

### CRITICAL: Do NOT Use Local Laptop Ollama
**KL's laptop** (`localhost:11434`) MUST NOT be used unless KL explicitly asks. All LLM inference should use the office 4090 GPU, NVIDIA NIM cloud, or OpenAI API.

Reason: Local Ollama runs on KL's laptop CPU and impacts system performance.

## ComfyUI (Image & Video Generation)

### Endpoint
```
http://100.113.43.60:8188
```

### Location
```
C:\Users\Damas\Documents\ComfyUI
```

### Installed Models
- **Flux.1-dev** (image generation): `C:\Users\Damas\Documents\ComfyUI\models\unet\flux1-dev.safetensors`
- **Flux text encoders**: `C:\Users\Damas\Documents\ComfyUI\models\text_encors\t5xxl_fp16.safetensors`, `clip_l.safetensors`
- **Flux VAE**: `C:\Users\Damas\Documents\ComfyUI\models\vae\ae.safetensors`
- **HunyuanVideo 1.5** (video generation): `C:\Users\Damas\Documents\ComfyUI\models\unet\hunyuan_video_1_5.safetensors`

### API Usage
```bash
# Check status
curl http://100.113.43.60:8188/system_stats

# Get object info
curl http://100.113.43.60:8188/object_info

# Queue a prompt (POST to /prompt with workflow JSON)
curl -X POST http://100.113.43.60:8188/prompt -d '{"prompt": {...}}'
```

### Starting ComfyUI
If ComfyUI is not running, Scott needs to start it:
```powershell
cd C:\Users\Damas\Documents\ComfyUI
python main.py --listen 0.0.0.0 --port 8188
```

Or if auto-start is configured (startup folder: `shell:startup` -> `start-comfyui.bat`).

## OpenClaw (AI Task Router)

OpenClaw routes AI tasks between providers (local Ollama, ChatGPT, Gemini). Runs on Docker.

### Docker Commands
```powershell
# Check containers
docker compose ls

# Check health
docker compose exec openclaw-gateway openclaw doctor

# Restart
docker compose down && docker compose up -d

# Get config
docker compose run --rm openclaw-cli config get
```

## Assigning Tasks to Scott

When you need Scott to do something on the office PC, create a task in the "For Scott from Qoder Quests" To Do list. See the **microsoft-todo** skill for details.

Scott's setup document: `D:\Projects\office-computer-setup-for-scott.md`

### Task Rules
1. Include clear step-by-step instructions (Scott is non-technical)
2. Set `"importance": "high"` for urgent tasks
3. Ask Scott to send results/screenshots back to "Master KL"
4. Reference the setup doc when applicable

## Tailscale
- Encrypted mesh network connecting all KL devices
- Office PC: `100.113.43.60` (hp-pc1)
- Old Azure VM: `100.87.60.60`
- Status check: `tailscale status`
