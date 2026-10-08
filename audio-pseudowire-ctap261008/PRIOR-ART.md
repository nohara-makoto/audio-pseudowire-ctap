# Prior Art and Adjacent Work

**Status:** Working survey  
**Last reviewed:** October 2026  
**Purpose:** Identify prior work, avoid novelty claims, and invite corrections

This document records public work found near the APW-CTAP idea.

It is intentionally conservative.

> **Current conclusion:** We have not found a public project or standards proposal that clearly matches the complete APW-CTAP combination:
>
> **CTAP2 transported over a software audio modem using ordinary bidirectional audio endpoints, with analog TRRS, USB Audio, Bluetooth Audio, and redirected/virtual audio treated as interchangeable underlays.**
>
> This is **not** a claim that no such work exists.

If you know of earlier or parallel work, please open an issue with a reference.

---

# 1. What counts as a close match?

For this survey, a strong match would include most of the following:

1. FIDO2 / CTAP2 semantics are preserved.
2. Audio is used as the actual client-to-authenticator transport.
3. The audio waveform carries framed digital CTAP traffic.
4. The design is not limited to ambient-sound comparison or voice calls.
5. Standard audio endpoints are intentionally reused as the platform compatibility layer.
6. The design considers more than one audio underlay, such as:
   - analog audio;
   - USB Audio Class;
   - Bluetooth audio;
   - virtual audio; or
   - VDI redirected audio.
7. The proposal is intended as a transport binding or transport adapter rather than a new authentication ceremony.

No public project found so far satisfies this complete set.

---

# 2. FIDO CTAP transport architecture

FIDO CTAP already separates authenticator protocol semantics from transport-specific bindings.

Known native transport families include USB HID, NFC, BLE, and hybrid mechanisms.

This architectural separation is important because APW is intended as an **additional transport binding**, not a replacement for WebAuthn or CTAP semantics.

Relevant source:

- FIDO Alliance, Client to Authenticator Protocol (CTAP):  
  https://fidoalliance.org/specifications/

At the time of this survey, no standardized CTAP audio transport binding was identified.

---

# 3. FIDO hybrid / proximity work

FIDO hybrid and proximity work is conceptually adjacent because it demonstrates that:

- credential operations;
- proximity establishment; and
- the data-transfer channel

do not necessarily have to be represented by one monolithic physical transport.

The FIDO Proximity Exchange Protocol (PXP) is therefore relevant architectural prior art.

However, the public PXP work reviewed for this survey does not define ordinary audio endpoints as a CTAP transport underlay.

Relevant source:

- FIDO Alliance specifications / hybrid and proximity work:  
  https://fidoalliance.org/specifications/

---

# 4. VDI and remote FIDO redirection

Remote use of FIDO authenticators is an established problem.

Existing solutions typically solve it by adding a dedicated remote-desktop/WebAuthn/FIDO redirection mechanism.

Examples include:

## 4.1 Microsoft RDP / FreeRDP

FreeRDP has implemented WebAuthn/FIDO2 redirection using the RDP WebAuthn virtual-channel architecture.

Relevant project:

- https://github.com/FreeRDP/FreeRDP

This is a close **use-case** match but not a transport match.

APW differs by attempting to reuse ordinary bidirectional audio redirection rather than a FIDO-specific remote virtual channel.

## 4.2 Citrix

Citrix provides FIDO2 redirection support in its VDI stack.

Relevant documentation:

- https://docs.citrix.com/

Again, this demonstrates demand for remote FIDO but uses VDI-specific integration rather than an audio modem transport.

## 4.3 Apache Guacamole

Guacamole has had work around WebAuthn/passkey forwarding and relay behavior.

Relevant project:

- https://github.com/apache/guacamole-client

This is adjacent remote-authentication work, not an audio CTAP transport.

---

# 5. Acoustic / audio authentication

There is substantial prior work using sound for authentication.

Examples include:

- acoustic challenge/response systems;
- ultrasonic pairing;
- ambient-sound proximity verification;
- audio-jack transaction/authentication tokens;
- software modems for arbitrary data;
- telephony-based out-of-band authentication.

These establish that audio can carry authentication-related information.

They do **not** by themselves establish CTAP2-over-audio.

---

# 6. Ambient-sound second-factor research

Research such as **Sound-Proof** uses ambient sound similarity as a second factor or proximity signal.

Representative paper:

- Nikolaos Karapanos et al., *Sound-Proof: Usable Two-Factor Authentication Based on Ambient Sound*  
  https://arxiv.org/abs/1503.03790

