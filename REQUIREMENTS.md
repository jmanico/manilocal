# Requirements — ManiLocal

## Hardware Requirements

- **Minimum:** Apple Silicon Mac (M1+) with 16GB unified memory
- **Recommended:** Apple Silicon Mac (M2 Pro/Max+) with 32GB+ unified memory
- **Storage:** 50GB+ free for model files

## Software Prerequisites

- macOS 14+ (Sonoma or later)
- Homebrew
- Python 3.11+
- Git LFS (for any tracked model artifacts)

## Runtime Requirements

### R-RUNTIME-01: Ollama Support
The project must document installation, configuration, and usage of Ollama for local inference.

### R-RUNTIME-02: llama.cpp Support
The project must document building and running llama.cpp with Metal acceleration on Apple Silicon.

### R-RUNTIME-03: Model Format Compatibility
All evaluated models must be documented with their format (GGUF, safetensors, etc.) and quantization level.

## Content Management Requirements

### R-CM-01: Summarization Pipeline
Define a reproducible pipeline for document summarization using local models.

### R-CM-02: Content Classification
Define a pipeline for classifying and tagging content by topic, type, and priority.

### R-CM-03: Batch Processing
Support batch processing of multiple documents with configurable concurrency.

### R-CM-04: Output Quality Benchmarks
Each pipeline must include quality benchmarks comparing output against reference results.

## Security Requirements

### R-SEC-01: No Cloud Dependencies
All pipelines must function fully offline with no cloud API calls.

### R-SEC-02: No Credential Storage
No API keys, tokens, or credentials stored in this repository.

### R-SEC-03: Localhost-Only Inference
Any HTTP-based inference endpoints must bind to 127.0.0.1 by default.

### R-SEC-04: Model Provenance
Document the source, hash, and license for every model file referenced.
