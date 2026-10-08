# FIDO CTAP2向け Audio Pseudowire Transport Binding

## APW-CTAP: リモート・仮想化・制約環境向けAudio Endpoint Transport

**文書ステータス:** Experimental Technical Specification  
**バージョン:** 0.3  
**想定ステータス:** Experimental  
**著者:** TBD  
**日付:** 2026年10月

---

# 概要

本書は、FIDO Client to Authenticator Protocol（CTAP2）向けの実験的な **Audio Pseudowire（APW）Transport Binding** を定義する。

APWは、FIDO roaming authenticatorとCTAP clientとの通信を、既存の双方向Audio Pathを利用して搬送する。

Audio Pathは、例えば以下によって提供され得る。

- Analog Headset Interface
- USB Audio Class Device
- Bidirectional Bluetooth Audio Profile
- Virtual Audio Endpoint
- Virtual Desktop Infrastructure（VDI）のRedirected Audio
- Thin Client / Zero Client
- Audio Input / Outputを利用可能なGame ConsoleやAppliance
- その他、双方向Audio Render / Capture Streamを提供可能なPlatform

APWはPhysical Attachment Mechanismを意図的に抽象化する。

重要なのは、

**「どのConnector、Bus、RadioでAuthenticatorが接続されたか」**

ではない。

重要なのは、

**「Platformから利用可能なAudio Endpointとして認識・操作できるか」**

である。

Platformが選択されたStandard Audio Class / Profileを既にサポートしている場合、APW Authenticatorは、新しいAPW専用Peripheral Device ClassやEdge-side Device Driverを追加することなく利用できる可能性がある。

本仕様が主として対象とするのは、

- Generic USB Device Redirection
- Bluetooth Peripheral Forwarding
- FIDO-specific Device Redirection

などが禁止、利用不能、またはPlatform依存である一方、

**Audio Input / Outputは既に利用可能**

という環境である。

APWはCTAP Transport Messageを低速Digital Audio Waveformへ変換し、既存Audio Path上で搬送する。

APWは以下を定義しない。

- 新しいWebAuthn Ceremony
- 新しいCredential Type
- 新しいAuthenticator Cryptographic Model
- Generic USB Tunnel
- Bluetooth Forwarding Protocol
- IP Tunnel
- General-purpose Covert Communication Channel

初期APW ProfileはBandwidth Efficiencyよりも、

- Robustness
- Portability
- Implementation Simplicity
- Heterogeneous Audio Infrastructureとの互換性

を優先する。

---

# 1. 本文書の位置付け

本書は独立したExperimental Proposalであり、現時点ではFIDO Alliance、W3C、IETF、USB-IF、Bluetooth SIG、その他の標準化団体による正式仕様ではない。

目的は以下を可能にする程度に十分明確なProtocolを定義することである。

- Prototype Implementation
- Interoperability Test
- Security Analysis
- VDI Evaluation
- Cross-platform Evaluation
- 将来のCTAP Transport Bindingに関する議論

**Audio Pseudowire** という名称は、Audio Input / Output Facility上に構築されるPoint-to-Point Logical Communication Pathの抽象化を表す。

本書におけるPseudowireという用語は記述的な名称であり、IETF PWE3 Architectureへの適合を主張するものではない。

---

# 2. 規範用語

本書における、

**MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, **OPTIONAL**

は、すべて大文字で記述されている場合に限り、BCP 14 [RFC2119] [RFC8174] に従って解釈される。

---

# 3. Design Premise

APWの中心となる観察は、

**多くのPlatformでは、任意Peripheral TransportよりAudio Abstractionの方が広く利用可能である**

という点にある。

Platformによっては以下が禁止、制限、または未実装である。

```text
Generic USB Forwarding
FIDO-specific USB Forwarding
BLE Peripheral Forwarding
NFC Forwarding
Vendor-specific Device Driver
Arbitrary Local Peripheral Access
```

しかし同じPlatformで、

```text
Audio Render
Audio Capture
```

は普通に提供されている場合がある。

その理由として、

- Web会議
- Voice Communication
- Accessibility
- Gaming
- Media
- Remote Desktop
- Headset Support

などが挙げられる。

APWは、このAudio AbstractionをCTAP2専用の限定的Communication Substrateとして利用する。

設計目標は、

> USBをAudio上にTunnelする

ことではない。

また、

> 任意データをSoundに偽装する

ことでもない。

目的は、

> CTAP2が必要とする最小限のReliable Point-to-Point Link Semanticsだけを、既存のBidirectional Audio Endpoint上に構成する

ことである。

---

# 4. 適用範囲

APWは以下を定義する。

1. Audio Endpoint Abstraction
2. Logical Point-to-Point APW Link
3. Baseline Voice-band Signaling
4. Link Training
5. Capability Negotiation
6. Packet Framing
7. Fragmentation / Reassembly
8. Error Detection
9. Retransmission
10. Transport Control Operation
11. CTAP Request / Response Messageのカプセル化
12. Optional Enhanced Media Profile
13. 最低限のTransport Security Requirement

APWは以下を定義しない。

- WebAuthn API Behavior
- WebAuthn Origin Processing
- Credential Format
- Attestation Format
- Authenticator Private-key Storage
- User Presence Mechanism
- User Verification Mechanism
- General-purpose Network Interface
- Arbitrary Serial Access
- USB Forwarding
- Bluetooth Peripheral Forwarding
- Remote Desktop Session Security
- Physical Proximity Proof

---

# 5. WebAuthnおよびCTAP2との関係

APWはWebAuthn Client Layerの下位で動作し、CTAP2 Transport Bindingとして機能する。

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

APWはCTAP MessageをOpaque Payloadとして搬送する。

