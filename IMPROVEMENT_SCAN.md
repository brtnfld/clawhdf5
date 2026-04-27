# Rust Improvement Scan: clawhdf5
**Date:** 2026-04-26 UTC
**Rust:** 1.95 stable -- Edition 2024
**Iteration:** 20

## Changes Made

Added deny(unsafe_code) to 6 pure-safe lib.rs files:
- crates/clawhdf5-ann/src/lib.rs
- crates/clawhdf5-derive/src/lib.rs
- crates/clawhdf5-gpu/src/lib.rs
- crates/clawhdf5-netcdf4/src/lib.rs
- crates/clawhdf5-py/src/lib.rs
- crates/clawhdf5-types/src/lib.rs

Crates skipped (have unsafe code in submodules):
- crates/clawhdf5-android/ (57 unsafe -- Android JNI bindings)
- crates/clawhdf5-accel/ (9 unsafe -- hardware acceleration)
- crates/clawhdf5-napi/ (1 unsafe -- Node.js bindings)
- crates/clawhdf5-format/ (unsafe extern C -- C filter FFI)
- crates/clawhdf5-io/ (unsafe mmap -- memory-mapped I/O)
- crates/clawhdf5-filters/ (unsafe extern C -- compression codecs)
- crates/clawhdf5/ main crate (unsafe in reader.rs -- raw pointer slice)
- crates/clawhdf5-agent/ (unsafe in accelerate_search.rs)

## Security Notes
No cargo audit findings.

## Files Over Limit
None.

## Remaining Opportunities
- io_uring + O_DIRECT support in clawhdf5-io for NVMe throughput (architecture priority)
- Safe wrapper types to reduce unsafe surface in clawhdf5-format C FFI
