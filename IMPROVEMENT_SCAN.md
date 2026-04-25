# Rust Improvement Scan: clawhdf5
**Date:** 2026-04-25 05:38 UTC
**Rust:** 1.95 stable -- Edition 2024 capable

## Changes Made

### crates/clawhdf5-agent/src/knowledge.rs:3 & :287
- Added `use std::cmp::Reverse;` import
- Changed `pairs.sort_by(|a, b| b.0.len().cmp(&a.0.len()))` to `pairs.sort_by_key(|b| Reverse(b.0.len()))`
- Why: clippy::unnecessary_sort_by lint violation blocking compilation with -D warnings

### crates/clawhdf5-agent/src/strategy.rs:103
- Changed `"gpu".to_string()` to `"gpu".to_owned()`
- Why: .to_owned() is semantically clearer for string literals

### crates/clawhdf5-format/tests/reference_tests.rs:206-243
- Fixed h5py_object_reference_roundtrip test to handle both v1 (SymbolTable/B-tree) and v2 (Link) group formats
- h5py defaults to v0 superblock files using B-tree + local heap for group storage
- Test now uses clawhdf5_format::group_v1::resolve_v1_group_entries() as fallback
- Why: Test panicked because h5py creates old-style HDF5 groups using SymbolTable messages, not Link messages (0x0006). The group_v1 and symbol_table modules already had correct parsing infrastructure; the test just was not using it.

## Security Notes
cargo audit not installed on this host. No known CVEs through manual review.
- No unsafe blocks found in non-test production code (excluding GPU/FFI crates)
- No hard-coded credentials or secrets found

## Files Over Limit (>1300 lines)
Files exceeding the 1300-line guidance:
- crates/clawhdf5-bench/src/bin/memory_arena.rs: 4699 lines (bench binary, low priority)
- crates/clawhdf5-format/src/chunked_read.rs: 2077 lines
- crates/clawhdf5-format/src/data_read.rs: 1913 lines
- crates/clawhdf5-format/src/chunked_write.rs: 1492 lines
- crates/clawhdf5-agent/src/lib.rs: 1439 lines
- crates/clawhdf5-format/src/datatype.rs: 1414 lines
- crates/clawhdf5-agent/src/openclaw.rs: 1378 lines

Splitting these requires careful sub-module design to preserve public API.

## Remaining Opportunities
1. File splits: 7 source files >1300 lines could be split into sub-modules
2. .to_string() on literals: ~15 instances in bench crates (minor)
3. unwrap() in clawhdf5-migrate: 20+ unwraps in test helper sections
4. Edition 2024: already enabled workspace-wide (no upgrade needed)
