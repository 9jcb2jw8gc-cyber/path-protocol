# PATH — Supplementary Material: Phase 1 Demonstration Hardware and Test Procedures

*This supplement accompanies the PATH preprint and documents the Phase 1 demonstration build referenced in the main paper. It is provided as part of the full defensive disclosure.*

## Appendix B — C2 Demo Hardware Inventory

*Documents the physical hardware used in the Phase 1 military C2 demonstration build. Supersedes prior references to LilyGo T-Lora Pager as the operator terminal.*

### B.1 Confirmed Hardware Inventory

The Phase 1 C2 demonstration uses six boards across two hardware families, all running ESP32-S3 host processors.

**Muzi Works Baseband Duo (×4)**

- Radio: Semtech LR1121 — dual-band CSS, TDD architecture
- Transport paths: 868/915 MHz sub-GHz CSS + 2.4 GHz CSS
- Form factor: compact, solar-charging capable
- Expansion: Super IO board attached
- Role in demo: two assigned as fixed infrastructure nodes (one sub-GHz primary, one 2.4 GHz primary); two assigned as mobile relay nodes with both bands active, demonstrating mesh interior path diversity and session reconstitution when a fixed node goes offline.

**LilyGo T-Beam Supreme (×2)**

- Radio: Semtech SX1262 — sub-GHz only (433/868/915/923 MHz)
- Additional radios: ESP32-S3 integrated 2.4 GHz Wi-Fi (802.11 b/g/n) + Bluetooth 5 LE
- GPS: L76K
- Form factor: 18650 battery holder, 1.3-inch SH1106 OLED (128×64), IMU (QMI8658), 8MB Flash, 8MB PSRAM
- Role in demo: operator terminals. Initiate PATH sessions, display mesh topology, monitor band health across the CSS network, show the IABO interference map in real time. GPS provides operator position data for the C2 common operating picture.

### B.2 Radio Capability Matrix

Full radio and broadcast method inventory across the six-board demo stack:

|Radio / Method     |Band                |T-Beam Supreme ×2|Muzi Baseband Duo ×4|Notes                                    |
|-------------------|--------------------|-----------------|--------------------|-----------------------------------------|
|LoRa CSS           |915 MHz             |✓ SX1262         |✓ LR1121            |All 6 boards — common substrate          |
|LoRa CSS           |433 MHz             |✓ SX1262         |—                   |T-Beams only                             |
|LoRa CSS           |868 MHz             |✓ SX1262         |✓ LR1121            |EU/international band                    |
|LoRa CSS           |923 MHz             |✓ SX1262         |—                   |T-Beams only                             |
|LoRa CSS           |2.4 GHz             |—                |✓ LR1121            |Duos only — path diversity vs sub-GHz    |
|LoRa CSS           |1.9 GHz (NTN n256)  |—                |✓ LR1121            |Duos only — satellite uplink, 3GPP Rel-17|
|(G)FSK             |Sub-GHz             |✓ SX1262         |✓ LR1121            |Higher throughput, shorter range         |
|(G)FSK             |2.4 GHz             |—                |✓ LR1121            |Duos only                                |
|Wi-Fi              |2.4 GHz 802.11 b/g/n|✓ ESP32-S3       |¹                   |802.11s mesh capable, batman-adv         |
|Bluetooth 5 LE     |2.4 GHz             |✓ ESP32-S3       |¹                   |BLE mesh capable                         |
|Cellular (tethered)|LTE                 |✓ via USB-C      |—                   |Hotspot or USB modem via T-Beam          |

¹ ESP32-S3 on the Duos has integrated Wi-Fi and BLE silicon, but whether Muzi routes those antenna pins on the Baseband Duo PCB is unconfirmed — requires hardware doc verification.

**PATH-relevant transport paths for share dispatch:** The demo environment supports up to seven distinct broadcast methods. At minimum, a (2, 4) threshold scheme can be demonstrated using CSS 915 MHz, CSS 2.4 GHz, Wi-Fi mesh, and cellular — four physically independent paths across divergent spectrum and infrastructure. The NTN satellite path (1.9 GHz, LR1121 Duos) is available as a fifth path pending Skylo integration.

### B.3 Radio Architecture Note — SX1262 vs LR1121