APW Transport Layerは以下のApplication Semanticを解釈してはならない。

- Credential Creation
- Assertion Generation
- RP Identifier
- Challenge
- Credential Identifier
- Attestation Object

Layeringは以下となる。

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

Browser-side Audio ImplementationはExperimental Testingには有用であり得る。

ただしGeneric Browser Audio APIだけでは、Platform WebAuthn Authenticator Subsystemへの統合は自動的には行われない。

Complete Implementationには、WebAuthn Clientの下位または隣接層にCTAP/APW Transport Adapterが必要である。

---

# 6. Audio Endpoint Abstraction

## 6.1 基本要件

APWは、EndpointがAudio Render / Capture Streamを得る物理的方法に依存してはならない。

APW-capable Endpointには以下の2方向が必要である。

```text
APW Client  ------audio------>  APW Authenticator
APW Client  <-----audio-------  APW Authenticator
```

この2方向は、別々のPhysical / Logical Facilityで提供してもよい。

CTAP LayerからUnderlying Mechanismが見える必要はない。

## 6.2 Audio Endpoint Criterion

Physical / Virtual Interfaceは、以下を直接または間接的に提供可能であればAPW Underlayとして利用可能である。

1. Audio Render Path
2. Audio Capture Path
3. Negotiated APW Profileに必要なBandwidth
4. APW Endpoint Softwareが利用可能なAudio Access

したがってAPWが依存するのは、

**Connector IdentityではなくAudio Semantics**

である。

## 6.3 APW専用Peripheral Profileを必須としない

Operating System、Thin Client、Appliance、Console、その他Edge Platformが選択されたAudio Class / Profileを既にサポートしている場合、APWはその既存実装を再利用することをSHOULDとする。

例えば、

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

を、

```text
APW Authenticator
       |
       | USB
       v
Vendor-specific APW Device Class
       |
       v
Custom Kernel Driver
```

より優先する。

PlatformがStandard Audio Deviceを、APW Softwareから利用可能なAudio Interfaceとして既に公開している場合、

**APW専用Device Driverを新たに追加する必要はない。**

ただし、これはAPW Software自体が不要という意味ではない。

Host側には依然として、

- APW Modem
- CTAP Transport Adapter

が必要である。

---

# 7. Underlay Profile

APW Protocolは個々のPhysical Attachment Technologyから独立する。

初期Underlay Classとして以下を想定する。

| Profile | Local Attachment例          | Platformから見えるもの                          |
| ------- | -------------------------- | ---------------------------------------- |
| APW-A   | Analog TRRS / Analog Audio | Headphone + Microphone                   |
| APW-U   | USB                        | Standard USB Audio Endpoint              |
| APW-BH  | Bluetooth Classic          | Bidirectional Headset / Hands-Free Audio |
| APW-BL  | Bluetooth LE Audio         | Bidirectional LE Audio Endpoint          |
| APW-V   | Virtual Audio              | Virtual Render / Capture Device          |
| APW-R   | Redirected Audio           | Remote / VDI Audio Stream                |

特定Physical ProfileのSupportは、別途Implementation Profileで指定されない限りOPTIONALとする。

APW-VB1は、これらUnderlayより上位でModem Interoperabilityを定義する。

---

# 8. Local TransportとRemote Transportの分離

APWでは、

1. Local Attachment
2. Remote Transport Exposure

を明確に区別する。

例えば、

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

という構成では、USBはLocal Connectionとして利用される。

しかしUSB DeviceそのものをRemote VMへRedirectしているわけではない。

Remote Execution Environmentが受け取るのは、

**USB TransactionではなくAudio**

である。

同様に、

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

ではBluetooth Peripheral Forwardingを必要としない。

これはAPW Architectureの重要な性質である。

APWの目的は、

**USBやBluetoothをLocal Attachment Technologyとして排除すること**

ではない。

目的は、

**Remote ApplicationがそれらPeripheral Transportを理解またはForwardする必要をなくすこと**

である。

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

VDI Protocol自身は以下を理解する必要がない。

- FIDO
- CTAP2
- USB HID
- BLE FIDO
- AuthenticatorのLocal Physical Attachment

必要なのはAudioを搬送することだけである。

---

# 10. 非VDI・非PC Platform

Audio Endpoint AbstractionはConventional Desktop OSに限定されない。

Potential APW Hostには以下が含まれる。

- Thin Client
- Zero Client
- Kiosk
- Embedded Terminal
- Industrial Console
- Smart Display
- Game Console
- Media Appliance
- Mobile Device
- その他Bidirectional Audio Facilityを持つSystem

例えばStandard HeadsetやUSB Audio Endpointを既に認識できるPlatformであれば、新しいFIDO専用Hardware Device Classを追加せずにAPW Authenticatorを接続できる可能性がある。

ただし、

**Audio Deviceとして認識されるだけではWebAuthn機能は完成しない。**

Platformにはさらに、

1. Audio StreamへアクセスするSoftware Execution Path
2. APW Modem
3. CTAP2 ImplementationまたはCTAP Client
4. Application / WebAuthn EnvironmentとのIntegration

が必要である。

したがってAPWは、

**FIDO2 Authenticatorを実装可能なPlatformの範囲を広げる**

可能性を持つが、

**Audio-capable Deviceを自動的にWebAuthn-capable Deviceへ変換する**

ものではない。

---

# 11. Bidirectionality Requirement

APWには2方向のLogical Communication Pathが必要である。

したがって、

```text
Audio Playback Only
```

のTechnologyだけではAPW Underlay要件を満たさない。

例えばBluetooth AudioがPlayback Directionのみ提供するPlatformでは、そのBluetooth Path単独ではAPWを利用できない。

ただし、

```text
Playback : Endpoint A
Capture  : Endpoint B
```

