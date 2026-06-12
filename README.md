# PATH — Protocol for Autonomous Traversal and Handshake

**A Threshold-Cryptographic Overlay for Heterogeneous and Non-Terrestrial Transport**

James Pan · 8Bit Labs LLC · Irvine, California
Version 4.0 — June 2026

[![DOI](https://img.shields.io/badge/DOI-pending-blue.svg)](https://zenodo.org)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

---

## What this is

PATH is an overlay construction for **session-establishment traffic** — authentication
handshakes, credential exchange, session-token issuance, and command authorization — in
which an ephemeral session key is split via a (t, n) threshold secret-sharing scheme and
the resulting shares are dispatched across deliberately divergent transport paths,
including at least one non-terrestrial (satellite) path. An adversary who intercepts any
single path holds a share that is mathematically worthless below the reconstruction
threshold.

The cryptographic primitives are individually mature. The contribution is their
**composition** on a transport substrate — cellular, Wi-Fi, NTN satellite, and short-range
mesh, all available on commodity hardware under open standards — combined with a
self-routing payload model and an Interference-Adaptive Band Orchestration (IABO) layer
that together remove the infrastructure dependencies that make current alternatives
fragile.

This repository is a **defensive publication**. It is released publicly with a verifiable
timestamp to establish prior art, so that the architecture described here cannot be
patented by others. See [Citation & prior-art status](#citation--prior-art-status).

## Contents

| File | Description |
|------|-------------|
| `PATH_v4.0.pdf` | The full preprint (also on arXiv / Zenodo — see below). |
| `PATH_v4.0.md` | Markdown source of the preprint. |
| `PATH_v4.0.tex` | LaTeX source. |
| `PATH_supplement_demo_hardware.md` | Phase 1 demonstration build: hardware inventory, radio abstraction / IABO bridge architecture, and test scenarios. Part of the full disclosure. |
| `LICENSE` | CC BY 4.0. |

## Abstract

We describe an overlay construction for session-establishment traffic in which an
ephemeral session key is split via a (t, n) threshold secret-sharing scheme and the
resulting shares are dispatched across deliberately divergent transport paths, including
at least one non-terrestrial path. The construction targets a narrow class of traffic —
authentication handshakes, credential exchange, session-token issuance, and command
authorization — for which the ratio of per-message value to per-message size is high and
conventional TLS-over-IP concentrates vulnerability at the handshake.

The construction extends a core cryptographic model with a proximity-mesh relay layer, a
self-routing payload architecture, an Interference-Adaptive Band Orchestration (IABO)
model for power-efficient multi-band dispatch, a CSS physical layer with satellite link
budget analysis, indoor penetration characterization, a universal mesh architecture
thesis, a return path architecture, a tiered federation operator model, and a phased
commercialization roadmap.

## Status

This is an early-stage architecture document. Several claims are explicitly marked **OPEN**
in the text — they are derivations and analogies awaiting empirical validation, not
measured results. Section 12 ("Open Problems") catalogues them. Treat the security
analysis as informal: formal proofs are future work.

## Citation & prior-art status

This work is released as a defensive publication. If you build on it, please cite it:

> Pan, J. (2026). *PATH — Protocol for Autonomous Traversal and Handshake: A
> Threshold-Cryptographic Overlay for Heterogeneous and Non-Terrestrial Transport*
> (Version 4.0). Zenodo. https://doi.org/[DOI-here]

A canonical, timestamped copy is archived at:

- **Zenodo (DOI of record):** `[paste DOI after upload]`
- **arXiv:** `[paste arXiv ID after acceptance]`
- **OpenTimestamps proof:** `PATH_v4.0.pdf.ots` in this repo (Bitcoin-anchored proof of the exact file bytes and date)

## License

This work is licensed under a
[Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).
You are free to share and adapt the material for any purpose, including commercially,
provided you give appropriate credit.

## Contact

James Pan — 8Bit Labs LLC — `[your contact email]`