The T-Beam Supreme’s SX1262 radio is sub-GHz only and does not support the 2.4 GHz CSS path or the 1.9 GHz NTN satellite uplink band available on the LR1121. This is not a demo limitation — the operator terminal role does not require multi-band CSS capability. The T-Beam Supreme contributes sub-GHz CSS (as a share recipient/initiator), Wi-Fi mesh, and BLE as its transport paths, while the Muzi boards carry the dual-band CSS and future NTN path responsibilities.

For Phase 1 NTN integration, the LR1121-equipped Muzi Baseband Duos are the NTN-capable nodes. The 1.9–2.0 GHz NTN uplink path (3GPP Release 17, n256 band) requires the LR1121’s extended frequency range and is not available via the SX1262.

### B.4 Demo Topology

```
[Muzi Baseband Duo]      [Muzi Baseband Duo]
  Fixed Node A             Fixed Node B
  Primary: 915 MHz         Primary: 2.4 GHz
  Super IO board           Super IO board
       |                        |
       +--------+  +------------+
                |  |
           [Mesh Interior]
     [Muzi Baseband Duo ×2]
       Mobile Relay Nodes
       Both bands active
       Super IO boards
                |
       +--------+--------+
       |                 |
[T-Beam Supreme A]  [T-Beam Supreme B]
  Operator Terminal    Operator Terminal
  L76K GPS + OLED      L76K GPS + OLED
  Sub-GHz CSS          Sub-GHz CSS
  WiFi mesh            WiFi mesh
  BLE                  BLE
```

### B.5 Cellular Path (Addendum)

The T-Beam Supreme supports USB-C and the ESP32-S3 supports USB host mode, enabling a tethered cellular modem (4G USB dongle or smartphone hotspot) as a fifth transport path via the operator terminal. This path carries full-size ML-KEM-768 key material (1,184 bytes) in a single packet, contrasting with the CSS path which requires the fragmentation and reassembly protocol (currently Open — Blocking per Section roadmap). For demo purposes, the cellular path illustrates the transport heterogeneity thesis directly: the same share that CSS delivers in fragments, cellular delivers whole.

-----

## Appendix C — Radio Abstraction Layer and IABO Bridge Architecture

*Documents the software abstraction layer and optional hardware bridge required to enable PATH’s IABO scheduler to orchestrate transmissions across all radios in the Phase 1 demo stack. Each device currently controls its own radio independently; this appendix specifies the architecture that inverts that model.*

-----

### C.1 The Orchestration Problem

The Phase 1 demo hardware comprises six boards across two radio families — four Muzi Baseband Duos (LR1121, dual-band CSS) and two LilyGo T-Beam Supremes (SX1262, sub-GHz CSS + ESP32-S3 Wi-Fi/BLE). In their default firmware state, each device owns its radio stack locally: it decides when to transmit, which channel to use, and manages its own timing. This is the Meshtastic model — correct for a peer mesh, wrong for PATH.

PATH requires the inverse: a single IABO scheduler that sees all radios as fungible transport pipes and issues time-coordinated dispatch commands to each. No individual node decides when to send a share. The scheduler decides. Nodes execute.

This requires three functional layers:

1. **PATH Session Controller** — generates shares, runs IABO scoring, computes dispatch schedules
1. **IABO Bridge** — translates dispatch schedules into per-device commands, manages timing coordination across devices
1. **PATH Radio Agent** — lightweight firmware on each node that receives commands and executes radio operations

-----

### C.2 Functional Layer Definitions

#### C.2.1 PATH Session Controller (T-Beam Supreme)

The T-Beam Supreme is the Session Controller. It owns all session-layer logic:

- **Share generation** — Shamir (t, n) split of the ML-KEM-768 session key into n shares
- **Assembly manifest** — attaches routing metadata to each share (destination catcher address, expiry timestamp, reassembly index)
- **IABO band scoring** — maintains a scored band map updated by CAD reports from Duo Radio Agents and SDR spectral feed (see C.4)
- **Dispatch schedule computation** — determines which share goes over which radio on which node at which time offset
- **Operator interface** — OLED display, GPS position, mesh topology view, live interference map visualization
- **Control plane gateway** — issues dispatch commands to the IABO Bridge over Wi-Fi; receives acknowledgments and CAD reports back

The Session Controller does not manage radio timing directly. It produces a schedule and hands it to the Bridge. This separation keeps the Session Controller portable — the same software runs on any ESP32-S3 host regardless of which radio nodes are attached.

#### C.2.2 IABO Bridge (dedicated ESP32-S3)

