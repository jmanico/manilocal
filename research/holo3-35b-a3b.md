# Holo3-35B-A3B — Local Inference Research

**Date:** 2026-03-31
**Status:** Initial evaluation
**Issue:** #1

## Summary

Holo3-35B-A3B is H Company's open-weight Vision-Language Model (VLM) fine-tuned from **Qwen3.5-35B-A3B** (Alibaba). It uses a Sparse Mixture-of-Experts (MoE) architecture with 35B total parameters but only **3B active per token**, making it very efficient for local inference.

**Important caveat:** This model is specialized for **GUI agent / computer-use tasks** (web navigation, desktop automation, UI grounding) — not general-purpose content generation. Its suitability for this project's content management workflows (summarization, editing, classification) is uncertain and would need benchmarking.

## Architecture

| Property | Value |
|---|---|
| Base model | Qwen3.5-35B-A3B |
| Architecture | Sparse Mixture-of-Experts (MoE) |
| Total parameters | 35B |
| Active parameters | ~3B (per token) |
| Expert count | 256 total, 9 active per token |
| Context length | 262,144 tokens (native), extensible to 1M |
| Precision | BF16 |
| Modality | Vision-Language (VLM) |
| License | Apache 2.0 |

## Primary Use Case

Holo3-35B-A3B is trained for **computer-use agents**:
- GUI navigation (web, desktop, mobile)
- Visual interface interpretation
- UI element localization and grounding
- Multi-step reasoning for perception and decision-making

Benchmarks:
- **OSWorld-Verified:** 77.8% (state-of-the-art for computer use)
- **ScreenSpot-Pro:** top-tier UI grounding
- **H Corporate Benchmark:** 486 multi-step enterprise tasks

The base model (Qwen3.5-35B-A3B) scores well on general benchmarks too:
- MMLU-Pro: 85.3%
- GPQA Diamond: 84.2%
- SWE-bench Verified: 69.2%

## VRAM / RAM Requirements

Despite only 3B active parameters per token, **all 35B parameters (all 256 experts) must be loaded into memory**. This is the key constraint.

| Quantization | File Size | Est. VRAM/RAM |
|---|---|---|
| BF16 (full) | 69.4 GB | ~72 GB |
| Q8_0 | 36.9 GB | ~40 GB |
| Q6_K | 28.9 GB | ~32 GB |
| Q5_K_M | 26.2 GB | ~28 GB |
| Q4_K_M | 22.0 GB | ~24 GB |
| UD-IQ4_XS | 17.5 GB | ~20 GB |
| UD-IQ3_S | 13.6 GB | ~16 GB |
| UD-IQ2_XXS | 10.7 GB | ~13 GB |

**Sweet spot:** Q4_K_M (22 GB) for quality, or UD-IQ4_XS (17.5 GB) for constrained hardware.

## Runtime Compatibility

| Runtime | Support | Notes |
|---|---|---|
| **llama.cpp** | Yes | GGUF quantizations available via Unsloth. Primary recommended runtime. |
| **Ollama** | Partial | Qwen3.5 MoE models have had issues with separate mmproj vision files. Recent preview builds (MLX backend on Apple Silicon) show progress. May work for text-only mode. |
| **vLLM** | Yes | Supports Qwen3.5 architecture natively. Good for serving. |
| **LM Studio** | Likely | Supports GGUF models via llama.cpp backend. Should work if llama.cpp supports it. |

### Quantization Sources

GGUF files available from [unsloth/Qwen3.5-35B-A3B-GGUF](https://huggingface.co/unsloth/Qwen3.5-35B-A3B-GGUF) with Unsloth Dynamic 2.0 quantization (improved accuracy over standard quants).

Note: These are quantizations of the **base** Qwen3.5-35B-A3B model. Holo3's fine-tuned weights would need separate GGUF conversion unless H Company or the community provides them.

## Feasibility for Content Management

### Concerns
- **Specialization mismatch:** Holo3 is fine-tuned for GUI agents, not content tasks. The fine-tuning may have degraded general language capabilities compared to base Qwen3.5-35B-A3B.
- **Vision overhead:** As a VLM, it carries vision encoder weights that are unnecessary for text-only content management.
- **Better alternatives likely exist:** For summarization, editing, and classification, a general-purpose model (e.g., base Qwen3.5-35B-A3B, or a dense model like Qwen3.5-27B) would likely perform better.

### Recommendation

**Low priority for content management use cases.** The underlying Qwen3.5-35B-A3B base model is worth evaluating separately — it has excellent general benchmarks and the same efficient MoE architecture. Holo3 itself is better suited if the project scope expands to include automated UI testing or browser-based content workflows.

## References

- Model card: https://huggingface.co/Hcompany/Holo3-35B-A3B
- H Company announcement: https://hcompany.ai/holo3
- Base model: https://huggingface.co/Qwen/Qwen3.5-35B-A3B
- GGUF quantizations (base): https://huggingface.co/unsloth/Qwen3.5-35B-A3B-GGUF
- Cookbook: https://github.com/hcompai/hai-cookbook/tree/main/holo3
