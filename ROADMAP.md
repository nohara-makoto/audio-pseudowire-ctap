# Roadmap

**Status:** Suggested experimental sequence  
**Important:** No implementation commitment is implied.

APW-CTAP is currently a design proposal and defensive publication.

The roadmap below exists so that anyone interested in experimenting can see the shortest path from idea to evidence.

---

# 1. Phase 0 — Public review

Goals:

- publish the architecture;
- collect prior art;
- collect security objections;
- identify interested implementers;
- identify obviously bad assumptions before writing code.

Deliverables:

```text
README.md
SPEC.md
SPEC-ja.md
PRIOR-ART.md
SECURITY.md
ROADMAP.md
```

Success condition:

> At least one independent reviewer can understand the proposal well enough to criticize the transport model.

---

# 2. Phase 1 — Raw audio link

Do **not** begin with WebAuthn.

First prove that the intended real audio path can carry bits reliably.

Recommended initial setup:

```text
Linux host
   |
Audio output / input
   |
TRRS or controlled audio loop
   |
Embedded Linux / Raspberry Pi
```

Suggested first modem profile:

- voice band;
- approximately 1–3 kHz;
- simple BFSK / AFSK-like signaling;
- approximately 1200 symbols/s;
- half duplex;
- fixed known audio levels.

Measure:

- acquisition time;
- raw BER;
- frame error rate;
- clock mismatch tolerance;
- resampling tolerance;
- behavior with AGC;
- behavior with AEC;
- behavior with noise suppression.

Success condition:

> Repeatable bidirectional binary transfer with error detection.

---

# 3. Phase 2 — APW framing

Implement only:

```text
HELLO
HELLO_ACK
PING
DATA
ACK
NACK
RESET
```

Add:

- Session ID;
- Sequence Number;
- fragmentation;
- CRC-32C;
- retransmission;
- bounded timeouts;
- bounded buffers.

Do not add FEC until measurements show it is needed.

Success condition:

> Reliable arbitrary binary payload transfer over the real audio path.

---

# 4. Phase 3 — Channel characterization

Test realistic underlay distortion.

Suggested matrix:

```text
Local lossless PCM
Direct analog TRRS
48 kHz -> 16 kHz -> 48 kHz
Lossy voice codec
AGC ON / OFF
AEC ON / OFF
Noise suppression ON / OFF
Artificial jitter
Artificial packet loss
```

Optional work:

- Goertzel-based detection;
- adaptive thresholds;
- channel sounding;
- fallback between modulation profiles.

Success condition:

> Baseline APW-VB1 remains usable across at least one non-transparent voice-style audio path.

---

# 5. Phase 4 — CTAP transport adapter

Only after the link works should CTAP be introduced.

A practical first integration target is a CTAP client library with custom transport hooks.

Candidate:

- libfido2  
  https://github.com/Yubico/libfido2

Initial CTAP sequence:

```text
authenticatorGetInfo
```

Success condition:

> A complete valid CTAP response is transported over APW without changing CTAP semantics.

---

# 6. Phase 5 — MakeCredential / GetAssertion

Extend the prototype to:

```text
authenticatorMakeCredential
authenticatorGetAssertion
```

Add:

- keepalive handling;
- cancellation;
- physical User Presence;
- optional PIN/UV path as supported by the authenticator.

Success condition:

> End-to-end credential creation and assertion through APW.

---

# 7. Phase 6 — VDI path

Now insert a real remote audio path.

Target architecture:

```text
Remote VM
   |
Virtual speaker / microphone
   |
Existing VDI audio redirection
   |
Thin / Zero Client
   |
Local audio endpoint
   |
APW Authenticator
```

Critical demonstration:

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

Success condition:

> The same CTAP transaction succeeds through real VDI audio redirection.

---

# 8. Phase 7 — USB Audio underlay

Demonstrate that APW is not a TRRS-specific trick.

Implement an authenticator that appears to the edge device as a standard USB Audio endpoint.

Desired property:

> No APW-specific device driver is needed where the platform already exposes standard USB Audio.

Architecture:

```text
APW Authenticator
      |
   USB Audio
      |
Existing OS Audio Stack
      |
    APW Client
```

Success condition:

> Same APW and CTAP semantics as the analog profile, with only the underlay changed.

---

# 9. Phase 8 — Bluetooth Audio underlay

Where the platform exposes usable bidirectional Bluetooth audio:

```text
APW Authenticator
      |
Bluetooth Audio
      |
Existing Audio Stack
      |
    APW Client
```

Important:

- playback-only Bluetooth is insufficient;
- actual microphone/capture exposure must be verified;
- codec behavior may be much more destructive than local USB Audio.

Success condition:

> CTAP transaction completes over a standard bidirectional Bluetooth audio path without defining a new Bluetooth FIDO profile.

---

# 10. Phase 9 — Cross-platform / closed-device experiments

Candidate environments:

- thin clients;
- zero clients;
- kiosks;
- embedded Linux appliances;
- smart displays;
- industrial consoles;
- game consoles where application/platform audio access is available.

The objective is not to claim universal compatibility.

The objective is to test the hypothesis:

> **A platform that already exposes controllable bidirectional audio may be closer to external FIDO2 support than its native peripheral API suggests.**

---

# 11. Phase 10 — Security hardening

Before any production claim:

- fuzz APW framing;
- fuzz fragmentation;
- validate all length fields;
- bound retries and buffers;
- analyze downgrade;
- analyze discovery abuse;
- analyze endpoint hijacking;
- analyze VDI recording/injection;
- define authenticated secure-link requirements if needed;
- test fail-closed behavior.

Success condition:

> Security properties and non-goals are explicit and independently reviewable.

---

# 12. Phase 11 — Interoperability

Once two independent implementations exist:

- freeze a baseline frame encoding;
- freeze APW-VB1 minimum parameters;
- define conformance vectors;
- define timing bounds;
- define capability-negotiation behavior;
- define negative tests;
- define underlay qualification tests.

Success condition:

> Two independent implementations can interoperate without private coordination.

---

# 13. Phase 12 — Standards discussion

Only after implementation evidence exists should formal standardization be considered.

Possible discussion points:

- CTAP transport binding;
- WebAuthn transport hint, if actually useful;
- FIDO Alliance working-group relevance;
- remote/VDI interoperability;
- registry allocation;
- security profile;
- certification requirements.

The project should **not** begin by asking standards bodies to bless an untested modem.

---

# 14. Suggested repository milestones

```text
v0.3   Initial public design / defensive publication
v0.4   Prior-art and security-review corrections
v0.5   Frame format frozen for PoC
v0.6   Raw APW reference modem
v0.7   libfido2 / CTAP proof of concept
v0.8   VDI proof of concept
v0.9   Second audio underlay
v1.0-exp
       Experimental interoperable profile
```

These are suggested labels only.

No release schedule is implied.

---

# 15. Help wanted

Useful contributions include:

- prior-art research;
- threat-model review;
- modem/DSP experiments;
- libfido2 integration;
- Raspberry Pi / embedded authenticator work;
- USB Audio gadget implementation;
- VDI testing;
- Bluetooth audio testing;
- console/kiosk/platform API research;
- interoperability test tooling;
- documentation.

Independent implementations are particularly valuable.

---

# 16. Non-goal: project ownership

This repository does not require the original author to implement every stage.

The design is intentionally public so that interested people can:

- fork it;
- challenge it;
- build a PoC;
- replace bad assumptions;
- or take the useful parts in a different direction.

That outcome is considered success.