のように複数Facilityを組み合わせてもよい。

---

# 12. APW Voice-Band Baseline Profile

Mandatory Baseline Modem Profileを以下とする。

**APW-VB1**

APW-VB1はSpeech-oriented Audio Systemを通過することを優先する。

Mandatory Signaling Rangeは概ね、

**1 kHz ～ 3 kHz**

とする。

Baseline ImplementationはUltrasonic Frequencyへ依存してはならない。

Wideband / Ultrasonic ProfileはExperimental Extensionとして定義してもよい。

---

# 13. Baseline Modulation

APW-VB1はNon-transparent Audio Channelに適したRobust Binary Frequency Shift Signalingを使用する。

初期実装では、例えば以下を利用できる。

```text
Logical 0   約1200 Hz
Logical 1   約2200 Hz
```

Signal Rateの初期目標は、

```text
約1200 symbols/second
```

とする。

Receiverは以下に耐えなければならない。

- Amplitude Variation
- AGC
- Codec Quantization
- Resampling
- Clock Offset
- Phase Discontinuity
- Jitter
- Limited Media Loss

Sample-accurate PCM Preservationを前提としてはならない。

---

# 14. Half Duplex Baseline

APW-VB1実装はHalf DuplexをMUST Supportとする。

Full DuplexはMAYとする。

Half Duplexを初期Modeとして推奨する理由は以下である。

- Modem Designが単純
- Echo Cancellationとの干渉が少ない
- Crosstalk Sensitivityを低減
- Voice-oriented Linkとの相性がよい
- CTAPのRequest / Response Characterと一致

---

# 15. Channel Training

APW EndpointはData Transfer前にChannel Soundingを実施してよい。

推奨Probe例:

```text
1000 Hz
1400 Hz
1800 Hz
2200 Hz
2600 Hz
3000 Hz
```

BaselineではSimultaneous Multi-toneよりSequential Toneを推奨する。

Receiverは、

- FFT
- Goertzel Filter
- Matched Filter

などを利用してよい。

DSP ImplementationはNormativeではない。

---

# 16. Capability Negotiation

Baseline Synchronization後、Endpointは以下を交換することをSHOULDとする。

```text
APW Protocol Version
Maximum APW Frame Payload
Maximum CTAP Message Size
Supported Modulation Profiles
Supported FEC Profiles
Supported Duplex Modes
Optional Secure-Link Profiles
```

Enhanced Modeが失敗した場合、Session Termination前にAPW-VB1へFallbackすることをSHOULDとする。

---

# 17. Link-layer Architecture

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

CTAP Message全体が1 Frameへ収まる必要はない。

---

# 18. Frame Format

APW Data Frameは論理的に以下を含む。

```text
+----------------------+-------------------------------+
| Physical Preamble    | Modem Synchronization         |
+----------------------+-------------------------------+
| Start Delimiter      | Frame Boundary                |
+----------------------+-------------------------------+
| Version              | APW Protocol Version          |
+----------------------+-------------------------------+
| Type                 | DATA / ACK / CONTROL / ERROR  |
+----------------------+-------------------------------+
| Flags                | Fragment / Control Flags      |
+----------------------+-------------------------------+
| Session ID           | Logical APW Session           |
+----------------------+-------------------------------+
| Sequence Number      | ARQ / Duplicate Suppression   |
+----------------------+-------------------------------+
| Message ID           | CTAP / APW Message ID         |
+----------------------+-------------------------------+
| Fragment Offset      | Position Within Message       |
+----------------------+-------------------------------+
| Total Length         | Complete Message Length       |
+----------------------+-------------------------------+
| Payload Length       | Current Fragment Length       |
+----------------------+-------------------------------+
| Payload              | Opaque CTAP / Control Data    |
+----------------------+-------------------------------+
| CRC-32C              | Error Detection               |
+----------------------+-------------------------------+
```

Multi-byte IntegerはNetwork Byte OrderをSHOULDとする。

---

# 19. Transport Message Type

初期APW Message Classは以下を含むことをSHOULDとする。

| Type        | 用途                       |
| ----------- | ------------------------ |
| `HELLO`     | Link Initialization      |
| `HELLO_ACK` | Capability Response      |
| `DATA`      | CTAP Fragment            |
| `ACK`       | Positive Acknowledgement |
| `NACK`      | Retransmission Request   |
| `KEEPALIVE` | Long-running Operation   |
| `CANCEL`    | Operation Cancel         |
| `PING`      | Link Validation          |
| `ERROR`     | Transport Error          |
| `RESET`     | Link Reset               |

APWは、

- Challenge
- Credential
- Assertion

のようなWebAuthn Application SemanticをTransport Typeとして定義してはならない。

---

# 20. Fragmentation / Reassembly

APWはCTAP Message FragmentationをMUST Supportとする。

Baseline Payload Fragmentは初期値として、

**64～256 byte程度**

をSHOULDとする。

Receiverは以下を実施する。

1. Frame Integrity Verification
2. Invalid OffsetのReject
3. Inconsistent LengthのReject
4. Duplicate Frame Suppression
5. Correct Reassembly
6. Complete MessageのみCTAPへDelivery

Incomplete DataをCTAP Messageとして扱ってはならない。

---

# 21. Error Detection / Recovery

すべてのAPW FrameはError Detection Mechanismを持つ。

APW-VB1ではCRC-32CをSHOULDとする。

CRCはCryptographic Authenticationではない。

Baseline実装はRetransmissionをMUST Supportとする。

以下を推奨する。

- Stop-and-Wait ARQ
- Small-window ARQ

---

# 22. Forward Error Correction

FECはOPTIONALとする。

Burst Errorが多いUnderlayではNegotiated FEC Profileを利用してよい。

