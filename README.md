# SGLang Internals: Interactive Tutorial

An interactive, single-file HTML tutorial that explains how [SGLang](https://github.com/sgl-project/sglang) works under the hood -- from first principles to implementation details.

**[Open the Tutorial](https://odysa.github.io/mini-sglang/)**

## What's Inside

12 chapters covering the full SGLang serving stack:

| # | Chapter | Key Concepts |
|---|---------|-------------|
| 1 | The Big Picture | 3-process architecture, ZMQ pipeline |
| 2 | Tokenization & Lifecycle | Req data structure, 3 input paths |
| 3 | KV Cache & Memory | ReqToTokenPool, TokenToKVPoolAllocator |
| 4 | RadixAttention | Prefix caching, radix tree, LRU eviction |
| 5 | Scheduling | LPM policy, PrefillAdder budgets |
| 6 | Continuous Batching | Chunked prefill, TTFB control |
| 7 | Forward Pass | CUDA graph replay, ForwardMode dispatch |
| 8 | Attention Backends | AttentionBackend ABC, FlashInfer/Triton |
| 9 | Sampling | Temperature, top-k/top-p/min-p pipeline |
| 10 | Constrained Decoding | FSM masking, XGrammar, jump-forward |
| 11 | Parallelism | TP (AllReduce), DP (replicas), hybrid |
| 12 | Speculative Decoding | Draft-verify loop, EAGLE, bonus tokens |

Each chapter includes:
- Real source code snippets from the SGLang codebase with file paths
- Step-by-step "How It Works" explanations with pseudocode
- Interactive demos (memory grid, radix tree visualizer, sampling explorer, speculative decoding simulator)

## Features

- Single self-contained HTML file -- no build step, no dependencies
- Warm parchment design inspired by Claude (Anthropic)
- Sidebar navigation with progress tracking
- Mobile responsive
- 5 interactive JavaScript demos
- 15+ annotated source code blocks from the real SGLang codebase

## Quick Start

```bash
# Clone and open
git clone https://github.com/odysa/mini-sglang.git
open mini-sglang/tutorial.html

# Or just download tutorial.html and double-click it
```

## How It Was Built

This tutorial was generated using a [Claude Code skill](https://code.claude.com/docs/en/skills) that automates multi-phase codebase investigation, chapter writing, and interactive HTML generation. The skill is available at `skill.md`.

## License

MIT
