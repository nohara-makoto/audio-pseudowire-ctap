# Security Considerations and Threat Model

**Status:** Non-normative security analysis for APW-CTAP  
**Version:** aligned with specification v0.3

APW is a transport experiment.

It does **not** make audio a trusted medium.

Its security value, if any, comes from narrowing the exposed transport surface while preserving the security properties of FIDO/CTAP above it.

---

# 1. Security objective

APW should expose only the minimum functionality required to transport CTAP messages.

It should **not** implicitly create:

- an IP interface;
- a generic serial tunnel;
- arbitrary file transfer;
- a shell;
- generic USB forwarding;
- arbitrary peripheral control; or
- an unrestricted data tunnel.

The intended security proposition is:

> **Scope reduction, not inherent trust.**

---

# 2. Trust boundaries

A typical remote deployment contains several distinct trust boundaries:

```text
Relying Party / Service
        |
WebAuthn Client
        |
CTAP Client
        |
APW Transport Adapter
        |
Virtual / Physical Audio Stack
        |
VDI / Remote Audio Path
        |
Edge Audio Stack
        |
Physical / Wireless Audio Underlay
        |
APW Authenticator
        |
Authenticator Secure Boundary
```

APW must assume that one or more intermediate layers may be observable or controllable by an attacker.

---

# 3. Assets to protect

Primary assets include:

- authenticator private keys;
- credential integrity;
- User Presence state;
- User Verification state;
- CTAP request/response integrity;
- session state;
- user privacy;
- RP and credential metadata;
- authenticator availability.

APW must not weaken the authenticator's existing key-protection boundary.

---

# 4. Attacker capabilities

A realistic threat model should include attackers that can:

- record the audio path;
- inject arbitrary audio;
- replay prior recordings;
- alter audio gain;
- enable or change AGC/AEC/noise suppression;
- switch audio routes;
- occupy capture or playback devices;
- modify virtual audio drivers;
- compromise the VDI guest;
- compromise the thin/zero client;
- compromise the local OS;
- interfere with Bluetooth or analog connections;
- delay, reorder, duplicate, or drop frames;
- force repeated fallback behavior;
- cause denial of service.

APW must not assume that a private-looking audio path is confidential or authentic.

---

# 5. Authenticator security boundary

Credential private keys MUST remain inside the authenticator security boundary.

APW transport software MUST NOT be able to:

- export credential private keys;
- synthesize successful User Presence;
- synthesize successful User Verification;
- bypass authenticator policy;
- alter authenticator-generated signatures.

A physical button, biometric sensor, PIN mechanism, or equivalent authenticator-controlled UV/UP mechanism remains independent of APW.

---

# 6. Eavesdropping

Audio should be treated as observable unless the deployment proves otherwise.

Potential observers include:

- the endpoint OS;
- the remote VM;
- the VDI host;
- remote-desktop infrastructure;
- virtual audio drivers;
- recording software;
- conferencing middleware;
- compromised local applications.

Baseline APW therefore provides **no transport confidentiality guarantee** unless an authenticated secure-link profile is added.

Higher-layer FIDO protections remain important, but they should not be confused with link confidentiality.

---

# 7. Injection and corruption

An attacker who can inject audio may:

- forge APW preambles;
- inject bogus frames;
- corrupt valid frames;
- trigger retransmissions;
- cause spurious resets;
- attempt downgrade;
- keep the link permanently busy.

Mitigations should include:

- frame CRC;
- strict length validation;
- sequence numbers;
- session identifiers;
- bounded retransmission;
- bounded reassembly buffers;
- state-machine validation;
- fail-closed behavior.

CRC is for accidental corruption detection, **not** cryptographic authentication.

---

# 8. Replay

Previously recorded audio may be replayed.

Mitigations should include APW session identifiers and sequence numbers for intra-session replay suppression.

However, APW replay protection must not be confused with WebAuthn freshness.

WebAuthn challenge freshness helps prevent simple reuse of old assertions, but replayed transport messages may still cause:

- denial of service;
- confusing state transitions;
- repeated user prompts;
- downgrade attempts.

---

# 9. Active man-in-the-middle

Unauthenticated Diffie-Hellman is not sufficient.

A future secure-link profile must not claim active MITM protection unless key establishment is authenticated.

Potential approaches may include an established authenticated key-exchange construction followed by AEAD.

The baseline specification intentionally avoids inventing a new cryptographic handshake.

---

# 10. Relay attacks and proximity

APW is **not** proof of physical proximity.

In VDI use, remote communication is intentional.

A valid APW session may traverse:

- a physical cable;
- USB Audio;
- Bluetooth Audio;
- a local OS mixer;
- a VDI audio channel;
- a remote host.

Therefore:

> **"APW link established" MUST NOT be interpreted as "authenticator is physically near the relying party."**