例えば、

- Shortened Reed-Solomon
- Interleaving

などを候補とする。

FEC非対応Endpointでも、

**CRC + Retransmission**

によりInteroperateできなければならない。

---

# 23. CTAP Mapping

Complete APW Messageは1個のComplete CTAP RequestまたはResponseを搬送する。

APWはCTAP Byte Semanticsを維持する。

CTAP CBORをTransport都合で変更してはならない。

```text
CTAP Client
    |
    | Request
    v
APW Message
    |
    | Audio Frames
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

# 24. Keepalive / User Interaction

APWは、

```text
Broken Transport
```

と、

```text
AuthenticatorがUser Action待ち
```

を区別しなければならない。

`KEEPALIVE`を利用して、

- User Presence
- User Verification
- Biometric
- PIN
- その他Interaction

待ちを表現してよい。

---

# 25. Discovery

Audio Endpoint自身はFIDO Authenticatorであることを本質的には示さない。

したがってAPWにはExplicit Discovery MechanismまたはPolicyが必要である。

候補:

1. Administrator Configuration
2. Explicit User Selection
3. Jack / Connection Detection
4. Local Policy
5. Known Audio Endpoint Identity
6. Explicit APW Mode
7. Authentication Operation中のみ行うBounded Active Probe

Background Probeによって任意Speakerから突然Modem Toneを発生させてはならない。

---

# 26. Analog Wired Profile — APW-A

APW-AはSimple Wired Reference Profileである。

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

4極Headset-style ConnectorはInitial Implementationとして実用的である。

ただしAPW Protocol自身は特定Analog Connectorを要求しない。

Analog Circuitは以下を考慮することをSHOULDとする。

- AC Coupling
- Microphone Bias
- Attenuation
- Impedance
- Over-voltage Protection
- Headset Detection

APW-AはPrototype / Fallbackとして有用である。

しかし、

**APW Architectureそのものを4極ケーブルで定義するものではない。**

---

# 27. USB Audio Profile — APW-U

APW-UはLocal USB Audio DeviceをAudio Endpointとして利用する。

AuthenticatorはEdge Platformに対して、FIDO-specific USB HID Deviceではなく、

**Standard Audio Device**

としてEnumerateしてよい。

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
APW Client / VDI Redirection
```

USB DeviceそのものをRemote VMへRedirectする必要はない。

したがってAPW-Uは、

```text
Local USB Audio       : Allowed
Generic USB Forwarding: Prohibited
```

というEnvironmentとも両立可能である。

PlatformがSuitable Standard USB Audio Driverを持つ場合、APW-UはAPW専用Kernel Driverを要求しないことをSHOULDとする。

---

# 28. Bluetooth Audio Profile — APW-BH / APW-BL

PlatformがSuitable Bidirectional Audio Pathを公開する場合、APWはBluetooth Audio Technologyを利用してよい。

候補:

- Bidirectional Classic Bluetooth Headset / Hands-Free Audio
- Bidirectional LE Audio Configuration

Output-only Bluetooth AudioはAPW Requirementを満たさない。

APWは新しいBluetooth FIDO Profileを定義しない。

目的は、

**Platformが既に理解しているAudio Serviceを再利用すること**

である。

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

したがってBluetoothは、

**CTAP Peripheral Forwarding Mechanism**

ではなく、

**Local Audio Attachment Technology**

として扱われる。

---

# 29. Virtual / Redirected Audio Profile

APW-V / APW-Rは一方または両方のEndpointがVirtualなEnvironmentを対象とする。

例:

- Virtual Audio Cable
- Containerized Audio Endpoint
- Remote Desktop Virtual Microphone
- Remote Desktop Virtual Speaker
- Browser-accessible Audio Endpoint
- Conferencing Redirection Path
- Zero-client Audio Channel

APWはこれらがPCM Sampleを完全保存することを前提としてはならない。

---

# 30. Voice Processingとの相互作用

Audio Infrastructureには以下が介在する可能性がある。

- AGC
- AEC
- Noise Suppression
- Voice Activity Detection
- Lossy Codec
- Resampling
- Dynamic Bitrate
- Routing Change

可能であれば破壊的Processingを無効化することをSHOULDとする。

ただしAPW-VB1はNon-transparent Audio Transportに耐えるよう設計されなければならない。

---

# 31. Security Considerations

## 31.1 Narrow Protocol Surface

APWはCTAP Transportに必要なOperationだけを公開する。

以下を暗黙的に公開してはならない。

- IP Networking
- Arbitrary Serial Communication
- File Transfer
- Shell
- Generic USB Forwarding
- Unrestricted Peripheral Control

Security Objectiveは、

**Audioは安全である**

という主張ではない。

目的は、

**Scope Reduction**

である。

---

# 32. Local Audio DeviceとGeneric Peripheral Forwardingの違い

Local USB / Bluetooth Attachmentと、Generic USB / Bluetooth Forwardingを混同してはならない。

例えば、

```text
Authenticator --USB Audio--> Edge Device
```

の後、

```text
Edge Device --Audio--> Remote VM
```

とする構成では、Remote VMへUSB Protocolは公開されない。

Remote Sideが受け取るのはAudio Abstractionのみである。

これはAPW Architectureの主要なPropertyの一つである。

---

# 33. Authenticator Security Boundary

Credential Private KeyはAuthenticator Control下に留まらなければならない。

APWは以下を弱体化してはならない。

- User Presence
- User Verification
- Credential Protection
- Signature Generation

Modem LayerはTransport Infrastructureであり、Authentication Authorityではない。

---

# 34. Eavesdropping

Audio PathはDeployment-specific Controlにより証明されない限りObserve可能とみなす。

Potential Observer:

