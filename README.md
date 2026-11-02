# ⚡ LFM3-1.7B × Serena Agent Bridge

**A local 1.7B LLM driving real code tools — one-shot HTML5 game generation on a Jetson Orin Nano 8GB.**

This harness connects **LFM3-1.7B QAD-Q4_0** (served by llama.cpp at the full native 256K context)
to **Serena's MCP tools**, so the model does not just *print* a game — it calls
`create_text_file` / `read_file` / `list_dir` like a proper coding agent and saves the
result itself.

Built from the proven E4B+Serena bridge (`e4b-serena-settings`), retargeted to the
fastest tool-calling model in the 8GB zoo: BFCL v2 **48/52 (92.3%)**, ~31.7 t/s with the
GNOME desktop still running.

## Files

| File | Purpose |
|------|---------|
| `agent_bridge.py` | Config-driven agentic loop: llama.cpp OpenAI API ↔ Serena MCP (stdio). Incremental JSON log, context trimming, fallback save. |
| `agent_config.yaml` | All model/agent/Serena settings — edit for your own model. |
| `game_prompt.txt` | 3,053-char one-shot game spec (top-down dungeon crawler, 13 features). |
| `system_prompt.txt` | Game-developer persona + output rules. |

## Verified launch

```bash
# 1. Serve LFM3 at full native context (256K, q4_0 KV) — profile from lfm3-jetson-hermes/
bash ~/projects/lfm3-jetson-hermes/launch_lfm3.sh qad 8090 256k

# 2. Run the agent bridge (needs: python venv with mcp, httpx, pyyaml + serena-agent)
cd lfm3-serena-html-test
~/.venvs/serena-bridge/bin/python -u agent_bridge.py
```

The bridge reads `agent_config.yaml`, spawns Serena (`start-mcp-server --project .`),
discovers the toolset, and runs up to 30 turns until the game is written and verified.

## The settings that matter (LFM3-specific)

- **Sampling: temp 0.1 / top_k 50 / repeat-penalty 1.1 (vendor rec)** — never greedy.
  At temp 0.0 LFM3 loops inside its think block; thinking is unsuppressible but bounded
  (~100–4.6K tok) and lands cleanly in `reasoning_content`.
- **max_tokens ≥ 8000** per turn: the model plans the *whole answer* in reasoning first,
  then emits complete output in one pass. 17,000 covers a full HTML page comfortably.
- **Context: 262,144 native** at q4_0 KV (~3.2 GB RSS, runs beside the full desktop);
  `64k/f16` is the lighter verified profile.
- Full profile + benchmarks: `~/projects/lfm3-jetson-hermes/`.

## Results

See `RESULTS.md` after the first run — agent turn-by-turn log, timing, and the generated game.

## Credits

Bridge pattern originally proven with Gemma-4-E4B in [e4b-serena-settings](https://github.com/drwjkirkpatrick-web).