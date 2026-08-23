# SPMV_Optimization

Numerical Computing (CS-2008) project submission focused on **sparse matrix-vector multiplication (SpMV)** optimization: reports, presentation, rubric, and a captured **system architecture / host profile** used for performance experiments.

**Course:** Numerical Computing · **Folder:** `NC_PROJECT_SUBMISSION`

---

## Overview

SpMV (`y = A x` for sparse `A`) dominates many scientific kernels. This repository packages the project deliverables for analyzing and optimizing SpMV on a constrained laptop-class host (documented in `system_architecture.txt`), including full and summary reports plus a presentation deck.

The host profile recorded for experiments:

- **CPU:** Intel Core i5-5300U @ 2.30 GHz (2 cores / 4 threads)
- **Caches:** 64 KiB L1d, 512 KiB L2, 3 MiB L3
- **Memory:** ~7.7 GiB RAM
- **Environment:** x86_64 Linux guest (Microsoft Hyper-V)

---

## Features

- Full project report and condensed summary (`FULL PROJECT REPORT_NC.pdf`, `SUMMARY REPORT_NC.pdf`)
- Course project brief / statement (`CS-2008_ Numerical Computing-Project.pdf`)
- Presentation slides (`numerical project presentation.pptx`)
- Rubric and supporting Word notes
- Machine fingerprint for reproducibility (`system_architecture.txt` - `lscpu` / `free`-style dump)

---

## Repository structure

```text
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

## Build / run

This submission is **document-centric**. To reproduce related SpMV experiments on a similar machine:

1. Use Linux/WSL with `gcc`/`g++` and optional OpenMP/BLAS as described in the report.
2. Capture your own host profile:

```bash
lscpu > system_architecture.txt
free -h >> system_architecture.txt
```

3. Follow the methodology in `FULL PROJECT REPORT_NC.pdf` (formats such as CSR/COO, blocking, threading, and timing protocols).

For a related hands-on SpMV code path in this portfolio, see the **Multithreading** repo Task 2 (`webbase-1M.mtx` CSR SpMV with pthreads).

---

## Usage

- Open the PDFs/PPTX with any modern PDF/Office viewer.
- Cite `system_architecture.txt` when comparing timings so readers know the 2C/4T U-series baseline.
- Use the summary report for a quick walkthrough; the full report for algorithm details and result tables.

---

## Extending

- Check in reference SpMV source (CSR/ELL/SELL-C-σ) alongside the reports.
- Automate microbenchmarks across matrix suites (SuiteSparse) with a CSV logger.
- Re-run on a modern desktop/server NUMA node and contrast with the i5-5300U profile.

---

## License

Course project materials - respect academic integrity and any third-party matrix dataset licenses.
