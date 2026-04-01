# Architecture — ManiLocal

## System Overview

```
┌─────────────────────────────────────────────┐
│                Content Input                 │
│  (documents, notes, knowledge base files)    │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│           Pipeline Orchestrator              │
│  (scripts/pipelines — bash + python)         │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│          Local Inference Runtime             │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐ │
│  │  Ollama   │  │llama.cpp │  │  vLLM     │ │
│  └──────────┘  └──────────┘  └───────────┘ │
│         (localhost:11434 etc.)               │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│            Content Output                    │
│  (summaries, tags, edited drafts, reports)   │
└─────────────────────────────────────────────┘
```

## Design Decisions

### D-01: Runtime Abstraction
Pipelines interact with models via HTTP API (Ollama-compatible `/api/generate` or OpenAI-compatible `/v1/chat/completions`). This allows swapping runtimes without changing pipeline logic.

### D-02: Configuration-Driven Pipelines
Each pipeline is defined by a YAML config specifying: model, prompt template, input format, output format, and quality thresholds. No hardcoded model references in scripts.

### D-03: Apple Silicon First
All benchmarks and configurations target Apple Silicon with Metal acceleration. Other platforms are out of scope unless explicitly added.

### D-04: File-Based I/O
Pipelines read from and write to the filesystem. No database dependencies. Input and output directories are configurable per pipeline.

## Model Selection Criteria

1. **Task quality** — output accuracy on content management tasks
2. **Speed** — tokens/second on target hardware
3. **Memory footprint** — VRAM/unified memory usage
4. **License** — must permit local/private use
5. **Quantization tradeoffs** — quality vs. resource usage at Q4, Q5, Q6, Q8 levels
