# PATH — Defensive Publication Record

**Protocol for Autonomous Traversal and Handshake**
James Pan · 8Bit Labs LLC · Irvine, California
Publication date: June 13, 2026

This document is the permanent record of all artifacts and proofs establishing public
prior art for the PATH protocol. Keep this file alongside the paper.

---

## Publication Artifacts

### ORCID (Author Identity)
- **ID:** `0009-0005-2160-9434`
- **URL:** https://orcid.org/0009-0005-2160-9434
- **Name on record:** James Pan
- **Purpose:** Permanent, institution-independent author identifier. Linked to the Zenodo record and all future publications.

---

### Zenodo (Legal Prior-Art Timestamp)
- **DOI:** `10.5281/zenodo.20673141`
- **URL:** https://doi.org/10.5281/zenodo.20673141
- **Published:** June 13, 2026
- **Archive:** CERN (European Organization for Nuclear Research)
- **License:** Creative Commons Attribution 4.0 International (CC BY 4.0)
- **Resource type:** Publication / Preprint
- **Files archived:**
  - `PATH_v4.0.pdf` — full 52-page preprint
  - `PATH_v4.0.md` — markdown source
  - `PATH_v4.0.tex` — LaTeX source
  - `PATH_supplement_demo_hardware.md` — Phase 1 demo build supplement
- **Legal effect:** The moment of Zenodo publication establishes the prior-art date under patent law. Anyone attempting to patent the PATH architecture after June 13, 2026 will find this timestamped record blocking novelty claims.
- **Canonical citation:**
  > Pan, J. (2026). *PATH — Protocol for Autonomous Traversal and Handshake: A Threshold-Cryptographic Overlay for Heterogeneous and Non-Terrestrial Transport* (Version 4.0). Zenodo. https://doi.org/10.5281/zenodo.20673141

---

### OpenTimestamps (Bitcoin Blockchain Proof)
- **File stamped:** `PATH_v4.0.pdf`
- **File size:** 237.5 KB
- **SHA256 hash:** `2f3997f133c9022292e37b44216189aec8b5ad045ecee3e8a6fc26b64628f554`
- **Proof file:** `PATH_v4.0.pdf.ots` (stored in GitHub repo)
- **Timestamp submitted:** June 13, 2026
- **Anchor:** Bitcoin blockchain (attestation finalizes within a few hours of block confirmation)
- **Verify at:** https://opentimestamps.org — drag `PATH_v4.0.pdf.ots` onto the page
- **Purpose:** Trustless, institution-free cryptographic proof that the exact bytes of `PATH_v4.0.pdf` existed on June 13, 2026. Independent of Zenodo, CERN, or any centralized authority. Unforgeable: changing a single byte of the PDF produces a completely different hash that will not verify against this proof.
- **How it works:** OpenTimestamps hashes the file, submits that hash to a Bitcoin transaction, and the resulting block inclusion provides a cryptographic timestamp anchored to the Bitcoin blockchain's proof-of-work. The `.ots` file contains the Merkle path proving inclusion.

---

### GitHub Repository (Discoverable Public Home)
- **URL:** https://github.com/9jcb2jw8gc-cyber/path-protocol
- **Visibility:** Public
- **License:** CC BY 4.0
- **Release:** v4.0 — "PATH v4.0 — Initial Public Release" (tagged June 13, 2026)
- **Release permalink:** https://github.com/9jcb2jw8gc-cyber/path-protocol/releases/tag/4.0
- **Files in repo:**
  - `PATH_v4.0.pdf` — full preprint
  - `PATH_v4.0.md` — markdown source
  - `PATH_v4.0.tex` — LaTeX source
  - `PATH_supplement_demo_hardware.md` — demo build supplement
  - `PATH_v4.0.pdf.ots` — OpenTimestamps Bitcoin proof
  - `LICENSE` — CC BY 4.0
  - `README.md` — full publication readme with DOI badge

---

## arXiv (Pending)
- **Status:** Endorsement outreach in progress
- **Target category:** `cs.CR` (Cryptography and Security), cross-list `cs.NI`
- **Contact:** Dr. Prineha Narang, UCLA / arXiv Science Advisory Council (referral/endorser identification)
- **Update this record** with the arXiv ID once accepted.

---

## Summary Proof Chain

| Layer | Identifier | Date | Authority |
|-------|-----------|------|-----------|
| Author ID | ORCID `0009-0005-2160-9434` | June 13, 2026 | ORCID.org |
| Prior-art timestamp | DOI `10.5281/zenodo.20673141` | June 13, 2026 | CERN Zenodo |
| Cryptographic proof | SHA256 `2f3997f...28f554` | June 13, 2026 | Bitcoin blockchain |
| Public disclosure | github.com/9jcb2jw8gc-cyber/path-protocol | June 13, 2026 | GitHub (Microsoft) |
| Academic preprint | arXiv ID TBD | Pending | arXiv (Cornell) |

The first three layers are independently verifiable without trusting any single institution.
The SHA256 hash ties the exact PDF bytes to the Bitcoin blockchain proof-of-work — any
dispute about the date or content of the publication can be resolved cryptographically.

---

## How to Verify

**Verify Zenodo record:**
Visit https://doi.org/10.5281/zenodo.20673141 — the record is permanently public.

**Verify Bitcoin timestamp:**
1. Download `PATH_v4.0.pdf.ots` from the GitHub repo.
2. Download the original `PATH_v4.0.pdf`.
3. Go to https://opentimestamps.org
4. Drag both files onto the verify page.
5. The tool will confirm the SHA256 hash matches and show the Bitcoin block attestation.

**Verify SHA256 hash independently:**
On macOS: `shasum -a 256 PATH_v4.0.pdf`
On Linux: `sha256sum PATH_v4.0.pdf`
Expected: `2f3997f133c9022292e37b44216189aec8b5ad045ecee3e8a6fc26b64628f554`

---

*Record created June 13, 2026. Update with arXiv ID when available.*