- Endpoint OS
- VDI Host
- Virtual Audio Driver
- Recording Software
- Remote Desktop Infrastructure
- Compromised Software

Baseline Cleartext APWはHigher-layer FIDO Protectionを超えるConfidentialityを提供しない。

---

# 35. Link Encryption

APW v0.2では新規Cryptographic Handshakeを定義しない。

Unauthenticated Ephemeral Diffie-Hellmanのみを利用してActive MITM Protectionを主張してはならない。

将来Secure-link ProfileとしてAuthenticated Key Establishment + AEADを定義してよい。

---

# 36. Replay / Injection / Denial of Service

Audio Endpointを操作できる攻撃者は、

- Frame Injection
- Corruption
- Replay
- Retransmission Trigger
- Link Reset
- Denial of Service

を実行し得る。

Session ID / Sequence Numberを用いてStale TrafficをRejectすることをSHOULDとする。

Invalid MessageはFail Closedしなければならない。

---

# 37. Relay / Proximity

APWはPhysical Proximityの証明ではない。

特にRemote DesktopではRemote Operationそのものが目的である。

Cryptographic Proximity Assuranceが必要な場合、別Mechanismで提供する。

---

# 38. Administrative Policy

APWはExternal AuthenticatorまたはAudio上のNon-voice Data Signalingを明示的に禁止するPolicyを回避する目的で利用すべきではない。

Enterprise DeploymentではAPW利用をExplicitly AuthorizeすることをSHOULDとする。

APWは、

> Existing Audio Facility上のConstrained Application-specific Transport

として評価されるべきである。

---

# 39. Cross-platform Design Objective

APWの重要なDesign Goalの一つは、

**Host固有Peripheral Interfaceへの不要な依存を減らすこと**

である。

SoftwareからAudio Input / Output Endpointを利用できるPlatformであれば、Native FIDO Roaming-authenticator Interfaceを持たないPlatformでもAPWへ参加できる可能性がある。

その対象には例えば、

```text
Thin Client
Zero Client
Kiosk
Game Console
Embedded Appliance
Smart Display
Special-purpose Terminal
Mobile Device
```

が含まれる。

必要条件は、

```text
Standard Audio Support
        +
APW Software
        +
CTAP Integration
        =
Potential FIDO2 Authenticator Transport
```

である。

これは、

```text
Standard Audio Support
        =
Automatically FIDO2-capable
```

という意味ではない。

この違いは重要である。

---

# 40. Prototype Conformance Stage

## Stage 1 — Audio Link

APW-A上でRandom Binary DataのBidirectional Transportを確認する。

## Stage 2 — Framing

以下を実証する。

```text
HELLO
PING
DATA
ACK
```

および、

- CRC
- Retransmission

の正常動作。

## Stage 3 — CTAP Basic Operation

```text
authenticatorGetInfo
```

を正常に搬送する。

## Stage 4 — Credential Creation

```text
authenticatorMakeCredential
```

を正常に搬送する。

## Stage 5 — Authentication

```text
authenticatorGetAssertion
```

を正常に搬送する。

## Stage 6 — VDI

Remote Boundaryを越えるIntentional Transportとして、

**Ordinary Bidirectional VDI Audio Redirectionのみ**

を利用してStage 3～5を再実行する。

## Stage 7 — Alternative Audio Underlay

以下の少なくとも1つでCTAP Operationを再実行する。

- Standard USB Audio
- Supported Bidirectional Bluetooth Audio Profile

これによりAPWがAnalog Cable専用Protocolではなく、

**Audio Abstraction**

であることを実証する。

---

# 41. Recommended Reference Prototype

最初のReference Authenticatorは以下のような構成を利用できる。

```text
Raspberry Pi / Embedded Linux
        |
CTAP2 Authenticator
        |
APW Daemon
        |
Audio Codec
```

Local Attachmentとして以下を利用できる。

```text
TRRS
USB Audio
Bluetooth Audio
```

Initial PoCでは観測・制御しやすいAPW-Aから開始することをSHOULDとする。

Second-stage PrototypeではAPW-Uまたは他Standard Audio Device Profileを実証することをSHOULDとする。

---

# 42. Experimental Browser Implementation

Web Audio APIは以下に利用してよい。

- Modem Experiment
- BER Test
- Visual Debugging
- Channel Sounding
- Demonstration
- PoC Application

ただしNative WebAuthn / CTAP Integrationが存在しない限り、Complete WebAuthn Authenticator Transportと説明してはならない。

---

# 43. Performance Objectives

初期Target:

```text
Baseline Gross Rate:          約1200 bit/s
Enhanced Rate:                2400 bit/s以上
Typical CTAP Transaction:     数百byte
Supported CTAP Message Size:  Connected Authenticator要件以上
```

Maximum ThroughputよりReliabilityを優先する。

---

# 44. Design Principles

## 44.1 Audio Is the Abstraction

APWはConnectorではなくAudio SemanticsへBindingする。

## 44.2 Existing Device Classを再利用

可能な限りStandard Audio Supportを再利用することをSHOULDとする。

## 44.3 不要なDriverを追加しない

Platformが利用可能なAudio Interfaceを既に持つ場合、新しいAPW-specific Edge Device Driverを要求しないことをSHOULDとする。

## 44.4 Local AttachmentとRemote Redirectionを分離

USB / BluetoothをLocalで使用しても、それらをRemoteへForwardする必要はない。

## 44.5 CTAP Semanticsを維持

APWはFIDO Authentication Semanticsを再定義してはならない。

## 44.6 Privilege Minimization

CTAP Transportに必要な機能だけを公開する。

## 44.7 Fail Closed

Invalid / Incomplete TrafficをSuccessful CTAP Operationとして扱ってはならない。

## 44.8 BandwidthよりRobustness

