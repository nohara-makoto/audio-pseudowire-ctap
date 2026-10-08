# Audio Pseudowire Transport Binding for FIDO CTAP2

## APW-CTAP: An Audio Endpoint Transport for Remote, Virtualized, and Constrained Client Environments

**Document Status:** Experimental Technical Specification  
**Version:** 0.3  
**Intended Status:** Experimental  
**Author:** makoto nohara  
**Date:** October 2026

---

# Abstract

This document specifies an experimental **Audio Pseudowire (APW) transport binding for the FIDO Client to Authenticator Protocol (CTAP2)**.

APW enables a FIDO roaming authenticator to communicate with a CTAP client through an existing bidirectional audio path.

The audio path may be provided by:

- an analog headset interface;
- a USB Audio Class device;
- a bidirectional Bluetooth audio profile;
- a virtual audio endpoint;
- redirected audio in Virtual Desktop Infrastructure (VDI);
- a thin or zero client;
- a game console or appliance exposing suitable audio input/output facilities; or
- another platform capable of presenting bidirectional audio render and capture streams.

APW deliberately abstracts the physical attachment mechanism.

The essential requirement is not that the authenticator be connected by a particular connector, bus, or radio technology.

The essential requirement is that the platform can expose the connection as a usable **audio endpoint**.

Where the platform already supports the selected standard audio class or profile, an APW authenticator may therefore operate without introducing a new APW-specific peripheral device class or edge-side device driver.

The primary deployment scenario addressed by this specification is an environment in which arbitrary peripheral forwarding—particularly generic USB device redirection—is prohibited, unavailable, undesirable, or platform-specific, while bidirectional audio input and output are already supported.

APW converts CTAP transport messages into a low-rate digital waveform suitable for transmission through such audio paths.

APW does not define:

- a new WebAuthn ceremony;
- a new credential type;
- a new authenticator cryptographic model;
- a generic USB tunnel;
- a Bluetooth forwarding protocol;
- an IP tunnel; or
- a general-purpose covert communications channel.

The initial APW profile intentionally favors robustness, portability, implementation simplicity, and compatibility with heterogeneous audio infrastructure over bandwidth efficiency.

---

# 1. Status of This Document

This document is an independent experimental proposal.

It is not currently a specification of the FIDO Alliance, W3C, IETF, USB-IF, Bluetooth SIG, or any other standards organization.

Its purpose is to define a protocol sufficiently precisely to support:

- prototype implementation;
- interoperability testing;
- security analysis;
- VDI evaluation;
- cross-platform evaluation; and
- discussion of a possible future CTAP transport binding.

The term **Audio Pseudowire** describes the abstraction of a point-to-point logical communications path constructed over audio input and output facilities.

The use of the term “pseudowire” is descriptive.

It does not claim conformance with the IETF PWE3 architecture.

---

# 2. Requirements Language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals.

---

# 3. Design Premise

The central APW observation is that many computing platforms expose an audio abstraction more broadly than they expose arbitrary peripheral transports.

A platform may reject, restrict, or lack:

```text
Generic USB forwarding
FIDO-specific USB forwarding
BLE peripheral forwarding
NFC forwarding
Vendor-specific device drivers
Arbitrary local peripheral access
```

while still providing:

```text
Audio Render
Audio Capture
```

for ordinary purposes such as:

- conferencing;
- voice communication;
- accessibility;
- gaming;
- media;
- remote desktop operation; and
- headset support.

APW treats this audio abstraction as a narrow communications substrate for CTAP2.

The design objective is not:

> Tunnel USB through audio.

Nor is the objective:

> Make arbitrary data look like sound.

The objective is:

> Emulate only the reliable point-to-point link semantics required by CTAP2 over an already supported bidirectional audio endpoint.

---

# 4. Scope

APW defines:

1. an Audio Endpoint Abstraction;
2. establishment of a logical point-to-point APW link;
3. baseline voice-band signaling;
4. link training;
5. capability negotiation;
6. packet framing;
7. fragmentation and reassembly;
8. error detection;
9. retransmission;
10. transport control operations;
11. encapsulation of CTAP request and response messages;
12. optional enhanced media profiles; and
13. minimum transport-related security requirements.

APW does not define:

- WebAuthn API behavior;
- WebAuthn origin processing;
- credential formats;
- attestation formats;
- authenticator private-key storage;
- User Presence mechanisms;
- User Verification mechanisms;
- a general-purpose network interface;
- arbitrary serial access;
- USB forwarding;
- Bluetooth peripheral forwarding;
- remote desktop session security; or
- proof of physical proximity.

---

# 5. Relationship to WebAuthn and CTAP2

APW operates below the WebAuthn client layer and acts as a transport binding for CTAP2.

Conceptually:

```text
+-------------------------------------------------------+
| Web Application / Relying Party                       |
+-------------------------------------------------------+
| WebAuthn Client / User Agent                          |
+-------------------------------------------------------+
| CTAP2 Client                                          |
+-------------------------------------------------------+
| APW Transport Adapter                                 |
+-------------------------------------------------------+
| Audio Endpoint Abstraction                            |
+-------------------------------------------------------+
| Audio Underlay                                        |
+-------------------------------------------------------+
| APW Authenticator Modem                               |
+-------------------------------------------------------+
| CTAP2 Authenticator                                   |
+-------------------------------------------------------+
```

APW carries CTAP messages as opaque payloads.

The APW transport layer MUST NOT reinterpret application semantics such as:

- credential creation;
- assertion generation;
- RP identifiers;
- challenges;
- credential identifiers; or
- attestation objects.

The layering is:

```text
WebAuthn
   |
   v
CTAP2
   |
   v
APW Transport
   |
   v
Audio Endpoint Abstraction
   |
   v
Physical / Virtual Audio Underlay
```

