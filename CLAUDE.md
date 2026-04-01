# ManiLocal — Local AI Model Research & Content Management

## Project Overview

ManiLocal is Jim Manico's research and planning repository for running local AI models, with a primary focus on content management workflows. The goal is to evaluate, configure, and operationalize local/self-hosted models for content creation, editing, summarization, and knowledge management — without relying on cloud-hosted APIs.

## Key Objectives

- Evaluate local model options (llama.cpp, Ollama, vLLM, LM Studio, etc.)
- Benchmark models for content management tasks (writing, editing, summarization, classification)
- Document hardware requirements and performance characteristics
- Build reproducible local inference setups
- Develop content management pipelines powered by local models

## Project Structure

```
manilocal/
├── CLAUDE.md              # This file — project instructions for Claude
├── README.md              # Project overview and quickstart
├── REQUIREMENTS.md        # Machine-readable requirements
├── ARCHITECTURE.md        # System design and decisions
├── research/              # Model evaluations, benchmarks, notes
├── configs/               # Model configurations and runtime settings
├── scripts/               # Setup, benchmark, and utility scripts
├── pipelines/             # Content management pipeline definitions
└── docs/                  # Extended documentation
```

## Engineering Standards

- All requirements must be testable and explicit
- Scripts must be POSIX-compatible or clearly document dependencies
- Configuration files use YAML unless a tool requires otherwise
- Document hardware specs and OS for all benchmark results
- No cloud API keys or credentials in this repo
- Pin model versions and quantization levels in all references

## Content Management Scope

This project covers local-model-powered workflows for:
- Long-form content drafting and editing
- Document summarization and extraction
- Content classification and tagging
- Knowledge base maintenance
- Batch content processing pipelines

## Security Considerations

- No secrets, API keys, or credentials committed to this repo
- Model files are excluded via .gitignore (large binaries)
- Any web-facing inference endpoints must be localhost-only by default
- Document supply-chain provenance for downloaded models