This is security prior art involving sound, but it does not transport CTAP request/response messages over an audio modem.

---

# 7. Acoustic challenge/response prototypes

Public prototypes exist that use FSK or related acoustic signaling for challenge/response authentication between devices.

These are important adjacent work because they show that:

- laptops and phones can exchange structured authentication traffic over audio;
- commodity audio hardware can support a modem-like link; and
- robust authentication protocols can be layered over acoustic signaling.

However, the examples found during this survey use their own authentication protocols rather than preserving FIDO CTAP2 semantics.

A representative public project found during the survey:

- https://github.com/yaidarbek/acoustic-authentication

This should be treated as adjacent technical prior art, not an APW-CTAP implementation.

---

# 8. Audio-jack authentication and payment tokens

Audio-jack-connected authentication and transaction devices existed before modern FIDO2.

These are especially relevant because they demonstrate that analog headset interfaces can be used as bidirectional digital links to security peripherals.

Examples include commercial and patented audio-jack token/payment-reader designs.

This is strong prior art for the **physical idea of data-over-audio to a security device**.

APW differs in the intended abstraction:

```text
Existing work:
    application-specific audio security token

APW:
    preserve CTAP2 semantics
    +
    define an audio transport abstraction
    +
    allow TRRS / USB Audio / BT Audio / VDI Audio
    as interchangeable underlays
```

Because this area contains patents and proprietary systems, additional prior-art review is welcome.

---

# 9. Software modems and data-over-audio libraries

Many existing projects can provide part of the APW physical layer.

Examples include:

- minimodem
- libquiet / Quiet
- liquid-dsp
- AFSK/FSK modem implementations
- amateur-radio modem software
- telephony modem DSP

These projects are implementation building blocks, not FIDO/CTAP transports.

Representative links:

- minimodem: https://github.com/kamalmostafa/minimodem
- Quiet: https://github.com/quiet/quiet
- liquid-dsp: https://github.com/jgaeddert/liquid-dsp

APW should reuse mature DSP work where practical rather than inventing modulation algorithms unnecessarily.

---

# 10. USB Audio Class and Bluetooth Audio

Standard audio device classes/profiles are not themselves authentication prior art, but they are central deployment primitives.

They demonstrate that many platforms already know how to expose audio render/capture paths without vendor-specific drivers.

Relevant standards organizations:

- USB-IF: https://www.usb.org/
- Bluetooth SIG: https://www.bluetooth.com/

APW's proposed use of those existing audio abstractions as a CTAP transport substrate is the part for which no direct public match has yet been identified.

---

# 11. libfido2 custom transport hooks

libfido2 provides transport abstraction hooks that can be used to experiment with non-standard authenticator transports.

Relevant project:

- https://github.com/Yubico/libfido2

This is highly relevant implementation infrastructure because an APW proof of concept may be possible without first modifying the higher-level FIDO semantics.

The existence of custom transport hooks is not itself an audio transport proposal.

---

# 12. Why the distinction matters

The following ideas already exist separately:

```text
Audio modem                           Yes
Audio-jack security token             Yes
Acoustic authentication               Yes
FIDO2 / CTAP2                         Yes
Remote FIDO redirection               Yes
Standard USB Audio                    Yes
Bluetooth audio                       Yes
Virtual / redirected VDI audio        Yes
Custom CTAP transport abstraction     Yes
```

The combination being proposed here is:

```text
CTAP2
  over
APW framing / reliability
  over
software audio modem
  over
standard bidirectional audio endpoint
  over
TRRS / USB Audio / Bluetooth Audio / VDI Audio
```

That complete combination is the specific item for which matching public work has not yet been found.

---

# 13. Search limitations

This survey cannot prove non-existence.

Possible blind spots include:

- unpublished internal projects;
- abandoned prototypes;
- patents using unexpected terminology;
- non-English publications;
- vendor research;
- standards mailing-list discussions;
- private FIDO Alliance material;
- conference demos without archived source;
- projects described as "acoustic", "modem", "headset", "voice channel", or "side channel" rather than "audio transport".

For that reason, this project should avoid claims such as:

> "This is the first such system."

Preferred wording is:

> **"We have not found a prior public implementation or proposal matching this architecture. Please point us to prior art we missed."**

---

# 14. Requested prior-art feedback

Useful references include:

- standards drafts;
- GitHub repositories;
- academic papers;
- patents;
- conference talks;
- vendor whitepapers;
- discontinued products;
- archived mailing-list discussions; and
- proprietary products with public technical documentation.

Please include enough information to determine whether the work transports actual CTAP/FIDO messages or merely uses sound as an authentication signal.