A browser-side audio implementation MAY be useful for experimental testing.

However, generic browser audio APIs alone do not automatically integrate an APW device into a platform WebAuthn authenticator subsystem.

A complete platform implementation still requires a CTAP/APW transport adapter somewhere below or alongside the WebAuthn client.

---

# 6. Audio Endpoint Abstraction

## 6.1 Fundamental Requirement

APW SHALL NOT depend on the physical mechanism by which an endpoint obtains its audio render and capture streams.

An APW-capable endpoint requires two logical directions:

```text
APW Client  ------audio------>  APW Authenticator
APW Client  <-----audio-------  APW Authenticator
```

These directions MAY be implemented by separate physical or logical facilities.

The underlying mechanism is not visible to CTAP.

## 6.2 Audio Endpoint Criterion

A physical or virtual interface is suitable as an APW underlay when it can provide, directly or indirectly:

1. an audio render path;
2. an audio capture path;
3. sufficient bandwidth for the negotiated APW profile; and
4. sufficient software access for an APW endpoint implementation.

The transport therefore depends on **audio semantics**, not on connector identity.

## 6.3 No APW-Specific Peripheral Profile Requirement

If an operating system, thin client, appliance, console, or other edge platform already supports the selected audio class or profile, APW SHOULD reuse that existing implementation.

For example:

```text
APW Authenticator
       |
       | USB
       v
Standard USB Audio Device
       |
       v
Existing OS Audio Stack
```

is preferred over:

```text
APW Authenticator
       |
       | USB
       v
New Vendor-Specific APW USB Device Class
       |
       v
Custom Kernel Driver
```

when both provide equivalent APW functionality.

No APW-specific device driver is required when the endpoint platform already exposes the selected standard audio device through an interface usable by the APW implementation.

This does not imply that no APW software is required.

The host still requires an APW modem and CTAP transport adapter.

---

# 7. Underlay Profiles

The APW protocol is independent of individual physical attachment technologies.

The following underlay classes are initially contemplated.

| Profile | Example local attachment | Platform-visible abstraction |
|---|---|---|
| APW-A | Analog TRRS / analog audio | Headphone + microphone |
| APW-U | USB | Standard USB audio endpoint |
| APW-BH | Bluetooth Classic | Bidirectional headset / hands-free audio |
| APW-BL | Bluetooth LE Audio | Bidirectional LE audio endpoint |
| APW-V | Virtual audio | Virtual render/capture device |
| APW-R | Redirected audio | Remote/VDI render and capture streams |

Support for a particular physical profile is OPTIONAL unless separately stated by an implementation profile.

APW-VB1 defines modem interoperability above these underlays.

---

# 8. Local Transport Versus Remote Transport

APW distinguishes between:

1. **local attachment**, and
2. **remote transport exposure**.

For example:

```text
Authenticator
     |
     | USB Audio
     v
Thin Client
     |
     | PCM / Audio Redirection
     v
Remote VM
```

uses USB locally but does **not** redirect the USB device into the VM.

The remote execution environment receives audio, not USB transactions.

Likewise:

```text
Authenticator
     |
     | Bluetooth Audio
     v
Edge Device
     |
     | Audio Stream
     v
Remote Environment
```

does not require Bluetooth peripheral forwarding.

This distinction is fundamental to APW.

APW does not attempt to eliminate USB or Bluetooth as local attachment technologies.

It attempts to eliminate the requirement that the remote application understand or forward those peripheral transports.

---

# 9. Representative VDI Deployment

```text
REMOTE EXECUTION ENVIRONMENT
===========================================================

 Browser / Application
         |
      WebAuthn
         |
     CTAP Client
         |
   APW Transport
         |
 Virtual Audio Render / Capture
         |
========= VDI AUDIO REDIRECTION ==========================
         |
      Edge Device
         |
   Audio Endpoint
     /    |     \
    /     |      \
 TRRS    USB     Bluetooth
 Audio   Audio   Audio
    \     |      /
     \    |     /
    APW Authenticator
         |
      CTAP2

===========================================================
LOCAL USER ENVIRONMENT
```

The VDI protocol does not need to understand FIDO, CTAP2, USB HID, BLE FIDO, or the authenticator's local physical attachment.

---

# 10. Non-VDI and Non-PC Deployment

The Audio Endpoint Abstraction is intentionally not limited to conventional desktop operating systems.

Potential APW hosts include:

- thin clients;
- zero clients;
- kiosks;
- embedded terminals;
- industrial consoles;
- smart displays;
- game consoles;
- media appliances;
- mobile devices; and
- other systems exposing usable bidirectional audio facilities.

For example, a platform that already recognizes a standard headset or USB audio endpoint may potentially attach an APW authenticator without requiring a new FIDO-specific hardware device class.

However, recognition as an audio device alone is **not sufficient** to provide WebAuthn functionality.

The platform must additionally provide a software execution path capable of:

1. accessing the audio streams;
2. implementing the APW modem;
3. implementing or invoking CTAP2; and
4. integrating the resulting authenticator with the application or WebAuthn environment.

Therefore APW expands the set of platforms on which a FIDO2 authenticator can potentially be implemented, but does not automatically make every audio-capable device a WebAuthn client.

---

# 11. Bidirectionality Requirement

APW requires two logical communication directions.

An output-only audio technology is insufficient.

For example:

```text
Audio playback only
```

does not satisfy the APW underlay requirement.

A platform MAY combine independent facilities, such as:

```text
Playback through one audio endpoint
Capture through another audio endpoint
```

provided that the resulting logical APW link meets timing and security requirements.

---

# 12. APW Voice-Band Baseline Profile

The mandatory baseline modem profile is:

**APW-VB1**

