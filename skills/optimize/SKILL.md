---
name: optimize
description: Profile-driven performance optimization loop. Baseline, profile, optimize one thing, measure, repeat.
user-invocable: true
---

# Optimize

Profile-driven performance optimization loop. Every optimization starts with measurement and ends with measurement. Never guess where the bottleneck is -- profile first, then fix the thing that actually matters.

## Core Rule

**No optimization without a baseline number.** If you cannot measure it, you cannot optimize it. Establish a number before touching any code.

## Process

### Step 1: Understand the Workload

Ask targeted questions in batches of 2-3. Focus on what matters for performance.

**What and where:**
- "What operation is slow? What is the current latency/throughput/resource usage?"
- "What is the target? Give me a number -- X ms, Y requests/sec, Z MB."
- "What is the hot path? Where does this code spend most of its time?"

**Workload characteristics:**
- "What does the input look like? Size, shape, frequency, concurrency level."
- "Is this CPU-bound, memory-bound, I/O-bound, or GPU-bound?"
- "What is the expected load? Peak vs sustained."

**Constraints:**
- "What cannot change? API signatures, data formats, dependencies."
- "What hardware are we targeting? CPU cores, GPU memory, available RAM."
- "Are there correctness properties that must be preserved?"

**Guidelines:**
- 2-3 questions at a time
- Read the code before asking -- do not ask what you can figure out
- If the user says "it's slow" without numbers, the first task is getting numbers

### Step 2: Baseline

Before any optimization, establish measurable baselines. This is non-negotiable.

1. **Find or write a benchmark** -- use existing benchmarks if available. If not, write a minimal reproducible one.
2. **Run the benchmark** -- capture numbers: latency (p50/p99), throughput, memory usage, CPU/GPU utilization.
3. **Identify the bottleneck** -- use profiling tools, not intuition.

**Profiling approaches by language:**

| Language | CPU | Memory | Tracing |
|---|---|---|---|
| Python | `py-spy`, `cProfile`, `line_profiler` | `tracemalloc`, `memray` | `viztracer` |
| Rust | `perf`, `flamegraph`, `criterion` | `heaptrack`, `DHAT` | `tracing` + `chrome-tracing` |
| CUDA/GPU | `nsys`, `ncu`, `nvidia-smi dmon` | `nvidia-smi` | `nsys` timeline |

3. **Record the baseline** -- write it down explicitly:

```
Baseline (before optimization):
- Metric: [what you measured]
- Value: [number with units]
- Conditions: [input size, concurrency, hardware]
- Bottleneck: [what profiling identified]
```

Present to the user before proceeding.

### Step 3: Optimize One Thing

Pick the single highest-impact optimization based on profiling. One change at a time -- never batch multiple optimizations before measuring.

**Common optimization categories (check in this order):**

1. **Algorithm/data structure** -- wrong algorithm dominates everything else. O(n^2) -> O(n log n) beats any micro-optimization.
2. **Unnecessary work** -- are we computing things we do not need? Redundant copies, repeated calculations, unused results.
3. **Memory layout** -- cache misses, fragmentation, excessive allocation. Reuse buffers, prefer contiguous memory, avoid pointer chasing.
4. **Concurrency** -- can work be parallelized? Are there lock contention issues? Is async I/O being serialized?
5. **I/O** -- batching, buffering, async. Are we making round trips we could batch? Blocking where we could be async?
6. **Language-level** -- Python -> Rust for CPU-bound work. Vectorization. SIMD. These are last resort, not first.

**Implementation rules:**
- Minimal diff. Change only what the profiler told you to change.
- Do not "clean up" surrounding code. Do not refactor. Optimize.
- If the optimization requires architectural changes, stop and discuss with the user first.
- Keep the old code path available (behind a flag or comment) until the new one is validated.

### Step 4: Measure

Run the exact same benchmark from Step 2 with the exact same conditions.

```
Iteration N result:
- Change: [one sentence describing what changed]
- Before: [baseline or previous iteration number]
- After: [new number]
- Delta: [% improvement or regression]
- Bottleneck shifted to: [what profiling now shows as the top cost]
```

**If regression:** revert immediately. Understand why before retrying.

**If improvement < 5%:** flag it. Ask the user if it is worth the added complexity.

**If improvement is significant:** commit the change as a checkpoint.

### Step 5: Loop or Finish

After each iteration, present the cumulative results:

```
Optimization summary:
- Baseline: [original number]
- Current: [latest number]
- Total improvement: [%]
- Iterations: [N]
- Remaining bottleneck: [what profiling shows]
```

**Continue if:**
- Still below the target number from Step 1
- Profiling shows a clear next bottleneck worth attacking
- User wants more

**Stop if:**
- Target is met
- Remaining bottleneck is outside our control (hardware limit, external dependency)
- Amdahl's law -- the hot path is now <10% of total time and further gains are marginal
- User says stop

If continuing, go back to Step 3 with the new profiling data. The bottleneck usually shifts after each optimization -- re-profile, do not assume.

## Behavior Notes

- **Profile, do not guess.** The most common optimization mistake is optimizing the wrong thing. Always let profiling data drive decisions.
- **One change at a time.** If you batch two optimizations and get a 20% improvement, you do not know which one mattered. Worse, one might be a regression masked by the other.
- **Numbers in every message.** Every update to the user should include a measurement. "It's faster" is not acceptable -- "p99 latency dropped from 12ms to 8ms" is.
- **Preserve correctness.** Run the test suite after every change. A fast wrong answer is worse than a slow right one.
- **Commit after each validated iteration.** Do not accumulate uncommitted optimization changes -- if something goes wrong you lose all progress.
- **Be autonomous between checkpoints.** Do not ask permission to read files, run benchmarks, or profile. Ask at decision points: which optimization to pursue, whether to continue, whether an architectural change is acceptable.
