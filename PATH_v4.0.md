*Preprint. Released as a defensive publication to establish public prior art. Licensed under CC BY 4.0 — you may share and adapt with attribution.*

-----

## Abstract

We describe an overlay construction for session-establishment traffic in which an ephemeral session key is split via a (t, n) threshold secret-sharing scheme and the resulting shares are dispatched across deliberately divergent transport paths, including at least one non-terrestrial path. The construction targets a specific, narrow class of traffic — authentication handshakes, credential exchange, session-token issuance, and command authorization — for which the ratio of per-message value to per-message size is high and conventional TLS-over-IP is known to concentrate vulnerability at the handshake.

The construction extends a core cryptographic model with a proximity-mesh relay layer, a self-routing payload architecture, an Interference-Adaptive Band Orchestration (IABO) model for power-efficient multi-band dispatch, a CSS physical layer with satellite link budget analysis, indoor penetration characterization, a universal mesh architecture thesis, a return path architecture, a tiered federation operator model, and a phased commercialization roadmap anchored to MCDC (Mission-Critical Data Channel) as the Phase 1 deployment vehicle.

The cryptographic primitives are individually mature and well-studied. The contribution of this paper is their composition on a transport substrate — one that includes commercial non-terrestrial network (NTN) paths alongside terrestrial cellular and short-range radio — that has only recently become available on commodity devices, combined with a self-routing payload model and an intelligent radio orchestration layer that together eliminate the infrastructure dependencies that make every current alternative fragile.

-----

## 1. Introduction

### 1.1 Motivation

TLS handshakes concentrate the cryptographic value of a session in a small number of exchanged messages. An adversary who captures those messages — and who later acquires the long-term private key or sufficient computational capability — recovers the session key and, from it, the entire session. This is the harvest-now-decrypt-later (HNDL) threat driving the current migration to post-quantum key-encapsulation mechanisms.

> **Plain language:** Think of a TLS handshake as exchanging a safe combination over a phone call. If someone records that call today and later cracks the encryption protecting the recording, they have the combination and can open everything sent afterward. PATH addresses this by never sending the full combination over any single path.

This paper does not argue against post-quantum migration. It argues for an orthogonal defensive layer for a narrow class of traffic where the consequences of HNDL compromise are asymmetrically severe.

### 1.2 The Foundational Inversion

PATH is not an incremental improvement to existing network architecture. It represents a foundational inversion of where intelligence lives in a network.

Current networking is infrastructure-centric. Packets are dumb. Routers are smart. The substrate carries the cognitive load of routing, security inspection, and delivery management. The entire carrier, ISP, CDN, firewall, and load-balancing industry exists to manage that centralized intelligence layer.

PATH is message-centric. The packet carries intelligence. The substrate provides capacity. Each share fragment carries everything a relay node needs to forward it toward its destination — no routing tables, no topology maps, no coordinator required. This inversion has compounding implications: infrastructure becomes fungible, trust becomes portable, and session establishment no longer requires stable, addressable endpoints.

> **Plain language:** Consider air travel. Today, most network packets are like passengers who show up at the airport with no knowledge of where they’re going. The airport — the infrastructure — manages all routing decisions. PATH inverts this: the message knows its destination, carries its own credentials, and negotiates each hop based on what it finds in the environment. The airport’s job shrinks to providing runways and gates. Infrastructure becomes commodity.

The implications extend beyond security. When messages self-route, infrastructure scales differently. When trust travels with the message, surveillance requires defeating the message rather than monitoring the substrate. When any substrate can carry a PATH fragment, coverage becomes an emergent property of device density rather than infrastructure investment.

### 1.3 Scope

This construction is restricted to session establishment. It is not proposed for bulk data transport. A session whose establishment is protected by PATH continues, after setup, over conventional TLS-over-IP on a single path. This is a hybrid-handshake architecture: threshold dispersal at setup, TLS for payload.

The architectural vision extends further. While the cryptographic construction remains narrow, the universal mesh architecture described in Section 11 represents the long-term destination toward which MCDC and PATH are early steps. The development roadmap in Section 12 sequences from today’s purpose-built mission-critical devices to that destination.

### 1.4 Relationship to Prior Work

The cryptographic primitives in this construction are individually well-studied.

**Threshold secret sharing over multiple paths.** Boloorchi et al. [14] formalized the combination of threshold secret sharing with multipath dispatch in sensor network contexts (STM, 2014). Their scheme splits a session key into shares and routes them through different network paths. PATH’s core key-splitting and dispatch mechanism is structurally similar. We cite it as foundational.

**Multipath key exchange integrated with TLS.** Costea et al. [15] presented SMKEX (Secure Multipath Key Exchange) at ACM CCS 2018 — a peer-reviewed, implemented protocol combining multipath transport with key exchange to protect TLS session establishment. A subsequent paper [16] showed that multipath security for TLS 1.3 can be achieved with minimal protocol modifications. PATH’s security argument for the handshake layer is compatible with and builds on this body of work.

**What this construction adds.** PATH differs from prior multipath key exchange literature in three specific respects:

1. **NTN as a structural anchor.** Prior work operates exclusively over terrestrial paths. PATH requires at least one non-terrestrial path, which physically bypasses terrestrial aggregation points and provides path diversity that survives terrestrial outages. This became a practical design option with 3GPP Release 17 NTN integration and the commercial availability of NTN-capable commodity devices.
1. **Post-quantum per-share encapsulation.** PATH wraps each share under ML-KEM (FIPS 203) [6], providing defense in depth against HNDL adversaries even on paths that are individually compromised. Its application per share in a multipath threshold construction is a deliberate composition choice not present in prior work.
1. **Self-routing payload with proximity-mesh relay.** Prior work assumes a fixed set of paths from a single sender. PATH’s assembly manifest makes each share self-describing — carrying its own routing intelligence — and the proximity-mesh extension allows co-located devices to contribute independent radio access paths as additional share-transport substrates. Neither mechanism requires topology tables or infrastructure coordination.

The core claim is not that individual components are novel. It is that their composition on a transport substrate simultaneously including cellular, Wi-Fi, NTN satellite, and short-range mesh — all available on commodity hardware under open standards — produces a practically deployable construction for the first time.

-----

## 2. Threat Model

> **Plain language:** We state precisely what PATH defeats, what it degrades, and what it does not address. A construction that claims to protect everything protects nothing. Intellectual honesty about scope is a load-bearing property of any deployable security protocol.

### 2.1 Adversary Capabilities We Defend Against

- **Bounded passive observation.** The adversary has passive wiretap access to some subset S of the n dispatched transport paths, with |S| < t. Captured shares are individually worthless below the reconstruction threshold.
- **Single-domain active compromise.** The adversary controls one transport operator and may modify, delay, or drop traffic within that operator’s domain. PATH’s path diversity ensures no single operator controls enough shares to reconstruct.
- **Future quantum capability.** The adversary may at some future time possess a cryptographically relevant quantum computer. Per-share ML-KEM-768 wrapping provides post-quantum confidentiality on each path.
- **Bounded relay compromise.** The adversary controls up to k < t relay nodes in the mesh. Honest relay majority above threshold is preserved.
- **Harvest-now-decrypt-later (HNDL).** The adversary records encrypted traffic today and decrypts later using future capability. ML-KEM-768 per-share wrapping is designed specifically against this threat class.

-----

### 2.2 Attack Catalog

The following documents all known attacks against PATH, their mechanism, current mitigation status, and open work required. Attacks are grouped by layer: cryptographic, protocol, network, physical/RF, and operational.

-----

#### 2.2.1 Timing Correlation Attack

**Layer:** Network / Traffic Analysis
**Crypto required:** None
**Cost to execute:** Medium — requires wideband passive SIGINT coverage across multiple bands simultaneously

**Mechanism:** A passive adversary with multi-band RF monitoring observes that multiple radio bursts — on 915 MHz CSS, 2.4 GHz CSS, and Wi-Fi UDP — occur within a narrow time window from the same geographic area. No share content is readable. But the adversary now knows: a PATH session was initiated, the probable originator location, the session timing, and which paths were used. At Phase 1 software TDD switching speeds (100–500ms), the dispatch window is wide enough that a capable SIGINT platform can correlate across bands without breaking any cryptography. Once the timing window is identified, the adversary knows precisely when to focus collection on subsequent sessions.

**Current mitigation:** Partial. Section 2.3 documents IABO’s emergent timing randomization as a power-optimization side effect. This provides degraded timing correlation, not elimination.

**Required mitigation:** Explicit randomized dispatch timing with mandatory inter-share delay jitter, independent of IABO power optimization. Cover traffic on non-transmitting bands during active sessions to obscure the correlation window. Formal analysis of minimum jitter required to defeat wideband correlation at specified adversary capability levels.

**Status: OPEN — requires formal specification of jitter and cover traffic protocol.**

-----

#### 2.2.2 Byzantine Relay: Corrupt-and-Retransmit

**Layer:** Protocol
**Crypto required:** None
**Cost to execute:** Medium — requires physical node placement in relay path; no cryptographic capability needed

**Mechanism:** A malicious node positioned in the relay path receives a share, introduces a targeted corruption (single bit flip or structured modification to the Shamir share value), and retransmits immediately at full signal strength with correct timing. The share appears legitimate at the RF layer — correct frequency, correct timing, correct format. The catcher receives t shares, all appearing valid. Shamir reconstruction proceeds. The reconstructed key is garbage. ML-KEM decapsulation fails silently or produces an invalid session key. The session appears to complete but is cryptographically dead. Neither party detects the attack at the session layer without explicit share authentication.

The attack is surgical: because the assembly manifest (Section 3.2) carries the share index, a malicious relay that captures share i knows exactly which position it poisoned. In a (t, n) scheme, corrupting any single share above the threshold guarantees reconstruction failure regardless of which other shares arrive clean.

**Current mitigation:** None. ML-KEM wrapping provides per-share confidentiality but not per-share integrity from the originator’s perspective. A relay node that holds the ML-KEM ciphertext cannot modify the plaintext share — but it can modify the ciphertext itself, causing decapsulation to fail or produce a wrong key.

**Required mitigation:** Per-share originator signature using ML-DSA (FIPS 204, reference [7]). The session initiator signs each share’s ciphertext and manifest before dispatch. The catcher verifies all n signatures before attempting Shamir reconstruction. Any share failing signature verification is discarded, not assembled. A malicious relay can still corrupt a share, but the catcher detects and drops it rather than assembling poisoned material. The session falls back to remaining clean shares, provided t clean shares survive.

Note: ML-DSA signature verification at the catcher requires the catcher to hold or retrieve the sender’s public signature key — introducing a key distribution requirement not currently specified.

**Status: OPEN — ML-DSA per-share signature specified as required mitigation; key distribution mechanism unspecified.**

-----

#### 2.2.3 Share Suppression (Denial by Selective Drop)

**Layer:** Protocol / Network
**Crypto required:** None
**Cost to execute:** Medium — requires node placement; no cryptographic capability

**Mechanism:** A malicious relay node correctly receives a share and silently discards it without retransmitting. Unlike corrupt-and-retransmit, no modified packet appears on the network. If the adversary controls or influences enough relay nodes to suppress shares on more than (n - t) paths, the catcher never accumulates t valid shares and the session starves. The attack is undetectable at the originator — the sender dispatched n shares correctly. The failure appears as a network reliability problem, not a security event.

A sophisticated variant: suppress selectively on high-reliability paths (sub-GHz CSS with good link budget) and allow shares to arrive on degraded paths (high-loss NTN uplink), maximizing the probability that fewer than t shares complete delivery.

**Current mitigation:** Partial. The (t, n) threshold scheme with n > t provides some redundancy. Increasing n relative to t raises the number of paths an adversary must suppress simultaneously.

**Required mitigation:** No cryptographic counter exists for share suppression — an honest node and a dishonest suppressing node are indistinguishable at the network layer. Mitigations are architectural: (1) increase n significantly above t to raise suppression cost; (2) dispatch redundant shares over independent paths including paths not controlled by relay nodes (direct NTN uplink, direct cellular); (3) session retry with path rotation on failure, making sustained suppression require sustained adversary presence across rotating path sets.

**Status: OPEN — no cryptographic solution. Architectural redundancy parameters (n, t ratio) require formal analysis against suppression adversary model.**

-----

#### 2.2.4 Assembly Manifest Exposure

**Layer:** Protocol
**Crypto required:** None
**Cost to execute:** Low — capturing any single share reveals the manifest

**Mechanism:** The assembly manifest (Section 3.2) must be readable by relay nodes to enable self-routing — a relay cannot forward a share toward a catcher it cannot identify. This means the manifest cannot be encrypted under the session key PATH is trying to establish (circular dependency). A passive adversary who captures any single share recovers the manifest in full: total share count n, reconstruction threshold t, share index i, assembly deadline, catcher identity, and blinded message identifier. The adversary now knows the complete session structure: how many shares exist, which index was captured, when the session expires, and where all shares are converging.

This is not content exposure — the session key remains protected below threshold. But it is a complete map of the session, enabling targeted collection, catcher identification for DoS, and confirmation that a PATH session is in progress.

**Current mitigation:** The manifest uses a “blinded message identifier” described as opaque to relay nodes but recognizable to the catcher federation. The blinding mechanism is not fully specified.

**Required mitigation:** Two-layer manifest architecture. An outer manifest (relay-readable) carries only the minimum required for forwarding: hop count, assembly deadline, and a catcher locator that is meaningful only to catcher nodes (e.g., a pseudonymous catcher identifier resolved only within the federation). An inner manifest (catcher-readable only) carries share index, n, t, and message identifier, encrypted under the catcher federation’s public key. Relay nodes forward on outer manifest alone. Only catchers can read inner manifest content.

**Status: OPEN — two-layer manifest architecture specified as required; blinding mechanism and catcher public key infrastructure not yet designed.**

-----

#### 2.2.5 Catcher Denial of Service

**Layer:** Network
**Crypto required:** None
**Cost to execute:** Low — catcher endpoint must be addressable to receive shares

**Mechanism:** The catcher is structurally required to have a stable, reachable address — shares from all n paths must converge somewhere. This makes the catcher a high-value, identifiable DoS target. An adversary who identifies the catcher endpoint (possible via manifest exposure per 2.2.4, or via traffic analysis of multi-path convergence) can saturate the catcher’s inbound interface during the assembly window. Shares arrive but cannot be processed. The session starves regardless of how successfully shares were dispersed.

**Current mitigation:** The three-layer catcher federation architecture specified in Section 3.5 structurally eliminates the single-receiver vulnerability. Senders dispatch shares to hundreds of geographically distributed edge nodes selected randomly per session. Edge nodes hold no reassembly state and forward shares via a private routing fabric to interior reassembly nodes that are not directly addressable from the public network. An attacker has no single endpoint to flood and no reachable assembly target. Volumetric attacks must scale across hundreds of operator-diverse edges to deny session establishment.

**Required mitigation:** Catcher inbound interfaces should additionally enforce cryptographic admission control: incoming shares must present a Proof-of-Work (PoW) token tied to the current epoch and message identifier — for example, a partial SHA3-256 collision against a published difficulty target — before the edge node allocates buffer memory or forwards the share inward. The sender computes a nonce X such that the first d bits of H(epoch ∥ message_id ∥ X) = 0; the catcher verifies this with a single O(1) hash evaluation while the sender’s average cost is O(2^d) hash operations. This creates exponential computational asymmetry.

Client puzzles are standardized for analogous DoS protection in RFC 8019 (IKEv2, November 2016) and RFC 7401 (HIPv2, April 2015). However, commercial deployment of these mechanisms in production IPsec/IKEv2 infrastructure is rare — most enterprise installations do not enable client puzzles due to interoperability complexity and the client-starvation problem described below. PATH’s adoption of PoW admission control therefore draws on standardized protocol mechanisms with limited deployed precedent, rather than widely-used industry practice.

**Critical constraints on PoW parameter selection:**

For PATH’s heterogeneous client population, the difficulty parameter d must be carefully bounded:

- **d ≤ 8 bits (~128 hash operations average):** Computationally cheap on IoT-class hardware (Cortex-M at 100 MHz solves in <5ms with negligible power draw). Recommended default for IoT senders.
- **d ≥ 16 bits (~32,768 hash operations average):** Can block constrained IoT devices for several seconds and significantly degrade battery life. Should only be invoked during active DoS triage, not as steady-state policy.
- **d ≥ 20 bits:** Imposes only ~1 second on modern CPUs but completely excludes low-power IoT senders, creating self-inflicted denial of service against legitimate constrained clients.

**Two counter-attack vectors PoW does not solve:**

1. **Client starvation under adaptive difficulty.** When difficulty is scaled up to deter large botnets, the same difficulty starves legitimate IoT senders. PoW therefore cannot be a static gateway policy; it must be dynamically scaled and only invoked under active DoS conditions.
1. **Energy depletion attack on senders.** A malicious catcher (or an adversary capable of injecting forged difficulty challenges into the directory) could force IoT devices to solve continuous puzzles, draining battery as an energy-exhaustion attack. The defensive PoW mechanism becomes a vulnerability vector if difficulty announcements are not authenticated.

**Operational specification:**

PoW admission control should be deployed as a **temporary, dynamically-scaled triage mechanism** rather than as a static admission policy. Catchers should announce current difficulty in a signed directory entry that senders verify before computing puzzles. Difficulty should be set at d ≤ 8 under normal operation, scaling upward only when volumetric attack is detected, and reverting to baseline once attack signatures abate. Network operators deploying interior nodes within 5G/6G infrastructure should additionally isolate them on dedicated network slices, ensuring volumetric traffic on public channels cannot exhaust reassembly resources.

The residual attack surface — interior node compromise and edge-to-interior traffic analysis — is documented as attacks 2.2.11 and 2.2.12 below.

**Status: PARTIALLY RESOLVED by three-layer architecture (Section 3.5). PoW admission control specified with dynamic difficulty scaling as required additional mitigation; signed difficulty announcement protocol not yet designed; energy-depletion defense against malicious or spoofed difficulty challenges open.**

-----

#### 2.2.6 Share Fingerprinting via Airtime Signature

**Layer:** RF / Traffic Analysis
**Crypto required:** None
**Cost to execute:** Medium — requires RF monitoring; no cryptographic capability

**Mechanism:** ML-KEM-768 produces ciphertext of known, fixed size (1,088 bytes). Wrapped in a Shamir share and dispatched over CSS, the total airtime required to transmit each share is deterministic and computable from public parameters. On CSS paths where the payload ceiling (51–255 bytes per packet) forces fragmentation, the number of fragments per share is a fixed, predictable value. An adversary monitoring the RF environment can identify PATH sessions by packet size distribution and airtime signature — even without reading any payload content. The fragmentation pattern itself is a fingerprint that distinguishes PATH traffic from ambient LoRa sensor traffic.

**Current mitigation:** None explicitly. Section 2.3 notes CSS transmissions are indistinguishable from civilian LoRa at the RF layer — this is attribution difficulty at the chirp level but does not address size-based fingerprinting at the packet level.

**Required mitigation:** Fixed-size block padding regardless of actual payload size, applied before CSS fragmentation. Randomized fragment counts with null padding appended to genuine share content. This adds airtime overhead but eliminates the deterministic size signature. The ML-KEM fragmentation protocol (currently Open — Blocking) must incorporate padding as a first-class requirement, not an afterthought.

**Status: OPEN — requires integration into fragmentation protocol specification, which is itself unresolved.**

-----

#### 2.2.7 RF Fingerprinting via Hardware Signature

**Layer:** Physical / RF
**Crypto required:** None
**Cost to execute:** Low to Medium — RTL-SDR (~$35) sufficient for proof-of-concept; high-quality I/Q capture requires USRP-class hardware (~$1,500+)

**Mechanism:** Every radio transmitter has unique physical-layer signatures introduced by manufacturing variation in its oscillators, mixers, power amplifiers, and antenna feed network. These manifest as device-specific carrier frequency offsets, phase noise profiles, transient turn-on behavior, and In-phase/Quadrature-phase (I/Q) imbalances. They exist below the digital layer and are not removed by encryption, padding, or timing randomization.

A passive adversary captures raw I/Q waveforms from PATH transmissions using software-defined radio. Deep learning models — convolutional or recurrent neural networks trained on labeled transmitter populations — classify the captured waveforms against device fingerprints with high accuracy. Published research on RF fingerprinting of nano-satellite transmitters demonstrates RNN classification accuracy of 99.34% maintained over 198 days under environmental degradation (Qiu, Savva, Guo, *Modern Physics Letters B*, Vol. 36, No. 12, April 2022, DOI: 10.1142/S0217984922500439). Terrestrial RF fingerprinting against LoRa CSS transmitters specifically — the physical layer used by PATH’s CSS paths — achieves up to 96.40% classification accuracy across 25 distinct devices using spectrogram-CNN models (Shen, Zhang, Peng, Marshall, *IEEE Journal on Selected Areas in Communications*, Vol. 39, No. 8, August 2021, DOI: 10.1109/JSAC.2021.3082227). This is not a theoretical attack — it is a documented capability against the exact radio architecture PATH uses.

The consequence for PATH: an adversary who fingerprints any single share transmission can subsequently identify the same originating device across all other PATH transmissions, regardless of which band, frequency, or path is used. Physical path diversity becomes operationally meaningless if all paths are emitted by the same physically-identifiable hardware. The threshold security argument requires the adversary to collect t shares; RF fingerprinting allows the adversary to identify and target collection at the originator with certainty, then collect all subsequent traffic from the same source.

This attack is particularly severe for PATH because the protocol’s core design intent is multi-path diversity. RF fingerprinting collapses that diversity at the physical layer.

**Current mitigation:** None. Standard timing randomization, padding, and band hopping address the digital and timing layers but do not affect analog hardware signatures.

**Required mitigation:** Baseband Pre-Distortion (BPD) with randomized I/Q variation per packet. The transmitter’s digital signal processor introduces small, cryptographically randomized variations into the I/Q components and local oscillator phase offsets before each transmission. These synthetic distortions exceed the magnitude of the device’s natural manufacturing variations, masking the hardware’s true signature beneath induced noise. Each packet appears to originate from a different statistical device.

Published research (Jing et al., IEEE Transactions on Information Forensics and Security, July 2024) demonstrates that joint I/Q imbalance compensation and DNN-based digital pre-distortion can reduce specific emitter identification (SEI) classifier accuracy from near-95% to approximately 17%. This is an active area of post-quantum physical-layer security research; commercial and military integration into deployed software-defined transceivers remains an emerging protective mechanism rather than a universally deployed standard. PATH’s adoption of BPD would be among the early operational implementations of a research-stage technique, not the application of an off-the-shelf military capability.

BPD cannot run on stock LR1121 or SX1262 firmware; it requires either a custom RF front-end with digital pre-distortion capability or migration to an FPGA-based SDR transceiver for the PATH transmit path. This has material implications for the Phase 2 silicon roadmap.

**Two supplementary mitigations:**

A secondary mitigation: training-set denial. If the adversary cannot capture sufficient labeled samples to train a classifier (e.g., because PATH transmits rarely enough to prevent classifier training in the operational window), the attack degrades. This is not a substitute for BPD but provides defense-in-depth for low-frequency session establishment.

A tertiary defensive opportunity worth investigating: classifier blind-spot exploitation. Published research indicates that RFFI classifiers trained on warmed-up, stable transmitters exhibit significant accuracy degradation when classifying frames transmitted during the first hour of device operation, due to instantaneous carrier frequency offset (CFO) drift during hardware warm-up. PATH session establishment that occurs immediately after device cold-boot may evade classification by classifiers trained on steady-state signals. This is an operational tactic, not a protocol guarantee, but worth specifying as a deployment recommendation for high-threat environments.

**Sub-attack distinction:** The published threat model encompasses two distinct fingerprinting vectors. Transceiver-circuit anomalies (oscillator drift, mixer non-linearity, PA distortion, I/Q imbalance) originate in the radio front-end and apply to all transmissions from a given device. Antenna structural fingerprinting (radiation pattern variation due to manufacturing or environmental degradation, as demonstrated in Qiu, Savva, and Guo, *Modern Physics Letters B*, 2022) operates at the antenna layer and survives even if the transceiver is replaced. BPD mitigates the transceiver layer; antenna-layer fingerprinting requires either antenna randomization, antenna pattern masking, or operational antenna rotation across a population of identical antennas.

**Differential fingerprinting difficulty across PATH’s radio stack.** Not all radios in PATH’s transport stack are equally susceptible to fingerprinting. Adversary cost, receiver complexity, and published research maturity vary significantly by radio type, which drives threat-proportionate allocation of defensive hardware:

|Radio                            |Receiver cost               |Published research                         |Fingerprinting difficulty                             |
|---------------------------------|----------------------------|-------------------------------------------|------------------------------------------------------|
|Sub-GHz CSS (LoRa, 915 MHz)      |RTL-SDR ~$35                |Heavy (Shen et al. 2021, ~96% accuracy)    |**Easiest**                                           |
|Wi-Fi 802.11                     |HackRF / PlutoSDR ~$230     |Extensive, >95% accuracy                   |Easy                                                  |
|2.4 GHz CSS (LoRa)               |PlutoSDR ~$230              |Limited but applicable from sub-GHz studies|Easy-Moderate                                         |
|Bluetooth / BLE                  |SDR or BLE sniffer ~$50-150 |Moderate, 85-98% accuracy                  |Moderate (frequency hopping complicates capture)      |
|Cellular LTE (uplink)            |LimeSDR / USRP $1,500+      |Less mature, 80-90% accuracy               |Hard (bandwidth and protocol complexity)              |
|NTN uplink (ground-based passive)|Specialized SatCom equipment|Minimal published research                 |Hardest (directional uplink, weak ground-side leakage)|

Sub-GHz CSS is the easiest fingerprinting target by a wide margin. The combination of long chirp symbols (millisecond-duration signal for feature extraction), narrow bandwidth (capturable with $35 hardware), clear preamble structure, and abundant public research datasets makes it nearly ideal for fingerprinting attacks. Every public fingerprinting tutorial and toolkit published in the next several years is likely to use LoRa as the example physical layer.

**Proximity-Mesh Fingerprint Break.** The proximity-mesh relay layer creates a structural fingerprint break between session originator and long-range transport. When BLE is used as a device-to-device handoff to a relay device, shares emitted by long-range radios (cellular, Wi-Fi, CSS, NTN) carry the relay device’s RF fingerprint, not the originator’s. An adversary capable of long-range RF fingerprinting therefore identifies relay devices rather than originators. To identify the originator, the adversary must additionally conduct close-proximity (≤30m) BLE fingerprinting collection — a substantially harder operational problem requiring physical access or distributed local sensors.

This is an emergent security property of the proximity-mesh architecture, not a designed defense. It is partial: close-proximity BLE collection remains a viable attack against originators in environments where the adversary has physical access, and the relay device population becomes identifiable over time as PATH usage matures. However, it materially raises the bar for adversaries seeking to fingerprint originators specifically rather than relay populations generally. This protection applies only when BLE is used for relay handoff; if BLE ever carries share-bearing traffic directly to non-relay endpoints, the fingerprint break does not apply.

**Layered Defense Architecture.** Antenna diversity and BPD operate at different physical layers and combine multiplicatively against fingerprinting attacks when implemented with three properties: independent randomization sources, per-packet rotation, and uncorrelated rotation patterns. The full defensive stack:

- **Layer 1 — Antenna Diversity (Phase 1, achievable on current hardware):** Two antennas per protected radio with cryptographically-randomized per-packet selection. Defeats single-snapshot antenna fingerprinting; bimodal antenna signatures remain identifiable across many samples. Implementation cost: ~$15-30 per protected radio (additional antenna, RF switch, GPIO wiring).
- **Layer 2 — Baseband Pre-Distortion (Phase 2, requires silicon):** Per-packet cryptographically-randomized I/Q distortion at the digital baseband layer. Defeats transceiver-circuit fingerprinting. Empirically reduces SEI classifier accuracy to ~17% (Jing et al. 2024). Not implementable on LR1121/SX1262; requires FPGA-based SDR transceiver or custom silicon.
- **Phase 1.5 software approximation:** Software I/Q perturbation at the modulation layer rather than true baseband pre-distortion. Available on current hardware via firmware modification. Provides partial transceiver-layer protection while waiting for Phase 2 silicon.
- **Combined Phase 2 deployment:** Both layers operating in parallel with independent per-packet randomization sources. Attacker must defeat both layers simultaneously. No published empirical data on combined classifier accuracy.

**Threat-proportionate antenna diversity allocation.** Antenna diversity hardware is allocated to radios whose effective transmission range makes long-distance RF fingerprinting collection feasible. Short-range radios are excluded where architectural mitigations (proximity-mesh break) or alternative attack surfaces (protocol-layer identity, operator visibility) make antenna diversity inappropriate as the defense mechanism:

|Radio                      |Antenna diversity in Phase 1 demo?|Rationale                                                                                                     |
|---------------------------|----------------------------------|--------------------------------------------------------------------------------------------------------------|
|Sub-GHz CSS (LoRa 915 MHz) |**Yes**                           |Easiest fingerprinting target; antenna diversity is the right defense                                         |
|2.4 GHz CSS (LoRa)         |Defer to Phase 2 silicon          |Moderate threat; integration solves cost problem                                                              |
|Wi-Fi (T-Beam ESP32-S3)    |Defer to Phase 2 silicon          |Moderate threat; PCB rework intensive on current hardware                                                     |
|BLE (T-Beam ESP32-S3)      |**No**                            |Proximity-mesh fingerprint break; BLE serves control plane only in demo                                       |
|Cellular (tethered modem)  |**No**                            |Antenna diversity does not address protocol-layer identity (IMEI/SIM)                                         |
|NTN uplink (LR1121 1.9 GHz)|**No**                            |Antenna diversity does not protect against operator-side fingerprinting; correct defense is operator selection|

This allocation scopes the Phase 1 demo antenna count from 32 antennas (full coverage across all transmit-capable radios) to **12 antennas covering sub-GHz CSS only** (8 antennas across 4 Muzi Duos + 4 antennas across 2 T-Beam Supremes). The defensive argument is threat-proportionate: hardware concentrates on the radio class where the attack is cheapest, most studied, and most likely to be deployed by future adversaries, while skipping radios where antenna diversity is either redundant (proximity-mesh break covers BLE) or wrong-tool (operator-layer attacks dominate for cellular and NTN).

**Status: OPEN — BPD specified as required Phase 2 mitigation for transceiver-layer fingerprinting; Phase 1 antenna diversity scoped to sub-GHz CSS paths; antenna-layer fingerprinting mitigation for radios beyond sub-GHz CSS deferred to Phase 2 silicon. Phase 2 silicon roadmap must incorporate BPD across all PATH-transport radios as a baseline requirement for the PATH radio module.**

-----

#### 2.2.8 NTN Path Operator Compelled Disclosure

**Layer:** Operational / Legal
**Crypto required:** None
**Cost to execute:** Low for nation-state adversary with jurisdiction over satellite operator

**Mechanism:** The NTN satellite uplink (1.9 GHz, LR1121) traverses a commercial satellite operator — Skylo or equivalent. The operator has full visibility into: uplink originator device identity, GPS coordinates at time of transmission, session timing, and traffic volume. A nation-state adversary with legal authority over the satellite operator obtains this metadata via compelled disclosure without breaking any cryptography. The NTN path, intended as the most physically diverse path in the stack, is simultaneously the least anonymous path — it routes through a single commercial chokepoint with a known legal domicile.

**Current mitigation:** None. PATH’s security argument for NTN is physical path diversity (jurisdictional orthogonality from terrestrial paths), not metadata anonymity.

**Required mitigation:** Explicit scope boundary in threat model. PATH does not protect against adversaries with legal authority over infrastructure operators on any path — NTN, cellular, or otherwise. This is a design boundary, not a flaw, but must be stated explicitly so deployers understand the threat classes PATH addresses and does not address.