APW-VB1 prioritizes survivability through speech-oriented audio systems.

The mandatory signaling range SHOULD remain approximately within:

**1 kHz to 3 kHz**

A baseline implementation MUST NOT depend on ultrasonic frequencies.

Additional wideband or ultrasonic profiles MAY be defined experimentally.

They MUST NOT be required for APW-VB1 interoperability.

---

# 13. Baseline Modulation

APW-VB1 SHALL use robust binary frequency-shift signaling suitable for non-transparent audio channels.

An initial implementation MAY use frequencies approximately corresponding to:

```text
Logical 0   ~1200 Hz
Logical 1   ~2200 Hz
```

with a signaling rate near:

```text
1200 symbols/second
```

Exact parameters SHOULD be finalized through interoperability measurements.

The receiver MUST tolerate reasonable:

- amplitude variation;
- AGC behavior;
- codec quantization;
- resampling;
- clock offset;
- phase discontinuity;
- jitter; and
- limited media loss.

Sample-accurate PCM preservation MUST NOT be assumed.

---

# 14. Half-Duplex Baseline

All conforming APW-VB1 implementations MUST support half-duplex operation.

Full duplex MAY be supported.

Half duplex is the RECOMMENDED initial mode because it:

- simplifies modem design;
- reduces echo-cancellation interaction;
- reduces crosstalk sensitivity;
- works well over voice-oriented links; and
- naturally matches the request/response character of CTAP.

---

# 15. Channel Training

An APW endpoint MAY perform Channel Sounding before data transfer.

A recommended Channel Sounding Sequence consists of sequential short probe tones, for example:

```text
1000 Hz
1400 Hz
1800 Hz
2200 Hz
2600 Hz
3000 Hz
```

Sequential probing is preferred to simultaneous multi-tone bursts for the baseline profile.

The receiver MAY use:

- FFT;
- Goertzel filters;
- matched filters; or
- equivalent DSP techniques.

The DSP implementation is not normative.

---

# 16. Capability Negotiation

After baseline synchronization, endpoints SHOULD exchange:

```text
APW Protocol Version
Maximum APW Frame Payload
Maximum CTAP Message Size
Supported Modulation Profiles
Supported FEC Profiles
Supported Duplex Modes
Optional Secure-Link Profiles
```

Both endpoints MUST retain the ability to return to APW-VB1.

Failure of an enhanced mode SHOULD cause fallback to the baseline profile before session termination.

---

# 17. Link-Layer Architecture

```text
CTAP Message
    |
    v
APW Message
    |
fragmentation
    v
APW Frame(s)
    |
optional FEC
    v
Modem Symbols
    |
    v
Audio Stream
    |
    v
Audio Underlay
```

CTAP messages are not required to fit in a single APW frame.

---

# 18. Frame Format

An APW data frame SHALL logically contain:

```text
+----------------------+-------------------------------+
| Physical Preamble    | Modem synchronization         |
+----------------------+-------------------------------+
| Start Delimiter      | Frame boundary                |
+----------------------+-------------------------------+
| Version              | APW protocol version          |
+----------------------+-------------------------------+
| Type                 | DATA / ACK / CONTROL / ERROR  |
+----------------------+-------------------------------+
| Flags                | Fragment/control flags        |
+----------------------+-------------------------------+
| Session ID           | Logical APW session           |
+----------------------+-------------------------------+
| Sequence Number      | ARQ / duplicate suppression   |
+----------------------+-------------------------------+
| Message ID           | CTAP/APW message identifier   |
+----------------------+-------------------------------+
| Fragment Offset      | Position within message       |
+----------------------+-------------------------------+
| Total Length         | Complete message length       |
+----------------------+-------------------------------+
| Payload Length       | Current fragment length       |
+----------------------+-------------------------------+
| Payload              | Opaque CTAP/control data      |
+----------------------+-------------------------------+
| CRC-32C              | Error detection               |
+----------------------+-------------------------------+
```

Multi-byte integers SHOULD use network byte order.

---

# 19. Transport Message Types

Initial APW message classes SHOULD include:

| Type | Purpose |
|---|---|
| `HELLO` | Link initialization |
| `HELLO_ACK` | Capability response |
| `DATA` | CTAP message fragment |
| `ACK` | Positive acknowledgement |
| `NACK` | Retransmission request |
| `KEEPALIVE` | Long-running operation |
| `CANCEL` | Cancel outstanding operation |
| `PING` | Link validation |
| `ERROR` | Transport error |
| `RESET` | Reset link state |

APW MUST NOT define transport types corresponding to WebAuthn application semantics such as:

- Challenge;
- Credential; or
- Assertion.

Those remain opaque higher-layer data.

---

# 20. Fragmentation and Reassembly

APW MUST support CTAP message fragmentation.

A baseline implementation SHOULD initially use payload fragments in the approximate range of:

**64 to 256 bytes**

The receiver MUST:

1. verify frame integrity;
2. reject invalid offsets;
3. reject inconsistent message lengths;
4. suppress duplicate frames;
5. reassemble fragments correctly; and
6. deliver only complete messages to CTAP.

Incomplete data MUST NOT be delivered as a CTAP message.

---

# 21. Error Detection and Recovery

Every APW frame MUST contain an error-detection mechanism.

APW-VB1 SHOULD use CRC-32C.

The CRC is not cryptographic authentication.

Baseline implementations MUST support retransmission.

A simple:

- Stop-and-Wait ARQ; or
- small-window ARQ

is RECOMMENDED.

---

# 22. Forward Error Correction

FEC is OPTIONAL.

An implementation MAY negotiate a FEC profile when the underlay produces burst corruption.

Possible profiles include shortened Reed-Solomon codes with interleaving.

An endpoint without FEC support MUST remain interoperable through CRC plus retransmission.

