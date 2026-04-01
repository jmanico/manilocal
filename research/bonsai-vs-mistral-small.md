# Local Model Evaluation: PrismML Bonsai 8B vs Mistral Small 3.1 (24B)

**Date:** 2026-04-01
**Status:** Initial evaluation

---

## PrismML Bonsai — Native 1-Bit Models

**Source:** PrismML (emerged from stealth 2026-03-31)
**License:** Apache 2.0

### Summary

Bonsai is a family of **native 1-bit large language models** from PrismML, a Caltech-founded startup backed by Khosla Ventures, Cerberus, and Google. The key differentiator: these are trained end-to-end with 1-bit precision — not post-training quantized. Weights are ternary {-1, 0, +1} stored as 1 sign bit with an FP16 scale factor per 128-weight group (effective 1.125 bits/weight).

All three models are dense architectures based on Qwen3, licensed Apache 2.0.

### Models

| Model | Parameters | Layers | Context | Heads (Q/KV) | GGUF Size | FP16 Equivalent | Compression |
|---|---|---|---|---|---|---|---|
| **Bonsai 8B** | 8.19B | 36 | 65,536 | 32/8 GQA | **1.15 GB** | 16.38 GB | 14.2x |
| **Bonsai 4B** | 4.0B | 36 | 32,768 | 32/8 GQA | **0.57 GB** | 8.04 GB | 14.1x |
| **Bonsai 1.7B** | 1.7B | 28 | 32,768 | 16/8 GQA | **0.24 GB** | 3.44 GB | 14.3x |

All share: SwiGLU MLP, RoPE positional encoding, RMSNorm, vocab size 151,936.

### Benchmarks (Bonsai 8B)

EvalScope v1.4.2:

| Model | Disk Size | Avg | MMLU-R | MuSR | GSM8K | HumanEval+ | IFEval | BFCL |
|---|---|---|---|---|---|---|---|---|
| **Bonsai 8B** | **1.15 GB** | **70.5** | 65.7 | 50.0 | 88.0 | 73.8 | 79.8 | 65.7 |
| Qwen 3 8B | 16 GB | 79.3 | 83.0 | 55.0 | 93.0 | 82.3 | 84.2 | 81.0 |
| Mistral 3 8B | 16 GB | 71.0 | 73.9 | 53.8 | 87.2 | 67.4 | 75.4 | 45.4 |
| Llama 3.1 8B | 16 GB | 67.1 | 72.9 | 51.3 | 87.9 | 75.0 | 51.5 | -- |

**Takeaway:** Bonsai 8B (70.5 avg) is competitive with Mistral 3 8B (71.0) and beats Llama 3.1 8B (67.1), while being 14x smaller. Trails Qwen3 8B by ~9 points — significant for knowledge-intensive tasks (MMLU-R: 65.7 vs 83.0).

No benchmark scores published for 4B or 1.7B models yet.

### Throughput

#### Bonsai 8B