**Status: RESOLVED by scope definition — PATH’s NTN value proposition is physical and jurisdictional path diversity against passive adversaries and single-domain active compromise, not protection against compelled operator disclosure. Deployers requiring anonymity from infrastructure operators must layer PATH over an anonymization network (e.g., Tor over the IP paths) — a configuration outside PATH’s current scope.**

-----

#### 2.2.9 Global Passive Adversary

**Layer:** Cryptographic
**Crypto required:** None (passive collection only)
**Cost to execute:** High — requires simultaneous monitoring of all n paths

**Mechanism:** If the adversary captures t or more of the n dispatched shares, Shamir reconstruction succeeds. ML-KEM decapsulation recovers each share. The adversary reconstructs the session key. PATH’s threshold guarantee fails entirely at or above the collection threshold.

**Current mitigation:** PATH’s security reduces to the adversary’s ability to monitor t or more independent paths simultaneously. Path divergence (Section 3.3) is designed to maximize the cost of this collection. NTN structural inclusion ensures at least one path requires satellite monitoring capability.

**Required mitigation:** None beyond path divergence maximization and appropriate (t, n) parameter selection for the threat environment. For military deployments, t should be set conservatively relative to n, accepting higher session establishment overhead to raise collection cost.

**Status: RESOLVED by design — acknowledged as out-of-scope for the construction. PATH degrades gracefully: as adversary collection capability increases from 0 to t-1 paths, the construction holds. At t paths, it fails. There is no cryptographic construction that defeats an adversary who can observe all paths.**

-----

#### 2.2.10 Endpoint Compromise

**Layer:** Operational
**Crypto required:** None
**Cost to execute:** Variable

**Mechanism:** If the sender or catcher host is compromised before or during session establishment, PATH provides no protection. The session key exists in plaintext on both endpoints at the moment of use. An adversary with code execution on either endpoint recovers the key regardless of how securely it was transported.

**Status: RESOLVED by scope definition — endpoint security is outside PATH’s construction boundary. PATH protects the transport of session establishment material. It assumes honest endpoints.**

-----

#### 2.2.11 Interior Node Compromise

**Layer:** Operational / Architectural
**Crypto required:** None (post-compromise)
**Cost to execute:** High — interior nodes have no public route and require either insider access or supply-chain compromise

**Mechanism:** Interior reassembly nodes (Section 3.5) hold the decryption key for the inner manifest and the reconstruction authority for sessions routed to them. An adversary who compromises an interior node — through insider access, supply chain attack, or physical compromise — gains the ability to read session identities, reassemble keys for sessions arriving at that node, and potentially identify session participants. Because interior nodes handle many sessions concurrently (gossip-with-deterministic-hint convergence routes all shares with the same routing tag bucket to the same interior), a single interior compromise exposes a larger session population than a single edge compromise would.

This is the residual risk created by the three-layer architecture. The single-catcher DoS vulnerability (2.2.5) is structurally eliminated, but the assembly authority that was previously distributed across all catcher federation nodes is now concentrated in the smaller interior layer.

**Current mitigation:** Partial. Mode B federated deployments (Section 3.5.5) limit blast radius — an interior compromise in one operator’s network does not expose other operators’ sessions. Geographic and jurisdictional distribution of interior nodes preserves the path divergence argument and prevents single-jurisdiction compromise of the entire interior layer.

**Required mitigation:** Interior nodes require HSM-grade operational security. Specifically: session reassembly state must expire aggressively (recommended: within seconds of TLS handoff completion); session keys must never be persisted to durable storage; interior node access must require multi-party authorization for any operational change; audit logs must record administrative actions without recording session material; interior node firmware should be measured and attested at boot. The interior layer should be operationally treated as a Hardware Security Module installation, not as a general-purpose application server.

For Mode B deployments, federation operators should publish interior node attestation policies so customers can evaluate the operational security posture before selecting a catcher federation.

**Status: OPEN — operational security requirements specified at a high level; concrete attestation protocol, key rotation policy, and audit requirements not yet designed.**

-----

#### 2.2.12 Edge-to-Interior Traffic Analysis

**Layer:** Network / Traffic Analysis
**Crypto required:** None
**Cost to execute:** Medium — requires monitoring access to the operator’s internal forwarding fabric, or correlation of edge ingress with interior ingress timing

**Mechanism:** Even with dynamic per-share forwarding, an adversary with monitoring access to the catcher federation’s internal network can observe traffic flows from many edges converging on a smaller set of interior nodes. Over time, this convergence pattern identifies interior node locations even if their IP addresses are not published. Once interior locations are identified, they become targets for direct compromise (2.2.11) or for legal compulsion in the jurisdiction where they are hosted. A nation-state adversary with monitoring access at the operator’s network perimeter can map the interior topology without compromising any node.

A secondary variant: timing correlation between edge ingress and interior ingress. If a share arrives at an edge node at time t and arrives at an interior node at time t + δ where δ is the forwarding latency, an external observer with monitoring access to both edge and interior network segments can correlate ingress events to identify which interior node received which share, even without reading any payload content.

**Current mitigation:** Section 3.5 specifies dynamic per-share routing to break static edge-to-interior associations. This degrades but does not eliminate the traffic analysis surface.

**Required mitigation:** Onion-style encrypted forwarding through multiple hops within the forwarding layer, similar to Tor circuit construction. Each forwarding hop decrypts only enough to determine the next hop, preventing any single observer (including the operator’s own forwarding nodes) from associating edge ingress with interior destination. Additional mitigations: forwarding latency randomization to defeat timing correlation; cover traffic between edges and interior nodes during sessions and during idle periods to prevent statistical traffic flow analysis.

These mitigations add significant operational complexity and forwarding latency overhead. For Mode A deployments where operator visibility is already accepted, these mitigations may be deferred. For Mode B deployments protecting against adversaries with monitoring access to the operator’s network, they are required.

**Status: OPEN — onion forwarding architecture specified as required for Mode B; cover traffic protocol not designed; latency overhead budget not established.**

-----

### 2.3 Attack Summary

|Attack                             |Layer           |Crypto Required|Adversary Cost    |Mitigation Status                                    |
|-----------------------------------|----------------|---------------|------------------|-----------------------------------------------------|
|Timing correlation                 |Network/RF      |None           |Medium            |**OPEN**                                             |
|Byzantine corrupt-and-retransmit   |Protocol        |None           |Medium            |**OPEN** — ML-DSA specified, not implemented         |
|Share suppression (denial)         |Protocol/Network|None           |Medium            |**OPEN** — no cryptographic solution                 |
|Assembly manifest exposure         |Protocol        |None           |Low               |**OPEN** — two-layer manifest specified, not designed|
|Catcher denial of service          |Network         |None           |Low               |Partially resolved by 3.5; PoW open                  |
|Share fingerprinting (size)        |RF              |None           |Medium            |**OPEN** — requires fragmentation protocol           |
|RF fingerprinting (hardware)       |Physical        |None           |Low–Medium        |**OPEN** — BPD specified, requires hardware revision |
|NTN operator compelled disclosure  |Operational     |None           |Low (nation-state)|Resolved by scope                                    |
|Global passive adversary (≥t paths)|Cryptographic   |None           |High              |Resolved by design                                   |
|Endpoint compromise                |Operational     |None           |Variable          |Resolved by scope                                    |
|Interior node compromise           |Operational     |None           |High              |**OPEN** — HSM-grade operational security specified  |
|Edge-to-interior traffic analysis  |Network         |None           |Medium            |**OPEN** — onion forwarding required for Mode B      |

**Key observation:** Of the strongest attacks against PATH, only one — RF fingerprinting (2.2.7) — requires hardware architecture changes beyond the current Phase 1 demo stack. The remaining open attacks are addressable through protocol, network, firmware, and operational engineering. The cryptographic primitives — Shamir secret sharing, ML-KEM-768, and the proposed ML-DSA-65 per-share signatures — are individually sound. The work agenda for PATH v5 is: timing jitter specification, ML-DSA integration with key distribution mechanism, fragmentation protocol with size padding, two-layer manifest design, federated catcher specification with PoW admission control, onion forwarding for Mode B catcher deployments, SAGIN-aware path divergence measurement, interior node HSM-grade operational security specification, and Baseband Pre-Distortion as a Phase 2 silicon requirement.

-----

### 2.4 Emergent SIGINT Resistance

The Interference-Adaptive Band Orchestration model described in Section 8 provides an additional defensive property: emergent resistance to signals intelligence (SIGINT) traffic analysis.

SIGINT does not require content decryption to be operationally valuable. Transmission timing, frequency patterns, burst duration, and correlation between transmission events and real-world activities allow adversaries to identify command nodes and map network topology against encrypted communications.

Choreographed multi-band PATH transmission disrupts several of these analytical inputs. Timing is partially randomized by the interference-avoidance scheduler. Frequency varies across bands and channels. Burst length varies with fragmentation decisions. Fragments appear as unrelated native traffic on each substrate.

Note: Section 2.2.1 documents the limits of this protection. SIGINT resistance from IABO is emergent and partial — not a designed security guarantee. A formal timing jitter specification (currently open) is required before SIGINT resistance can be claimed as a deliberate property.

**Attribution ambiguity — precise framing.** CSS transmissions in ISM bands are detectable by calibrated SIGINT equipment scanning for chirp patterns. The operative defensive property is attribution difficulty, not detection immunity. Attributing a CSS transmission to a PATH military mesh rather than civilian LoRa sensor traffic requires correlating timing, geography, and content across many intercepts against dense civilian ISM background traffic with identical RF signatures. In military-exclusive spectrum, any transmission is military by definition. In ISM bands, detection alone solves nothing — attribution remains an unsolved problem for the adversary in high-noise terrain.

-----

## 3. Construction

### 3.1 Overview

> **Plain language:** PATH works in three steps. Break the session key into pieces mathematically. Send each piece over a different path. Reassemble the pieces at a trusted endpoint. An adversary who intercepts any single path gets a piece that is mathematically worthless without the others.

The construction operates as a two-phase handshake followed by standard TLS. Call the sender A and the receiver B, where B is implemented as a federated catcher.

**Phase 1 — Share dispersal.** A generates an ephemeral session key K. A splits K via a (t, n) Shamir threshold scheme into shares s₁, …, sₙ. Each share sᵢ is wrapped under a post-quantum KEM keyed to B’s published KEM public key, yielding ciphertext cᵢ. A dispatches each cᵢ, along with an assembly manifest, over a distinct transport path Pᵢ. At least one Pᵢ is non-terrestrial.

**Phase 2 — Reassembly and TLS handoff.** B’s federated catcher accumulates cᵢ as they arrive, decapsulates each to recover sᵢ, and upon receiving any t shares reconstructs K. B then establishes a conventional TLS session with A using K as pre-shared key material.

### 3.2 The Assembly Manifest and Self-Routing Architecture

> **Plain language:** Every piece carries a label telling it where to go, what it is, and when to give up. Relay nodes don’t need to know the network topology — they just read the label and forward the piece toward the nearest catcher. The packet routes itself.

Each dispatched share carries an assembly manifest identifying: total share count n, reconstruction threshold t, share index i, an assembly deadline, the catcher identity, and a blinded message identifier that is recognizable to the catcher federation but opaque to relay nodes.

The assembly manifest is the architectural foundation of PATH’s self-routing model. A relay node receiving a share does not need a routing table, topology map, or prior coordination. It reads the manifest and makes a single local decision: can I get this fragment closer to a catcher endpoint? If yes, forward. If no, hold and retry within the assembly deadline.

This is a DTN-adjacent (Delay-Tolerant Networking) store-and-forward relay model, not a traditional mesh routing problem. The distinction matters:

- **Traditional mesh routing** requires global topology discovery, route convergence, and routing table maintenance across all nodes. This is what AODV and OLSR solve — and what makes flooding-based meshes degrade above ~80 nodes.
- **PATH’s relay model** requires only local knowledge: which outbound paths are available from this node, and which catcher endpoints are reachable via those paths. The manifest provides the destination. The relay provides the transport. No global state is required.

This is the foundational inversion thesis applied at the routing layer. The packet carries the routing intelligence. The substrate provides the capacity. Infrastructure — including routing infrastructure — becomes fungible.

**Relay forwarding policy requirements.** The self-routing model requires specification of four relay behaviors:

1. **Relay decision logic.** A node receiving a share evaluates available outbound paths against the manifest’s catcher identity and assembly deadline. If at least one outbound path can plausibly reach a catcher before the deadline, the node forwards. Otherwise it holds.
1. **Catcher reachability.** Relay nodes maintain a lightweight catcher directory — a list of known catcher endpoint identifiers and their associated radio access characteristics — updated via periodic beacon. No full topology map required.
1. **TTL and loop prevention.** The manifest includes a hop count field, decremented at each relay. Shares exceeding the hop limit are discarded. This prevents indefinite circulation without requiring loop-detection state.
1. **Store-and-forward policy.** A relay node holds an unforwarded share until either a suitable outbound path becomes available, the assembly deadline passes, or the hop count expires. Hold duration is bounded by the manifest deadline, not by protocol state.

### 3.3 Path Divergence

> **Plain language:** Two paths that look different on the surface may actually merge into the same pipe at the backbone level. Real path diversity means the paths stay separate all the way through, including at the major transit hubs where most internet traffic consolidates. Satellite is valuable here because it physically bypasses those terrestrial consolidation points.

The security argument requires paths to be divergent at four tiers, in increasing strength:

- **Radio-access divergence.** Different physical media: cellular LTE, 5G, satellite, Wi-Fi, Bluetooth.
- **Operator divergence.** Different commercial carriers, even if using the same medium.
- **Autonomous-system divergence.** Paths that do not share transit AS operators at any point.
- **Jurisdictional divergence.** Paths whose surveillance would require legal instruments in multiple distinct national regimes.

NTN satellite transport contributes most strongly to tiers 3 and 4.

**Convergence risk under SAGIN integration.** Path divergence at the radio-access layer does not guarantee divergence at the backhaul layer. Under 3GPP Release 17 (TS 38.101-5, TS 38.108) and Space-Air-Ground Integrated Network (SAGIN) architectures formalized in the O-RAN Alliance’s “Deployments of O-RAN-based Non-Terrestrial Networks” white paper (May 2025), terrestrial and non-terrestrial network operators increasingly share infrastructure: regional internet exchange points, subsea cable landing stations, and — most consequentially — virtualized network functions (VNFs) running on shared O-RAN platforms. The VNF service chain under disaggregated architectures runs Active Antenna Unit → virtualized Distributed Unit (vDU) → virtualized Centralized Unit (vCU) → 5G Core, with these software blocks deployed as containers or VMs running on General Purpose Processors within shared processing pools.

PATH secures the air gap by dispersing shares across diverse physical radio links, but this transport-layer diversity converges at the ground segment. The dominant near-term deployment pattern is the **Transparent (non-regenerative) payload model**, where the satellite serves as a bent-pipe analog repeater and the entire gNB — including RU, DU, and CU — resides at the terrestrial ground station. This architecture heavily concentrates processing at the terrestrial gateway, reinforcing convergence risk: an adversary intercepting traffic within the shared terrestrial cloud infrastructure or at the gateway captures both the terrestrial cellular path and the “satellite” path simultaneously.

The alternative **Regenerative payload model** places the RU and DU on-board the satellite, with only the CU or 5GC remaining terrestrial. This reduces ground-segment RAN sharing but is severely constrained by satellite power, thermal limits, and in-orbit computational budgets. It is the architecturally cleaner option for PATH but is rare in current commercial deployment.

**Operational implication:** Deployments must affirmatively select operators with non-converging backhaul, not merely diverse radio access. The technology demonstration framework should include AS-level traceroute analysis across all candidate paths to identify shared transit, gateway termination points, and shared O-RAN VNF pools. Operators marketing “satellite + cellular” as physically diverse paths must be challenged on their backhaul topology — specifically, whether they operate transparent or regenerative satellite payloads, and where their satellite gateway processing terminates. For high-assurance deployments, the NTN path should terminate at a dedicated ground station and route via independent transit, not via the shared 5G core that already terminates the cellular path. Where available, regenerative-payload NTN architectures should be preferred over transparent-payload architectures.

