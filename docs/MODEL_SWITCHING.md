# OMTAE local model switching (vLLM + desk)

Jay's stack: **vLLM :8000**, **OMTAE desk :4200**, dual RTX 3090. Only one vLLM weights bundle can run at a time.

## One command (full switch)

```bash
omtae-model use qwen-coder-32b          # default coder AWQ
omtae-model use qwen36-obliterated-27b  # after HF download completes
```

This will:

1. Write `~/.omtae/active_model.txt`
2. Set `[default_model].model` in `~/.omtae/config.toml`
3. Point `omtae-vllm.service` at the profile's `start_script` and restart vLLM
4. POST `/api/config/reload` when the desk API is up

## List / status

```bash
omtae-model list
omtae-model status
```

## Files

| File | Purpose |
|------|---------|
| `~/.omtae/models.toml` | Profile definitions (paths, TP, served name) |
| `~/.omtae/active_model.txt` | Last `omtae-model use` profile id |
| `~/.omtae/custom_models.json` | Catalog entries for dashboard/chat |
| `~/.omtae/config.toml` | `[default_model]` used by agents without override |

Install CLI helper:

```bash
cp scripts/omtae-model ~/.local/bin/omtae-model && chmod +x ~/.local/bin/omtae-model
cp scripts/models.toml.example ~/.omtae/models.toml
```

## Dashboard (one click)

**Runtime** page → **Local vLLM profile** card → choose profile → **Apply to OMTAE**.

That updates the OMTAE default model immediately. For a **full** switch (restart vLLM on :8000), run the CLI command shown after Apply, or enable `OMTAE_ALLOW_VLLM_RESTART=1` on the daemon and check **Restart vLLM** before Apply.

## API

```bash
curl -s http://127.0.0.1:4200/api/models/profiles
curl -s http://127.0.0.1:4200/api/models/active
curl -s -X PUT http://127.0.0.1:4200/api/models/active \
  -H "Content-Type: application/json" \
  -H "X-OMTAE-Pin: YOUR_PIN" \
  -d '{"profile_id":"qwen-coder-32b"}'
```

## Agents (`agent.toml`)

The `model` field must match the **served** id (same as `omtae_model_id` in `models.toml` / `custom_models.json`), or omit it to use `[default_model]` from config.

```toml
# Matches vLLM --served-model-name / custom_models.json id
model = "Qwen2.5-Coder-32B-Instruct-AWQ"
```

After switching to Qwen3.6:

```toml
model = "OBLITERATUS/Qwen3.6-27B-OBLITERATED"
```

Aliases from `custom_models.json` (e.g. `qwen36-obliterated`) also work in chat when registered in the catalog.

## Qwen3.6 download (optional profile)

```bash
hf download OBLITERATUS/Qwen3.6-27B-OBLITERATED \
  --local-dir ~/llm-models/Qwen3.6-27B-OBLITERATED \
  --exclude 'gguf/*'
```

BF16 27B may OOM on 2×24GB; prefer AWQ/GGUF variants when available. The start script is `~/start-vllm-qwen36-obliterated.sh`.
