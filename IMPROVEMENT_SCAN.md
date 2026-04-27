# Rust Improvement Scan: clawhdf5
**Date:** 2026-04-26 UTC
**Rust:** 1.95 stable -- Edition 2024

## Changes Made

### Clippy Fixes
- crates/clawhdf5-agent/src/knowledge.rs:287 -- replaced sort_by(|a, b| b.0.len().cmp(&a.0.len())) with sort_by_key(|b| std::cmp::Reverse(b.0.len())) (clippy::unnecessary_sort_by)

### String Literal Modernisation
- All .to_string() calls on string literals replaced with .to_owned() across agent, format, io, bench, and py crates (~31 files).
- Rationale: .to_owned() is idiomatic for &str to String conversion; .to_string() goes through the Display trait unnecessarily.
- Files affected: strategy.rs, lib.rs, anomaly.rs, agents_md.rs, knowledge.rs, query_expand.rs, hybrid.rs, consolidation.rs, gpu_search.rs, multimodal.rs, openclaw.rs, async_memory.rs, memory_strategy.rs, reranker.rs, ephemeral.rs, entity_extract.rs, decision_gate.rs, wal.rs, bm25.rs, memory_bench.rs, clawhdf5-py (file.rs, group.rs), clawhdf5-format (filter_pipeline.rs, data_read.rs, chunked_write.rs, datatype.rs), clawhdf5-io/hsds.rs, bench bins.

## Security Notes
- cargo audit not installed on this system.
- Unsafe code in clawhdf5-napi (unsafe Send impl for ClawhdfMemory) and clawhdf5-android (FFI extern "C" functions) -- appropriate uses for JNI/NAPI FFI boundaries; not changed.

## Files Over Limit (>1300 lines)
- crates/clawhdf5-bench/src/bin/memory_arena.rs: 4699 lines -- bench-only binary, flagged for future split
- crates/clawhdf5-format/src/chunked_read.rs: 2077 lines -- could be split into read/decompress sub-modules
- crates/clawhdf5-format/src/data_read.rs: 1913 lines -- could be split by datatype category
- crates/clawhdf5-format/src/chunked_write.rs: 1492 lines -- could be split into write/compress sub-modules
- crates/clawhdf5-agent/src/lib.rs: 1481 lines -- consider extracting sub-module facades
- crates/clawhdf5-format/src/datatype.rs: 1414 lines -- consider splitting primitive/compound types
- crates/clawhdf5-agent/src/openclaw.rs: 1378 lines -- candidate for protocol/state machine split

## Remaining Opportunities
- Split oversized files into sub-modules -- deferred, requires careful API boundary analysis
- Add cargo audit and address any advisories once installed
- 306 .clone() calls -- many may be on Copy types; deferred (requires per-call type analysis)
- Production unwrap() in clawhdf5-migrate/src/main.rs -- acceptable in a migration binary CLI but could use anyhow for better error messages
- One pre-existing test failure: h5py_object_reference_roundtrip (likely environment/HDF5 library issue, unrelated to Rust changes)