**Status: OPEN** — formal AS-level path divergence measurement methodology required; recommended for inclusion in the empirical validation work flagged in Section 12.

### 3.4 Transport: UDP and Choreographed Multipath Dispatch

> **Plain language:** TCP waits for a reply before the next message is sent. On a 600ms satellite link, that waiting is prohibitively expensive. UDP just sends. PATH takes advantage of this by coordinating share dispatch across paths — each share goes out on the best available channel at the right moment, not sequentially and not all at once.

PATH uses UDP transport over choreographed multipath dispatch for four load-bearing reasons:

- **No handshake per path.** UDP requires no connection establishment before sending.
- **Choreographed dispatch.** Shares are dispatched in a coordinated sequence that avoids co-channel interference and optimizes power draw. See Section 8 for the full Interference-Adaptive Band Orchestration model.
- **Tolerance for loss.** The (t, n) threshold scheme already assumes some shares will not arrive.
- **Stateless relay nodes.** UDP’s statelessness aligns with the blind-relay model.

### 3.5 Catcher Federation Architecture

> **Plain language:** A single catcher is a sitting duck — one address, one target. PATH’s catcher is not one machine. It is a large public ingestion layer of hundreds of edge nodes that receive shares, plus a small protected interior layer of reassembly nodes that the public network cannot directly address. Edges are sacrificial and replaceable. Interior nodes are hidden and stable. No single node holds enough to reconstruct a session alone.

The catcher is decomposed into three functional layers to address the catcher denial-of-service attack (Section 2.2.5) and to remove the single-point-of-compromise property of monolithic catcher designs.

#### 3.5.1 Three-Layer Decomposition

**Edge Layer (public).** A directory-published set of ingestion endpoints distributed across geographies and network operators. Senders randomly select edges from the directory for each session — different shares within a session may go to different edges. Edge nodes:

- Receive incoming shares from PATH transports
- Read only the outer manifest, including the routing tag (see 3.5.2)
- Forward shares inward to the interior layer via dynamic routing
- Hold no reassembly state and cannot decrypt share contents
- Are sacrificial: an edge node can be lost, compromised, or DoS’d without affecting session integrity at the interior

**Forwarding Layer (private).** Dynamic routing fabric between edge and interior. Each share is forwarded along a path selected per-share, not per-session, breaking the static topology that would otherwise allow traffic analysis to identify interior node locations. Forwarding can be implemented as onion-style encrypted hops, network slice isolation, or operator-internal VPN — the requirement is that no single observer external to the operator can map edge-to-interior associations.

**Interior Layer (protected).** A smaller set of reassembly nodes not directly addressable from the public network. Interior nodes:

- Receive shares forwarded from edges via the forwarding layer
- Decrypt the inner manifest (see 3.5.2) to determine session identity
- Correlate shares belonging to the same session
- Perform Shamir reconstruction once t shares are accumulated
- Execute the post-handshake operation (TLS handoff, authentication forwarding, etc.)
- Hold session reassembly state for the minimum duration required and never persist session keys

The interior layer is the trust root of the catcher federation. Interior nodes require HSM-grade operational security: aggressive state expiry, no persistent logging of session material, geographic distribution across jurisdictions to preserve the path divergence argument established in Section 3.3.

#### 3.5.2 Two-Layer Manifest Applied

The three-layer decomposition requires the two-layer manifest design specified as the mitigation for attack 2.2.4. Concretely:

**Outer manifest (readable by edges):**

- Hop count and TTL for relay routing
- Assembly deadline (so edges discard expired shares)
- Routing tag: a truncated HMAC of the session identifier, typically 8–16 bits, computed by the sender as `HMAC(catcher_federation_key, session_id)[0:k]`
- No session identity, no share index, no (t, n) parameters

**Inner manifest (readable only by interior):**

- Full 256-bit session identifier
- Share index i
- Total share count n, threshold t
- Catcher federation identity
- Encrypted under the interior layer’s collective public key using ML-KEM-768

#### 3.5.3 Edge-to-Interior Routing via Truncated Routing Tag

The truncated routing tag is the load-bearing mechanism that allows efficient routing without exposing session identity to edges.

When a sender prepares a share, it computes the routing tag from the session ID using an HMAC keyed to the catcher federation’s published key. The tag length k is a deployment parameter:

- **k = 0 (no truncation, full session ID exposure):** Edges can correlate all shares of a session. Maximum routing efficiency, minimum edge trust. Appropriate only for Mode A deployments (Section 3.5.5).
- **k = 8 to 16 bits (truncated tag):** Edges see only the routing bucket. Shares from a single session collide into the same bucket; shares from many sessions also collide into shared buckets. An edge node performing traffic analysis sees bucket-level distribution, not unique sessions.
- **k = full session ID (no truncation, edges blind):** Edges cannot route efficiently and must rely on gossip between interior nodes. Maximum trust separation, lowest routing efficiency.

The recommended default is k = 8 bits. This produces 256 routing buckets, sufficient to distribute load across interior nodes while ensuring each bucket carries many sessions concurrently. An adversary compromising an edge node learns only which bucket each share belongs to, not which session.

Edges route by hashing the routing tag to an interior node via consistent hashing. The interior node receiving a share reads the inner manifest, recovers the full session identifier, and correlates the share with others matching that session ID. Interior nodes gossip session state across the federation so that a misrouted share (e.g., due to bucket collision drift or interior node failover) is re-forwarded to the correct assembler.

#### 3.5.4 Session Convergence Without Centralized Coordination

Convergence — ensuring all t shares of a session arrive at the same interior assembler — is achieved through three mechanisms operating in concert:

1. **Routing tag determinism.** Shares from the same session carry the same routing tag and therefore route to the same interior bucket by default.
1. **Interior gossip.** Interior nodes broadcast session IDs they are currently assembling. A misrouted share arriving at the wrong interior is forwarded to the correct one.
1. **Assembly deadline.** All shares for a session must arrive within the deadline. After expiry, partial assemblies are discarded.

This is gossip-with-deterministic-hint convergence: the routing tag provides the hint, gossip provides the recovery mechanism for hint failures. No centralized session-to-interior mapping is required.

#### 3.5.5 Deployment Profiles

The catcher architecture supports two deployment profiles distinguished by trust model, not by wire protocol. The same protocol serves both with deployment-time parameter selection.

**Mode A — Single Trusted Operator**

- One organization operates both edge and interior layers
- Routing tag length k can be set to full session ID exposure for maximum efficiency
- Trust model: customers trust the operating organization with full session metadata visibility
- Example: Skylo offering catcher federation as a managed service to consumer authentication customers
- Appropriate for: high-throughput consumer authentication, financial services, any deployment where the operator’s commercial trust relationship with customers already covers session metadata access

**Mode B — Federated Operators**

- Multiple organizations operate independent catcher networks
- Each organization runs its own edge and interior layers
- Routing tag length k set to 8–16 bits to limit edge trust
- Senders choose which organization’s catcher to use per session
- Trust model: no single organization sees all sessions; cross-organization correlation requires colluding operators
- Example: defense customers running their own catcher networks alongside commercial operators
- Appropriate for: high-assurance deployments, multi-tenant federations, jurisdictionally diverse customer bases

The same client library, the same wire protocol, and the same edge/interior architecture serve both modes. Deployment selects the trust profile via the routing tag length parameter and the directory selection logic.

#### 3.5.6 Why This Defeats Catcher DoS

The single-receiver vulnerability (2.2.5) is structurally eliminated rather than mitigated:

- **No single endpoint to flood.** An attacker must DoS hundreds of edge nodes simultaneously to deny session establishment. Volumetric attacks scale poorly across geographically distributed, operator-diverse edges.
- **No reachable assembly target.** Interior nodes have no public route. An attacker cannot direct traffic at them. The forwarding layer is operator-internal.
- **Edge compromise yields bucket-level metadata, not sessions.** An adversary who compromises an edge node sees routing buckets and forwarding patterns, not session identities or content.
- **Interior compromise is the residual risk.** Interior nodes hold reassembly authority and must be protected accordingly. This becomes attack 2.2.11 in the threat model — a smaller, harder, and more contained attack surface than the original “compromise the catcher” attack.

### 3.6 Return Path Architecture

The construction as described in Sections 3.1–3.4 dispatches shares from originating device to catcher federation. Two return path models serve different deployment contexts.

#### Type 1 — Mesh-Native Return

**Target context:** Denied or degraded environments. MCDC deployments. Military operations. Disaster response.

Every relay node that forwarded an inbound share observed the originating device’s mesh identifier and the hop path it arrived from. By the time the catcher federation is ready to return a payload, the mesh has already constructed a return route as a passive byproduct of forward traffic. The routing table builds itself.

Return payloads are encrypted to the originating device’s public key before entering the mesh. Relay nodes carry opaque ciphertext they cannot read. The blind relay model applies in both directions.

> **Plain language:** The mesh built a return address by watching where the original message came from. The reply travels back along a path the network already knows — assembled automatically, without anyone planning it.

#### Type 2 — Hybrid TCP Handoff

**Target context:** Consumer authentication, browser-based login, financial services. Environments where TCP infrastructure exists and PATH is used only for session establishment.

PATH handles the credential handshake. TCP handles everything after. The Skylo-managed catcher federation reconstructs K, authenticates the credential material, and forwards the assembled authentication payload to the home network over conventional TCP. The remainder of the session runs over conventional TLS-over-TCP directly between the device and the home network.

The federation’s role is bounded to the handshake duration. It never sees bulk session content and never stores credentials.

### 3.7 Browser Integration and Consumer Authentication

#### The WebAuthn Parallel

WebAuthn established the pattern for integrating hardware-backed authentication into the browser without requiring browser rebuilds. PATH follows the same pathway. A PATH browser API intercepts credential submission events on PATH-enabled endpoints. Instead of submitting credentials over a single TLS connection, the browser PATH library:

1. Hashes the credential material
1. Splits the hash via the PATH threshold scheme
1. Dispatches shares across divergent paths to the nearest catcher federation endpoint
1. Waits for the federation ACK confirming session establishment
1. Proceeds with the authenticated session over conventional TLS

The website operator declares PATH support in authentication response headers. No other change is required. The user experiences no visible change.

#### Scale Implications

Every login event on the internet is a potential PATH authentication. Browser-integrated PATH authentication operating at consumer scale means billions of handshakes per day through the Skylo catcher federation. At that scale, the federation is not routing packets — it is brokering trust at global authentication scale. The revenue model, the valuation multiple, and the strategic position all change accordingly.

#### Credential Privacy

The federation reconstructs K — the session key — but not the credential itself. The credential is hashed by the browser PATH library before threshold splitting. The federation receives shares of a hash, verifies it against the home network’s authentication system, and confirms the session. It never holds a credential that could be replayed or exfiltrated.

-----

## 4. Security Analysis

### 4.1 Confidentiality of K Under Bounded Passive Observation

Against an adversary controlling fewer than t of the n transport paths, K is information-theoretically hidden. This derives directly from Shamir’s construction [1]: any set of fewer than t shares reveals zero information about the shared secret. This is not a computational hardness assumption.

### 4.2 HNDL Resistance

Against an HNDL adversary with future quantum capability, K is protected if either (a) the network assumption holds — fewer than t paths captured — or (b) the post-quantum KEM on each share resists the future adversary. The construction provides defense in depth: the adversary must defeat both assumptions simultaneously to succeed.

### 4.3 What PATH Does Not Defend Against

Timing correlation against a global passive adversary. Endpoint compromise. Legal compulsion of catcher operators. These are explicit residuals, not oversights. The Interference-Adaptive Band Orchestration model substantially mitigates timing correlation as an emergent property, but does not eliminate it against a sufficiently resourced global observer.

### 4.4 Delivery Confirmation

PATH handles delivery confirmation at the application layer. The confirmation signal for a complete and valid transmission is the successful reconstruction of K and completion of the TLS handshake. This eliminates a separate acknowledgment round trip over a constrained link — TLS handshake completion is the confirmation.

-----

## 5. NTN as a First-Class Anchor Substrate

The path-diversity argument requires at least one path whose surveillance is administratively or jurisdictionally orthogonal to the other paths. In most deployment contexts, that path is most naturally provided by a non-terrestrial network. Satellite ground stations are geographically concentrated, operated under identifiable national legal regimes, and reach terrestrial infrastructure through identifiable gateways.

This is not a claim that satellite links are inherently confidential. The claim is more precise: satellite transport routes around terrestrial aggregation points. The adversary who has invested in terrestrial tap infrastructure does not automatically convert that investment into satellite tap capability.

The commercial availability of NTN-capable commodity devices under 3GPP Release 17 is what makes this substrate practically accessible for the first time. CSS integration (Section 9) strengthens the NTN anchor argument further: spreading gain lowers the minimum detectable signal threshold at the satellite receiver, extending the NTN path into marginal conditions — remote locations, low-elevation geometry, adverse weather — where conventional modulation fails.

-----

## 6. Proximity-Mesh Relay Extension

### 6.1 The Core Insight

> **Plain language:** Imagine four people in a building with one bar of signal. Each person’s phone has degraded cellular, a satellite MCDC connection, and Bluetooth. Instead of one person fighting a bad signal alone, all four phones become the transmission infrastructure — each carrying one share of the session key over its own path. The recipient receives a clean, authenticated delivery. Four bad paths together are more reliable than one bad path alone.

In the standard PATH construction, a single sender A dispatches n shares across n transport paths. In the proximity-mesh extension, A enlists k co-located relay devices via short-range radio. Each relay receives one or more shares and dispatches them over its own independent radio access paths.

### 6.2 Security Properties

- **Share confidentiality is unchanged.** Relay devices receive KEM-wrapped share ciphertext. They cannot decapsulate without the catcher’s private key.
- **The threshold property is preserved.** An adversary who compromises all relay devices still cannot reconstruct K unless they accumulate t shares. Relay compromise is an availability attack, not a confidentiality attack.
- **Physical proximity creates a new attack surface.** Relay devices communicate with the sender over Bluetooth, which has limited range. Deployments in adversarial physical environments should consider encrypted Bluetooth transport or Wi-Fi Direct with WPA3.

-----

## 7. Deployment Contexts

### 7.1 Integration with MCDC

> **Plain language:** MCDC is a wholesale satellite-cellular service providing a backup data path when terrestrial cellular fails. It is designed for public safety and fleet operators. PATH’s proximity-mesh extension fits naturally because every MCDC-capable device already has satellite connectivity.

MCDC (Mission-Critical Data Channel) is a wholesale satellite-cellular make-before-break data delivery service targeted at public safety and fleet operators. It operates at 1–5 kbps, approximately 600ms latency, UDP-only, across 37 countries, via 3GPP NTN infrastructure.

The fit is natural on four dimensions. MCDC already provides the NTN substrate PATH requires as an anchor path. MCDC’s UDP-only constraint aligns with PATH’s choreographed multipath dispatch model. MCDC’s target buyers routinely operate in groups of co-located personnel — the proximity-mesh relay model maps directly to a team of first responders. MCDC’s Mission tier is exactly the traffic class for which PATH’s session-establishment protection is most valuable.

**MCDC as the PATH beachhead.** MCDC is not merely a compatible deployment vehicle — it is the strategic beachhead from which PATH’s broader architecture becomes commercially viable. The mission-critical context provides customers who accept battery tradeoffs because the alternative is mission failure, a proving ground that validates PATH under the most demanding real-world conditions, government and defense contract revenue that funds Phase 2 silicon development, and field data on orchestration performance and mesh relay reliability under adversarial conditions.

### 7.2 Military Deployment

#### 7.2.1 The Infrastructure Dependency Problem

Today’s battlefield communications are vulnerable at the infrastructure layer. Jam the tower, kill the network. Spoof the GPS, blind the unit. Destroy the forward operating base relay, isolate the unit. Every centralized dependency is a target. Adversaries with peer or near-peer electronic warfare capability have spent decades cataloging and developing weapons against exactly those targets.

