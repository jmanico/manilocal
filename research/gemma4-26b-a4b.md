# Gemma 4 26B A4B — Local Inference Research

**Date:** 2026-04-10
**Status:** Planned
**Goal:** Run Gemma 4 26B A4B (MoE) locally on the Mac Mini (24GB unified memory, M4 Pro)

## Summary

Gemma 4 26B A4B is Google's Mixture-of-Experts model with 25.2B total parameters but only **3.8B active per token**, delivering near-31B quality at 4B-class inference speed. This makes it an excellent fit for the Mac Mini's 24GB unified memory budget.

## Key Details

| Property | Value |
|---|---|
| Model | Gemma 4 26B A4B |
| Architecture | Mixture-of-Experts (MoE) |
| Total parameters | 25.2B |
| Active parameters | 3.8B (per token) |
| Quantization | Q4 (Dynamic 4-bit, recommended by Unsloth) |
| Runtime | Ollama 0.20+ (day-one Gemma 4 support) |
| Context limit | 8192 tokens (to stay within memory budget) |
| Target hardware | Mac Mini, M4 Pro, 24GB unified memory |

## Why This Model

- Near-31B quality at 4B-class inference speed
- MoE architecture keeps active parameter count low (3.8B per token)
- Fits comfortably in 24GB unified memory with Q4 quantization
- Ollama 0.20+ provides day-one support
- Strong candidate for secure coding prompt library tasks

## Tasks

- [ ] Update Ollama to 0.20+
- [ ] Pull model: `ollama pull gemma4:26b-a4b`
- [ ] Test inference with `--ctx-size 8192`
- [ ] Benchmark tok/s and memory usage
- [ ] Evaluate for secure coding prompt library tasks

## Hardware Target

| Component | Spec |
|---|---|
| Machine | Mac Mini |
| Chip | M4 Pro |
| Unified memory | 24 GB |
| Context size | 8192 tokens (memory-constrained) |

## Notes

- `--ctx-size 8192` is recommended to stay within the 24GB memory budget
- Dynamic 4-bit quantization (Unsloth) provides the best quality-to-size ratio for this model
- Ollama 0.20+ is required for Gemma 4 support
