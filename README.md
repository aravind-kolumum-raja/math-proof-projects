# Mathematical Proof Projects

Formal verification and computational mathematics research workspace centered on **Lean 4** (Apple Silicon M5 Pro, 48 GiB unified memory), Python numerical/symbolic tooling, and Rust/C high-performance verification utilities.

> **Workspace Layout Convention**: This top-level repository tracks workspace documentation and ignore/editor configuration. Standalone sub-repositories (`zeta3-closed-form/`) and large external clones / multi-gigabyte proof archives (`external/`) are intentionally git-ignored here so each project maintains clean independent history.

## Workspace Structure

```text
math-proof-projects/
├── README.md                  # Workspace overview, hardware benchmarks, and verification runbooks
├── .gitignore                 # Excludes nested repos, external archives, Lean/Python/Rust/LaTeX outputs
├── .editorconfig              # Shared formatting defaults across Lean, Python, Rust, LaTeX, and Markdown
├── zeta3-closed-form/         # Standalone repo (aravind-kolumum-raja/zeta3-closed-form, git-ignored here)
│   ├── Zeta3/                 # Lean 4 modules (leanprover/lean4:v4.36.0)
│   ├── docs/                  # Compendium, strategy notes, and experiments report
│   ├── scripts/               # High-precision Python verification & discovery scripts
│   └── tools/                 # Rust search engines (rv_hunter, pcf_hunter)
└── external/                  # External clones & proof export archives (git-ignored here)
    ├── riemann-hypothesis/    # leanprover-community/riemann-hypothesis
    ├── rh-export/             # Pre-compiled NDJSON/Zstd proof export & SHA256SUMS
    ├── con-leche/             # leanprover/con-leche independent Lean verifier
    ├── con-ron/               # leanprover/con-ron Rust port of con-leche
    └── arxiv-2609.26980/      # Reference TeX & C verification bundle (Giraudin 2026)
```

## Projects

### 1. `zeta3-closed-form/` — Closed Form & Arithmetic Nature of $\zeta(3)$