The response has been to harden infrastructure — more redundant towers, more frequency-agile radios, more sophisticated jamming-resistant waveforms. This approach works until it doesn’t. Infrastructure, however hardened, remains infrastructure. It has a location. It has a signature. It can be found and targeted.

PATH inverts this dependency. There is no tower to jam. No central node to target. No infrastructure to destroy. The network is the devices. Destroy half of them and the mesh reconstitutes around what remains. The system degrades gracefully rather than failing catastrophically. This is not an incremental improvement to survivability. It is a change in the survivability calculus at the architectural level.

#### 7.2.2 Spectrum Freedom in Contested Environments

Commercial spectrum regulation — FCC Part 15, ITU coordination frameworks — governs peacetime civilian use. In a contested operational theater, the applicable legal framework is the Law of Armed Conflict (LOAC). Operational spectrum use is governed by NTIA authority and theater commander discretion.

This removes the primary constraint on PATH’s multi-band operation. The Semtech LR1121 provides CSS operation across sub-GHz bands (150–960 MHz), 2.4 GHz ISM, S-Band, and L-Band satellite on a single die — with spreading factor programmable from SF5 to SF12 and bandwidth from 62.5 kHz to 500 MHz. In an operational theater, a PATH-equipped unit operates across the full LR1121 band stack simultaneously without regulatory friction.

#### 7.2.3 Passive Attribution Ambiguity

ISM bands — 915 MHz and 2.4 GHz — are saturated with civilian traffic in any populated operational area: Wi-Fi routers, Bluetooth devices, LoRa agricultural sensors, consumer IoT. A PATH mesh operating in ISM bands is swimming in this traffic. Its CSS transmissions, recoverable from below the noise floor by an intended receiver, are structurally indistinguishable from civilian LoRa sensor traffic to any receiver not specifically configured to correlate PATH’s chirp parameters.

The distinction from conventional LPI waveforms matters. Traditional LPI attempts to hide a signal’s existence through technical means. PATH’s attribution ambiguity operates at the semantic level: even if the signal is detected, attributing it to a military command mesh rather than a civilian sensor network requires correlating timing, geography, and content across many intercepts in dense civilian traffic generating identical signatures. The civilian infrastructure provides attribution cover by existing — it requires no manufactured deception.

#### 7.2.4 Directed Injection Architecture for Denied-Area Operations

In some operational scenarios, mesh nodes cannot maintain direct NTN uplinks — building interiors, underground positions, heavily shielded environments, or situations where individual node NTN transmission creates unacceptable RF exposure.

The Directed Injection Architecture addresses this through a two-layer model. A high-gain directional antenna — phased array, parabolic dish, or electronically steerable array — transmits toward the general geographic area occupied by the mesh. The beam is aimed at a zone, not a specific node. Any node within the beam footprint receives the injected signal and relays it into the mesh interior via the standard store-and-forward relay model.

The injection point can be a forward drone, a disposable relay, or a vehicle that repositions after each transmission burst. It holds no message content between transmissions — it is a one-time-use pipe, not a persistent node. This model applies in reverse for exfiltration: a mesh node near the zone boundary can transmit a short CSS burst toward a designated collection point without exposing interior mesh topology.

#### 7.2.5 Interference-Adaptive Band Orchestration as Jamming Cost Imposition

Rather than dispatching shares simultaneously across all bands — which would be detectable and power-wasteful — PATH’s IABO scheduler retransmits shares across successive band cycles on a randomized schedule derived from the real-time interference map. The scheduler selects the cleanest available band for each dispatch, threading transmission bursts between interference windows.

The effect on an adversary attempting to jam delivery is asymmetric. To block delivery, a jammer must sustain broadband noise across every band PATH is cycling through, for the entire duration of the retry window. Attacker cost scales as (number of bands) × (retry window duration) × (power per band). For a PATH mesh cycling across four bands with a 60-second delivery window, this is a sustained, high-power, broadband jamming operation.

Defender cost scales as the power draw of a single CSS burst per retry per band — milliwatts, not watts. The asymmetry between attacker jamming cost and defender retransmission cost is several orders of magnitude. This is not passive resilience. It is active cost imposition on the adversary’s electronic warfare resources.

#### 7.2.6 Jammer Self-Nomination as a Targeting Signature

The combination of attribution ambiguity and IABO produces an emergent property with direct fires integration value: Jammer Self-Nomination.

In an environment where PATH mesh traffic is indistinguishable from civilian ISM activity, a jammer attempting to suppress PATH must emit broadband noise across ISM bands continuously and at sufficient power to overwhelm CSS spreading gain. This jamming signature is not naturally occurring. Civilian devices do not generate broadband ISM noise at the power levels required to suppress below-noise-floor CSS transmissions.

The moment an adversary begins jamming PATH, they distinguish themselves from the ambient civilian RF environment. The PATH mesh’s IABO scheduler — already maintaining a real-time interference map across all monitored bands — detects the anomalous broadband emission, characterizes its power and spectral signature, and reports it as a detected emitter through the mesh to command.

The adversary has nominated themselves as a target. This property requires no additional sensing hardware. The interference map that enables power-efficient share dispatch is the same map that detects anomalous jamming emissions. The targeting value is a byproduct of the efficiency optimization, not a purpose-built sensor capability.

#### 7.2.7 The Complete Military Value Stack

|Property                   |Mechanism                                            |Adversary Problem                                                    |
|---------------------------|-----------------------------------------------------|---------------------------------------------------------------------|
|Cryptographic fragmentation|PATH threshold share dispatch                        |No single intercept carries a complete message                       |
|Mesh resilience            |No fixed infrastructure                              |No node or link to target or destroy                                 |
|CSS jamming resistance     |Spreading gain, recoverable below noise floor        |Must jam entire spread bandwidth to suppress                         |
|Attribution ambiguity      |ISM band operation in civilian RF environment        |Cannot attribute military mesh from civilian traffic                 |
|Asymmetric jamming cost    |IABO band cycling, randomized retry                  |Sustaining jamming costs orders of magnitude more than retransmission|
|Jammer self-nomination     |Interference map detects anomalous broadband emission|Jamming attempt generates a targeting signature                      |

The first three properties make the network hard to defeat. The last three make the attempt to defeat it operationally costly and potentially fatal for the adversary’s electronic warfare assets.

### 7.3 Emergency Communications

#### 7.3.1 The Disaster Zone as a Contested Battlefield

The 2018 Camp Fire — which destroyed the town of Paradise, California in under four hours — demonstrated a failure mode that has since repeated in every major wildland-urban interface disaster: simultaneous, total, multi-layer communications collapse.

Cellular towers burned or lost grid power. Backhaul fiber severed. Repeater infrastructure offline. Landlines dead. Every communications system shared a single architectural failure mode: dependency on infrastructure outside the disaster zone. When the grid failed, every system that required the grid failed with it.

This is the civilian equivalent of the military communications problem: infrastructure-dependent systems collapsing under adversarial conditions. The adversary in Paradise was fire. The result was identical to what a peer electronic warfare campaign achieves against conventional military communications.

PATH’s architecture addresses this at the root. A PATH mesh requires nothing outside the mesh to function. There is no backhaul to sever. No tower to burn. No grid dependency. As long as devices with stored power exist within relay range of each other, the mesh operates.

> **Plain language:** In Paradise, the communications network died because it was built on infrastructure that burned. A PATH mesh keeps running as long as any two devices can hear each other. You don’t need a tower. You don’t need power lines. You don’t need cell service. The phones — or dedicated mesh nodes — are the network.

#### 7.3.2 Spectrum Constraints in Disaster Zones

When cellular towers are dark and Wi-Fi routers have lost power, the spectrum they occupied is empty. There is no co-existing civilian traffic to interfere with. A PATH mesh operating across 433 MHz, 915 MHz, and 2.4 GHz in a burned community is not stomping on anyone’s network. Those networks no longer exist.

The FCC recognizes this in existing emergency communications doctrine. Part 97 permits operators to use any means necessary to protect life and property in declared emergencies. In practice, no enforcement action has ever been taken against emergency operators for interference with infrastructure that was already offline. The regulatory constraints that govern civilian PATH deployment effectively suspend themselves when the infrastructure those regulations protect no longer exists.

#### 7.3.3 CSS Indoor Penetration as a Life-Safety Property

At MCDC data rates with 15–21 dB of spreading gain, CSS extends link viability through construction materials that block conventional satellite and cellular signals entirely. A victim trapped in collapsed concrete — which attenuates conventional signals by 20–30 dB — is invisible to cellular search and rescue coordination. A PATH mesh node in the same structure can reach nodes outside through walls and debris that conventional signals cannot penetrate.

At the attenuation levels typical of collapsed concrete, the difference between CSS spreading gain and conventional modulation is binary at the link level: the link closes or it doesn’t. First responders inside a structure face the same attenuation from the other direction. PATH CSS mesh nodes — pre-positioned or carried by responders — maintain communication through the structure to incident command outside.

#### 7.3.4 Directed Injection for Out-of-Zone Coordination

Emergency management command — outside the disaster zone with full connectivity — needs to push information into the affected area: evacuation routes, shelter locations, resource staging, search and rescue priorities, hazmat boundaries. In a total infrastructure blackout, no existing system provides a reliable path for this traffic.

The Directed Injection Architecture resolves this. A satellite NTN uplink from outside the disaster zone injects messages into the local PATH mesh via directed transmission. The injection platform can be an aircraft orbiting the zone perimeter, a drone at altitude, a vehicle at a staging area on the zone boundary, or a satellite with sufficient ground footprint. The return path — mesh nodes near the zone boundary relaying survivor and responder status back — closes the bidirectional loop.

#### 7.3.5 Pre-Positioning as Community Infrastructure

A wildfire-prone community can deploy a PATH-capable mesh as permanent community infrastructure, dormant under normal conditions and autonomous under disaster conditions. LR1121-class nodes — similar to the Muzi Works Duo, with CSS across sub-GHz and 2.4 GHz bands, solar charging, and weatherproof enclosures — cost under $100 per node at current retail pricing and are falling.

Deployed at utility pole spacing throughout a community, with battery backup and solar charging, they operate indefinitely without grid power. Under normal conditions, the infrastructure is invisible. When the grid goes down and cellular fails, the mesh activates automatically around surviving nodes. No operator intervention required.

Community members with a PATH-compatible application are automatically part of the mesh when cellular fails. Their device joins the nearest surviving node and becomes both a consumer of mesh services and a relay for other devices.

The cost of pre-positioning this infrastructure at the Paradise scale — approximately 27,000 residents across a dispersed community — at current node pricing is within the range of a municipal emergency preparedness budget. The cost of not having it was 85 lives and $16.5 billion in losses.

### 7.4 PATH as IoT Substrate

#### 7.4.1 The IoT Fragmentation Problem

The current IoT protocol landscape is fragmented across incompatible standards: Zigbee, Z-Wave, BLE Mesh, LoRaWAN, Thread, Sigfox, and Matter each own distinct deployment segments with no interoperability between them. More critically, every major IoT protocol shares a structural dependency: a hub, gateway, coordinator, or controller that is a single point of failure.

The infrastructure dependency that Section 7.2 identifies as the foundational military vulnerability is equally present in every consumer and industrial IoT protocol deployed today. Remove the LoRaWAN gateway and the network dies. Remove the Zigbee coordinator and the mesh loses its management layer. This is not an edge case — it is the architectural norm.

#### 7.4.2 Why CSS over 915 MHz and 2.4 GHz Changes the IoT Calculus

PATH operating on LR1121-class hardware with CSS across 915 MHz and 2.4 GHz ISM bands has structural properties that no current IoT protocol achieves simultaneously:

**CSS operates outside the Wi-Fi collision domain.** Zigbee, BLE Mesh, and Thread all operate at 2.4 GHz and participate in the CSMA/CA contention environment that also hosts Wi-Fi. In a dense 2.4 GHz environment — an apartment building, a commercial facility, an urban deployment — these protocols degrade as channel contention increases. CSS transmissions at 2.4 GHz are recoverable from below the noise floor of Wi-Fi receivers. They do not participate in CSMA/CA contention. A PATH mesh does not get slower as the Wi-Fi environment gets busier.

**Sub-GHz propagation penetrates where 2.4 GHz cannot.** LoRa’s success in industrial and agricultural IoT is largely a 915 MHz physics story: sub-GHz wavelengths penetrate construction materials, foliage, and terrain that block 2.4 GHz entirely. PATH on LR1121 hardware operates at 915 MHz and 2.4 GHz simultaneously, selecting the optimal band per transmission via IABO. The mesh uses 2.4 GHz where the propagation geometry is favorable and drops to 915 MHz where penetration or range matters.

**No gateway. No coordinator. No SPOF.** The PATH mesh is the network. Every node is simultaneously a consumer of mesh services and a relay for other nodes. The mesh reconstitutes around failed or destroyed nodes automatically. LoRaWAN’s gateway dependency — the architectural feature that makes it unsuitable for disaster or military deployment — makes it equally unsuitable for any IoT deployment where the gateway is a maintenance burden, a reliability risk, or a cost center.

**NTN uplink native.** No competing IoT protocol has native satellite connectivity on the same hardware die. Every LoRaWAN, Zigbee, BLE, Thread, or Z-Wave deployment terminates at the mesh boundary — its data reaches the internet only through a gateway with a terrestrial connection. PATH’s LR1121-based nodes carry L-Band and S-Band satellite connectivity natively.

#### 7.4.3 Comparison with LoRaWAN

LoRa is PATH’s closest IoT competitor — both use CSS modulation. The differentiation is architectural, not physical-layer:

|Dimension          |PATH (LR1121)                         |LoRaWAN                                   |
|-------------------|--------------------------------------|------------------------------------------|
|Topology           |Infrastructure-free mesh              |Star-of-stars, gateway mandatory          |
|Gateway dependency |None                                  |Hard SPOF — gateway loss = network failure|
|Operating bands    |915 MHz + 2.4 GHz + NTN on one die    |Single band per gateway deployment        |
|NTN uplink         |Native (L-Band + S-Band)              |Requires separate satellite modem         |
|Mesh resilience    |Self-healing around node loss         |No mesh — gateway is non-redundant        |
|Cryptographic model|Threshold shares, ML-KEM, post-quantum|AES-128 symmetric, single ciphertext      |

LoRa proved the CSS physical layer works at scale. PATH inherits that proof and resolves LoRaWAN’s architectural limitations.

#### 7.4.4 IoT Deployment Targets

The combination of CSS congestion immunity, sub-GHz penetration, infrastructure-free mesh, and native NTN uplink positions PATH as a viable substrate for IoT deployments that current protocols cannot serve:

- **Dense urban IoT** where 2.4 GHz contention degrades Zigbee and BLE performance
- **Industrial IoT** in facilities where concrete construction blocks 2.4 GHz and gateway infrastructure is a maintenance burden
- **Agricultural and remote monitoring** where LoRaWAN gateways require power and backhaul that don’t exist
- **Disaster-resilient community infrastructure** where gateway dependency is a categorical failure mode
- **Military IoT** where gateway SPOFs are unacceptable by definition

The same hardware, the same protocol, the same mesh — deployed across these verticals without modification.

-----

## 8. Interference-Adaptive Band Orchestration

### 8.1 The Battery Problem

Multi-RAN simultaneous broadcasting — maintaining active connections across Wi-Fi, cellular, LoRa, Bluetooth, and NTN simultaneously — is the naive implementation of PATH’s path-diversity requirement. It is not the correct implementation. Simultaneous broadcast requires each radio to be powered and transmitting concurrently. The power drain would be prohibitive for any consumer or field-deployed device. All the security value in the world does not matter if devices can only operate for a fraction of their current battery life.

### 8.2 Interference-Adaptive Band Orchestration

The solution is not simultaneous broadcast. It is Interference-Adaptive Band Orchestration (IABO).

Instead of all radios transmitting at once, the PATH firmware maintains a real-time interference and availability map across all accessible bands and channels — which channels are busy, which RANs are congested, which paths are clean at this moment, what the power cost is for each available option.

The firmware dispatches share fragments in coordinated bursts that thread between interference windows across available substrate. No two fragments compete for the same channel at the same time. Each fragment travels on the best available path at the moment it is dispatched.