The IABO Bridge is a small dedicated coordinator node — a plain ESP32-S3 devkit — that sits between the Session Controller and the Radio Agents. It handles the real-time coordination problem that the Session Controller abstracts away.

**Responsibilities:**

- Maintains persistent BLE connections to all four Muzi Baseband Duos simultaneously
- Receives dispatch schedules from the Session Controller over Wi-Fi (UDP, local network)
- Translates per-share dispatch instructions into per-device BLE commands with precise timing offsets
- Aggregates CAD reports and TX acknowledgments from all Duos and forwards to Session Controller
- Acts as the timing master for TDMA coordination across all nodes — each Duo’s Radio Agent syncs its local clock to the Bridge

**Why a dedicated device rather than embedding in the T-Beam:**
The Bridge handles four simultaneous BLE connections with real-time timing constraints. Running this alongside the Session Controller’s UI, GPS, and Wi-Fi stack on the same ESP32-S3 introduces scheduling jitter that degrades timing coordination. A dedicated $12 ESP32-S3 devkit with a single responsibility is the correct engineering choice. It also creates the right architectural separation for Phase 2, where the Bridge function moves into dedicated silicon.

#### C.2.3 PATH Radio Agent (Muzi Baseband Duo firmware)

Each Muzi Baseband Duo runs a PATH Radio Agent — a lightweight firmware layer that replaces Meshtastic for demo purposes. The Radio Agent is the simplest component:

- **Command listener** — receives dispatch commands from the IABO Bridge over BLE
- **Clock sync** — synchronizes local timer to the Bridge’s timing master
- **Radio executor** — on command, calls `lr1121_set_rf_frequency()` to the commanded band, then executes TX of the received share fragment at the commanded time
- **CAD reporter** — before each TX, runs hardware CAD on the candidate band; reports channel energy measurement back to Bridge
- **TX acknowledgment** — confirms successful transmission or reports failure back to Bridge
- **TDD enforcement** — maintains local hardware interlock (TX_EN/RX_EN GPIO) per Appendix A.3; Bridge commands cannot override the hardware safety layer

The Radio Agent does not make routing decisions. It executes instructions. All intelligence lives in the Session Controller.

-----

### C.3 Control Plane Architecture

The control plane — the channel over which the Session Controller commands the Radio Agents — must be independent of the PATH share transport paths. Using a transport path to coordinate that same transport path creates a dependency loop: if the control channel is congested, share dispatch degrades.

**Selected control plane: BLE (Bridge ↔ Duos)**

BLE is the correct choice for Bridge-to-Duo control:

- Does not consume any LoRa airtime or ISM band capacity reserved for PATH shares
- ESP32-S3 BLE stack supports up to 9 simultaneous connections; four Duos is well within limit
- Round-trip latency under 10ms in a co-located demo environment — sufficient for TDMA coordination at Phase 1 software switching speeds (100–500ms per Appendix A.4)
- No additional hardware required on the Duos if ESP32-S3 BLE pins are exposed by Muzi PCB

**Selected control plane: Wi-Fi UDP (Session Controller ↔ Bridge)**

The T-Beam’s ESP32-S3 Wi-Fi and the Bridge ESP32-S3 Wi-Fi form a local AP/station pair. The Session Controller runs a Wi-Fi AP; the Bridge connects as a station. Dispatch schedules are sent as compact UDP datagrams. This keeps the Session Controller’s control link on a separate radio from the BLE mesh used to command the Duos.

**Control plane topology:**

```
[T-Beam Supreme — Session Controller]
  Wi-Fi AP (192.168.4.1)
       |
       | UDP — dispatch schedules
       | UDP — CAD reports, ACKs (return)
       |
  [ESP32-S3 — IABO Bridge]
  Wi-Fi station + BLE central
       |
       | BLE — per-device dispatch commands
       | BLE — CAD reports, ACKs (return)
       +----------+----------+----------+
       |          |          |          |
  [Duo A]    [Duo B]    [Duo C]    [Duo D]
  Radio      Radio      Radio      Radio
  Agent      Agent      Agent      Agent
```

-----

### C.4 SDR Integration — IABO Spectral Feed

A software-defined radio (SDR) in passive RX-only mode provides the IABO scheduler with a real-time wideband spectral picture that no individual node can produce. Each Duo sees only its current band via hardware CAD — a single narrowband snapshot. The SDR monitors all PATH transport bands simultaneously and feeds measured noise floor and interference energy per band into the IABO scoring function.

