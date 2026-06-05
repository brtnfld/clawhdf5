# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose
Pure-Rust HDF5 format implementation with HNSW vector search, WAL-backed persistence, agent memory storage, and GPU-accelerated I/O. Used by ZeroClaw as its persistent memory and knowledge graph backend.

## Architecture

Cargo workspace (edition 2024, resolver 2) with 17 crates under `crates/`:

| Crate | Role |
|-------|------|
| `clawhdf5-types` | Shared type definitions and physical constants |
| `clawhdf5-format` | HDF5 binary spec parser/writer — no_std compatible, no C deps |
| `clawhdf5-io` | I/O abstraction: mmap, async (tokio), HSDS |
| `clawhdf5-filters` | Compression filter pipeline (deflate, LZ4, Zstd, Blosc, Apple) |
| `clawhdf5-derive` | Proc-macro derive for HDF5-serializable structs |
| `clawhdf5` | High-level facade (`File`, `FileBuilder`, `Dataset`, `Group`) |
| `clawhdf5-netcdf4` | NetCDF-4 compatibility layer |
| `clawhdf5-ann` | HNSW approximate nearest-neighbor index stored as HDF5 |
| `clawhdf5-agent` | Agent memory engine — 29 modules, ~16K lines |
| `clawhdf5-gpu` | GPU-accelerated vector ops via wgpu compute shaders |
| `clawhdf5-accel` | SIMD acceleration (AVX2, AVX-512, NEON) |
| `clawhdf5-migrate` | CLI tool: migrate SQLite agent memory → HDF5 |
| `clawhdf5-android` | Android JNI bridge (cdylib) for clawhdf5-agent |
| `clawhdf5-cli` | Command-line interface (9 subcommands) |
| `clawhdf5-napi` | Node.js native addon via napi-rs |
| `clawhdf5-py` | PyO3 Python bindings |
| `clawhdf5-bench` | Criterion.rs benchmark harnesses |

## Commands

### Build & Check
```bash
cargo build --release
cargo check --workspace
cargo fmt --check          # CI gate
cargo clippy --all -D warnings  # CI gate — zero warnings required
```

### Test
```bash
# Full workspace (excludes clawhdf5-py, which requires maturin)
cargo test --workspace --exclude clawhdf5-py

# Single test
cargo test -p clawhdf5-agent test_name

# Validate no_std compatibility of clawhdf5-format
bash scripts/check-nostd.sh

# Full CI suite (fmt + clippy + test)
bash scripts/ci-test.sh
```

### CLI
```bash
cargo run -p clawhdf5-cli -- --help
# Subcommands: create, save, search, recall, stats, flush-wal, agents-md, export, snapshot
```

### Benchmarks
```bash
bash scripts/run-benchmarks.sh   # generates BENCHMARKS.md
cargo bench -p clawhdf5-bench
```

### Python bindings
```bash
cd crates/clawhdf5-py
maturin develop
python -c "import clawhdf5; print(clawhdf5.__version__)"
```

## Key Feature Flags

### `clawhdf5` (facade)
- `mmap` (default) — memory-mapped file access
- `fast-deflate` (default) — optimized deflate compression
- `parallel` — Rayon parallel decompression
- `zstd`, `lz4`, `apple-compression`, `blake3_hash`

### `clawhdf5-agent`
- `agent` — full agent memory layer (enable this to use the memory engine)
- `float16` (default) — half-precision embedding storage (2× compression)
- `parallel` — Rayon parallel search
- `fast-math` / `openblas` — BLAS matrix-vector multiply
- `accelerate` — Apple Accelerate / AMX (macOS only)
- `gpu` — GPU search via wgpu
- `async` — Tokio async with background WAL flush

## Agent Memory Architecture (`clawhdf5-agent`)

The memory engine is the most complex subsystem. Core components:

**Search pipeline**: Vector search (cosine similarity with pre-computed norms) + BM25 keyword index → RRF fusion (k=60 reciprocal rank fusion) → multi-factor re-ranker (temporal, authority, activation) → confidence gate (gap filtering).

**Knowledge graph** (`knowledge.rs`): Entity/relation graph with BFS traversal, spreading activation, and entity resolution.

**Memory consolidation** (`consolidation.rs`): Three-tier model — Working → Episodic → Semantic — with importance scoring. Triggered at `compact_threshold` (default 0.3).

**Persistence**: WAL (`wal.rs`) buffers writes up to `wal_max_entries` (default 500) before flushing to main HDF5. Flush via `flush-wal` CLI or `async` feature background flush.

**Security**: `anomaly.rs` rate-limits writes and runs 15 injection-pattern detectors. `provenance.rs` tracks source attribution with FNV-1a hashing and integrity verification.

**Advanced search**: `ivf.rs`/`pq.rs` for IVF with product quantization at scale; `multimodal.rs` for cross-modal search (text/image/audio/video embeddings); `query_expand.rs` for synonym/acronym/morphological expansion.

**`MemoryConfig` defaults**: `chunk_size=512`, `overlap=50`, `embedding_dim=384`, `float16=true`, `wal_enabled=true`.

## HDF5 Core Architecture (`clawhdf5-format`)

40 modules covering the full HDF5 spec: superblock, object headers, B-tree v1/v2, chunked/fixed arrays, data layout (compact/contiguous/chunked/external), fractal/local/global heaps, symbol tables, extensible arrays, and dictionary encoding. Checksums via Jenkins lookup3. Provenance via SHA-256.

Zero-copy read paths exist for contiguous, little-endian f64/f32 datasets (direct mmap slice access via `read_f64_zerocopy()`).

## Integration
ZeroClaw imports this as a Cargo feature (`clawhdf5` feature flag) to persist agent memory with HNSW vector search for context retrieval. See `docs/openclaw-integration.md` for the backend integration details.