The orchestration layer operates with three simultaneous objectives:

1. **Battery efficiency.** The scheduler minimizes radio activations by selecting the cleanest available band for each dispatch — one transmission on the right band at the right moment, rather than broadcasting across all bands wastefully.
1. **Interference avoidance.** A share dispatched on a congested channel has higher retransmission probability, which costs more total power than dispatching on a clean channel the first time. The interference map scores candidate bands before each dispatch.
1. **Path diversity preservation.** Despite optimizing for efficiency, the scheduler maintains the cryptographic requirement: shares must traverse genuinely divergent paths. Efficiency optimization operates within the constraint of path diversity, not at its expense.

> **Plain language:** Instead of every radio shouting at once — which is where the power drain comes from — the system reads the radio environment and sends each fragment through a clean channel at the right moment. The message still travels across multiple independent paths. The paths are just used intelligently rather than simultaneously.

This model preserves all of PATH’s path-diversity security properties. The catcher federation is indifferent to dispatch timing — it accumulates shares as they arrive and reconstructs K when threshold is reached. The orchestration is invisible to the cryptographic construction.

### 8.3 The Airtime Budget Constraint

CSS airtime is expensive. At SF12, a 50-byte payload occupies approximately 2.5 seconds of on-air time. LoRaWAN imposes 1% duty cycle limits in EU-regulated bands precisely because CSS airtime consumption at scale creates collision risk even below the CSMA/CA collision domain.

IABO addresses this directly: by selecting the cleanest band for each dispatch rather than retransmitting across all bands simultaneously, the scheduler minimizes total airtime consumed per message. The delivery guarantee comes from intelligent path selection, not brute-force retransmission.

The honest constraint: airtime budget is finite and must be managed. The orchestration layer is the management mechanism. In civilian regulated deployments, FCC Part 15 duty cycle limits apply and must be respected by the scheduler policy. In military operational contexts under LOAC, these constraints do not apply and the full band stack is available without duty cycle restriction.

### 8.4 Emergent Randomness as a SIGINT Defeat Property

Choreographed transmission across interference-avoiding channels produces transmission behavior that is inherently random in timing, frequency, and burst pattern. This randomness is a byproduct of the power optimization logic, not a designed security feature.

An adversary observing the RF environment across all relevant bands simultaneously sees what appears to be unrelated native traffic — some Wi-Fi activity here, a cellular burst there, a LoRa chirp somewhere else, an NTN ping at an irregular interval. Nothing correlates. Nothing patterns. The unified PATH communication is indistinguishable from background noise across multiple domains — not because it is encrypted, but because intelligent power management produces structurally undetectable transmission behavior as a byproduct.

### 8.5 The Two-Phase Orchestration Maturation Arc

PATH’s IABO is designed for a specific technology maturation sequence with clear precedent in wireless protocol history. Wi-Fi band steering started as software logic in access points and progressively embedded into chipsets. Bluetooth adaptive frequency hopping followed the same arc: protocol specification → software implementation → silicon. PATH’s IABO follows the same path.

#### Phase 1 — Software-Layer Orchestration (Current)

The LR1121 chipset provides the multi-band hardware substrate. PATH firmware implements the interference map, band scoring, and adaptive dispatch logic entirely in software on the application processor.

The Phase 1 implementation is deployable now on existing hardware. Its constraints are accepted tradeoffs for the mission-critical context: software polling latency limits switching speed to hundreds of milliseconds rather than microseconds; cross-radio coordination running through the application processor adds power overhead that purpose-built silicon would eliminate; the choreography is less precise than hardware would achieve but sufficient to validate the model under operational conditions.

Phase 1 deployment is not the target state — it is the instrument that defines what the target state must achieve. It generates the real interference environment data, real power consumption measurements, and real path availability characterization that informs the silicon design.

#### Phase 2 — Silicon-Embedded Orchestration (Target State)

The orchestration logic migrates from software into a dedicated hardware scheduler — a radio coordination processor integrated with the RF front end. Target capabilities: microsecond-latency band switching driven by hardware interference detection; orchestration logic on a dedicated low-power processor rather than the application CPU; antenna switching fabric integrated with band selection logic; power draw from orchestration itself approaching near-zero as a fraction of total device power budget.

**The pitch to silicon vendors.** PATH’s Phase 1 software implementation generates a validated algorithm with real-world interference data. The pitch to Semtech or a defense silicon vendor is not “build something unproven” — it is “embed this validated scheduling algorithm into your next-generation multi-band die.” The software phase de-risks the silicon investment.

**Strategic implication.** The orchestration algorithm is PATH’s durable IP. Hardware gets commoditized. The intelligence layer — the algorithm that decides when, which band, which spreading factor, and which share — is what survives chipset generations. PATH’s moat is the protocol and its orchestration logic, not the silicon it initially runs on.

### 8.6 Why Existing Silicon Is Insufficient for Phase 2

Current device architectures present three barriers to the Phase 2 target:

**Independent radio management.** Each radio on a current smartphone is managed by independent firmware with independent power states. Cross-radio coordination at the millisecond timing granularity required for interference-aware choreography is not available through existing software interfaces.

**Sequential power management assumptions.** Existing power management hardware assumes radios are used primarily one at a time, with handoffs. The power sequencing logic is not designed for rapid multi-radio coordination with shared interference awareness.

**Antenna design.** Current antennas are optimized for one dominant path. Multi-band choreography requires antenna design supporting rapid switching across frequency bands without the tuning delays that would break the interference-avoidance timing model.

The required chip architecture is an intelligent scheduler — a radio coordination processor that maintains a live interference map, manages rapid switching across radios with minimal latency, and optimizes power draw across the transmission burst sequence. The closest current analog is software-defined radio platforms, which demonstrate that the coordination logic is feasible but are not power-optimized for mobile deployment.

### 8.7 Silicon Design Requirements

The PATH multi-RAN coordination chip must provide:

- Real-time interference mapping across Wi-Fi, cellular (LTE/5G), CSS (sub-GHz and 2.4 GHz), Bluetooth, and NTN bands simultaneously
- Sub-millisecond radio switching latency to enable interference-window threading
- Coordinated power sequencing that minimizes peak draw by staggering radio activation
- Antenna switching fabric designed for rapid multi-band transitions
- Programmable dispatch scheduling with firmware-accessible interference policy
- Hardware-accelerated share signing and verification to keep cryptographic operations off the application processor
- Path quality monitoring with feedback to the interference map

-----

## 9. CSS Physical Layer and Link Budget

### 9.1 What LoRa Proved

LoRa demonstrated through Chirp Spread Spectrum (CSS) modulation:

- Communication recoverable from below the noise floor — the signal is detectable by receivers tuned for it but indistinguishable from background noise to others
- Inherent jamming resistance — CSS spreads the signal across a wide frequency band over time, making targeted jamming difficult without broadband power
- Long range at low power — spreading gain allows communication at distances that would require far more power using conventional modulation
- Low probability of intercept — the signal resembles noise to equipment not specifically configured for CSS reception

These properties were achieved at 1–5 kbps. LoRa is slow by deliberate design tradeoffs. The CSS modulation itself is not the bandwidth bottleneck.

### 9.2 CSS at Higher Throughput

CSS has been applied at high data rates before — early 802.11 Wi-Fi used direct-sequence spread spectrum, a CSS derivative. The physics of spreading signals across spectrum for noise resistance are not fundamentally limited to low bandwidth. The tradeoffs shift, but the core properties are preserved:

- **Jamming resistance by physics.** A jammer targeting a CSS signal must broadcast across the full spread spectrum bandwidth or acquire the spreading code — which PATH’s IABO randomizes per session.
- **Substrate orthogonality.** CSS transmissions on one band do not interfere with orthogonal CSS transmissions on another band, enabling the multi-channel choreography model.

### 9.3 Technical Challenges at High Throughput

High-throughput CSS is not a solved engineering problem:

- **Spreading factor tradeoffs.** Spreading gain compresses as data rate rises. At 50 Mbps with 100 MHz of bandwidth, residual gain is 3–6 dB — meaningful but not the 21 dB available at MCDC data rates.
- **Spectrum allocation.** High-throughput CSS requires more spectrum than LoRa’s narrow bands. Military and government deployments have access to spectrum allocations — AEHF, WGS — that commercial deployments do not.
- **Receiver complexity.** CSS receivers capable of high-throughput demodulation are more complex and power-intensive than LoRa receivers. This is a hardware design problem, not a fundamental physical constraint.

These challenges are appropriate for a DARPA-scale development program. The MCDC Phase 1 deployment generates the contract revenue and field evidence that justifies the investment.

### 9.4 CSS Satellite Link Budget

#### What a Link Budget Is

A link budget accounts for signal power from transmitter to receiver across a satellite path. The transmitting device starts with a fixed power level. That power attenuates across distance, atmospheric absorption, and geometric losses. The satellite receiver must collect enough signal power above its noise floor to decode the transmission successfully.

#### What CSS Changes

In conventional modulation, a signal must arrive above the noise floor to be decoded. If it arrives too weak, it is lost. CSS receivers decode signals that arrive below the noise floor. The spreading gain effectively lowers the minimum detectable signal power. This is not a software trick — it is a property of spread spectrum physics, demonstrated at scale in every LoRa deployment.

#### Spreading Gain Formula

**Spreading Gain (dB) = 10 × log₁₀(Bandwidth / Data Rate)**

At LoRa standard parameters — 125 kHz bandwidth, 1 kbps data rate:

Spreading Gain = 10 × log₁₀(125) ≈ **21 dB**

A CSS receiver at these parameters can close a link 21 dB weaker than conventional modulation requires. In satellite engineering terms, 21 dB is transformative — the difference between requiring a professional-grade dish antenna and using a small patch antenna on a field device.

#### The Spreading Gain vs. Data Rate Tradeoff

Spreading gain and data rate are locked in a direct tradeoff. Every tenfold increase in data rate costs 10 dB of spreading gain at constant bandwidth.

|Target Data Rate        |Bandwidth for 21 dB Gain|Bandwidth for 10 dB Gain|Bandwidth for 3 dB Gain|
|------------------------|------------------------|------------------------|-----------------------|
|1 kbps (MCDC Phase 1)   |125 kHz                 |10 kHz                  |2 kHz                  |
|100 kbps                |12.5 MHz                |1 MHz                   |200 kHz                |
|1 Mbps                  |125 MHz                 |10 MHz                  |2 MHz                  |
|10 Mbps                 |1.25 GHz                |100 MHz                 |20 MHz                 |
|50 Mbps (Phase 3 target)|6.25 GHz                |500 MHz                 |100 MHz                |


> **Plain language:** At MCDC’s 1–5 kbps, CSS delivers approximately 21 dB of link budget improvement using a narrow slice of spectrum — achievable within existing satellite allocations. At 50 Mbps, maintaining the same advantage would require 6.25 GHz of continuous spectrum, which does not exist as a usable allocation. The gain compresses as data rate rises. This is physics, not an engineering failure.

#### Phase-Specific Value Propositions

**Phase 1 — MCDC (1–5 kbps): Link Budget Story**

At MCDC data rates, 15–21 dB of spreading gain is achievable within existing NTN spectrum allocations. Devices close the satellite link at significantly lower transmit power, extending battery life. Coverage extends into marginal conditions — precisely where PATH’s NTN anchor path is most needed. Below-noise-floor transmission renders the uplink attribution-ambiguous to conventional monitoring equipment.

**Phase 3 — High-Throughput (10–50 Mbps): Jamming Resistance Story**

At high data rates, spreading gain compresses to 3–6 dB. The link budget improvement is modest. However, CSS retains meaningful value: a jammer targeting a 50 Mbps CSS signal must match a 100 MHz footprint — an expensive and conspicuous operation. The Phase 3 value proposition is communications survivability under active electronic attack, not link budget.

#### The Stealth Compounding Effect

A CSS signal below the noise floor is hard to detect before it is transmitted. Monitoring equipment scanning the uplink spectrum sees background noise, not a transmission event. CSS satellite uplinks extend PATH’s attribution ambiguity from the terrestrial layer to the NTN layer. An adversary attempting to identify PATH sessions must simultaneously defeat IABO-randomized terrestrial transmission and detect attribution-ambiguous CSS uplinks at the satellite layer — with no common transmission signature to correlate across both.

> **Validation note:** Spreading gain values above are derived from spread spectrum theory anchored to LoRa’s empirically validated performance. They have not been measured against actual NTN uplink conditions at satellite frequencies. NTN-specific validation via SDR testbed is required before these numbers are stated as performance claims.

### 9.5 Indoor Penetration

#### Why Indoors Is Hard for Satellite Today

Current NTN systems require line of sight or near-line of sight. Buildings attenuate satellite signals significantly:

|Construction Material |Typical Signal Loss|
|----------------------|-------------------|
|Glass window          |2–4 dB             |
|Drywall / wood frame  |5–10 dB            |
|Brick / concrete block|15–20 dB           |
|Reinforced concrete   |20–30 dB           |
|Underground / basement|30–40+ dB          |

This limits NTN deployment to predominantly outdoor use cases. For PATH specifically, this is a material limitation: the NTN anchor path must be available precisely in the environments where terrestrial infrastructure fails — building interiors during disasters, indoor military operations, urban first-responder deployments.

#### What CSS Spreading Gain Changes

At MCDC data rates with 15–21 dB of spreading gain, CSS effectively adds 15–21 dB of link margin to the budget:

|Construction Material |Typical Loss|15 dB Gain Margin |21 dB Gain Margin       |
|----------------------|------------|------------------|------------------------|
|Glass window          |2–4 dB      |✅ Full penetration|✅ Full penetration      |
|Drywall / wood frame  |5–10 dB     |✅ Full penetration|✅ Full penetration      |
|Brick / concrete block|15–20 dB    |⚠️ Marginal        |✅ Marginal to meaningful|
|Reinforced concrete   |20–30 dB    |❌ Insufficient    |❌ Insufficient          |
|Underground / basement|30–40+ dB   |❌ Insufficient    |❌ Insufficient          |


> **Plain language:** At full CSS spreading gain, PATH’s NTN anchor path works indoors through wood-frame and drywall construction — the majority of residential buildings in North America — without requiring the user to go outside or position near a window. This moves the NTN path from an outdoor emergency capability to a persistent indoor-capable background service for the majority of real-world deployment environments.

#### The Mesh Relay Compensation Effect

Where CSS spreading gain does not fully close the indoor link — brick structures, partially shielded environments — PATH’s proximity mesh relay provides a compensating mechanism. A device near a window or exterior wall with adequate satellite link conditions can relay shares for deeper indoor devices over Bluetooth LE or Wi-Fi Direct. The mesh does not require every device to independently close the satellite link — it requires enough nodes distributed across the environment to maintain adequate path diversity in aggregate.

In a team of first responders entering a concrete structure, a single device positioned near an exterior wall can serve as the NTN relay node for the entire team’s PATH sessions.

#### Consumer Authentication Implications

CSS indoor penetration removes the last behavioral barrier to mainstream consumer adoption of PATH-backed authentication. Today’s NTN-based services require behavior change — users must go outside or position near a window. People log into their bank from their couch. CSS spreading gain at MCDC data rates makes the NTN anchor path available indoors in standard residential and commercial construction, with no behavior change required. The login hardening PATH provides works transparently, everywhere the user already is.

#### Honest Caveats

- The gain is at the receiver, not against building loss directly. CSS lowers the noise floor at the satellite receiver. The device still transmits at its normal power level through the building. Building loss must be overcome by the total link budget — CSS spreading gain increases the margin available for that budget.
- Construction type varies significantly. North American residential wood-frame is favorable. European masonry is more challenging.
- Measured validation is required. The penetration estimates above are derived from published building attenuation figures combined with theoretical spreading gain projections.
- Reinforced concrete remains a hard limit. No commercially practical CSS implementation closes a 30 dB building loss at useful data rates within available spectrum.

-----

## 10. Universal Mesh Architecture

### 10.1 The Infrastructure Cost of Dumb Packets

