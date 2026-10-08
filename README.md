# Audio Pseudowire for FIDO CTAP2 (APW-CTAP)

**Status:** Experimental design proposal / defensive publication  
**Current draft:** v0.3  
**Implementation:** Not yet provided  
**Purpose:** Technical review, prior-art discovery, security review, and independent experimentation

> **APW-CTAP explores whether an ordinary bidirectional audio endpoint can serve as a narrowly scoped transport binding for FIDO CTAP2.**

The idea is intentionally simple:

```text
WebAuthn / Application
        |
       CTAP2
        |
        v
   APW Transport
        |
        v
Audio Endpoint Abstraction
   /       |        |        \
 TRRS   USB Audio  BT Audio  Virtual / VDI Audio
```

The physical connector is not the defining property.

The defining property is:

> **Can the platform expose a sufficiently controllable bidirectional audio render/capture path?**

If yes, APW may be able to construct a reliable logical link for CTAP2 without introducing a new peripheral device class.

---

## Why this exists

Many constrained platforms already support:

- audio playback;
- microphone capture;
- USB Audio Class devices;
- headset interfaces;
- Bluetooth audio;
- virtual audio; and
- remote/VDI audio redirection.

At the same time, those platforms may not support, permit, or forward:

- generic USB devices;
- external FIDO USB HID devices;
- BLE authenticator forwarding;
- NFC readers;
- vendor-specific peripheral drivers; or
- FIDO-specific remote-desktop virtual channels.

APW explores whether that asymmetry can be useful.

The proposal is **not**:

- "USB over audio";
- a general-purpose data tunnel;
- an IP tunnel;
- a new WebAuthn ceremony;
- a replacement for native FIDO transports; or
- an attempt to claim that audio is inherently secure.

The intended model is:

```text
Reuse the connector.
Reuse the audio class.
Reuse the driver.
Reuse the OS audio stack.
Reuse the VDI audio path.

Add only the protocol semantics
required for strong authentication.
```

---

## Core deployment property

Where a platform already exposes a compatible standard audio device, APW may avoid requiring an **APW-specific device driver**.

That does **not** mean "no software required."

A host still needs:

```text
Audio I/O
   |
APW Modem
   |
APW Link Layer
   |
CTAP Transport Adapter
   |
WebAuthn / Application Integration
```

The practical goal is to shift integration from **new peripheral enablement** toward **user-space or application-level protocol integration**.

---

## Underlay profiles considered

| Profile | Example attachment | Platform-visible abstraction |
| --- | --- | --- |
| APW-A | Analog TRRS | Headphone + microphone |
| APW-U | USB | Standard USB audio endpoint |
| APW-BH | Bluetooth Classic | Bidirectional headset / hands-free audio |
| APW-BL | Bluetooth LE Audio | Bidirectional LE audio endpoint |
| APW-V | Virtual audio | Virtual render/capture endpoint |
| APW-R | Redirected audio | Remote/VDI audio streams |

APW requires two logical directions. Playback-only audio is insufficient.

---

## Initial modem profile

The current draft proposes a conservative baseline profile:

- voice-band signaling;
- approximately 1 kHz to 3 kHz;
- robust binary FSK/AFSK-like signaling;
- approximately 1200 symbols/s as an initial target;
- mandatory half-duplex support;
- fragmentation and reassembly;
- CRC-based corruption detection;
- retransmission;
- optional negotiated FEC.

These parameters are intentionally provisional and should be validated experimentally.

---

## Target use cases

The original motivation is VDI / thin-client / zero-client environments where bidirectional audio is available but generic peripheral forwarding is restricted.

Potential additional environments include:

- kiosks;
- shared terminals;
- industrial HMIs;
- embedded appliances;
- smart displays;
- game consoles;
- media devices; and
- other closed or constrained platforms.

Audio support alone does **not** make a platform WebAuthn-capable. Platform or application software must still integrate APW and CTAP.

---

## Repository documents

- [`SPEC.md`](SPEC.md) — English technical specification
- [`SPEC-ja.md`](SPEC-ja.md) — Japanese technical specification
- [`PRIOR-ART.md`](PRIOR-ART.md) — known adjacent work and request for missing prior art
- [`SECURITY.md`](SECURITY.md) — threat model and security assumptions
- [`ROADMAP.md`](ROADMAP.md) — suggested PoC progression
- [`LICENSE`](LICENSE) — CC0-1.0 dedication for the current documentation

---

## What feedback is wanted

This repository is intentionally public early.

Feedback is especially welcome on:

1. **Prior art** — existing projects, papers, patents, standards work, or abandoned prototypes that are substantially similar.
2. **Security** — relay, injection, endpoint hijacking, downgrade, privacy, or threat-model weaknesses.
3. **Transport design** — framing, timing, FEC, modulation, discovery, and fallback behavior.
4. **Platform integration** — libfido2, browser/OS integration, VDI, thin clients, kiosks, consoles, or embedded systems.
5. **Independent PoCs** — implementations are welcome; no reference implementation is currently scheduled.

Please open an issue or discussion with references and technical criticism.

---

## Defensive-publication intent

This project is intentionally published as an **open technical proposal and defensive publication**.

The objective is to make the concept:

- publicly reviewable;
- discoverable;
- reusable;
- independently implementable; and
- attributable to a clear publication date.

The goal is not exclusivity.

If you know of earlier work that overlaps with this proposal, please point to it.

---

## Current implementation status

**Design exploration only.**

No reference implementation is currently scheduled.

A minimal proof of concept would ideally demonstrate:

```text
No generic USB redirection
No BLE peripheral forwarding
No NFC forwarding
No IP connectivity to the authenticator
No FIDO-specific VDI virtual channel

                    but

Successful CTAP2 authentication
through ordinary bidirectional audio.
```

A stronger demonstration would repeat the same CTAP flow over multiple local audio underlays such as TRRS and USB Audio.

---

## License

The current documentation is dedicated under **CC0 1.0 Universal**.

Future source code, if added, may use a separate software license stated in the relevant files or directories.