**Recommended hardware: ADALM-PLUTO (PlutoSDR)**

- Frequency range: 325 MHz–3.8 GHz — covers all PATH bands (433/868/915/923 MHz sub-GHz, 1.9 GHz NTN, 2.4 GHz CSS/Wi-Fi/BLE)
- Instantaneous bandwidth: 20 MHz — sufficient to park on 2.4 GHz and observe all 2.4 GHz PATH activity simultaneously
- Interface: USB, powered from host laptop or USB battery bank
- Software: GNU Radio or SDR++ for spectrum monitoring; custom Python script exports per-band power measurements to IABO scoring function via local UDP socket
- Role: **RX only in demo** — the Duos and T-Beams are the certified transmitters; the SDR is a passive sensor

**SDR integration into IABO scoring:**

```
[PlutoSDR — wideband RX]
  Monitors simultaneously:
  868/915 MHz (sub-GHz CSS)
  1.9 GHz (NTN uplink)
  2.4 GHz (CSS + Wi-Fi + BLE)
       |
       | USB → Host laptop
       |
  [GNU Radio / SDR++ + IABO Feed Script]
  Per-band noise floor measurement
  Interference spike detection
  Band scoring deltas → UDP socket
       |
       | Wi-Fi UDP
       |
  [T-Beam — Session Controller]
  IABO band scores updated in real time
  Dispatch schedule adjusted before each share TX
```

**Demo scenario enabled by SDR:** Introduce a 2.4 GHz interference source (Wi-Fi saturation, proximity to microwave oven, or deliberate jamming signal). The SDR detects the spike, the IABO scoring function penalizes 2.4 GHz CSS, the Session Controller routes the next share away from that band onto 915 MHz CSS or cellular. The session reconstitutes on clean paths. This is the jammer self-nomination property — an emergent behavior of the interference map — demonstrated in hardware rather than described in specification.

-----

### C.5 Bill of Materials — Demo Hardware

Hardware already owned is marked **[OWNED]**. New purchases required for the abstraction layer and SDR integration are marked **[NEW]**.

#### Owned Hardware

|Item                                    |Qty|Unit Cost|Total|Notes                       |
|----------------------------------------|---|---------|-----|----------------------------|
|Muzi Works Baseband Duo (LR1121)        |4  |—        |—    |[OWNED] With Super IO boards|
|LilyGo T-Beam Supreme (SX1262, L76K GPS)|2  |—        |—    |[OWNED] Operator terminals  |

#### New Hardware — IABO Bridge

|Item                        |Qty|Est. Unit Cost|Est. Total|Source           |Notes                                         |
|----------------------------|---|--------------|----------|-----------------|----------------------------------------------|
|Espressif ESP32-S3-DevKitC-1|1  |$12–15        |$12–15    |DigiKey / Amazon |IABO Bridge node; Wi-Fi + BLE, USB-C          |
|USB-C cable (short, data)   |1  |$8            |$8        |Any              |Bridge power + programming                    |
|18650 LiPo battery + holder |1  |$12           |$12       |Adafruit / Amazon|Bridge field power (optional; USB bank viable)|

#### New Hardware — SDR Spectral Sensor

|Item                          |Qty|Est. Unit Cost|Est. Total|Source                           |Notes                                  |
|------------------------------|---|--------------|----------|---------------------------------|---------------------------------------|
|ADALM-PLUTO (PlutoSDR)        |1  |$230–310      |$230–310  |Analog Devices / DigiKey / Mouser|325 MHz–3.8 GHz, 20 MHz BW, USB powered|
|SMA antenna — 915 MHz         |1  |$10–15        |$10–15    |Amazon / Taoglas                 |Sub-GHz monitoring antenna             |
|SMA antenna — 2.4 GHz         |1  |$10–15        |$10–15    |Amazon / Taoglas                 |2.4 GHz monitoring antenna             |
|SMA to SMA adapter (f-f)      |2  |$5            |$10       |Amazon                           |Antenna switching if single port used  |
|USB battery bank (≥10,000 mAh)|1  |$25–35        |$25–35    |Anker / Amazon                   |Powers PlutoSDR and Bridge in field    |

#### New Hardware — Cellular Path (Optional)