The current networking infrastructure exists in its current form because packets are dumb. Routers and switching fabric exist to read packet headers and make forwarding decisions. Firewalls and deep packet inspection exist because packets cannot prove their own legitimacy. Load balancers and CDNs exist because content cannot route itself. NAT infrastructure exists because IP addressing was not designed for current scale. The aggregate power consumption, physical infrastructure, and operational cost of these compensating systems is substantial — a meaningful fraction of global data center energy consumption is overhead, not useful data movement.

### 10.2 What Universal Mesh Changes

Universal mesh — every device a node, every device a relay, PATH as the intelligence layer, any available substrate as the transport — eliminates the overhead layer: the infrastructure that exists purely to compensate for protocol limitations.

When messages self-authenticate cryptographically, deep packet inspection becomes largely unnecessary. When messages self-route via the assembly manifest, routing tables shrink. When trust travels with the message, re-establishment at every hop through new certificate chains is eliminated. When fragmentation across divergent paths is built into the protocol, redundant infrastructure for delivery guarantees becomes optional.

More significant than the infrastructure savings is what becomes possible when the overhead is removed: networks that do not exist today because they are too expensive to secure and manage, edge deployments that do not pencil out under current cost models, and connectivity in environments where centralized infrastructure cannot be built, maintained, or trusted.

### 10.3 The Historical Parallel

The internet did to the telephone network what universal mesh does to the internet. Voice used to require dedicated circuit infrastructure — a physical path established end-to-end for the duration of a call. The intelligence lived in the circuit. The internet said: packetize it, route it through shared infrastructure, move the intelligence to the edges. The dedicated circuit business collapsed. Infrastructure became commodity.

PATH applied universally repeats this disruption at the next layer. The dedicated routing intelligence — routers, switching fabric, carrier backbone — gets replaced by emergent mesh intelligence carried in the messages themselves. The substrate becomes commodity. The protocol layer becomes the value.

### 10.4 The Incentive Barrier

5G sidelink — device-to-device communication without tower involvement — is already specified in the 5G standard (3GPP Release 16). It is not activated at scale because carriers have no incentive to enable it. Device-to-device mesh would cannibalize the infrastructure access revenue that carriers depend on.

This is not a technical barrier. It is an incentive barrier. Incentive barriers break from the outside, in contexts where the incumbent’s incentives do not apply. Military, disaster response, denied environments, and developing world infrastructure share the common property that centralized carrier infrastructure either does not exist, cannot be trusted, or has been destroyed. These are the contexts where universal mesh becomes undeniably necessary rather than merely preferable. Proof in those contexts generates the reference architecture and political momentum that forces universal mesh into the mainstream.

-----

## 11. Commercialization Roadmap

### 11.1 Sequencing Logic

The commercialization sequence is determined by two constraints:

1. **The battery constraint.** Consumer devices cannot accept the power tradeoff of Phase 1 software orchestration. New silicon is required before consumer deployment is viable.
1. **The silicon investment constraint.** New silicon development requires proof of concept at scale and government or institutional contract revenue to justify the investment.

These constraints force a specific sequence: deploy first in contexts where the battery tradeoff is acceptable, generate the field evidence and revenue that funds silicon development, then bring the consumer product to market with the tradeoff already solved.

### 11.2 Phase 1 — MCDC Deployment (Years 1–3)

**Objective:** Prove PATH in the most demanding real-world environment. Generate government reference customers and field data.

**Target customers:** Military units, public safety agencies, emergency management organizations, critical infrastructure operators.

**Product:** Purpose-built MCDC-PATH devices. Small form factor, single mission, known battery tradeoff. Software-layer IABO on existing LR1121-class hardware.

**Key milestones:**

- Reference implementation with measured handshake latency, share-loss tolerance, and relay reliability
- First government or public safety deployment contract
- Field data collection: interference environments, path availability, power consumption under operational conditions
- SIGINT resistance characterization in operational environments

**Revenue model:** Device sales, MCDC service subscription, backend catcher federation service.

**Strategic outcome:** PATH is proven under conditions civilian networking cannot replicate. Government reference customers provide the credibility that unlocks institutional investment for Phase 2.

### 11.3 Phase 2 — Silicon Development (Years 2–5, overlapping Phase 1)

**Objective:** Develop the multi-RAN coordination chip specified in Section 8. Remove the battery constraint from the PATH deployment model.

**Funding path:** Phase 1 contract revenue plus DARPA or defense innovation investment. Phase 1 field evidence provides the technical justification. Government reference customers provide the acquisition signal that defense R&D programs respond to.

**Development program:**

- SDR prototype implementation of the IABO coordination scheduler to validate the interference-avoidance model
- Power consumption modeling for the orchestration approach versus naive simultaneous broadcast
- ASIC design for the multi-RAN coordination processor
- Antenna design for rapid multi-band switching
- Integration validation with PATH protocol stack

**Parallel track — High-throughput CSS and Satellite Link Budget:**

- CSS feasibility study at 10, 25, and 50 Mbps target data rates
- Spreading factor optimization for NTN uplink geometry
- CSS satellite link budget characterization: empirical measurement of link margin improvement at NTN uplink frequencies
- Regulatory engagement for spectrum access in military and emergency management allocations

**Strategic outcome:** A chip that enables PATH on devices with consumer-acceptable battery life.

### 11.4 Phase 3 — Consumer and Enterprise Mesh (Years 4–8)

**Objective:** Deploy PATH as the universal mesh protocol layer.

**Enabling conditions:**

- Phase 2 silicon available for integration into consumer devices
- PATH protocol standardization (IETF or 3GPP track)
- Regulatory framework for high-throughput CSS spectrum access
- SDK available for third-party device integration

**Market entry sequence:**

1. Enterprise and government (path diversity and resilience as primary value)
1. Developing world (connectivity where carrier infrastructure does not exist)
1. Disaster response and emergency management (infrastructure-independent communication)
1. Consumer mainstream (mesh connectivity as a feature, carrier dependency as optional)

**Revenue model:** SDK licensing, protocol royalties, catcher federation service subscription, enterprise deployment contracts.

### 11.5 The Skylo Strategic Position

NTN is the one substrate that works when everything terrestrial fails. Every PATH-enabled device requires at least one NTN anchor path for the path-diversity argument to hold in contested or degraded environments. A PATH deployment at scale is a persistent NTN connectivity requirement at scale.

The operator of that NTN substrate — embedded in the protocol architecture as a required anchor path — is not a connectivity provider competing on price. It is foundational infrastructure for the next generation of networking. Revenue model, valuation multiple, and competitive moat all change when the NTN path is not optional but architecturally required.

### 11.6 Tiered Federation Operator Model

PATH is an open protocol. The federation architecture must balance two competing imperatives: openness maximizes network effects and adoption velocity; restriction of the NTN anchor path preserves Skylo’s structural moat. The tiered federation model resolves this tension.

#### Tier 1 — Anchor Federation Operators

Tier 1 operators provide at least one certified NTN path as part of their federation endpoint. The NTN path must satisfy the jurisdictional divergence requirements of Section 3.3 — it must physically bypass terrestrial aggregation points and operate under a separate national legal regime from the operator’s terrestrial paths.

Skylo holds Tier 1 status by virtue of existing NTN infrastructure. The certification standard for Tier 1 qualification is developed and maintained by the PATH protocol steward. Being first to define the standard means writing the rules that govern who can compete at this tier.

Tier 1 operators are structurally necessary to every PATH deployment. Every session requires at least one NTN anchor share. Every Tier 2 operator depends on a Tier 1 operator for that share.

#### Tier 2 — Terrestrial Federation Operators

Tier 2 operators run their own PATH catcher federation endpoints — banks securing their own authentication infrastructure, enterprises hardening their own login systems, government agencies operating sovereign authentication services. Tier 2 operators provide terrestrial path diversity but do not operate NTN substrate.

Every Tier 2 deployment has a Tier 1 dependency built into the protocol architecture. Tier 2 operators pay Tier 1 operators for NTN anchor path access on a per-session basis. This creates a persistent revenue stream for Skylo from every Tier 2 deployment globally, without Skylo operating the consumer-facing infrastructure.

#### Network Effects and the Open Protocol Argument

PATH’s value increases with adoption. Every new device that speaks PATH, every new federation operator that joins the network, every new substrate PATH can traverse makes the protocol more valuable for every existing participant. This is the same dynamic that made TCP/IP the universal protocol.

The recommended posture: open the protocol fully, open Tier 2 federation to any operator, and make Tier 1 qualification requirements technically rigorous but openly documented. Skylo’s head start in NTN infrastructure, government relationships, and protocol development is the durable advantage — not artificial gatekeeping.

#### ⚠️ Open Question — Satellite Operator Federation Rights

*Flagged for resolution before protocol standardization.*

The current tiered model does not resolve whether competing satellite operators (Viasat, AST SpaceMobile, Iridium, Telesat, and others) may qualify as Tier 1 Anchor Federation Operators.

**The case for allowing competing satellite operators at Tier 1:** More Tier 1 operators means more NTN path diversity, which strengthens the PATH security argument. Exclusion could be read as anticompetitive. A competitor locked out of Tier 1 has strong incentive to fork the protocol.

**The case for restricting Tier 1 to Skylo (or requiring licensing):** Skylo’s NTN position is the primary commercial moat. Allowing competitors to operate equivalent Tier 1 endpoints erodes the structural dependency. Tier 1 certification could require a licensing relationship with the PATH protocol steward — open in principle, revenue-generating in practice.

*Recommendation: This question requires legal, competitive strategy, and business model input before resolution. The tiered architecture accommodates either outcome. The protocol specification should defer this decision — Tier 1 qualification requirements should be defined technically, with licensing and competitive access policy resolved through Skylo’s go-to-market strategy process.*

-----

## 12. Open Problems

Open problems are organized by priority. Implementation-blocking issues must be resolved before technical due diligence can proceed. Pre-deployment issues must be resolved before field deployment. Research issues are appropriate for the Phase 2 and Phase 3 program tracks.

### 12.1 Implementation-Blocking

**Relay forwarding policy specification.** The self-routing relay model described in Section 3.2 requires formal specification of four relay behaviors: relay decision logic (when to forward vs. hold), catcher reachability (how relay nodes maintain a lightweight catcher directory), TTL and loop prevention (hop count field in the manifest), and store-and-forward policy (hold duration bounded by manifest deadline). The assembly manifest provides the destination intelligence — the relay forwarding policy completes the routing model. This is the correct framing: PATH uses a DTN-adjacent store-and-forward model, not a traditional mesh routing protocol requiring topology convergence.

**ML-KEM fragmentation and reassembly protocol.** ML-KEM-768 public key material (1,184 bytes) exceeds maximum CSS payload sizes at high spreading factors (approximately 51 bytes at SF12, 255 bytes at SF7). A specified fragmentation and reassembly protocol is required before the cryptographic architecture claim is fully deployable. This protocol must handle partial share delivery, reassembly timeout, and replay protection across a lossy CSS mesh.

### 12.2 Pre-Deployment

**LR1121 simultaneous vs. TDD multi-band confirmation.** Confirm with Semtech FAE whether the LR1121 supports true simultaneous multi-band transmission or time-division switching between bands. If TDD, the IABO model is unchanged in concept but the timing model requires revision — switching latency between bands affects the minimum interference window that can be exploited.

**Airtime budget modeling under IABO.** A quantitative model of total airtime consumption per message under Interference-Adaptive Band Orchestration, across the full band stack, at representative spreading factors, is required. This model must verify compliance with FCC Part 15 duty cycle limits for civilian deployment and characterize the operational envelope for military deployment where those limits do not apply.

**Node-to-satellite uplink link budget at device power levels.** A specific link budget calculation for a small-form-factor node uplink to LEO satellite at each LR1121-supported band is needed to confirm which uplink paths are closable without directional antenna hardware. Free-space path loss to LEO at 600 km altitude at 2.4 GHz is approximately 165 dB. The directed injection downlink is credible and well-characterized. The node uplink requires explicit demonstration that CSS spreading gain closes the link at practical TX power levels.

**Silicon power modeling.** A quantitative comparison of power consumption between naive simultaneous multi-RAN broadcast and the IABO model across representative PATH workloads is required to validate the battery life improvement claim.

**Type 1 return path routing table convergence.** The mesh-native return model relies on passive routing table construction from forward share traffic. The convergence properties of this table — how quickly it builds, how long entries remain valid, how it degrades under node churn — require formal analysis and empirical measurement.

### 12.3 Research Track (Phase 2–3)

**Formal security proofs.** The security claims in Section 4 are informal. A full proof requires a formal adversary model and a reduction from the confidentiality of K to the hardness of the underlying KEM and the information-theoretic property of Shamir.

**CSS high-throughput feasibility.** The spreading factor optimization for 50 Mbps CSS requires empirical measurement. Theoretical models for noise-floor properties at high data rates need validation against hardware implementations.

**AS-level divergence with NTN.** Costea et al. [15] provide empirical measurements of AS-level divergence for terrestrial paths. A comparable study for NTN-inclusive path sets in target deployment geographies is a prerequisite for quantitative security claims.

**Federation Byzantine bounds.** The federated-catcher model tolerates fewer than t_c compromised federation members. The precise relationship between t_c, federation member count, and synchronization channel integrity deserves formal treatment.

**CSS satellite link budget empirical validation.** Section 9.4 derives the link budget improvement argument from spread spectrum physics and LoRa field evidence. Quantitative characterization at NTN uplink frequencies — L-band, S-band, Ka-band — with measured spreading factor optimization for LEO and GEO geometries is required before the link margin improvement claim can be stated with precision.

**Indoor penetration empirical measurement.** Section 9.5 maps CSS spreading gain against published building attenuation figures. Actual penetration performance at NTN satellite uplink frequencies in representative building types requires controlled measurement. Published building loss figures are derived from terrestrial frequency bands and may not transfer directly to satellite uplink geometry.

**Proximity-mesh Bluetooth attack surface.** The security properties of Bluetooth LE as a share-distribution medium in adversarial physical environments require dedicated analysis.

**NB-IoT-NTN share-size budget.** ML-KEM-768 ciphertext at 1,088 bytes exceeds common NIDD transaction sizes. An engineering study of which PQC parameter sets are practical for this substrate is needed before NB-IoT-NTN deployment.

**Regulatory pathway for high-throughput CSS.** The spectrum regulatory framework for CSS operation above LoRa’s current band allocations, particularly for commercial consumer devices, requires engagement with FCC and international equivalents.

**Browser API standardization pathway.** The WebAuthn parallel suggests a standards track through W3C or IETF. Timeline, stakeholder engagement, and interim deployment model (extension vs. native) need scoping.

**Type 2 federation availability and failover.** The hybrid TCP handoff model introduces a dependency on Skylo federation availability for session establishment. Failover behavior when the nearest federation endpoint is unreachable, and latency implications of routing to a more distant endpoint, require specification.

**⚠️ Open Question — Satellite operator Tier 1 federation rights.** See Section 11.6. Requires legal, competitive strategy, and business model resolution before protocol standardization.

-----

## 13. Conclusion

The construction presented here is narrow on purpose at the cryptographic layer and deliberately expansive at the architectural layer.

The cryptographic core protects session-establishment material against the harvest-now-decrypt-later adversary. It composes threshold secret sharing of session keys over deliberately divergent transport paths, with non-terrestrial networks as the substrate that makes the divergence argument plausible in a world of terrestrial transit consolidation. The assembly manifest makes each share self-describing — carrying its own routing intelligence — so that relay nodes require no topology state, no routing tables, and no infrastructure coordination to forward shares toward their destination.

The physical layer adopts Chirp Spread Spectrum modulation, which provides spreading gain that moves PATH’s transmissions to the attribution-ambiguous zone in dense civilian ISM band traffic, extends satellite link budgets into marginal conditions including building interiors, and creates the asymmetric jamming cost structure that makes electronic attack on PATH self-defeating.

The Interference-Adaptive Band Orchestration layer manages the radio environment intelligently — selecting clean bands at the right moments, preserving battery life, maintaining path diversity as a constraint on efficiency rather than sacrificing it — and produces, as an emergent byproduct of power optimization, transmission behavior that defeats traffic analysis without any deliberate deception.