FEC SHALL be negotiated as a coding profile rather than represented by a fixed arbitrary-size “ECC field”.

---

# 23. CTAP Mapping

One complete APW message SHALL carry one complete:

- CTAP request; or
- CTAP response.

APW MUST preserve CTAP byte semantics.

APW MUST NOT modify CTAP CBOR merely to accommodate the transport.

The conceptual flow is:

```text
CTAP Client
    |
    | Request
    v
APW Message
    |
    | one or more audio frames
    v
Authenticator
    |
    | Response
    v
APW Message
    |
    v
CTAP Client
```

---

# 24. Keepalive and User Interaction

APW MUST distinguish a broken link from an authenticator waiting for:

- User Presence;
- User Verification;
- biometric interaction;
- PIN input; or
- another CTAP-defined user action.

The transport MAY provide `KEEPALIVE` indications.

---

# 25. Discovery

Audio endpoints do not inherently identify themselves as FIDO authenticators.

APW therefore requires an explicit discovery mechanism or policy.

Possible strategies include:

1. administrative configuration;
2. explicit user selection;
3. jack or connection detection;
4. local policy;
5. known audio endpoint identity;
6. explicit APW mode selection; or
7. bounded active probing during an authentication attempt.

Background probing MUST NOT cause arbitrary loudspeakers or headphones to emit unexpected modem signaling.

---

# 26. Analog Wired Profile — APW-A

APW-A defines a simple wired reference profile.

A typical logical connection is:

```text
Host / Edge Audio Output
          |
          v
Authenticator Audio Input


Authenticator Audio Output
          |
          v
Host / Edge Audio Input


Ground ---------------- Ground
```

A headset-style 4-pole connector is a practical initial implementation.

The APW protocol itself does not require a particular analog connector.

Analog circuitry SHOULD account for:

- AC coupling;
- microphone bias;
- attenuation;
- impedance;
- over-voltage protection; and
- headset detection behavior.

APW-A is a useful prototype and fallback profile.

It is **not** the architectural definition of APW.

---

# 27. USB Audio Profile — APW-U

APW-U uses a locally attached USB audio device as the Audio Endpoint.

The authenticator MAY enumerate to the edge platform as a standard audio device rather than as a FIDO-specific USB HID device.

Conceptually:

```text
APW Authenticator
       |
       | Local USB
       v
Standard Audio Device
       |
       v
Edge Audio Stack
       |
       | Audio / PCM
       v
APW Client or VDI Redirection
```

The USB device itself need not be redirected to a remote VM.

APW-U therefore remains compatible with environments in which:

- local USB audio is permitted; but
- generic USB device forwarding is prohibited.

Where the platform provides a suitable standard USB audio driver, APW-U SHOULD avoid requiring an APW-specific kernel driver.

---

# 28. Bluetooth Audio Profiles — APW-BH and APW-BL

APW MAY operate over a Bluetooth audio technology when the platform exposes a suitable bidirectional audio path.

Possible profiles include:

- bidirectional Classic Bluetooth headset / hands-free audio; and
- bidirectional LE Audio configurations.

Output-only Bluetooth audio does not satisfy the APW requirement.

APW does not define a new Bluetooth FIDO profile.

The intent is to reuse an audio service that the platform already understands.

Conceptually:

```text
APW Authenticator
       |
   Bluetooth Audio
       |
       v
Existing Audio Stack
       |
       v
APW Modem
```

Bluetooth is therefore treated as a local audio attachment technology rather than as a CTAP peripheral forwarding mechanism.

---

# 29. Virtual and Redirected Audio Profiles

APW-V and APW-R cover environments in which one or both endpoints are virtual.

Examples include:

- virtual audio cables;
- containerized audio endpoints;
- remote desktop virtual microphones;
- remote desktop virtual speakers;
- browser-accessible audio endpoints;
- conferencing redirection paths; and
- zero-client audio channels.

APW MUST NOT assume that such paths preserve PCM samples exactly.

---

# 30. Voice Processing Interaction

Audio infrastructure may introduce:

- AGC;
- AEC;
- noise suppression;
- voice activity detection;
- lossy codecs;
- resampling;
- dynamic bitrate changes; and
- routing changes.

Implementations SHOULD disable destructive processing where possible.

APW-VB1 MUST nevertheless be designed for non-transparent audio transport.

---

# 31. Security Considerations

## 31.1 Narrow Protocol Surface

APW MUST expose only operations needed for the CTAP transport.

It MUST NOT implicitly expose:

- IP networking;
- arbitrary serial communication;
- file transfer;
- command shells;
- generic USB forwarding; or
- unrestricted peripheral control.

The security objective is **scope reduction**, not a claim that audio is inherently secure.

---

# 32. Local Audio Device Versus Generic Peripheral Forwarding

A local USB or Bluetooth attachment MUST NOT be confused with generic USB or Bluetooth forwarding.

For example:

```text
Authenticator --USB Audio--> Edge Device
```

followed by:

```text
Edge Device --Audio--> Remote VM
```

does not expose the USB device protocol to the VM.

The remote side receives the audio abstraction only.

This separation is one of the principal architectural properties of APW.

---

# 33. Authenticator Security Boundary

Credential private keys MUST remain under authenticator control.

APW MUST NOT weaken:

- User Presence;
- User Verification;
- credential protection; or
- signature generation.

The modem layer is transport infrastructure, not an authentication authority.

---

# 34. Eavesdropping

An audio path MUST be considered observable unless deployment-specific controls establish otherwise.

Potential observers include:

- endpoint operating systems;
- VDI hosts;
- virtual audio drivers;
- audio recording software;
- remote desktop infrastructure; and
- compromised endpoint software.

