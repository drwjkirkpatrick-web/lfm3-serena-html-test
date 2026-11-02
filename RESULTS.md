# Results — LFM3-1.7B × Serena, 3,053-char one-shot prompt

**Run date:** 2026-11-02 · Jetson Orin Nano 8GB · llama.cpp build e0f142d5
**Server:** LFM3-1.7B QAD-Q4_0, full native **262,144 ctx** at q4_0 KV, GUI (GNOME) ON
**Pace:** 29–32 tok/s generation, zero GPU alloc failures, ~3.2 GB RSS

## Attempt 1 — single-shot mega-emission (archived in `attempt1/`)

| Metric | Value |
|---|---|
| Server profile | 256K / q4_0 KV, GUI on |
| Bridge budget | 17,000 tok/turn → raised mid-run |
| Outcome | **Failed** — 46,676 tokens generated in 1,377 s (23 min) |
| Think block | 116,363 chars (~45K tokens of planning) |
| File emitted | **3,051 chars / 90 lines** — structurally valid skeleton, stopped cleanly (not budget-truncated) |
| Diagnosis | Over-plan / under-emit: the planner described the whole 13-feature game, wrote ~4% of it, declared done |

## Attempt 2 — staged build (archived in `attempt2/`, the new standard)

System prompt mandates: skeleton with named stubs → `replace_content` one stub per
turn (~120 lines) → `read_file` verify.

| Metric | Value |
|---|---|
| Bridge settings | max_tokens 65,536/turn · trimmer window 262,144 · vendor sampling (0.1 / 50 / 1.1) |
| Turns | 4 (3 tool calls: 2× create_text_file, 1× read_file) — then converged to a full-file content emission, bridge fallback-saved |
| Tokens | 60,747 in 675 s (11.3 min) |
| File emitted | **8,141 chars / 251 lines**, structurally valid (DOCTYPE, closes `</html>`, canvas, balanced braces/parens, WebAudio) |
| Features | **10 / 13** — missing: F2 procedural corridors, F7 localStorage high-score, F11 exact enemy roster (skeleton/bat/wraith) |

## What was proven

1. **Infra works end-to-end:** local llama-server ⇄ bridge ⇄ Serena 2.1.0 MCP (29 tools
   discovered, `create_text_file`/`read_file` round trips clean at 256K context).
2. **The 1.7B screamer codes, but not one-shot.** Single-shot generation collapses into
   plan-heavy thinking with tiny output. Staged builds with tool-mediated turns
   tripled the output size and feature count on the same prompt.
3. **The bridge is the standard:** `agent_bridge.py` + `agent_config.yaml` are model-
   agnostic — point `base_url`/`model` at any llama.cpp endpoint with native tool calls.