Authentication Trafficは小さいため、Aggressive Modem OptimizationよりReliabilityとSimple Recoveryを優先する。

---

# 45. 他のFIDO Transportとの関係

APWは追加CTAP Transport Abstractionとして提案される。

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

APWは既存Native FIDO Transportを置き換えるものではない。

Native Transportが利用可能かつPolicy上許可される場合、それを利用した方が適切な場合もある。

APWは、

**Native Authenticator TransportよりAudio Abstractionの方がDeployment Reachを持つEnvironment**

を対象とする。

---

# 46. WebAuthn Transport Identifier

v0.2では新しいWebAuthn `AuthenticatorTransport` Identifierを要求しない。

Prototype DiscoveryはImplementation Detailとして扱うことをSHOULDとする。

Interoperable Implementationが成立した場合、将来、

```text
"audio"
```

のようなTransport Hintを検討してよい。

---

# 47. Interoperability Test Matrix

以下をTestすることをSHOULDとする。

```text
Local Lossless PCM
Analog TRRS
USB Audio
Supported Bidirectional Bluetooth Audio
Virtual Audio Endpoint
VDI Audio Redirection
48 kHz -> 16 kHz -> 48 kHz
Lossy Voice Codec
AGC ON / OFF
AEC ON / OFF
Noise Suppression ON / OFF
Artificial Packet Loss
Artificial Jitter
```

Measurement:

- Acquisition Time
- BER
- Frame Error Rate
- Retransmission Count
- CTAP Completion Time
- Authentication Success Rate
- Fallback Behavior

---

# 48. Success Criteria

APW ConceptのPrimary Technical Validationは、

**Remote側が通常のAudio Transportだけを必要とするEnvironmentでCTAP2 Credential CreationおよびAuthenticationが完了すること**

である。

代表的Demonstration:

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

さらに強いDemonstrationは、

```text
TRRS
USB Audio
Bluetooth Audio
```

など複数Local Audio Attachment上で、

**CTAP Semanticsを変更せず同じAPW Implementationが動作すること**

である。

---

# 49. IANA / Registry Considerations

v0.2ではIANA Allocationを要求しない。

Experimental IdentifierはPrivate Experimental Valueを使用することをSHOULDとする。

将来Standardizationへ進む場合、Relevant FIDO / WebAuthn / Audio / USB / Bluetooth / Protocol Registryとの調整を検討する。

---

# 50. Open Issues

今後検証すべき項目:

1. APW-VB1 Final Waveform
2. Frame Timing
3. Optimum Fragment Size
4. Codec Adaptation
5. Endpoint Discovery
6. Audio-device Arbitration
7. Bluetooth Profile Compatibility
8. Console / Appliance API Accessibility
9. Browser Integration
10. OS WebAuthn Integration
11. Authenticated Secure-link Profile
12. Remote-origin Threat Analysis
13. User Experience
14. Automatic Fallback
15. Transport Identifier
16. Certification / Conformance Testing

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

# Appendix A — Architecture Summary

APWはCTAPとPhysical Connectivityの間にStable Abstractionを配置する。

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

重要なのは、

- Wire
- Connector
- Bus
- Radio

の種類ではない。

重要なのは、

> **Platformが十分に制御可能なBidirectional Audio Endpointを提供できるか**

である。

それが可能であればAPW Linkを構築できる可能性がある。

---

# Appendix B — Core Proposition

```text
多くのSystemは既にAudio Deviceを理解している。

例えば、

    Headset Output
    Microphone Input
    USB Audio
    Bluetooth Audio
    Virtual Audio

を理解できる。


一方で、

    New Peripheral Bus上のCTAP
    Vendor-specific Authentication Hardware
    Generic USB Forwarding
    Generic Bluetooth Forwarding

は理解できない、または許可されない場合がある。


したがって、

    Existing Audio-device Support

をAPWのDeployment Substrateとして利用できる。


Edge Platformは、

    「このAudio EndpointがFIDO Authenticatorである」

ことを理解する必要すらない。


Edge Platformに必要なのは、

    Audio Pathを公開すること

だけである。


APW SoftwareがそのAudio Pathを、

    CTAP2用Reliable Logical Wire

として解釈する。


その結果、

    Bidirectional Audioを既にSupportするPlatform

は、

    Native Peripheral APIだけを見た場合よりも、
    External FIDO2 Authenticator Supportへ
    実は近い可能性がある。
```

---

# Appendix C — Deployment and Commercial Rationale

> **非規範（Non-Normative）**

本Appendixは、APWの導入上の合理性および潜在的な事業価値を整理するものである。  
Protocol Conformance Requirementを定義するものではない。

## C.1 なぜAudioが戦略的に有用なのか

APWの価値は、AudioがUSB、NFC、BLEその他のNative FIDO Transportより技術的に優れていることではない。

価値は、

**専用Authenticator Transportを公開・許可・ForwardしていないSystemであっても、Audio Supportだけは既に広くDeployされている**

点にある。

多くのPlatformは既に以下をSupportしている。

```text
Audio Playback
Microphone Capture
USB Audio Class
Headset Interface
Bluetooth Audio
Virtual Audio
Remote Audio Redirection
```

一方、以下は未実装、制限、または製品依存である場合がある。

```text
External FIDO USB HID
Generic USB Device Redirection
BLE Authenticator Forwarding
NFC Reader Access
Vendor-specific Peripheral Driver
Remote WebAuthn / FIDO Virtual Channel
```

APWは、このDeployment上の非対称性をInteroperability上の利点へ変換することを目的とする。

基本命題は以下である。

```text
Existing Audio Support
        +
APW Software
        +
CTAP Integration
        =
Potential Strong-Authentication Interface
```