| Hardware | tok/s | vs FP16 Speedup |
|---|---|---|
| RTX 4090 | 368 | 6.2x |
| RTX L40S | 327 | 6.3x |
| RTX 3060 Laptop (6GB) | 81 | 23x (FP16 doesn't fit) |
| M4 Pro 48GB | 85-136 | 5.4x |
| iPhone 17 Pro Max | 44 | N/A |
| Samsung S25 Ultra | 19.6 | N/A |

#### Bonsai 4B / 1.7B

| Model | RTX 4090 tok/s | M4 Pro tok/s |
|---|---|---|
| Bonsai 4B | 440 | 136 |
| Bonsai 1.7B | 674 | 250 |

#### Energy Efficiency

| Platform | Bonsai mWh/tok | FP16 mWh/tok | Advantage |
|---|---|---|---|
| RTX 4090 | 0.276 | 1.134 | 4.1x |
| M4 Pro | 0.091 | 0.471 | 5.1x |

### Hardware Requirements

#### Model Weights Only

| Model | GGUF | MLX |
|---|---|---|
| Bonsai 8B | 1.15 GB | 1.28 GB |
| Bonsai 4B | 0.57 GB | 0.63 GB |
| Bonsai 1.7B | 0.24 GB | 0.27 GB |

#### KV Cache (Bonsai 8B, additional memory)

| Context Length | Additional Memory | Total (weights + KV) |
|---|---|---|
| 8,192 tokens | ~2.5 GB | ~3.7 GB |
| 32,768 tokens | ~5.9 GB | ~7.1 GB |
| 65,536 tokens | ~10.5 GB | ~11.7 GB |

**Practical minimums:**
- Bonsai 8B at 8K context: 4 GB RAM/VRAM
- Bonsai 8B at full 65K context: 16 GB system
- Bonsai 4B / 1.7B: runs on phones — any modern device

### How to Run Locally

#### Critical caveat: requires PrismML's llama.cpp fork

The `Q1_0_g128` quantization type is **not in upstream llama.cpp**. Standard Ollama, LM Studio, and GPT4All will not load these files. You must build from PrismML's fork.

#### Quick start (macOS, Metal)

```bash
git clone https://github.com/PrismML-Eng/llama.cpp
cd llama.cpp
cmake -B build && cmake --build build -j
./build/bin/llama-cli \
    -m Bonsai-8B-Q1_0_g128.gguf \
    -p "Your prompt here" \
    -n 256 --temp 0.5 --top-p 0.85 --top-k 20 -ngl 99
```

#### Quick start (Linux/Windows, CUDA)

```bash
git clone https://github.com/PrismML-Eng/llama.cpp
cd llama.cpp
cmake -B build -DGGML_CUDA=ON && cmake --build build -j
./build/bin/llama-cli -m Bonsai-8B-Q1_0_g128.gguf -p "Your prompt" -n 256 -ngl 99
```

#### Demo setup script

```bash
git clone https://github.com/PrismML-Eng/Bonsai-demo.git
cd Bonsai-demo
export BONSAI_MODEL=8B  # or 4B, 1.7B
./setup.sh
```

#### Server mode (OpenAI-compatible API)

```bash
./build/bin/llama-server -m Bonsai-8B-Q1_0_g128.gguf --host 127.0.0.1 --port 8080 -ngl 99
```

Web UI at `http://127.0.0.1:8080`. Compatible with Open WebUI.

#### MLX (Apple Silicon)

Separate MLX-format downloads available (e.g., `Bonsai-8B-mlx-1bit`).

#### Supported Backends

| Backend | Platform |
|---|---|
| CUDA | NVIDIA GPUs |
| Metal | macOS / iOS (Apple Silicon) |
| CPU | Any platform |
| OpenCL | Android |
| MLX | Apple Silicon (Python) |
| mlx-swift | iOS/macOS (Swift) |
| vLLM | Confirmed on H100 (v0.15.1) |

### Bonsai Downloads

All on HuggingFace under `prism-ml`:

- **Collection:** `https://huggingface.co/collections/prism-ml/bonsai`
- **Bonsai 8B GGUF:** `https://huggingface.co/prism-ml/Bonsai-8B-gguf`
- **Bonsai 8B MLX:** `https://huggingface.co/prism-ml/Bonsai-8B-mlx-1bit`
- **Bonsai 4B GGUF:** `https://huggingface.co/prism-ml/Bonsai-4B-gguf`
- **Bonsai 4B MLX:** `https://huggingface.co/prism-ml/Bonsai-4B-mlx-1bit`
- **Bonsai 1.7B GGUF:** `https://huggingface.co/prism-ml/Bonsai-1.7B-gguf`
- **Bonsai 1.7B MLX:** `https://huggingface.co/prism-ml/Bonsai-1.7B-mlx-1bit`
- **Demo repo:** `https://github.com/PrismML-Eng/Bonsai-demo`
- **Custom llama.cpp fork:** `https://github.com/PrismML-Eng/llama.cpp`
- **Whitepaper:** `https://github.com/PrismML-Eng/Bonsai-demo/blob/main/1-bit-bonsai-8b-whitepaper.pdf`

---

## Mistral Small 3.1 (24B) — Multimodal Dense Transformer

**Source:** Mistral AI, released 2025-03-17
**License:** Apache 2.0

### Summary

Mistral Small 3.1 is a 24B-parameter dense multimodal transformer from Mistral AI. It supports text and vision inputs, 128K context, native function calling, and 24+ languages. It was positioned as a competitor to GPT-4o Mini, Claude 3.5 Haiku, and Gemma 3 in its weight class. An updated 3.2 version was released in June 2025 with instruction-following improvements, but 3.1 remains the more widely benchmarked and deployed version.

### Architecture

| Parameter | Value |
|---|---|
| **Total Parameters** | 24B |
| **Architecture** | Mistral3ForConditionalGeneration (dense transformer + vision encoder) |
| **Hidden Size** | 5120 |
| **Num Hidden Layers** | 40 |
| **Num Attention Heads** | 32 |
| **Num KV Heads** | 8 (Grouped Query Attention, 4:1 ratio) |
| **Head Dimension** | 128 |
| **Intermediate Size (FFN)** | 32,768 |
| **Vocabulary Size** | 131,072 (Tekken tokenizer) |
| **Max Position Embeddings** | 131,072 (128K tokens) |
| **Sliding Window** | None (full attention) |
| **Activation** | SiLU |
| **Positional Encoding** | RoPE (theta=1,000,000,000) |
| **Precision** | bfloat16 |

#### Vision Encoder (Pixtral)

| Parameter | Value |
|---|---|
| Hidden Size | 1024 |
| Layers | 24 |
| Attention Heads | 16 |
| Head Dim | 64 |
| FFN Intermediate | 4096 |
| Image Size | 1540 |
| Patch Size | 14 |
| Channels | 3 |

### Text Benchmarks (Instruct)

| Benchmark | Score |
|---|---|
| **MMLU** (5-shot) | 80.62 |
| **MMLU Pro** (5-shot CoT) | 66.76 |
| **HumanEval** | 88.41 |
| **MBPP** | 74.71 |
| **MATH** | 69.30 |
| **GPQA Main** (5-shot CoT) | 44.42 |
| **GPQA Diamond** (5-shot CoT) | 45.96 |
| **SimpleQA** | 10.43 |
| **GSM8K** | ~69 (reported in reviews) |

### Multimodal Benchmarks

| Benchmark | Score |
|---|---|
| MMMU | 62.8 - 64.0 |
| MMMU Pro | 49.25 |
| MathVista | 68.91 |
| ChartQA | 86.24 |
| DocVQA | 94.08 |
| AI2D | 93.72 |
| MM-MT-Bench | 73.0 |

### Long Context

| Benchmark | Score |
|---|---|
| LongBench v2 | 37.18 |
| RULER 32K | 93.96 |
| RULER 128K | 81.20 |

### Multilingual MMLU

| Region | Score |
|---|---|
| Overall Average | 71.18 |
| European Languages | 75.30 |
| East Asian Languages | 69.17 |
| Middle Eastern Languages | 69.08 |

### Quantization Options & Sizes (GGUF)

Source: bartowski/mistralai_Mistral-Small-3.1-24B-Instruct-2503-GGUF on HuggingFace.

| Quantization | File Size | Quality Level | Recommended? |
|---|---|---|---|
| IQ2_XXS | 6.55 GB | Very low | No |
| IQ2_XS | 7.21 GB | Low | No |
| Q2_K | 8.89 GB | Very low, surprisingly usable | No |
| IQ3_XXS | 9.28 GB | Lower quality | No |
| IQ3_XS | 9.91 GB | Lower quality | No |
| Q3_K_S | 10.40 GB | Low | No |
| Q3_K_M | 11.47 GB | Low | No |
| Q3_K_L | 12.40 GB | Lower but usable | Marginal |
| **IQ4_XS** | **12.76 GB** | Decent | **Yes** |
| Q4_K_S | 13.55 GB | Good | **Yes** |
| **Q4_K_M** | **14.33 GB** | Good, default | **Yes** |
| Q4_1 | 14.87 GB | Legacy, good for Apple Silicon | Yes (Apple) |
| Q4_K_L | 14.83 GB | Good, Q8 embed/output | **Yes** |
| Q5_K_S | 16.30 GB | High | **Yes** |
| **Q5_K_M** | **16.76 GB** | High | **Yes** |
| Q5_K_L | 17.18 GB | High, Q8 embed/output | **Yes** |
| **Q6_K** | **19.35 GB** | Very high, near perfect | **Yes** |
| Q6_K_L | 19.67 GB | Very high, Q8 embed/output | **Yes** |
| Q8_0 | 25.05 GB | Extremely high | Yes |
| BF16 | 47.15 GB | Full precision | Requires 55 GB+ VRAM |

#### Key Quant Notes

- **Apple Silicon:** Q4_1 offers improved tokens/watt; K-quants recommended generally
- **NVIDIA (cuBLAS):** I-quants (IQ4_XS, IQ3_M) perform well below Q4
- **AMD Vulkan:** Use K-quants only, not I-quants
- **Embed/output weights:** *_L variants use Q8_0 for embed/output layers for better quality

### Memory Requirements

#### Model Weights Only

| Quantization | Model Size |
|---|---|
| Q4_K_M | 14.33 GB |
| Q5_K_M | 16.76 GB |
| Q6_K | 19.35 GB |
| Q8_0 | 25.05 GB |
| BF16 | 47.15 GB |

#### Total Memory (Weights + KV Cache, Q4_K_M)

| Context Length | Approx Total Memory |
|---|---|
| 2,048 tokens | ~14.3 GB |
| 8,192 tokens | ~15.5 GB |
| 16,384 tokens | ~17.4 GB |
| 32,768 tokens | ~21.0 GB |
| 65,536 tokens | ~28 GB (estimated) |
| 128,000 tokens | ~42 GB (estimated) |

**Practical minimums:**
- Q4_K_M at 8K context: ~16 GB RAM (fits 16 GB Mac with swap pressure)
- Q4_K_M at 32K context: ~24 GB RAM (comfortable on 32 GB Mac)
- Q4_K_M at 128K context: ~48 GB RAM (requires 48+ GB Mac or multi-GPU)
- BF16 full precision: ~55 GB VRAM (multi-GPU required)

### Throughput on Consumer Hardware

#### API-level (Mistral's infrastructure)
- Output speed: 129.6 - 150 tok/s (Mistral's servers)
- Time to first token: 0.70s

#### Local Inference Estimates (Q4_K_M, ~14 GB)

| Hardware | Estimated tok/s | Notes |
|---|---|---|
| **RTX 4090 (24 GB VRAM)** | 40-55 | Fits entirely in VRAM at Q4; fast |
| **Mac M4 Pro 48 GB** | 20-30 | Memory bandwidth limited (~273 GB/s) |
| **Mac M4 Max 64/128 GB** | 30-45 | Higher bandwidth (~546 GB/s) |
| **Mac M3 Max 96 GB** | 25-40 | ~400 GB/s bandwidth |
| **Mac M2 Ultra 192 GB** | 35-50 | ~800 GB/s bandwidth |
| **Mac M4 Pro 24 GB** | 15-25 | Tight fit; swap likely at longer context |

**Key insight:** Memory bandwidth determines speed on Apple Silicon, not compute.

### Runtime Support

| Runtime | Support | Notes |
|---|---|---|
| **Ollama** | Yes | `ollama run mistral-small3.1` — requires Ollama 0.6.5+. Default tag is Q4_K_M (~15 GB). |
| **LM Studio** | Yes | Available in GGUF and MLX formats. Min 14 GB RAM. |
| **llama.cpp** | Yes | Full support via GGUF. All quantization levels available. |
| **vLLM** | Yes | Recommended for production. Requires v0.8.1+ and mistral_common v1.5.4+. |
| **HuggingFace Transformers** | Yes | v4.50.0+. Native `Mistral3ForConditionalGeneration`. |
| **MLX** | Yes | Apple Silicon native. Available via LM Studio MLX format. |

#### Ollama Quick Start

```bash
ollama run mistral-small3.1
# Specific quantization:
ollama run mistral-small3.1:24b-instruct-2503-q4_K_M
```

#### vLLM Production Setup

```bash
vllm serve mistralai/Mistral-Small-3.1-24B-Instruct-2503 \
  --tokenizer_mode mistral \
  --config_format mistral \
  --load_format mistral \
  --tool-call-parser mistral \
  --enable-auto-tool-choice \
  --limit_mm_per_prompt 'image=10' \
  --tensor-parallel-size 2
```

### Special Capabilities

1. **Vision/Multimodal:** State-of-the-art image understanding via Pixtral vision encoder. Processes text + images. Scores 94% on DocVQA, 86% on ChartQA — strong for document analysis.
2. **Function Calling:** Native tool calling with structured JSON output. Mistral tool-call parser in vLLM.
3. **128K Context:** Full 128K token context window. RULER 128K score of 81.2% shows functional long-context capability.
4. **Multilingual:** 24+ languages with strong European language support (75.3% multilingual MMLU).
5. **Agent-Centric:** Optimized for agentic workflows with tool use.

### Mistral Sources

- [Mistral Small 3.1 Official Announcement](https://mistral.ai/news/mistral-small-3-1)
- [HuggingFace Model Card: Mistral-Small-3.1-24B-Instruct-2503](https://huggingface.co/mistralai/Mistral-Small-3.1-24B-Instruct-2503)
- [HuggingFace config.json](https://huggingface.co/mistralai/Mistral-Small-3.1-24B-Instruct-2503/blob/main/config.json)
- [bartowski GGUF Quantizations](https://huggingface.co/bartowski/mistralai_Mistral-Small-3.1-24B-Instruct-2503-GGUF)
- [Ollama: mistral-small3.1](https://ollama.com/library/mistral-small3.1)
- [LM Studio: Mistral Small](https://lmstudio.ai/models/mistral-small)
- [Artificial Analysis: Mistral Small 3.1](https://artificialanalysis.ai/models/mistral-small-3-1)
- [OpenRouter: Mistral Small 3.1](https://openrouter.ai/mistralai/mistral-small-3.1-24b-instruct:free)
- [VentureBeat: Mistral Small 3.1 to 3.2 Update](https://venturebeat.com/ai/mistral-just-updated-its-open-source-small-model-from-3-1-to-3-2-heres-why/)

---

## Head-to-Head Comparison

### Benchmark Overlap

Unfortunately, Mistral did not publish scores on the exact same benchmark suite used by PrismML for Bonsai. The overlap is limited:

| Benchmark | Mistral Small 3.1 (24B) | Bonsai 8B (1.15 GB) | Comparison |
|---|---|---|---|
| **MMLU** / MMLU-R | 80.62 (MMLU) | 65.7 (MMLU-R) | Different variants; Mistral wins but not apples-to-apples |
| **GSM8K** | ~69 (reported) | **88.0** | Bonsai significantly higher — surprising |
| **HumanEval** / HumanEval+ | 88.41 (HumanEval) | 73.8 (HumanEval+) | Different variants; HumanEval+ is harder |
| **IFEval** | Not published* | 79.8 | Mistral uses internal IFEval variant, scores not comparable |
| **BFCL** | Not published | 65.7 | |
| **MuSR** | Not published | 50.0 | |

*Mistral states they use an internal version of IFEval that "fixes some issues with the public version" and scores are not comparable with publicly shared scores.

### Side-by-Side for ManiLocal

| Dimension | Mistral Small 3.1 (24B) | Bonsai 8B | Winner |
|---|---|---|---|
| **Disk Size (Q4)** | 14.33 GB | 1.15 GB | Bonsai (12.5x smaller) |
| **RAM at 8K ctx** | ~16 GB | ~3.7 GB | Bonsai (4.3x less) |
| **MMLU-class** | 80.62 | 65.7 (MMLU-R) | Mistral (different benchmarks) |
| **GSM8K** | ~69 | 88.0 | Bonsai (significantly) |
| **HumanEval-class** | 88.41 | 73.8 (HumanEval+) | Mistral (different benchmarks) |
| **IFEval** | Not published | 79.8 | Cannot compare |
| **BFCL** | Not published | 65.7 | Cannot compare |
| **Context Length** | 128K | 65K | Mistral |
| **Vision** | Yes (strong) | No | Mistral |
| **Function Calling** | Yes (native) | Yes | Both |
| **tok/s M4 Pro 48GB** | 20-30 (est.) | 85-136 | Bonsai (4-5x faster) |
| **tok/s RTX 4090** | 40-55 (est.) | 368 | Bonsai (7-9x faster) |
| **Ollama support** | Yes (stable) | No (custom fork) | Mistral |
| **LM Studio support** | Yes | No | Mistral |
| **Multilingual** | 24+ languages | Limited | Mistral |
| **License** | Apache 2.0 | Apache 2.0 | Tie |

---

## Combined Assessment for ManiLocal

### Bonsai 8B Strengths

- **Absurdly small footprint.** The 8B model at 1.15 GB is smaller than most 1B models. Trivially fits on any Mac.
- **Fast inference.** 85-136 tok/s on M4 Pro for the 8B — very usable for interactive work.
- **Apache 2.0.** No license restrictions.
- **65K context on 8B.** Useful for longer documents, though untested for content workflows.
- **Function calling support.** BFCL score of 65.7 indicates tool-use capability.

### Bonsai 8B Concerns

- **Quality gap vs full-precision peers.** The ~9 point average gap versus Qwen3 8B is real. Most noticeable on knowledge (MMLU-R: 65.7 vs 83.0) and coding (HumanEval+: 73.8 vs 82.3).
- **Custom runtime required.** Must build PrismML's llama.cpp fork — no Ollama/LM Studio support yet. This adds friction and means you're running a non-upstream binary.
- **Brand new (day 1).** No independent evaluations. No community testing of edge cases.
- **Content management suitability unproven.** No benchmarks for summarization, long-form editing, or classification — the core ManiLocal use cases.
- **No eval data for 4B/1.7B.** Only throughput numbers published.

### Mistral Small 3.1 Strengths

- **Mature ecosystem.** Full Ollama, LM Studio, llama.cpp, vLLM, MLX support. No custom forks needed. Zero friction to get running.
- **Vision capability.** DocVQA 94%, ChartQA 86% — genuinely useful for document understanding and content extraction from images/PDFs.
- **128K context.** Double Bonsai's context window. RULER 128K score of 81.2% means it actually works at long context, not just claims it.
- **Strong knowledge benchmarks.** MMLU 80.6%, MMLU Pro 66.8% — significantly better for knowledge-intensive content tasks than Bonsai's 65.7 MMLU-R.
- **Multilingual.** Real multilingual capability for content management across languages.
- **Function calling.** Native, well-tested tool use for pipeline integration.

### Mistral Small 3.1 Concerns

- **Size and speed.** At Q4_K_M (14.33 GB), it is 12.5x larger than Bonsai and 4-5x slower on M4 Pro. Interactive use will feel noticeably slower at 20-30 tok/s vs 85-136 tok/s.
- **Needs 32+ GB Mac.** Practical minimum is 32 GB for reasonable context lengths. Bonsai runs comfortably on 16 GB.
- **GSM8K surprisingly low.** The ~69 GSM8K score (if accurate) is well below Bonsai's 88.0. Math reasoning may be a weakness relative to model size.
- **Missing benchmark overlap.** Mistral did not publish IFEval, BFCL, or MuSR scores, making direct comparison with Bonsai difficult on the instruction-following and function-calling dimensions.
- **24B is the upper end for local comfort.** On an M4 Pro 48GB, it fits but leaves limited headroom for long context or concurrent tasks.

### Recommendation

**These models serve different niches — they are complementary, not competing.**

- **Use Mistral Small 3.1 for:** Quality-critical content work, document understanding with images, long-context summarization, multilingual tasks, and agentic pipelines where you need stable runtime support today.
- **Use Bonsai 8B for:** Fast drafting, high-throughput batch processing, constrained-memory environments, and tasks where speed matters more than peak quality.

For a 48GB M4 Pro setup:
1. Mistral Small 3.1 Q4_K_M (14 GB) as the primary quality model — comfortable fit with room for 32K context.
2. Bonsai 8B (1.15 GB) as the speed model — instant responses for iterative/interactive work.
3. Both can run simultaneously with memory to spare.

### Next Steps

1. Install Mistral via Ollama (`ollama run mistral-small3.1`) and test against actual content management prompts.
2. Benchmark tok/s on your specific hardware to replace estimates with measured numbers.
3. Test Mistral's vision capabilities on real document extraction tasks — strongest differentiator vs Bonsai.
4. Compare long-context summarization quality at 32K and 64K tokens.
5. Wait for independent Bonsai benchmarks and community feedback (give it 2-4 weeks).
6. Monitor upstream llama.cpp for Q1_0_g128 kernel merge — this would eliminate Bonsai's main friction point.
7. If evaluating Bonsai now, use the demo setup script and test against your actual content management prompts.
8. Compare Bonsai directly against Qwen3 8B Q4_K_M (~4.9 GB) on content tasks — the quality/size tradeoff may favor the larger quant.