If a deployment requires proximity assurance, it must use an independent mechanism.

---

# 11. Audio endpoint hijacking

The audio subsystem itself may be shared.

Threats include:

- another process opening the capture device;
- another process opening the render device;
- device routing changes;
- loopback capture;
- virtual cable insertion;
- injected conferencing DSP;
- endpoint substitution.

Implementations should prefer explicit endpoint selection and, where available, exclusive or policy-controlled access.

---

# 12. Discovery abuse

Continuous modem probing could create:

- audible nuisance;
- privacy leakage;
- device fingerprinting;
- unexpected speaker output;
- interference with legitimate audio.

Therefore APW discovery should be bounded and policy-driven.

Background probing SHOULD NOT cause arbitrary user speakers or headphones to emit repeated signaling.

Safer approaches include:

- administrator configuration;
- explicit user selection;
- known-device policy;
- connection detection;
- probing only during an authentication attempt.

---

# 13. Downgrade attacks

If APW supports multiple modem modes or secure-link profiles, an attacker may try to force the weakest one.

Capability negotiation should therefore be designed so that future authenticated profiles can bind negotiated parameters cryptographically.

Until such a profile exists, deployments should treat fallback as a potential attack surface.

---

# 14. Fragmentation and resource exhaustion

CTAP messages may require fragmentation.

Attackers may attempt:

- huge declared lengths;
- sparse fragments;
- duplicate fragments;
- fragment storms;
- many simultaneous Message IDs.

Receivers MUST impose strict limits on:

- total message size;
- number of fragments;
- number of concurrent messages;
- reassembly lifetime;
- retry count;
- memory allocation.

Incomplete or inconsistent messages must fail closed.

---

# 15. Denial of service

Availability attacks are unavoidable on a shared audio path.

Examples include:

- continuous tones;
- high-volume noise;
- audio endpoint capture;
- microphone muting;
- codec mode changes;
- deliberate packet loss;
- repeated malformed frames.

APW should detect link failure quickly and return a transport error rather than expose partially validated CTAP data.

---

# 16. Privacy and fingerprinting

APW should minimize stable identifiers.

Discovery or HELLO messages should avoid globally unique identifiers unless operationally necessary.

Unsolicited periodic signaling may reveal:

- authenticator presence;
- device type;
- implementation version;
- user behavior.

A privacy-preserving discovery model should therefore be preferred.

---

# 17. VDI-specific threat model

In VDI, the remote audio path may cross infrastructure controlled by:

- the endpoint;
- the VDI client;
- a connection broker;
- a gateway;
- the remote host;
- monitoring software.

APW should assume that audio may be:

- recorded;
- transcoded;
- resampled;
- inspected;
- delayed;
- injected.

APW does not turn an untrusted VDI path into a trusted one.

The intended advantage is that the remote environment receives a narrowly scoped audio-mediated CTAP transport rather than a generic forwarded peripheral.

---

# 18. Local USB/Bluetooth does not imply remote peripheral exposure

The following are different security models:

```text
Authenticator --USB--> Remote VM
```

and:

```text
Authenticator --USB Audio--> Edge Audio Stack
                           |
                           v
                    Audio Redirection
                           |
                           v
                       Remote VM
```

In the second model, the remote VM does not receive the USB transaction protocol.

The same distinction applies to local Bluetooth audio.

This separation is one of APW's intended security and deployment properties.

---

# 19. Administrative authorization

APW should not be deployed to defeat an explicit policy that forbids:

- external authenticators;
- non-voice signaling over audio;
- external security peripherals;
- unapproved authentication methods.

Enterprise deployment should be explicitly authorized.

The correct framing is:

> **purpose-limited CTAP transport over an existing authorized media facility**

not:

> **policy bypass using audio.**

---

# 20. Security review priorities

Before production use, the following areas require dedicated review:

1. exact frame state machine;
2. discovery behavior;
3. session establishment;
4. downgrade resistance;
5. authenticated secure-link profile;
6. buffer and fragmentation limits;
7. endpoint arbitration;
8. VDI threat model;
9. privacy leakage;
10. fuzzing and malformed-frame handling;
11. user-presence/user-verification binding;
12. relay expectations;
13. error and timeout behavior.

---

# 21. Security non-goals

APW does not attempt to:

- make compromised clients trustworthy;
- prevent all denial of service;
- provide physical proximity proof;
- protect a user from a malicious relying party;
- replace authenticator key isolation;
- replace WebAuthn origin security;
- make audio confidential by default.

These properties belong to other layers or require additional mechanisms.

---

# 22. Reporting security concerns

During the design-only phase, security issues should be reported publicly unless doing so would create immediate risk to an existing implementation.

Once reference code exists, this document should be updated with a coordinated vulnerability-reporting process.
