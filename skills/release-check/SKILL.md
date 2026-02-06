---
name: release-check
description: Check compatibility between Dynamo and SGLang versions before releases
user-invocable: true
---

# Release Compatibility Check

Verify compatibility between Dynamo and SGLang versions before a release. This catches breaking changes at the integration boundary.

## Step 1: Gather Version Information

Check current versions and recent changes:

!`cd /home/ubuntu/dynamo && git log --oneline -10`

!`cd /home/ubuntu/sglang && git log --oneline -10`

!`cd /home/ubuntu/dynamo && git describe --tags --always 2>/dev/null || echo "no tags"`

!`cd /home/ubuntu/sglang && git describe --tags --always 2>/dev/null || echo "no tags"`

## Step 2: Check API Compatibility

### ConfigArgumentMerger

The ConfigArgumentMerger is a key integration point where Dynamo merges its configuration with SGLang's argument parser. Check for breaking changes:

1. **Dynamo side**: Look at how Dynamo creates and passes configuration:
   - `components/src/dynamo/sglang/` - SGLang wrapper configuration
   - Check for changes to config structs or argument passing

2. **SGLang side**: Look at SGLang's argument parser:
   - `python/sglang/srt/server_args.py` - Server arguments
   - `python/sglang/srt/configs/` - Configuration files
   - Check for added/removed/renamed arguments

3. **Verify no mismatches**: Ensure every argument Dynamo passes is still accepted by SGLang.

### Argument Parsing

Check for changes in how arguments flow through the system:

1. Look at Dynamo's launch scripts: `examples/backends/sglang/launch/`
2. Compare with SGLang's expected arguments
3. Check for deprecated or renamed flags

## Step 3: Check Python Binding Compatibility

The Rust-Python boundary is a common source of breakage:

1. **Check PyO3 bindings**: `lib/bindings/python/src/`
2. **Check Python imports**: Ensure all SGLang modules that Dynamo imports still exist
3. **Check function signatures**: Ensure functions Dynamo calls have not changed their signatures

```bash
cd /home/ubuntu/dynamo && grep -r "from sglang" components/ lib/ --include="*.py" | head -20
cd /home/ubuntu/dynamo && grep -r "import sglang" components/ lib/ --include="*.py" | head -20
```

## Step 4: Run Integration Smoke Test

Build and run a minimal test to verify the integration works:

1. **Build Dynamo**:
   ```bash
   cd /home/ubuntu/dynamo/lib/bindings/python && maturin develop --uv && cd /home/ubuntu/dynamo && uv pip install -e .
   ```

2. **Run aggregated SGLang example** (simplest integration test):
   ```bash
   bash /home/ubuntu/dynamo/examples/backends/sglang/launch/agg.sh
   ```

3. **Send test request and verify response**

## Step 5: Report

Summarize findings:

- **Dynamo version**: [version/commit]
- **SGLang version**: [version/commit]
- **API compatibility**: [pass/fail with details]
- **Python binding compatibility**: [pass/fail with details]
- **Integration test**: [pass/fail with details]
- **Breaking changes found**: [list any issues]
- **Recommendation**: [safe to release / needs fixes]
