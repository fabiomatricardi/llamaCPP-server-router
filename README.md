# llamaCPP-server-router
new commands for llama.cpp server router option



[llama.cpp server now ships with router mode](https://huggingface.co/blog/ggml-org/model-management-in-llamacpp), which lets you dynamically load, unload, and switch between multiple models without restarting.
Reminder: llama.cpp server is a lightweight, OpenAI-compatible HTTP server for running LLMs locally.

This feature was a popular request to bring Ollama-style model management to llama.cpp. It uses a multi-process architecture where each model runs in its own process, so if one model crashes, others remain unaffected.

Quick Start
Start the server in router mode by not specifying a model:
```bash
llama-server
```
This auto-discovers models from your llama.cpp cache (LLAMA_CACHE or ~/.cache/llama.cpp). If you've previously downloaded models via llama-server -hf user/model, they'll be available automatically.

You can also point to a local directory of GGUF files:
```bash
llama-server --models-dir ./my-models
```
Features
Auto-discovery: Scans your llama.cpp cache (default) or a custom --models-dir folder for GGUF files
On-demand loading: Models load automatically when first requested
LRU eviction: When you hit --models-max (default: 4), the least-recently-used model unloads
Request routing: The model field in your request determines which model handles it


## Examples


Here are **three clean, working PowerShell commands** — one for each of your models — using the **reliable `Invoke-WebRequest`** method (recommended on Windows to avoid JSON quoting issues):

---

### ✅ 1. Load `LFM2-1.2B-Q6_K`
```powershell
Invoke-WebRequest -Uri "http://localhost:8080/models/load" -Method POST -Headers @{ "Content-Type" = "application/json" } -Body (@{ model = "LFM2-1.2B-Q6_K" } | ConvertTo-Json -Compress)
```

### ✅ 2. Load `ibm-granite.granite-4.0-h-1b.Q6_K`
```powershell
Invoke-WebRequest -Uri "http://localhost:8080/models/load" -Method POST -Headers @{ "Content-Type" = "application/json" } -Body (@{ model = "ibm-granite.granite-4.0-h-1b.Q6_K" } | ConvertTo-Json -Compress)
```

### ✅ 3. Load `zen-nano-0.6b.i1-Q6_K`
```powershell
Invoke-WebRequest -Uri "http://localhost:8080/models/load" -Method POST -Headers @{ "Content-Type" = "application/json" } -Body (@{ model = "zen-nano-0.6b.i1-Q6_K" } | ConvertTo-Json -Compress)
```

---

### 💡 Why this works:
- `@{ model = "..." }` creates a PowerShell hashtable.
- `ConvertTo-Json -Compress` turns it into a compact, valid JSON string like `{"model":"LFM2-1.2B-Q6_K"}`.
- `Invoke-WebRequest` sends it natively — **no shell quoting, no escaping, no corruption**.

---

### 🔄 Optional: Use `curl.exe` (less reliable, but possible)

If you **really** want to use `curl.exe`, here’s the correctly escaped version for one model (repeat for others):

```powershell
curl.exe -X POST "http://localhost:8080/models/load" -H "Content-Type: application/json" -d "{`"model`":`"LFM2-1.2B-Q6_K`"}"
```

But **avoid this** unless necessary — the `Invoke-WebRequest` version is safer and more maintainable.

---

### ✅ Bonus: One-liner to load any model (reusable)

```powershell
function Load-Model { param($Name); Invoke-WebRequest -Uri "http://localhost:8080/models/load" -Method POST -Headers @{ "Content-Type" = "application/json" } -Body (@{ model = $Name } | ConvertTo-Json -Compress) }
```

Then use:
```powershell
Load-Model "LFM2-1.2B-Q6_K"
Load-Model "ibm-granite.granite-4.0-h-1b.Q6_K"
Load-Model "zen-nano-0.6b.i1-Q6_K"
```

Let me know if you want to **unload models**, **list loaded models**, or **send chat requests** next!


Great! You want to convert the **model unload** `curl` command to **PowerShell** — just like the load command, but for unloading.

The structure is identical: same JSON format, same endpoint pattern — just a different URL path (`/models/unload`).

---

### ✅ Recommended PowerShell Version (using `Invoke-WebRequest`)

Replace `"my-model.gguf"` with each of your actual model names:

---

#### 1. Unload `LFM2-1.2B-Q6_K`
```powershell
Invoke-WebRequest -Uri "http://localhost:8080/models/unload" -Method POST -Headers @{ "Content-Type" = "application/json" } -Body (@{ model = "LFM2-1.2B-Q6_K" } | ConvertTo-Json -Compress)
```

#### 2. Unload `ibm-granite.granite-4.0-h-1b.Q6_K`
```powershell
Invoke-WebRequest -Uri "http://localhost:8080/models/unload" -Method POST -Headers @{ "Content-Type" = "application/json" } -Body (@{ model = "ibm-granite.granite-4.0-h-1b.Q6_K" } | ConvertTo-Json -Compress)
```

#### 3. Unload `zen-nano-0.6b.i1-Q6_K`
```powershell
Invoke-WebRequest -Uri "http://localhost:8080/models/unload" -Method POST -Headers @{ "Content-Type" = "application/json" } -Body (@{ model = "zen-nano-0.6b.i1-Q6_K" } | ConvertTo-Json -Compress)
```

---

### 🔄 Or use a reusable function (if you're doing this often)

```powershell
function Unload-Model {
    param([string]$ModelName)
    $body = @{ model = $ModelName } | ConvertTo-Json -Compress
    Invoke-WebRequest -Uri "http://localhost:8080/models/unload" -Method POST -Headers @{ "Content-Type" = "application/json" } -Body $body
}
```

Then run:
```powershell
Unload-Model "LFM2-1.2B-Q6_K"
Unload-Model "ibm-granite.granite-4.0-h-1b.Q6_K"
Unload-Model "zen-nano-0.6b.i1-Q6_K"
```

---

### ⚠️ Notes:
- The API expects the **exact model name** as listed in your server logs (e.g., `LFM2-1.2B-Q6_K`, not a `.gguf` filename unless that's how it was loaded).
- If you get a `404` or `400` error, double-check the model name — it must match **exactly** what the server recognizes.
- This assumes your backend (likely `llama.cpp` + `llama-cpp-python` or similar) supports the `/models/unload` endpoint with a JSON body containing `"model"`.

---

Let me know if you also need commands to **list loaded models** or **check server status**! 😊
