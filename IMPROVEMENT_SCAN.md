# Rust Improvement Scan: clawhdf5
**Date:** 2026-04-27 00:56 UTC
**Rust:** 1.95 stable -- Edition 2024 capable

## Changes Made

### SAFETY Comments -- Unsafe Block Coverage
- clawhdf5-agent/src/blas_search.rs -- Added SAFETY comments to all 4 matrixmultiply::sgemm unsafe calls (pointer validity, sizes, row-major layout)
- clawhdf5-agent/src/accelerate_search.rs -- Added SAFETY to cblas_sgemv (x3), cblas_snrm2 (x1), vDSP_dotpr (x1) calls
- clawhdf5-accel/src/lib.rs -- Added SAFETY to all 9 SIMD dispatch arms (NEON x3, AVX-512 x3, AVX2 x3)
- clawhdf5/src/reader.rs -- Added SAFETY to 4 from_raw_parts zerocopy casts (f64, f32, i32, i64)
- clawhdf5-format/src/data_read.rs -- Added SAFETY to 3 copy_nonoverlapping and 6 from_raw_parts blocks
- clawhdf5-filters/src/fast_deflate.rs -- Added SAFETY to 2 Apple Compression Framework blocks
- clawhdf5-format/src/filters.rs -- Added SAFETY to FFI call block
- clawhdf5-agent/src/knowledge.rs:287 -- Fixed sort_by to sort_by_key (clippy::unnecessary_sort_by regression)

## Security Notes
No known vulnerabilities. All unsafe blocks now have explicit soundness justifications.

## Files Over Limit
None -- all files within 1300-line limit.

## Remaining Opportunities
- avx2/avx512/neon inner SIMD loops inside unsafe fn bodies already covered by fn-level Safety doc
- clawhdf5-napi unsafe impl Send: already has SAFETY comment
- clawhdf5-android extern C fns: all have Safety doc comments
- Flaky timing tests (test_gate_under_1_microsecond, performance_10k_vectors_384d): pre-existing, not related to this scan