The deployment arc — MCDC in mission-critical contexts where the security value justifies the battery tradeoff, silicon development funded by those deployments, consumer mesh as the end state — sequences the technical and commercial dependencies in the only order that works. The beachhead markets prove the construction under the hardest conditions and generate the reference architecture that justifies the investment that makes the consumer product possible.

The individual components of this construction are not new. Threshold secret sharing, multipath key exchange, post-quantum key encapsulation, BLE mesh relay, and Chirp Spread Spectrum are all mature. What is new is their composition on a transport substrate that simultaneously supports cellular, Wi-Fi, NTN satellite, and short-range mesh on commodity devices available under open standards — combined with a self-routing payload model and an intelligent orchestration layer that together eliminate the infrastructure dependencies that make every current alternative fragile.

PATH treats that substrate as a first-class architectural resource. The roadmap treats MCDC as the wedge that proves the construction under the most demanding conditions and funds the development that makes it universal.

-----

## References

- [1] Shamir, A. “How to Share a Secret.” *Communications of the ACM* 22(11):612–613, 1979.
- [2] Desmedt, Y. “Threshold Cryptography.” *European Transactions on Telecommunications* 5(4):449–458, 1994.
- [3] Heinrich, A. et al. “Who Can Find My Devices?” *Proceedings on Privacy Enhancing Technologies* 2021(3):227–245, 2021.
- [4] Dingledine, R., Mathewson, N., and Syverson, P. “Tor: The Second-Generation Onion Router.” *USENIX Security Symposium*, 2004.
- [5] 3GPP TR 38.821. “Solutions for NR to Support Non-Terrestrial Networks (NTN).” Release 16, 2019.
- [6] NIST FIPS 203. “Module-Lattice-Based Key-Encapsulation Mechanism Standard.” 2024.
- [7] NIST FIPS 204. “Module-Lattice-Based Digital Signature Standard.” 2024.
- [8] NIST FIPS 205. “Stateless Hash-Based Digital Signature Standard.” 2024.
- [9] Rescorla, E. “The Transport Layer Security (TLS) Protocol Version 1.3.” RFC 8446, 2018.
- [10] Cerf, V. et al. “Delay-Tolerant Networking Architecture.” RFC 4838, 2007.
- [11] W3C. “Decentralized Identifiers (DIDs) v1.0.” W3C Recommendation, July 2022.
- [12] W3C. “Verifiable Credentials Data Model v1.1.” W3C Recommendation, March 2022.
- [13] Dorsey, J. et al. “Bitchat.” Open-source BLE mesh messaging application, released July 2025. Source: <https://github.com/jackjackbits/bitchat>. Operational deployment record: Madagascar, Nepal, Uganda, Iran (2025–2026).
- [14] Boloorchi, A.T., Samadzadeh, M.H., and Chen, T. “Symmetric Threshold Multipath (STM): An online symmetric key management scheme.” *Information Sciences* 268 (2014) 489–504.
- [15] Costea, S., Choudary, M.O., Gucea, D., Tackmann, B., and Raiciu, C. “Secure Opportunistic Multipath Key Exchange.” *Proceedings of the 2018 ACM SIGSAC Conference on Computer and Communications Security (CCS 2018)*, pp. 2077–2094.
- [16] Fischlin, M. and Günther, F. “Multipath TLS 1.3.” *Security and Cryptography for Networks (SCN 2021)*, Lecture Notes in Computer Science, vol. 13061, Springer, 2021.
- [17] 3GPP TS 23.287. “Architecture enhancements for 5G System (5GS) to support Sidelink Communication.” Release 16, 2020.
- [18] Semtech Corporation. “AN1200.22 — LoRa Modulation Basics.” Application Note, 2015.

-----

*Correspondence: James Pan, Irvine, California.*
*This preprint will be superseded by a subsequent version incorporating formal security proofs, prototype measurements, silicon feasibility study results, browser API standardization pathway, relay forwarding policy specification, ML-KEM fragmentation protocol, and resolution of the satellite operator federation rights open question.*

-----

## Appendix A — Phase 1 Hardware Architecture

*Confirmed via RF engineering review, May 2026. Supersedes any prior references to “simultaneous multi-band transmission.”*

### A.1 LR1121 Architecture — TDD Confirmed

The Semtech LR1121 is a single-die, half-duplex transceiver with a single internal synthesizer and shared modem architecture. Band switching is software-commanded via `lr1121_set_rf_frequency()`. The chip cannot transmit on multiple bands simultaneously.

**Why simultaneous is impossible at the physics level.** The 2nd harmonic of a 915 MHz transmission falls at 1,830 MHz. The 3rd harmonic falls at 2,745 MHz. At the power levels required for useful CSS range, these harmonics bleed into and desensitize a co-located 2.4 GHz receiver. True simultaneous dual-band TX would require two completely independent RF synthesizer chains and two separate modems on a single die — which does not exist in any current low-power IoT silicon.

**Why TDD is the correct architecture, not a limitation.** IABO’s time-division switching model is not a workaround for a hardware limitation — it is the correct RF engineering solution. Commercial multi-band LoRaWAN gateways are built today using exactly this approach: separate Semtech modules (SX1303 for sub-GHz, SX1280 for 2.4 GHz) coordinated by a host processor. IABO adds intelligence to that coordination — band scoring, interference awareness, path diversity preservation — that commercial gateways do not have.

### A.2 Phase 1 Node Hardware Stack

A Phase 1 PATH node requires multiple NiceRF modules because a single RF front end cannot achieve peak performance across all PATH bands simultaneously. The RF matching network — capacitors, inductors, baluns, filters — is frequency-specific. No single off-shelf module exposes all bands at full performance.

**Recommended Phase 1 module stack:**

- **NiceRF LoRa1121** (868/915 MHz BOM) — sub-GHz CSS path
- **NiceRF LoRa1121F33-2G4** — 2.4 GHz CSS path
- **NiceRF LoRa1121F33-1G9** — 1.9–2.0 GHz NTN satellite path

All three modules share a single SPI bus with dedicated Chip Select pins per module. The host MCU — ESP32-S3, STM32, or equivalent — runs the IABO scheduler software and coordinates all three modules.

### A.3 TDMA Coordination and Hardware Interlocking

The IABO scheduler enforces TDMA (Time-Division Multiple Access) across modules — when one module is transmitting, all others are locked in receive mode. This prevents harmonic interference between modules and satisfies antenna proximity constraints.

**Hardware safety layer:** NiceRF modules expose TX_EN and RX_EN GPIO pins. The host MCU uses these to implement a hardware interlock — a software crash cannot cause simultaneous transmission because the TX_EN line of a non-transmitting module is physically held low. This is standard practice in commercial multi-band radio systems.

**Channel Activity Detection (CAD):** The LR1121 includes a hardware CAD feature — a built-in listen-before-talk scan. Before each share dispatch, the IABO scheduler queries CAD on the candidate band to confirm channel availability. CAD results feed the interference map, providing fresh band scoring data at dispatch time.

### A.4 Implications for IABO Specification

The TDD architecture confirmation has one open implication for the IABO scheduler specification: the minimum exploitable interference window is bounded by the band-switching latency of the software scheduler, not by simultaneous-TX timing. At Phase 1 software switching speeds (100–500ms), the scheduler can exploit interference windows of roughly that duration. Phase 2 silicon reduces this to microseconds, enabling much finer-grained interference avoidance. The scheduler policy must be specified with awareness of the Phase 1 latency floor.

### A.5 Production Precedent

The multi-module bridge architecture is not experimental. Multi-band LoRaWAN gateways using SX1303 (sub-GHz) and SX1280 (2.4 GHz) modules on a shared host processor are deployed commercially at scale. PATH’s Phase 1 node is a miniaturized, intelligence-added version of existing production architecture — not a novel hardware design. The IABO scheduling algorithm is the new element, running on hardware whose fundamental approach is already production-proven.

-----


## Appendix E — PATH Forward Research Directions

*Documents concepts under active development that extend the core PATH architecture. These are not Phase 1 scope items. They are recorded here to establish conceptual provenance, identify prior art gaps, and inform Phase 2 hardware and firmware planning.*

-----

### E.1 PATH Mesh Acknowledgment Protocol (PMAP)

#### E.1.1 Motivation

The core PATH protocol operates over UDP exclusively. This is intentional — UDP’s stateless, connectionless model aligns with PATH’s design principle that the network is passive terrain. However, pure UDP with no acknowledgment layer leaves the Session Controller blind to assembly progress at the catcher. The Session Controller dispatches shares and has no native mechanism to know whether shares are accumulating, whether a path is silently failing, or whether the session is close to reconstruction threshold.

TCP is not the answer. TCP’s connection handshake, retransmission logic, and ordered delivery semantics reintroduce exactly the centralized session state PATH is designed to eliminate. A lightweight, purpose-built acknowledgment layer is required.

PMAP is that layer.

#### E.1.2 Core Concept

PMAP is a **UDP gossip beacon** protocol. Each catcher node periodically broadcasts a compact state beacon announcing its current share accumulation progress to any node within range. No handshake. No connection. No response required. The beacon is a broadcast — any node that hears it updates its local picture of network assembly state.

The Session Controller aggregates incoming beacons from all catcher nodes and constructs a live view of session progress across the full topology.

#### E.1.3 Beacon Structure

Each PMAP beacon is a minimal fixed-width UDP datagram:

```
[SESSION_ID 4B] [NODE_ID 2B] [SHARES_HELD 2B] [SHARES_NEEDED 2B] [STATUS 1B] [TIMESTAMP 4B]
```

Total: 15 bytes. Trivially small. Transmittable on any active path.

|Field        |Description                                                       |
|-------------|------------------------------------------------------------------|
|SESSION_ID   |Truncated session identifier — links beacon to active PATH session|
|NODE_ID      |Catcher node identifier                                           |
|SHARES_HELD  |Current share count accumulated at this node                      |
|SHARES_NEEDED|Threshold k — shares required for reconstruction                  |
|STATUS       |Enum: ACCUMULATING / STALLED / RECONSTRUCTED / SILENT             |
|TIMESTAMP    |Unix epoch seconds — beacon freshness                             |

#### E.1.4 Beacon Lifecycle

**ACCUMULATING** — Node is receiving shares. Beacon broadcasts on randomized interval within a defined window (see E.1.6). Progress is visible to the Session Controller.

**STALLED** — Share count has not increased for a configurable stall threshold (e.g., 3 beacon intervals). Node infers a path is degraded or lost. Session Controller interprets STALLED status as a reroute signal and adjusts dispatch to favor remaining active paths.

**RECONSTRUCTED** — Threshold k reached. Session key successfully reconstructed. Beacon broadcasts RECONSTRUCTED status once, then node goes silent for this session. Session Controller marks session complete.

**SILENT** — Node has detected anomalous RF monitoring or has been instructed to suppress beacons by the Session Controller. No beacon transmitted. Session Controller treats absence of expected beacon as a soft stall signal after timeout.

#### E.1.5 Session Controller Aggregation

The Session Controller on the T-Beam Supreme maintains a beacon table — a lightweight in-memory structure keyed by NODE_ID:

```
beacon_table[NODE_ID] = {
    shares_held,
    shares_needed,
    status,
    last_seen,
    stall_count
}
```

On each beacon receive event the Session Controller:

1. Updates the beacon table entry for the sending node
1. Computes aggregate progress across all catcher nodes
1. Evaluates stall conditions — any node STALLED triggers path rebalancing
1. Updates the operator UI via WiFi (laptop topology display)
1. Determines if session is complete — any node RECONSTRUCTED closes the session

#### E.1.6 AI-Managed Beacon Timing

A naive fixed-interval beacon creates a predictable RF signature. An adversary monitoring the spectrum can fingerprint a PMAP node by its transmission cadence — even without decoding the beacon content.

The edge inference layer (see E.2) manages beacon timing. Rather than transmitting on a fixed interval, the node draws the next beacon interval from a distribution shaped by local RF environment conditions. In a quiet RF environment, timing jitter is moderate. In a monitored or contested environment, the node widens the jitter window or selects intervals that blend with observed ambient transmission patterns.

The result is a beacon cadence that is statistically unpredictable to a passive observer while remaining functionally reliable for the Session Controller’s aggregation logic — the Session Controller expects beacons within a window, not at a precise moment.

This timing function is the primary intersection between PMAP and the Edge Inference Layer. It is the only AI dependency in the PMAP architecture. PMAP is functional without it — fixed-interval beacons work for Phase 1 — but AI-managed timing is the production hardening step.

#### E.1.7 Demo Integration

PMAP is implementable in Phase 1 and is the recommended data source for the laptop topology display’s share progress animation.

During demo Scenario 3 — both CSS paths disabled, cellular only — the topology display shows:

```
6 / 30 shares → 14 / 30 shares → 22 / 30 shares → RECONSTRUCTED
```

This animation is driven entirely by PMAP beacons received at the Session Controller and relayed to the laptop UI via WiFi. No additional instrumentation required. The beacon data that drives the demo UI is the same data the production protocol uses for session management.

This makes PMAP demonstrable at Phase 1 with fixed-interval beacons, and upgradeable to AI-managed timing at Phase 2 without any architectural change.

#### E.1.8 Prior Art Gap

PMAP occupies a novel position in the literature. Distributed gossip protocols are well established (Demers et al., 1987; Birman et al., 1999). UDP-based state broadcast is common in gaming and real-time systems. Threshold cryptographic session management is documented in Boloorchi et al. (2014) and Costea et al. (2018).

However, a purpose-built gossip acknowledgment layer for threshold-cryptographic share assembly — with session-scoped beacons, stall detection as a reroute signal, and AI-managed timing for emissions obfuscation — does not appear in the reviewed literature. The combination of these elements in service of a heterogeneous transport session protocol represents a meaningful extension beyond existing prior art.

-----

### E.2 Edge Inference Layer (EIL)

*Concept-stage roadmap item. Not Phase 1 scope.*

#### E.2.1 Concept

Each PATH node runs a lightweight frozen inference model — TensorFlow Lite Micro on the ESP32-S3 — that manages local radio behavior independently of the Session Controller. The Session Controller sets the dispatch envelope. The EIL operates within that envelope making local decisions the Session Controller does not need to know about.

Two primary functions:

**Radio Signature Obfuscation** — The EIL randomizes per-transmission characteristics including power level, timing jitter, burst duration, and where hardware permits, frequency selection. The goal is to prevent passive adversaries from fingerprinting the node by its emissions pattern. The inference model is trained offline on RF environment data and deployed as a frozen model. The node runs inference only — no field training.

**Path Randomization** — Within the set of paths the packet designates as acceptable, the EIL introduces controlled non-determinism in path selection. An adversary observing the network cannot learn the routing pattern because the node itself is intentionally non-deterministic within protocol bounds.

#### E.2.2 Hardware Feasibility

The ESP32-S3 on both the T-Beam Supreme and Muzi Baseband Duo is capable of running TFLite Micro inference. A quantized classification model for this use case — inputs: band RSSI, noise floor, transmission history; outputs: obfuscation profile index — is estimated at under 50KB of flash. This fits comfortably alongside existing Radio Agent or Session Controller firmware.

The SDR at each production node serves as the local sensor feeding real-time spectrum features to the inference engine. In Phase 1, the laptop SDR approximates this function for the Session Controller only.

#### E.2.3 Relationship to PMAP

The EIL’s primary intersection with PMAP is beacon timing management (see E.1.6). Beyond that, the two layers are independent. PMAP is a protocol. EIL is a local behavioral layer. Neither depends on the other for core function.

#### E.2.4 Phase 2 Milestone

EIL is explicitly a Phase 2 item. It requires offline model training on representative RF environment data, TFLite Micro integration into node firmware, and a closed SDR sensor loop at each production node. None of these dependencies exist in Phase 1 hardware.

The Phase 1 demo hardware — T-Beam Supreme and Muzi Baseband Duo — is already EIL-capable. No hardware change is required for Phase 2 EIL integration.