Standalone repository: [`aravind-kolumum-raja/zeta3-closed-form`](https://github.com/aravind-kolumum-raja/zeta3-closed-form)

An active **Lean 4** (`v4.36.0`), **Python**, and **Rust** research repository tackling the closed-form problem and irrationality measure for Apéry's constant $\zeta(3) = \sum_{n=1}^\infty n^{-3}$:

- **Mathematical Compendium & Reports**: `docs/ZETA3_COMPENDIUM.md`, `docs/EXPERIMENTS_REPORT.md`, `docs/HIGHEST_PROBABILITY_STRATEGY.md`, and `docs/LITERATURE_100_SEARCH_ARCHIVE.md`.
- **Lean 4 Formalization**: `Zeta3.lean` and `Zeta3/` formalizing Apéry's recurrence, rational approximations, Plouffe's Lambert equivalence, structural identities, Rhin–Viola coset splitting certificates (`Zeta3/RhinViolaCert.lean`), and exact open-problem statements ($\zeta(3)/\pi^3 \notin \mathbb{Q}$, transcendence, algebraic independence with $\pi$).
- **Computational Workbench**: High-precision Python scripts (`scripts/zeta3_lab.py`, `scripts/certify_rv_bound.py`, `scripts/rhin_viola_lab.py`, `scripts/cm_lambert_discovery.py`, `scripts/cdt_capacity_test.py`, `scripts/verify_pcf_hits.py`) and Rust engines under `tools/`.

#### Setup & Verification (`zeta3-closed-form`)

```bash
# Clone the standalone repository inside this workspace (if not already present)
git clone https://github.com/aravind-kolumum-raja/zeta3-closed-form.git
cd zeta3-closed-form

# Build and verify all Lean 4 targets (requires elan / Lean v4.36.0)
lake build

# Run high-precision Python verification scripts (Python 3.11+ with mpmath)
python3 scripts/zeta3_lab.py
python3 scripts/certify_rv_bound.py
```

### 2. Local Verification of the 2026 Riemann Hypothesis Lean Proof (`external/`, git-ignored)

Local checkout and independent verification tooling for the November 20, 2026 Lean 4 proof of RH & GRH for Dirichlet $L$-functions (Levent Alpöge & Claude):

- `external/riemann-hypothesis/` — Shallow clone of [`leanprover-community/riemann-hypothesis`](https://github.com/leanprover-community/riemann-hypothesis) (community-maintained cleanup of the 175,187-file proof).
- `external/rh-export/` — Pre-compiled export archive (`solution.ndjson.zst`, `challenge.ndjson.zst`, `comparator.json`, `SHA256SUMS`, `audit.log`) mirrored from Andrew Sutherland's [verification page](https://math.mit.edu/~drew/grh/).
- `external/con-leche/` — Independent Lean checker with a formal consistency proof ([`leanprover/con-leche`](https://github.com/leanprover/con-leche)).
- `external/con-ron/` — High-performance Rust port of `con-leche` ([`leanprover/con-ron`](https://github.com/leanprover/con-ron)).
- `external/arxiv-2609.26980/` — Reference TeX source (`zeta3_bound.tex`) and ancillary C verification code for Giraudin's September 2026 $\mu(\zeta(3))$ bound.

#### Local Hardware Notes (Apple M5 Pro, 48 GiB Unified Memory)

Measured peak RSS from [Drew Sutherland's logs](https://math.mit.edu/~drew/grh/) on the full 61.57 GB unpacked export (`solution.ndjson`):

| Verifier | Peak RSS | Threads | Fit on 48 GiB M5 Pro |
| --- | ---: | --- | --- |
| `con-ron --compare` (Rust) | **44.6 GiB** | Multi-threaded | **Best fit** — fits in RAM (close heavy apps first) |
| `con-leche --compare` (Lean) | **45.8 GiB** | Multi-threaded | Fits tightly in RAM (minimal macOS compression/swap) |
| `nanoda_bin` (Rust) | 84.8 GiB | Multi-threaded | Exceeds 48 GiB (heavy SSD swap) |
| `leanchecker --from-export` | 82.6 GiB | Single-threaded | Exceeds 48 GiB; use `MIMALLOC_GENERIC_COLLECT=1000000` on ≥96 GiB RAM |
| `leanchecker-paranoid --from-export` | 98.8 GiB | Single-threaded | Exceeds 48 GiB; requires ≥128 GiB RAM |

#### Quick Commands for RH Export Verification

```bash
# 1. Decompress exports (needs ~70 GB free disk; --long=30 is required for solution)
cd external/rh-export
zstd -d --long=30 solution.ndjson.zst
zstd -d --long=27 challenge.ndjson.zst
shasum -a 256 solution.ndjson challenge.ndjson
# Expected SHA-256 checksums:
# 5ea74b912147c5c93cc2b6d70cc727cf908df2f62ac6a20480a3fa02f6be244e  solution.ndjson
# bd484ac9886a96f26b1d9a7a59ebd45b7a70d102c015d4f31f9302bbdd6ab2b9  challenge.ndjson

# 2. Fastest local independent verification + statement match (con-ron, ~44.6 GiB RSS)
../con-ron/target/release/con-ron --progress=10% --compare challenge.ndjson solution.ndjson

# 3. Alternative independent Lean checker (con-leche, ~45.8 GiB RSS)
../con-leche/.lake/build/bin/con-leche --progress=5% --compare challenge.ndjson solution.ndjson

# 4. If running Lean's single-threaded default kernel on a ≥96 GiB machine:
MIMALLOC_GENERIC_COLLECT=1000000 leanchecker --from-export solution.ndjson
```

## Tooling Prerequisites

- **Lean 4 / Lake**: [`elan`](https://github.com/leanprover/elan) (auto-selects `lean-toolchain` per subproject)
- **Python**: Python 3.11+ with `mpmath` / virtual environment (`.venv/`)
- **Rust**: `rustup` / `cargo` (for `con-ron`, `rv_hunter`, `pcf_hunter`)
- **Archives & TeX**: `zstd` (for `.ndjson.zst` exports), TeX Live / `latexmk` (optional, for `.tex` manuscripts)
