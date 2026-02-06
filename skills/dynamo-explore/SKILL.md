---
name: dynamo-explore
description: Explore Dynamo codebase architecture and trace Rust/Python component interactions
user-invocable: true
---

# Dynamo Codebase Explorer

Explore the Dynamo codebase to understand architecture, trace component interactions, and find relevant code for a feature or investigation.

## Key Directory Map

### Core Runtime (Rust)
- `lib/llm/src/` - LLM library: preprocessing, KV management, model execution
- `lib/llm/src/block_manager/` - KV block management and memory allocation
- `lib/llm/src/block_manager/kv_consolidator/tracker.rs` - KV event parsing and consolidation tracking
- `lib/llm/src/preprocessor/` - Request preprocessing, tokenization, chat templates
- `lib/llm/src/preprocessor/prompt/template/` - Chat template handling (Jinja, tokcfg)

### Parsers (Rust)
- `lib/parsers/src/tool_calling/` - Tool call parsers (JSON, XML, Pythonic, Harmony, DSML)
- `lib/parsers/src/tool_calling/config.rs` - Parser configuration presets
- `lib/parsers/src/tool_calling/parsers.rs` - Parser registry and routing

### Python Bindings
- `lib/bindings/python/` - PyO3 bindings exposing Rust to Python
- `lib/bindings/python/src/` - Rust source for Python module

### SGLang Integration
- `components/src/dynamo/sglang/` - SGLang wrapper components
- This is where Dynamo wraps the SGLang runtime with optimized pre/post processing

### Examples and Deployment
- `examples/backends/sglang/` - SGLang backend examples (launch configs, benchmarks)
- `examples/backends/vllm/` - vLLM backend examples
- `examples/backends/trtllm/` - TensorRT-LLM backend examples
- `deploy/` - Deployment configurations (Kubernetes, Docker, Slurm)

### Configuration
- `components/src/dynamo/` - Component configurations and service definitions
- `lib/llm/src/config/` - LLM configuration structs

## Exploration Workflow

When asked about a feature or component:

1. **Identify the layer**: Is it Rust core, Python bindings, SGLang wrapper, or deployment?

2. **Trace the data flow**: For request handling, trace from:
   - Python API entry point (components/src/dynamo/sglang/)
   - Through Rust preprocessing (lib/llm/src/preprocessor/)
   - To KV management (lib/llm/src/block_manager/)
   - Through model execution
   - Back through post-processing

3. **Check cross-language boundaries**: PyO3 bindings in `lib/bindings/python/` bridge Rust and Python. When tracing a feature, check both sides.

4. **Look at tests**: Tests often reveal expected behavior and edge cases. Check both Rust tests (inline `#[cfg(test)]`) and Python tests.

5. **Read examples**: Examples in `examples/backends/` show real usage patterns and configuration options.

## Common Investigation Patterns

### "How does X work?"
1. Find the entry point (grep for function/struct name)
2. Trace the call chain
3. Identify the key data structures
4. Check configuration options
5. Present a flow diagram

### "Where should I add X?"
1. Find similar existing features
2. Identify the right layer (Rust vs Python)
3. Check existing interfaces and traits
4. Look at how similar additions were done (git log)

### "Why does X behave this way?"
1. Find the relevant code path
2. Check git blame for context on when/why it was written
3. Look for related issues or PRs
4. Check if there are configuration flags that affect behavior

## SGLang Relationship

Dynamo wraps SGLang and adds:
- Optimized tokenization (Rust-based, faster than Python)
- KV-aware routing (routes requests based on KV cache state)
- Block management (manages GPU memory for KV blocks)
- Disaggregated inference support (separate prefill/decode workers)

The wrapping layer is in `components/src/dynamo/sglang/`. When debugging SGLang-related issues, check both the Dynamo wrapper and the upstream SGLang code at `~/sglang`.
