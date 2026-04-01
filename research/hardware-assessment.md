# Hardware Assessment — Local Model Inference

**Date:** 2026-03-31
**Status:** Complete
**Issue:** #4

## Available Hardware

### Machine 1: MacBook M5 (32 GB)

| Property | Value |
|---|---|
| Chip | Apple M5 (base) |
| Unified Memory | 32 GB |
| Memory Bandwidth | 153.6 GB/s |
| CPU | 12-core |
| GPU | 10-core |
| Neural Engine | 16-core |
| Best Runtime | MLX (mlx-lm) or Ollama |

### Machine 2: Mac Mini M4 Pro (24 GB)

| Property | Value |
|---|---|
| Chip | Apple M4 Pro |
| Unified Memory | 24 GB |
| Memory Bandwidth | 273 GB/s |
| CPU | 12-core or 14-core |
| GPU | 20-core |
| Neural Engine | 16-core |
| Best Runtime | MLX (mlx-lm) or Ollama |

### Which Machine is Better for Inference?

**Speed: Mac Mini M4 Pro wins.** LLM inference is memory-bandwidth-bound, and the M4 Pro has **273 GB/s vs 153.6 GB/s** — nearly 1.8x faster. The M4 Pro also has double the GPU cores (20 vs 10).

**Capacity: MacBook M5 wins.** With 32 GB vs 24 GB, the laptop can load larger models that the Mac Mini cannot fit.

**Recommendation:** Use the Mac Mini as the primary inference machine for models that fit in 24 GB (fastest). Use the MacBook M5 for larger models (up to ~20 GB model files) that need the extra 8 GB headroom, accepting slower speed.

## Memory Budget

**Rule of thumb:** Model file should be no more than 60-70% of total memory.

### Mac Mini M4 Pro (24 GB)

| Allocation | Memory |
|---|---|
| macOS + apps | ~4-6 GB |
| Model weights | **~14-16 GB max** |
| KV cache (context) | ~2-4 GB |
| Framework overhead | ~1 GB |

### MacBook M5 (32 GB)

| Allocation | Memory |
|---|---|
| macOS + apps | ~4-6 GB |
| Model weights | **~19-22 GB max** |
| KV cache (context) | ~2-4 GB |
| Framework overhead | ~1 GB |

The extra 8 GB on the M5 opens up the 27B dense and 35B MoE models.

## What Models Fit

### Comfortable (14 GB or under)

| Model | Quant | Size | Est. tok/s (M4 Pro) | Content Quality |
|---|---|---|---|---|
| Qwen3.5-9B | Q4_K_M | ~6.5 GB | 60-80 | Very good |
| Qwen3.5-4B | Q4_K_M | ~3.4 GB | 100+ | Decent |
| DeepSeek-R1-Qwen-14B | Q4_K_M | ~8 GB | 35-50 | Very good (reasoning) |
| DeepSeek-R1-Qwen-7B | Q4_K_M | ~4 GB | 60-80 | Good |
| Llama 3.3 8B | Q4_K_M | ~5 GB | 60-80 | Good |
| Mistral Small 3 | Q4_K_M | ~13 GB | 25-35 | Good |
| Qwen3.5-35B-A3B | UD-IQ2_XXS | ~10.7 GB | 20-30 | Uncertain (extreme quant) |

### MacBook M5 Only (15-20 GB, needs the 32 GB machine)

| Model | Quant | Size | Notes |
|---|---|---|---|
| Qwen3.5-27B | Q4_K_M | ~15 GB | Dense, high quality. Fits on M5 only. |
| Qwen3.5-35B-A3B | UD-IQ4_XS | ~17.5 GB | MoE, good quality 4-bit. M5 only. |
| Qwen3.5-35B-A3B | Q4_K_M | ~22 GB | Tight on M5, no room for long context. |
| DeepSeek-R1-Qwen-32B | Q4_K_M | ~18 GB | R1 reasoning in 32B. M5 only. |

### Tight Fit on Mac Mini (13-16 GB)

| Model | Quant | Size | Notes |
|---|---|---|---|
| Qwen3.5-35B-A3B | UD-IQ3_S | ~13.6 GB | MoE, 3-bit — quality uncertain |
| Qwen3.5-35B-A3B | UD-IQ2_XXS | ~10.7 GB | Extreme quant, test against 9B Q4 |