|Item                                              |Qty|Est. Unit Cost|Est. Total|Source          |Notes                                 |
|--------------------------------------------------|---|--------------|----------|----------------|--------------------------------------|
|4G LTE USB modem (e.g. GL.iNet Mudi or Netgear M2)|1  |$80–150       |$80–150   |Amazon / GL.iNet|Tethered to T-Beam Supreme via USB-C  |
|SIM card (T-Mobile prepaid)                       |1  |$10 + data    |$10+      |T-Mobile        |Cellular path for share transport demo|

#### New Hardware — Sub-GHz CSS Antenna Diversity (Phase 1 Defense)

Phase 1 antenna diversity covers the sub-GHz CSS paths only per the threat-proportionate allocation specified in Section 2.2.7. Provides defense against single-snapshot antenna fingerprinting on the highest-priority threat surface (LoRa CSS). 2.4 GHz CSS, Wi-Fi, BLE, cellular, and NTN paths excluded per the rationale documented in 2.2.7.

|Item                                               |Qty|Est. Unit Cost|Est. Total|Source          |Notes                                                                     |
|---------------------------------------------------|---|--------------|----------|----------------|--------------------------------------------------------------------------|
|915 MHz SMA antenna (high-gain, omnidirectional)   |6  |$10–15        |$60–90    |Taoglas / Amazon|One additional antenna per protected radio: 4 Duos × 1 + 2 T-Beams × 1 = 6|
|SP2T RF switch IC (Skyworks SKY13453 or equivalent)|6  |$1.50         |$9        |DigiKey / Mouser|One switch per protected radio                                            |
|SMA pigtail / U.FL adapter cables                  |12 |$3–5          |$36–60    |Amazon          |Connects antennas to switch and switch to radio                           |
|GPIO wiring and connectors                         |—  |$5            |$5        |Any             |Switch control signals from LR1121 DIO5/DIO6 or SX1262 TXEN               |

#### BOM Summary

|Category                                   |Est. Cost (low)|Est. Cost (high)|
|-------------------------------------------|---------------|----------------|
|IABO Bridge hardware                       |$32            |$35             |
|SDR spectral sensor                        |$280           |$370            |
|Sub-GHz antenna diversity                  |$110           |$164            |
|Cellular path (optional)                   |$90            |$160            |
|**Total new hardware (core demo)**         |**$422**       |**$569**        |
|**Total with antenna diversity**           |**$532**       |**$733**        |
|**Total with antenna diversity + cellular**|**$622**       |**$893**        |

*All prices estimated as of May 2026. PlutoSDR pricing varies by source; $230 direct from Analog Devices educational pricing when available, $310 typical market rate. LimeSDR Mini 2.0 (~$399) is an alternative SDR with 40 MHz instantaneous bandwidth and 10 MHz–3.5 GHz range but at higher cost with no significant demo advantage over PlutoSDR for this application. Sub-GHz antenna diversity is recommended for any demo that intends to claim physical-layer security properties; without it, the spec’s RF fingerprinting threat model (2.2.7) is documented but not demonstrated.*

-----

### C.6 Firmware Architecture Summary

Three distinct firmware codebases are required:

|Firmware               |Target Hardware  |Primary Responsibilities                                                       |Complexity|
|-----------------------|-----------------|-------------------------------------------------------------------------------|----------|
|PATH Session Controller|T-Beam Supreme   |Share generation, IABO scoring, dispatch scheduling, operator UI, GPS, Wi-Fi AP|High      |
|IABO Bridge Agent      |ESP32-S3 DevKit  |BLE connection management, timing master, command fanout, ACK aggregation      |Medium    |
|PATH Radio Agent       |Muzi Baseband Duo|BLE command listener, clock sync, LR1121 radio executor, CAD reporter          |Low       |

The Radio Agent is the recommended starting point — it is the simplest codebase, runs on familiar ESP32-S3/LR1121 hardware, and can be validated against a single T-Beam issuing commands before the full Bridge layer is introduced.

-----

### C.7 Phase 2 Architectural Destiny

The IABO Bridge is explicitly a software emulation of a function that belongs in silicon. In Phase 2, the Bridge’s timing coordination and radio command fanout move into the PATH coordination die described in the technology roadmap. The Radio Agent firmware becomes ROM in each radio module. The Session Controller logic moves into the application processor.

## The demo stack is already the correct shape. Phase 1 runs on commodity hardware with software doing the work of future silicon. Phase 2 hardens that software into dedicated hardware. The architectural boundaries established in the demo — Session Controller / Bridge / Radio Agent — map directly to the die boundaries in Phase 2 silicon.

