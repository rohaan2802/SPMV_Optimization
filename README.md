# SPMV Optimization — Numerical Computing Project (CS-2008)

Course project on **sparse matrix–vector multiplication** `y = A x`: reports, slides, rubric, and a captured **host fingerprint** (`system_architecture.txt`) used as the experimental machine. Source kernels are not in this submission folder; methodology lives in the PDFs. Closely related runnable SpMV code is **Multithreading Task 2** (`webbase-1M.mtx`, COO→CSR, pthreads).

**Author:** Mohammad Rohaan · **Student ID:** 22I-2327 · **GitHub:** [rohaan2802](https://github.com/rohaan2802) · **Course:** CS-2008 Numerical Computing

---

## Table of contents

1. [Problem and context](#problem-and-context)
2. [Features](#features)
3. [Architecture (host + SpMV)](#architecture-host--spmv)
4. [Algorithms](#algorithms)
5. [File-by-file reference](#file-by-file-reference)
6. [I/O formats](#io-formats)
7. [Tech stack](#tech-stack)
8. [Repository structure](#repository-structure)
9. [Build and run](#build-and-run)
10. [Usage](#usage)
11. [Constants (machine profile)](#constants-machine-profile)
12. [Results](#results)
13. [Limitations](#limitations)
14. [How to extend](#how-to-extend)
15. [Author](#author)

---

## Problem and context

SpMV is the workhorse of iterative solvers (CG, GMRES), graph analytics, and FEM assembly. Dense BLAS `gemv` wastes work on zeros; sparse formats (COO, CSR, CSC, ELL, SELL-C-σ) trade storage for irregular gathers.

This repository is the **NC_PROJECT_SUBMISSION** packet: problem statement PDF, full report, summary report, PowerPoint, rubric, a Word note (`1.docx`), and `system_architecture.txt` — a concatenated `lscpu` + `free -h` dump of the laptop used for timings.

The recorded host is a **Broadwell U-series** dual-core (4 threads) under **Microsoft Hyper-V** (WSL/Windows guest). Cache sizes and SMT matter: CSR SpMV is typically **memory-bound**; 3 MiB L3 and 7.7 GiB RAM bound matrix size before swapping.

---

## Features

- Reproducible hardware appendix (`system_architecture.txt`).
- Full vs summary PDF split for graders.
- Presentation deck for viva.
- Rubric + project brief (`CS-2008_ Numerical Computing-Project.pdf`).
- CPU flags include **AVX2**, **FMA**, **BMI1/2** — relevant if the report’s kernels use SIMD FMA on packed nonzeros.
- Speculative-execution mitigations listed (PTI, retpolines) — they add a small overhead vs bare metal.

---

## Architecture (host + SpMV)

### Host (`system_architecture.txt` verbatim facts)

| Field | Value |
|-------|--------|
| Architecture | x86_64, little endian |
| Address | 39-bit physical, 48-bit virtual |
| Logical CPUs | **4** (online 0–3) |
| Model | Intel Core **i5-5300U @ 2.30 GHz** (family 6, model 61, stepping 4) |
| Topology | **2 cores / 2 threads per core / 1 socket** |
| Caches | L1d **64 KiB (2 instances)**, L1i 64 KiB (2), L2 **512 KiB (2)**, L3 **3 MiB (1)** |
| NUMA | 1 node, CPUs 0–3 |
| Hypervisor | **Microsoft**, virtualization type **full** |
| BogoMIPS | 4589.37 |
| Memory | **7.7 GiB** total, ~6.2 GiB free at capture, **2.0 GiB swap** unused |
| Selected flags | `sse sse2 ssse3 sse4_1 sse4_2 avx avx2 fma f16c popcnt aes bmi1 bmi2 rdseed adx smap hle rtm hypervisor` |

Security block (for honesty in a performance write-up): Meltdown **PTI**; Spectre v2 **retpolines**; SSB disabled via prctl; MDS/TAA buffer clearing; L1tf PTE inversion.

### SpMV dataflow (standard; as expected in the reports)

```
Matrix Market / SuiteSparse  →  COO (i, j, v)
                               →  CSR: row_ptr[0..m], col_idx[nz], val[nz]
x[n]  (dense)
loop i = 0 .. m-1:
    s = 0
    for k = row_ptr[i] .. row_ptr[i+1]-1:
        s += val[k] * x[col_idx[k]]
    y[i] = s
```

Optimization axes typically covered in NC SpMV projects: format choice, blocking/cache tiling, OpenMP over rows, SIMD on the inner product, alignment, NUMA (N/A here — 1 node), and **not** timing I/O.

---

## Algorithms

Without the PDF text in this extract, the **implementable** algorithms that match this course + the sibling PDC repo are:

1. **COO SpMV** — scatter `y[i] += v * x[j]` (poor cache on `y` if unsorted).
2. **CSR SpMV** — row-wise; good if rows are long enough; inner loop irregular.
3. **CSC** — better for `x` reuse, worse for `y` (atomic/race if parallel).
4. **ELL / padded rows** — SIMD-friendly, waste on power-law degree.
5. **Threaded CSR** — partition rows (static chunks of 1024 as in Multithreading Task 2 V3) vs dynamic steal.

**Complexity:** O(nnz) flops, O(nnz) irregular loads of `x`. Arithmetic intensity ≈ 2 flops per 12+ bytes (int index + double) — **DRAM bound** on i5-5300U.

**Deterministic test vector** used in the PDC twin: `x[j] = ((j+1) % 1000) / 1000.0`, checksum `Σ y[i]`.

---

## File-by-file reference

| File | Role |
|------|------|
| `system_architecture.txt` | Full `lscpu`-style dump + `free -h` two-line memory/swap table |
| `CS-2008_ Numerical Computing-Project.pdf` | Official project statement (GitHub) |
| `FULL PROJECT REPORT_NC.pdf` | Algorithms, tables, plots |
| `SUMMARY REPORT_NC.pdf` | Short walkthrough |
| `numerical project presentation.pptx` | Oral defense slides |
| `Project_Rubric.pdf` | Grading criteria |
| `1.docx` | Supporting notes |

This working directory only extracted `system_architecture.txt` plus TREE/META; open the PDFs from the GitHub `NC_PROJECT_SUBMISSION/` folder for equations and timing tables.

---

## I/O formats

### `system_architecture.txt`

Plain text. First block: `lscpu` keys (`Architecture:`, `CPU op-mode(s):`, … `Vulnerability Tsx async abort:`). Second block: `free -h` columns `total used free shared buff/cache available` for `Mem:` and `Swap:`.

### Typical SpMV matrix (not stored here)

Matrix Market coordinate (see Multithreading README):

```
%%MatrixMarket matrix coordinate real general
m n nnz
i j v
```

---

## Tech stack

x86_64 Linux guest (Hyper-V), Intel Broadwell, GCC/G++ and/or MATLAB as specified in the PDFs, optional OpenMP/BLAS. Reports in PDF/PPTX/DOCX.

---

## Repository structure

```
SPMV_Optimization/
└── NC_PROJECT_SUBMISSION/
    ├── CS-2008_ Numerical Computing-Project.pdf
    ├── FULL PROJECT REPORT_NC.pdf
    ├── SUMMARY REPORT_NC.pdf
    ├── numerical project presentation.pptx
    ├── Project_Rubric.pdf
    ├── system_architecture.txt
    └── 1.docx
```

---

## Build and run

This packet is **document-centric**. To reproduce a CSR SpMV microbenchmark on a similar 2C/4T machine:

```bash
lscpu > system_architecture.txt
free -h >> system_architecture.txt
```

Then follow the full report’s compile flags. Runnable cousin:

```bash
# from the Multithreading repo
gcc Task2_v1.c -O2 -lm -o Task2_v1
./Task2_v1 webbase-1M.mtx
```

SuiteSparse matrices should be timed **after** CSR conversion, best-of-N, same `-O3 -march=native` as the triangle-counting lab if SIMD is claimed.

---

## Usage

- Cite `system_architecture.txt` in any table of GFLOP/s so readers know the 2-core U-series + Hyper-V baseline.
- Use the summary PDF for a 5-minute pass; full report for format diagrams and result tables.
- Compare AVX2 (`avx2` in flags) vs scalar; do not claim AVX-512 (not in flags).

---

## Constants (machine profile)

| Quantity | Value |
|----------|--------|
| Cores / threads | 2 / 4 |
| Nominal clock | 2.30 GHz |
| L3 | 3 MiB |
| RAM | 7.7 GiB |
| Swap | 2.0 GiB |
| SIMD peak (theoretical) | AVX 256-bit, FMA (not AVX-512) |
| NUMA nodes | 1 |

---

## Results

Timing tables are in `FULL PROJECT REPORT_NC.pdf` / `SUMMARY REPORT_NC.pdf` (GitHub). This text file only proves **where** the experiments ran. Memory at capture: 1.2 GiB used, 6.5 GiB available — enough for million-row web graphs if not holding many copies.

---

## Limitations

- **No `.c`/`.m` kernels** in the extracted tree — cannot recompile from this folder alone.
- Hyper-V + PTI/retpoline: numbers are **not** bare-metal peak.
- 2 physical cores: OpenMP scaling past 4 threads will oversubscribe.
- 3 MiB L3: large power-law rows thrash; ELL padding can explode RAM on 7.7 GiB.

---

## How to extend

- Check in CSR/ELL reference implementations next to the reports.
- Automate SuiteSparse sweeps → CSV (matrix name, nnz, time, GFLOP/s, format).
- Re-run on a desktop with AVX-512 and dual NUMA; keep this i5-5300U file as the “laptop” column.

---

## Author

**Mohammad Rohaan**  
Student ID: **22I-2327**  
GitHub: **rohaan2802**  
Numerical Computing (CS-2008) project submission.

Academic integrity: do not copy report text. Matrix datasets keep SuiteSparse/University of Florida licenses.