### Will Not Fit (Either Machine)

- Qwen3.5-122B-A10B — 81 GB at Q4
- DeepSeek 671B — minimum 131 GB
- Any 70B model — needs 48 GB minimum

## Current Setup

**Mac Mini M4 Pro (24 GB)** is running **nemobot** — an agentic research pipeline from [aegisman/research](https://github.com/aegisman/research).

### Software Stack
- **Model:** Mistral Small 24B 2501 (`mistral-small:24b-instruct-2501-q4_K_M`)
- **Quantization:** Q4_K_M (4-bit), ~14 GB
- **Runtime:** Ollama 0.18.3
- **Search:** Local SearXNG instance (localhost:8888)
- **Pipeline:** Python agent loop with tool calling (web_search, scrape_page, summarize)

### How Nemobot Works
1. LLM plans research steps via Ollama tool calling
2. Searches the web via local SearXNG (no cloud search APIs)
3. Scrapes pages with readability extraction (truncated to 12K chars)
4. Summarizes content with the same local LLM
5. Iterates (3-12 rounds depending on depth: quick/normal/deep)
6. Synthesizes findings into structured report (executive summary, key findings, analysis, sources)
7. Emails results and populates a research wiki

### Model Requirements for This Pipeline
The LLM must handle:
- **Tool calling:** Deciding which tool to invoke and with what arguments
- **Multi-turn context:** Accumulating search results, page content, and summaries across iterations
- **Synthesis:** Compiling findings into coherent structured reports
- **Judgment:** Knowing when to search deeper vs. when to stop

### Current Limitations
- **32K context window:** Deep research (12 iterations) accumulates many messages and may approach the limit. Mistral Small 3.1 (128K context, same 24B) would be a low-effort upgrade.
- **Memory pressure:** 14 GB model on 24 GB machine leaves ~10 GB for OS + KV cache. Sufficient for 32K context but would not support 128K without swapping.

### Model Assessment for Research Pipeline

| Model | Params | Tool Calling | Context | Fits Mac Mini | Fits M5 | Verdict |
|---|---|---|---|---|---|---|
| Mistral Small 24B 2501 | 24B | Excellent | 32K | Yes (tight) | Yes | **Current — solid baseline** |
| Mistral Small 3.1 24B | 24B | Excellent | 128K | Yes (tight) | Yes | **Best upgrade — same size, 4x context** |
| Qwen3.5-9B | 9B | Good | 256K | Yes (roomy) | Yes | Downgrade — worse planning/synthesis |
| Qwen3.5-27B | 27B | Very good | 256K | No (15 GB) | Yes | **Quality upgrade on M5 laptop** |
| DeepSeek-R1-14B | 14B | Limited | 128K | Yes | Yes | Reasoning-focused, weak tool calling |

### Recommendations

1. **Keep Mistral Small 24B on Mac Mini** as the primary research model. It works, tool calling is solid, and 24B intelligence matters for research planning.
2. **Upgrade to Mistral Small 3.1** when available in Ollama — same model size, 128K context window eliminates the deep-research context ceiling.
3. **Test Qwen3.5-27B on the MacBook M5** as an alternative backend — better benchmarks, 256K context, same Ollama interface. Just change `OLLAMA_MODEL` in `.env`.
4. **Do NOT downgrade to Qwen3.5-9B** for research. The 9B is great for simple content tasks but would produce shallower research with worse tool-call planning.

## Recommendations for Content Management

### Already Running: Mistral Small 3 (24B, ~14 GB)

Good general-purpose model. The main limitations to test against:
- 32K context may be short for long-document summarization (3.1 variant extends to 128K)
- At 14 GB on 24 GB machine, context window may be memory-constrained in practice
- Compare content quality against Qwen3.5-9B which is smaller but leaves more room for context

### Recommended Addition: Qwen3.5-9B (Q4_K_M, ~6.5 GB)

Worth adding alongside Mistral Small:
- Fits with massive headroom (6.5 GB model leaves ~12 GB for context + OS)
- Can handle long documents with large KV cache
- GPQA Diamond: 81.7% (beats many 120B+ models)
- Fast: 60-80 tok/s on M4 Pro
- Full 256K context support
- Thinking + non-thinking modes

### Secondary Model: DeepSeek-R1-Qwen-14B (Q4_K_M, ~8 GB)

For tasks requiring deeper reasoning:
- Chain-of-thought reasoning from R1 distillation
- Better for complex editing decisions, nuanced classification
- Slower but more accurate than general-purpose models
- Comfortable fit at 8 GB

### On the MacBook M5 (32 GB): Qwen3.5-27B (Q4_K_M, ~15 GB)

The extra memory on the M5 unlocks a major quality tier:
- Dense 27B model — no MoE complexity
- High-quality Q4 quantization (no extreme compression needed)
- Fits with ~11 GB headroom for OS + context
- Slower than Mac Mini (~20-25 tok/s due to lower bandwidth) but higher quality
- Best option when quality matters more than speed

### Also Worth Testing on M5: DeepSeek-R1-Qwen-32B (Q4_K_M, ~18 GB)

- R1 reasoning distilled into 32B
- Fits on M5 with ~8 GB headroom
- For complex content decisions where chain-of-thought helps

### Skip These

- **Qwen3.5-4B:** Works but 9B is so much better and still fits easily
- **Qwen3.5-35B-A3B at Q4_K_M (22 GB):** Too tight on M5, barely any context room
- **Any 70B+ model:** Needs 48 GB minimum

## Runtime Recommendations

### For the Mac Mini M4 Pro (primary inference machine)

1. **MLX (mlx-lm):** 20-30% faster than llama.cpp on Apple Silicon. Best performance.
2. **Ollama:** Easiest setup, auto Metal acceleration, one command. Good for daily use.
3. **LM Studio:** GUI option with MLX backend. Good if you want a chat interface.

### For the MacBook M5 (portable + larger models)

Same software stack. Expect ~55% of the Mac Mini's speed due to lower bandwidth, but the 32 GB memory lets you run 27B-32B models the Mac Mini cannot fit.

### Setup Steps

```bash
# Option A: Ollama (easiest)
brew install ollama
ollama pull qwen3.5:9b
ollama run qwen3.5:9b

# Option B: MLX (fastest)
pip install mlx-lm
mlx_lm.generate --model mlx-community/Qwen3.5-9B-4bit --prompt "Summarize this:"
```

## Upgrade Path (If Needed Later)

If 9B-14B quality proves insufficient for content tasks:

| Upgrade | Cost | Unlocks |
|---|---|---|
| Mac Mini M4 Pro 48GB | ~$1,800 | 27B-35B at Q4, 70B at Q2 |
| Mac Studio M4 Max 128GB | ~$4,000 | 70B at Q4, comfortable 35B at Q8 |
| Mac Studio M3 Ultra 192GB | ~$5,000+ | 122B-A10B, 671B at 1.58-bit |

The 48 GB Mac Mini M4 Pro is the best value upgrade — doubles model capacity for ~$600 over the 24 GB model.

## Summary

| Machine | Best Model | Size | Speed | Quality |
|---|---|---|---|---|
| Mac Mini M4 Pro 24GB | Mistral Small 3 (Q4) | 14 GB | ~25-35 tok/s | Good (current) |
| Mac Mini M4 Pro 24GB | Qwen3.5-9B (Q4) | 6.5 GB | ~70 tok/s | Very good |
| Mac Mini M4 Pro 24GB | DeepSeek-R1-14B (Q4) | 8 GB | ~40 tok/s | Very good (reasoning) |
| MacBook M5 32GB | Qwen3.5-27B (Q4) | 15 GB | ~20-25 tok/s | Excellent |
| MacBook M5 32GB | DeepSeek-R1-32B (Q4) | 18 GB | ~15-20 tok/s | Excellent (reasoning) |
| MacBook M5 32GB | Qwen3.5-9B (Q4) | 6.5 GB | ~40 tok/s | Very good |

**Bottom line:** Two-machine strategy:
- **Mac Mini (current):** Already running Mistral Small 3 (24B). Add Qwen3.5-9B for comparison — half the size, likely faster, may match quality. The key test is whether 9B at Q4 with more context headroom beats 24B at Q4 with tight memory.
- **MacBook M5 for quality:** Qwen3.5-27B at ~20 tok/s when you need the best output. The 32 GB unlocks models the Mac Mini cannot run.
- **Both machines:** DeepSeek-R1 distills for reasoning-heavy tasks (14B on Mini, 32B on M5).