Baseline cleartext APW provides no additional confidentiality beyond higher-layer FIDO protections.

---

# 35. Link Encryption

APW version 0.2 does not define a new cryptographic handshake.

Implementations MUST NOT claim active MITM protection merely because they use unauthenticated ephemeral Diffie-Hellman.

A future secure-link profile MAY define an authenticated key-establishment mechanism and AEAD protection.

---

# 36. Replay, Injection, and Denial of Service

Attackers able to manipulate an audio endpoint may:

- inject frames;
- corrupt frames;
- replay frames;
- trigger retransmissions;
- reset the link; or
- deny service.

Session identifiers and sequence numbers SHOULD reject stale intra-session traffic.

Incomplete or invalid messages MUST fail closed.

---

# 37. Relay and Proximity

APW MUST NOT be interpreted as proof of physical proximity.

This is particularly important for remote desktop use, where remote operation is intentional.

If cryptographic proximity assurance is required, another mechanism must provide it.

---

# 38. Administrative Policy

APW is not intended to bypass a policy that explicitly forbids external authenticators or non-voice data over audio interfaces.

Enterprise use SHOULD be explicitly authorized.

APW should be evaluated as:

> a constrained application-specific transport over an existing audio facility

rather than:

> a method for covertly bypassing device-redirection controls.

---

# 39. Cross-Platform Design Objective

A significant APW design goal is to avoid unnecessary dependence on host-specific peripheral interfaces.

A platform that can already present an audio input/output endpoint to software may potentially participate in APW even when it does not expose a native FIDO roaming-authenticator interface.

This can lower the hardware integration barrier for platforms including:

```text
Thin Clients
Zero Clients
Kiosks
Game Consoles
Embedded Appliances
Smart Displays
Special-purpose Terminals
Mobile Devices
```

The remaining software requirement is an APW-capable execution path.

Thus:

```text
Standard Audio Support
        +
APW Software
        +
CTAP Integration
        =
Potential FIDO2 Authenticator Transport
```

rather than:

```text
Standard Audio Support
        =
Automatically FIDO2-capable
```

---

# 40. Prototype Conformance Stages

## Stage 1 — Audio Link

Verify bidirectional random binary transport over APW-A.

## Stage 2 — Framing

Demonstrate:

```text
HELLO
PING
DATA
ACK
```

with CRC and retransmission.

## Stage 3 — CTAP Basic Operation

Successfully transport:

```text
authenticatorGetInfo
```

## Stage 4 — Credential Creation

Successfully transport:

```text
authenticatorMakeCredential
```

## Stage 5 — Authentication

Successfully transport:

```text
authenticatorGetAssertion
```

## Stage 6 — VDI

Repeat Stages 3–5 using only ordinary bidirectional VDI audio redirection across the remote boundary.

## Stage 7 — Alternative Audio Underlay

Repeat CTAP operation over at least one additional local underlay, such as:

- standard USB audio; or
- a supported bidirectional Bluetooth audio profile.

This demonstrates that APW is an audio abstraction rather than an analog-cable-specific protocol.

---

# 41. Recommended Reference Prototype

A first reference authenticator may use:

```text
Raspberry Pi / Embedded Linux
        |
CTAP2 Authenticator
        |
APW Daemon
        |
Audio Codec
```

with one or more of:

```text
TRRS
USB Audio
Bluetooth Audio
```

The initial PoC SHOULD start with APW-A because its behavior is easy to observe and control.

A second-stage prototype SHOULD demonstrate APW-U or another standard audio-device profile.

---

# 42. Experimental Browser Implementation

Web Audio APIs MAY be useful for:

- modem experiments;
- BER testing;
- visual debugging;
- channel sounding;
- demonstrations; and
- proof-of-concept applications.

A browser audio implementation MUST NOT be described as a complete native WebAuthn authenticator transport unless the necessary WebAuthn/CTAP integration exists.

---

# 43. Performance Objectives

Initial target:

```text
Baseline gross rate:            approximately 1200 bit/s
Enhanced rate:                  2400 bit/s or greater
Typical CTAP transaction:       hundreds of bytes
Supported CTAP message size:    at least the connected authenticator requirement
```

Reliability takes precedence over maximum throughput.

---

# 44. Design Principles

## 44.1 Audio Is the Abstraction

APW is bound to audio semantics, not a connector.

## 44.2 Reuse Existing Device Classes

Existing standard audio support SHOULD be reused whenever practical.

## 44.3 Avoid Unnecessary Drivers

A new APW-specific edge-device driver SHOULD NOT be required where the platform already exposes a usable audio interface.

## 44.4 Separate Local Attachment from Remote Redirection

Using USB or Bluetooth locally does not require forwarding those transports remotely.

## 44.5 Preserve CTAP Semantics

APW MUST transport CTAP without redefining FIDO authentication semantics.

## 44.6 Minimize Privilege

Only the functionality necessary for CTAP transport SHOULD be exposed.

## 44.7 Fail Closed

Invalid or incomplete traffic MUST NOT become a successful CTAP operation.

## 44.8 Prefer Robustness to Bandwidth

Authentication traffic is small enough that reliability and simple recovery are preferable to aggressive modem optimization.

---

# 45. Relationship to Other FIDO Transports

APW is proposed as an additional CTAP transport abstraction.

Conceptually:

```text
              CTAP2 Application Protocol
                         |
       +-----------------+-----------------+
       |                 |                 |
    USB HID             NFC               BLE
                                             \
                                              \
                                           Future
                                             |
                                             APW
                                             |
                                  Audio Endpoint Abstraction
                                   /     |      |      \
                                TRRS    USB    BT     Virtual
```

APW does not replace existing native FIDO transports.

Where a native transport is available and acceptable, it may remain preferable.