重要なのは、CTAPを搬送するためだけに、新しいPhysical Peripheral CategoryをPlatformへ追加する必要を減らせる可能性があることである。

---

## C.2 「Driverless」は「Software不要」ではない

APWの最も強いDeployment Propertyの一つは、既存のAudio Device Class / Profileを再利用できる可能性である。

例えば、

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

または、

```text
APW Authenticator
        |
        | Bluetooth Audio
        v
Existing Headset / Audio Stack
```

として実装できる。

Platformが互換性のあるStandard Audio Driverを既に持つ場合、

**APW専用Device Driverを新たに必要としない。**

これにより、以下を新規導入せずに済む可能性がある。

- Kernel-mode Peripheral Driver
- 新しいUSB Device Class
- 新しいBluetooth Peripheral Profile
- Vendor-specific Transport Driver
- Edge側の新しいDevice-redirection Module
- Host Platformごとの専用Hardware Enablement

ただし、

**APW専用Driver不要 = APW Software不要**

ではない。

Host側には依然として、

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

が必要である。

したがって、より正確にはAPWの利点は、

> **新しいPeripheral Enablementの問題を、User-space / Application-level Protocol Integrationの問題へ移動できる可能性がある**

ことにある。

Managed System、Embedded Platform、Appliance、長期運用端末では、この違いがCertification、Deployment、Maintenance、OS Compatibility Costを大きく下げる可能性がある。

---

## C.3 既存Platform Qualificationの再利用

新しいPeripheral Classを導入する場合、以下のような追加作業が発生し得る。

- Driver Development
- Driver Signing
- Kernel Integration
- OS Compatibility Test
- Device Certification
- Security Review
- Endpoint-control Policy変更
- Remote-device Forwarding対応
- Long-term Maintenance

一方、Platform側では既に以下に対するQualificationやPolicyが成熟している場合がある。

```text
Headset
Microphone
USB Audio Device
Bluetooth Audio Device
Virtual Audio Device
```

APWは、この既存Qualification Boundaryを再利用できる可能性がある。

もちろん、APWそのものの評価は別途必要である。

しかしFIDO Authenticatorを接続するためだけに導入するSystem Component数を減らせる点に意味がある。

---

## C.4 VDI / Thin Client Deployment Model

VDIは、APWにとって最も直接的なInitial Use Caseである。

典型的なPolicyでは、

```text
Keyboard / Mouse       Allowed
Display                Allowed
Audio Playback         Allowed
Microphone Capture     Allowed

Generic USB Redirect   Restricted
External Storage       Restricted
Unknown HID Devices    Restricted
```

という構成があり得る。

この場合APWは、

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

という経路で動作できる可能性がある。

この方式では、VDI Protocol自身にFIDO専用Virtual Channelを追加する必要がない。

そのため、異なるRemote-access Technology間で利用可能なFallback / Portability Layerとなる可能性がある。

APWの価値はNative FIDO Redirectionより必ず優れていることではない。

以下のようなEnvironmentで価値が出る。

- Native FIDO Redirectionが存在しない
- VendorごとにFIDO Redirection実装が異なる
- Zero ClientがAudioは公開するがGeneric USBは公開しない
- Local Device DriverをInstallできない
- Legacy Remote-access Productを維持する必要がある
- 複数Remote-display Protocolで同一Authenticator Integrationを利用したい

---

## C.5 Closed / Constrained Platform

APWは、Standard Audio Deviceは公開するがPeripheral Extensibilityが小さいSystemでも有用になり得る。

例:

```text
Game Console
Smart TV
Kiosk
Point-of-Service Terminal
Industrial HMI
Embedded Appliance
Shared Terminal
Media Device
Special-Purpose Console
```

これらのPlatformでは、製品機能上Audioが必須であるため、成熟したAudio Stackが存在する場合がある。

一方、

- General-purpose FIDO Roaming-authenticator Interface
- Third-party USB HID Extension
- Programmable BLE Security-key Stack
- Vendor-neutral External-authenticator API

などは提供されていない可能性がある。

ApplicationまたはPlatform SoftwareがAudio Streamへアクセスできるならば、APWはそのPlatform向けに新しいPhysical Device Profileを標準化する前に、Strong External Authenticationを追加する経路となり得る。

重要な考え方は以下である。

```text
PlatformはPhysical Deviceを
AUDIOとして既に理解している。

APW SoftwareがそのAudio Streamに
CTAP Transportとしての意味を与える。
```

これにより、Hardware Integration ProblemをSoftware Integration Problemへ縮小できる可能性がある。

---

## C.6 Game Console / Consumer Appliance Opportunity

Consumer Platform上のAccountは、今後ますます大きな価値を持つ。

例:

- Stored Payment Credential
- Digital Purchase
- Cloud Save
- Subscription Entitlement
- Family Account
- Parental-control Authority
- Virtual Goods
- Tournament Identity
- Developer Credential
- Account-recovery Authority

したがって、Strong Authenticationの用途は単純なLoginだけではない。

APWによって実現可能性があるUse Caseとして以下が考えられる。

```text
Account Sign-In
Purchase Re-Authentication
Parental Approval
Account Recovery
High-Risk Settings Change
Developer / Administrative Mode
Tournament Identity
Shared-Device Account Selection
```

User Interactionの一例は以下である。

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

Integration上の大きな利点は、Physical AuthenticatorがPlatformにとって既にSupportedな、

- Standard USB Audio Device
- Headset-class Audio Endpoint

として見える可能性があることである。

ただし、Platform / Application側の協力なしにConsole Loginそのものを実現することはできない。

したがって、Locked Consumer Consoleにおける現実的なCommercialization Pathは以下となる可能性が高い。