APW addresses environments in which the audio abstraction has greater deployment reach than the native authenticator transport.

---

# 46. WebAuthn Transport Identifier

Version 0.2 does not request a new WebAuthn `AuthenticatorTransport` identifier.

Prototype discovery SHOULD remain an implementation detail.

If interoperable APW implementations emerge, a future proposal MAY evaluate an identifier such as:

```text
"audio"
```

Such registration is outside the present scope.

---

# 47. Interoperability Test Matrix

Implementations SHOULD test at least:

```text
Local lossless PCM
Analog TRRS
USB Audio
Bidirectional Bluetooth Audio where supported
Virtual audio endpoint
VDI audio redirection
48 kHz -> 16 kHz -> 48 kHz conversion
Lossy voice codec
AGC enabled / disabled
AEC enabled / disabled
Noise suppression enabled / disabled
Artificial packet loss
Artificial jitter
```

Measurements SHOULD include:

- acquisition time;
- BER;
- frame error rate;
- retransmission count;
- CTAP completion time;
- authentication success rate; and
- fallback behavior.

---

# 48. Success Criteria

The primary APW concept is technically validated when a CTAP2 authenticator successfully completes credential creation and authentication through an environment in which the remote side requires only ordinary audio transport.

A representative demonstration is:

```text
No USB Device Redirection
No BLE Peripheral Forwarding
No NFC Forwarding
No IP Connectivity to Authenticator
No FIDO-specific VDI Virtual Channel

                    but

Successful CTAP2 Authentication
through ordinary bidirectional audio.
```

A stronger demonstration additionally shows that the same APW/CTAP implementation operates over multiple local audio attachments:

```text
TRRS
USB Audio
Bluetooth Audio
```

without changing CTAP semantics.

---

# 49. IANA and Registry Considerations

Version 0.2 requests no IANA allocations.

Experimental identifiers SHOULD use private experimental values.

Future standardization MAY require coordination with relevant FIDO, WebAuthn, audio, USB, Bluetooth, or protocol registries.

---

# 50. Open Issues

Open issues include:

1. final APW-VB1 waveform parameters;
2. frame timing;
3. optimum fragment size;
4. codec adaptation;
5. endpoint discovery;
6. audio-device arbitration;
7. Bluetooth profile compatibility;
8. console and appliance API accessibility;
9. browser integration;
10. OS WebAuthn integration;
11. authenticated secure-link profiles;
12. remote-origin threat analysis;
13. user experience;
14. automatic fallback;
15. transport identifier registration; and
16. certification and conformance testing.

---

# 51. References

## Normative

**[RFC2119]**  
S. Bradner, *Key words for use in RFCs to Indicate Requirement Levels.*

**[RFC8174]**  
B. Leiba, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words.*

**[CTAP2]**  
FIDO Alliance, *Client to Authenticator Protocol (CTAP).*

## Informative

**[WebAuthn]**  
W3C, *Web Authentication: An API for Accessing Public Key Credentials.*

**[USBAUDIO]**  
USB-IF, *USB Device Class Definition for Audio Devices.*

**[BT-HFP]**  
Bluetooth SIG, *Hands-Free Profile.*

**[BT-LE-AUDIO]**  
Bluetooth SIG, *LE Audio Specifications.*

---

# Appendix A — Architectural Summary

APW intentionally places a stable abstraction between CTAP and physical connectivity.

```text
                    CTAP2
                      |
                      v
                APW Protocol
                      |
                      v
             Audio Stream Abstraction
                      |
       +--------------+---------------+
       |              |               |
      TRRS         USB Audio      Bluetooth Audio
       |              |               |
       +--------------+---------------+
                      |
                  Edge Device
                      |
               Audio Redirection
                      |
                 Remote Host
```

The important property is not the wire, connector, bus, or radio.

The important property is:

> **Can the platform present a sufficiently controllable bidirectional audio endpoint?**

If yes, it may be possible to construct an APW link.

---

# Appendix B — Core Proposition

```text
Many systems already understand audio devices.

They may understand:

    headset output
    microphone input
    USB audio
    Bluetooth audio
    virtual audio

even when they do not understand:

    CTAP over a new peripheral bus
    vendor-specific authentication hardware
    generic USB forwarding
    generic Bluetooth forwarding


Therefore:

    existing audio-device support can act
    as a deployment substrate for APW.


APW does not require the edge platform
to know that the audio endpoint is a
FIDO authenticator.

It only requires the edge platform
to expose the audio path.


The APW software interprets that path
as a reliable logical wire for CTAP2.


Consequently, a platform that already
supports bidirectional audio may be
closer to supporting an external FIDO2
authenticator than its native peripheral
API would otherwise suggest.
```


---

# Appendix C — Deployment and Commercial Rationale

> **Non-Normative**

This appendix describes the deployment rationale and potential commercial value of APW. It is informative only and does not define protocol conformance requirements.

## C.1 Why Audio Is Strategically Useful

The value of APW is not that audio is a technically superior transport to USB, NFC, BLE, or other native FIDO transports.

Its value is that **audio support is already deployed on a very large class of systems that do not necessarily expose, permit, or forward dedicated authenticator transports**.

Many platforms already support:

```text
Audio Playback
Microphone Capture
USB Audio Class
Headset Interfaces
Bluetooth Audio
Virtual Audio
Remote Audio Redirection
```

while support for the following may be absent, restricted, or product-specific:

```text
External FIDO USB HID
Generic USB Device Redirection
BLE Authenticator Forwarding
NFC Reader Access
Vendor-specific Peripheral Drivers
Remote WebAuthn/FIDO Virtual Channels
```

APW attempts to convert this deployment asymmetry into an interoperability advantage.

The basic proposition is:

```text
Existing Audio Support
        +
APW Software
        +
CTAP Integration
        =
Potential Strong-Authentication Interface
```

The important point is that APW does not require the platform to introduce a new physical peripheral category merely to transport CTAP.

---

## C.2 Driverless Does Not Mean Software-Free

One of the strongest potential deployment properties of APW is that it can reuse an existing audio device class or profile.

For example:

```text
APW Authenticator
        |
        | USB
        v
Standard USB Audio Class Device
        |
        v
Existing Platform Audio Driver
```

or:

```text
APW Authenticator
        |
        | Bluetooth Audio
        v
Existing Headset / Audio Stack
```

Where the platform already provides a compatible standard audio driver, **no APW-specific device driver is required**.

This can avoid the need to introduce:

- a new kernel-mode peripheral driver;
- a new USB device class;
- a new Bluetooth peripheral profile;
- a vendor-specific transport driver;
- a new device-redirection module at the edge; or
- dedicated hardware-enablement logic for each host platform.

This does **not** mean that APW requires no software integration.

The host still needs software capable of:

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

The practical advantage is therefore better described as:

> **APW shifts integration effort from new peripheral enablement toward user-space or application-level protocol integration.**

For managed systems, embedded platforms, appliances, and long-lived devices, that distinction may significantly reduce certification, deployment, maintenance, and operating-system compatibility costs.

---

## C.3 Reuse of Existing Platform Qualification

A new peripheral class may require substantial platform work, including:

- driver development;
- driver signing;
- kernel integration;
- operating-system compatibility testing;
- device certification;
- security review;
- endpoint-control policy changes;
- remote-device forwarding support; and
- long-term maintenance.

By contrast, a platform may already have mature qualification and policy for:

```text
Headsets
Microphones
USB Audio Devices
Bluetooth Audio Devices
Virtual Audio Devices
```

An APW implementation can potentially reuse this existing qualification boundary.

This does not remove the need to assess APW itself.

However, it can reduce the number of new system components introduced solely to attach a FIDO authenticator.

---

## C.4 VDI and Thin-Client Deployment Model

VDI is the most direct initial use case for APW.

A common policy arrangement is:

```text
Keyboard / Mouse       Allowed
Display                Allowed
Audio Playback         Allowed
Microphone Capture     Allowed

Generic USB Redirect   Restricted
External Storage       Restricted
Unknown HID Devices    Restricted
```

In such an environment, APW can potentially provide:

```text
Authenticator
     |
Standard Audio Attachment
     |
Thin / Zero Client
     |
Existing Audio Redirection
     |
Remote Session
     |
APW + CTAP2
```

without requiring the VDI protocol to implement a dedicated FIDO virtual channel.

This creates a potentially useful fallback and portability layer across heterogeneous remote-access technologies.

The commercial value is not that APW is necessarily superior to a native FIDO redirection feature.

Rather, APW may be useful where:

- no native FIDO redirection exists;
- FIDO redirection differs by vendor;
- a zero client exposes audio but not arbitrary USB;
- local device drivers cannot be installed;
- a legacy remote-access product must be supported; or
- one authenticator integration is desired across multiple remote-display protocols.

---

## C.5 Closed and Constrained Platforms

APW may also be relevant to systems that expose standard audio devices but provide limited peripheral extensibility.

Examples include:

```text
Game Consoles
Smart TVs
Kiosks
Point-of-Service Terminals
Industrial HMIs
Embedded Appliances
Shared Terminals
Media Devices
Special-Purpose Consoles
```

Such platforms may already provide a mature audio stack because audio is required for ordinary product functionality.

However, they may not expose:

- a general-purpose FIDO roaming-authenticator interface;
- a third-party USB HID extension mechanism;
- a programmable BLE security-key stack; or
- a vendor-neutral external-authenticator API.

If application or platform software can access the audio streams, APW may provide a path to strong external authentication without first standardizing a new physical-device profile for that platform.

The relevant distinction is:

```text
Platform already understands the physical device
as AUDIO.

APW software gives semantic meaning
to that audio stream as CTAP transport.
```

This can reduce the hardware integration problem to a software integration problem.

---

## C.6 Game-Console and Consumer-Appliance Opportunity

Consumer platforms increasingly carry accounts that may have substantial value.

Examples include:

- stored payment credentials;
- digital purchases;
- cloud saves;
- subscription entitlements;
- family accounts;
- parental-control authority;
- virtual goods;
- tournament identities;
- developer credentials; and
- account-recovery authority.

Accordingly, stronger authentication on such devices can support more than simple login.

Potential APW-enabled use cases include:

```text
Account Sign-In
Purchase Re-Authentication
Parental Approval
Account Recovery
High-Risk Settings Changes
Developer / Administrative Mode
Tournament Identity
Shared-Device Account Selection
```

A possible user interaction could be:

```text
Console requests strong authentication
            |
            v
User connects or activates APW authenticator
            |
            v
Console accesses existing audio endpoint
            |
            v
APW transports CTAP request
            |
            v
Authenticator performs UP / UV
            |
            v
Signed assertion returned
```

The principal integration advantage is that the physical authenticator may appear to the device as something the platform already supports, such as a standard USB audio device or headset-class audio endpoint.

However, APW cannot provide platform login functionality without cooperation from the platform or application software.

For locked consumer consoles, the realistic commercialization path is therefore likely to involve:

- platform-vendor integration;
- OEM integration;
- an approved application SDK;
- middleware licensing; or
- incorporation into a platform security framework.

---

## C.7 Kiosk, Shared-Terminal, and Regulated-Environment Opportunity

APW may be especially attractive in environments where users must authenticate strongly but administrators intentionally minimize local peripheral capability.

Examples include:

- public kiosks;
- factory terminals;
- medical workstations;
- call-center thin clients;
- shared engineering terminals;
- education terminals;
- controlled-access consoles; and
- regulated VDI environments.

In these environments, a narrow authentication transport may be easier to justify than generic peripheral forwarding.

APW can potentially support a policy model such as:

```text
Permitted:
    Display
    Keyboard
    Pointer
    Audio Playback
    Audio Capture
    APW CTAP Transport

Not Permitted:
    Generic USB Forwarding
    File Transfer
    Arbitrary Serial Devices
    Removable Storage
```

This policy distinction may be useful for organizations applying the Principle of Least Privilege to endpoint connectivity.

---

## C.8 Productization Models

APW does not inherently require a single product form.

Possible commercial models include:

### C.8.1 Authenticator Hardware

A physical APW-capable roaming authenticator containing:

```text
Secure Element / Protected Key Store
CTAP2 Authenticator
APW Protocol Engine
Software Modem / DSP
Audio Codec
TRRS / USB Audio / Bluetooth Audio Interface
```

### C.8.2 APW SDK

A software package for host or platform vendors containing:

```text
APW Modem Library
APW Framing / ARQ
CTAP Transport Adapter
Audio Endpoint Abstraction
Device Discovery Logic
Reference Integration Code
```

### C.8.3 Embedded IP / Firmware License

APW may be incorporated into:

- thin-client firmware;
- zero-client firmware;
- game-console operating systems;
- kiosk platforms;
- smart displays;
- embedded Linux systems; or
- proprietary appliance operating systems.

### C.8.4 Conformance and Qualification Tooling

A deployable technology would benefit from a test package covering:

```text
Audio Channel Qualification
Codec Survivability
BER / FER Measurement
Latency Measurement
Fallback Verification
CTAP Transaction Validation
Security Negative Tests
Underlay Compatibility Testing
```

### C.8.5 Enterprise Integration

APW may also be delivered as middleware for organizations that control both their client image and authenticator fleet.

---

## C.9 Commercial Value of Avoiding New Device Drivers

For many platforms, a new device driver is not a small implementation detail.

It can create recurring costs in:

- security certification;
- code signing;
- compatibility validation;
- endpoint-management policy;
- software distribution;
- patch maintenance;
- operating-system upgrades;
- incident response;
- device inventory;
- remote-session forwarding; and
- customer support.

APW's ability to reuse a pre-existing audio device model can therefore have economic value independent of the modem implementation itself.

The commercial proposition can be summarized as:

> **Introduce a new authentication transport without introducing a new peripheral class.**

A more implementation-oriented formulation is:

> **Reuse the platform's existing audio hardware-enablement boundary and add CTAP semantics above it.**

This is potentially more important than the fact that the underlying signals happen to be audio.

---

## C.10 Integration Boundary as a Business Advantage

A native hardware transport often requires support from multiple layers:

```text
Physical Device
     |
Kernel Driver
     |
OS Device Framework
     |
Remote Redirection
     |
Application Integration
```

APW attempts to reuse the first several layers:

```text
Standard Audio Device
     |
Existing Audio Driver
     |
Existing Audio Framework
     |
Existing Audio Redirection
     |
APW Software
     |
CTAP Integration
```

This can move the point of innovation upward in the stack.

In practical terms, it may enable a vendor to prototype and deploy new authentication behavior without first obtaining support for a new peripheral type throughout every layer of the system.

---

## C.11 Deployment Reach Versus Native Efficiency

APW is not intended to replace native FIDO transports where they are already available and acceptable.

Native USB HID, NFC, BLE, or platform-integrated WebAuthn redirection may provide:

- lower latency;
- better power efficiency;
- better discovery;
- standardized security semantics;
- mature certification; and
- superior user experience.

APW's comparative advantage is **deployment reach**.

The relevant engineering tradeoff is therefore:

```text
Native Transport
    -> Better efficiency and native integration

APW
    -> Potentially broader compatibility using
       already-deployed audio infrastructure
```

A practical system may support both.

---

## C.12 Suggested Initial Market Sequence

A realistic deployment sequence is:

```text
1. Laboratory PoC
   TRRS + software modem + CTAP2

2. VDI PoC
   Existing bidirectional audio redirection

3. Standard USB Audio Endpoint
   Demonstrate no APW-specific device driver

4. Thin / Zero Client Integration
   Demonstrate managed enterprise use

5. Kiosk / Appliance Integration
   Demonstrate non-PC deployment

6. OEM / Platform Integration
   Console, smart device, embedded platform

7. Standardization Discussion
   Transport registration / interoperability
```

This sequence intentionally starts with environments in which the implementer controls the software stack.

It avoids requiring platform-vendor cooperation before the basic transport value has been demonstrated.

---

## C.13 Reference Commercial Proposition

APW may be summarized for non-protocol audiences as:

> **Audio Pseudowire turns an already-supported bidirectional audio endpoint into a narrowly scoped transport for strong FIDO2 authentication.**

Or more specifically:

> **If a platform can already expose a controllable microphone and audio-output path, it may be possible to add an external FIDO2 authenticator without introducing a new peripheral device class or APW-specific device driver.**

The resulting value proposition is not based on audio novelty.

It is based on reuse:

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

## C.14 Business and Security Boundary

The commercial usefulness of APW depends on preserving its narrow scope.

If APW evolves into a general-purpose data tunnel, many of its operational and security advantages are weakened.

A production APW profile SHOULD therefore remain intentionally constrained to authentication-related communication.

This provides a clearer message to platform vendors, security reviewers, and enterprise administrators:

> APW is not a replacement network interface.

> APW is not arbitrary USB tunneling.

> APW is a purpose-limited CTAP transport implemented over an existing audio abstraction.

That distinction is central both to the technical architecture and to the potential business case.