- Platform Vendor Integration
- OEM Integration
- Approved Application SDK
- Middleware Licensing
- Platform Security Frameworkへの組込み

---

## C.7 Kiosk / Shared Terminal / Regulated Environment

APWは、Strong Authenticationを必要とする一方、Local Peripheral Capabilityを意図的に最小化しているEnvironmentで特に有用となる可能性がある。

例:

- Public Kiosk
- Factory Terminal
- Medical Workstation
- Call-center Thin Client
- Shared Engineering Terminal
- Education Terminal
- Controlled-access Console
- Regulated VDI Environment

このようなEnvironmentでは、Generic Peripheral Forwardingより、用途限定Authentication Transportの方がSecurity Review上説明しやすい可能性がある。

例えば以下のPolicy Modelを構成できる。

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

この区別は、Endpoint ConnectivityにPrinciple of Least Privilegeを適用するOrganizationにとって意味を持ち得る。

---

## C.8 Productization Model

APWは単一のProduct形態に限定されない。

### C.8.1 Authenticator Hardware

Physical APW-capable Roaming Authenticator。

```text
Secure Element / Protected Key Store
CTAP2 Authenticator
APW Protocol Engine
Software Modem / DSP
Audio Codec
TRRS / USB Audio / Bluetooth Audio Interface
```

### C.8.2 APW SDK

Host / Platform Vendor向けSoftware Package。

```text
APW Modem Library
APW Framing / ARQ
CTAP Transport Adapter
Audio Endpoint Abstraction
Device Discovery Logic
Reference Integration Code
```

### C.8.3 Embedded IP / Firmware License

以下への組込み。

- Thin-client Firmware
- Zero-client Firmware
- Game-console OS
- Kiosk Platform
- Smart Display
- Embedded Linux System
- Proprietary Appliance OS

### C.8.4 Conformance / Qualification Tooling

Production Deploymentには以下のTest Toolが有用である。

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

Client ImageとAuthenticator FleetをOrganization自身がControlできるEnvironment向けMiddlewareとして提供することも可能である。

---

## C.9 新規Device Driverを避けることのCommercial Value

多くのPlatformにとって、新しいDevice Driverは単なるImplementation Detailではない。

以下のRecurring Costを生み得る。

- Security Certification
- Code Signing
- Compatibility Validation
- Endpoint-management Policy
- Software Distribution
- Patch Maintenance
- OS Upgrade対応
- Incident Response
- Device Inventory
- Remote-session Forwarding
- Customer Support

APWが既存Audio Device Modelを再利用できるならば、この経済価値はModem技術そのものとは独立して存在する。

Commercial Propositionは以下のように要約できる。

> **新しいPeripheral Classを導入せずに、新しいAuthentication Transportを導入する。**

Implementation観点では、

> **Platformが既に持つAudio Hardware-enablement Boundaryを再利用し、その上へCTAP Semanticsを追加する。**

となる。

これは、Underlying SignalがAudioであるという事実以上に重要な可能性がある。

---

## C.10 Integration BoundaryそのものがBusiness Advantageになる

Native Hardware Transportを追加する場合、複数LayerへのSupportが必要になる場合がある。

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

APWは、これらの前半を既存Audio Stackとして再利用することを狙う。

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

これによりInnovation PointをStack上位へ移動できる。

実務上は、System全Layerで新しいPeripheral TypeをSupportしてもらう前に、新しいAuthentication BehaviorをPrototype / Deployできる可能性がある。

---

## C.11 Deployment ReachとNative EfficiencyのTradeoff

APWは、利用可能かつ許可されているNative FIDO Transportを置き換えることを目的としない。

Native USB HID、NFC、BLE、Platform-integrated WebAuthn Redirectionには一般に以下の利点がある。

- Lower Latency
- Better Power Efficiency
- Better Discovery
- Standardized Security Semantics
- Mature Certification
- Better User Experience

APWの比較優位は、

**Deployment Reach**

にある。

したがってTradeoffは以下となる。

```text
Native Transport
    -> Better efficiency and native integration

APW
    -> Potentially broader compatibility using
       already-deployed audio infrastructure
```

実用Systemでは双方をSupportしてもよい。

---

## C.12 推奨Initial Market Sequence

現実的なDeployment Sequenceの一例を以下に示す。

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

この順序は、初期段階ではImplementer自身がSoftware StackをControlできるEnvironmentから開始する。

Basic Transport Valueを証明する前にPlatform Vendor Cooperationを必須にしないことが目的である。

---

## C.13 Reference Commercial Proposition

非Protocol関係者向けには、APWを以下のように表現できる。

> **Audio Pseudowireは、既にSupportされているBidirectional Audio Endpointを、用途限定されたFIDO2 Strong Authentication Transportへ変換する。**

より具体的には、

> **Platformが制御可能なMicrophone InputとAudio Outputを既に公開できるなら、新しいPeripheral Device ClassやAPW専用Device Driverを追加せずにExternal FIDO2 Authenticatorを統合できる可能性がある。**

その価値はAudioという新奇性ではなく、

**Reuse**

にある。

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

## C.14 Business / Security Boundary

APWのCommercial Valueは、そのScopeを狭く維持することに依存する。

APWがGeneral-purpose Data Tunnelへ拡張されると、運用・Security上の利点の多くが弱くなる。

したがってProduction APW Profileは、Authentication-related Communicationへ意図的に限定されることが望ましい。

Platform Vendor、Security Reviewer、Enterprise Administratorへは、以下のように説明できることが重要である。

> APWはNetwork Interfaceではない。

> APWはGeneric USB Tunnelではない。

> APWはExisting Audio Abstraction上に実装されるPurpose-limited CTAP Transportである。

この区別はTechnical Architectureだけでなく、Potential Business Caseにおいても中心的である。
