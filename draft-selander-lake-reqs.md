---
title: Requirements for a Lightweight AKE for OSCORE
abbrev: Reqs-LAKE-for-OSCORE
docname: draft-selander-lake-reqs-latest

ipr: trust200902
cat: info

coding: utf-8
pi: # can use array (if all yes) or hash here
  toc: yes
  sortrefs: yes
  symrefs: yes
  tocdepth: 2

author:
      -
        ins: M. Vucinic
        name: Malisa Vucinic
        org: INRIA
        email: malisa.vucinic@inria.fr
      -
        ins: G. Selander
        name: Göran Selander
        org: Ericsson AB
        email: goran.selander@ericsson.com
      -
        ins: J. Mattsson
        name: John Mattsson
        org: Ericsson AB
        email: john.mattsson@ericsson.com


normative:


informative:

  RFC7228:
  RFC8613:
  I-D.ietf-6tisch-minimal-security:
  I-D.ietf-lpwan-coap-static-context-hc:
  I-D.ietf-cose-x509:

  AKE-for-6TiSCH:
    target: https://docs.google.com/document/d/1wLoIexMLG3U9iYO5hzGzKjkvi-VDndQBbYRNsMUlh-k
    title: AKE for 6TiSCH
    date: March 2019
    
  AKE-for-NB-IoT:
    target: https://github.com/EricssonResearch/EDHOC/blob/master/docs/NB%20IoT%20power%20consumption.xlsx
    title: AKE for NB-IoT
    date: March 2019

  HKDF:
    target: https://eprint.iacr.org/2010/264.pdf
    title: "Cryptographic Extraction and Key Derivation: The HKDF Scheme"
    author:
      -
        ins: H. Krawczyk
    date: May 2010


  LwM2M:
    target: http://www.openmobilealliance.org/release/LightweightM2M/V1_1-20180710-A/OMA-TS-LightweightM2M_Transport-V1_1-20180710-A.pdf
    title: OMA SpecWorks LwM2M
    date: August 2018

  Fairhair:
    target: https://www.fairhair-alliance.org/data/downloadables/1/9/fairhair_security_wp_march-2018.pdf
    title: Security Architecture for the Internet of Things (IoT) in Commercial Buildings, Fairhair Alliance white paper
    date: March 2018

  LoRaWAN:
    target: https://lora-alliance.org/resource-hub/lorawantm-regional-parameters-v102rb
    title: LoRaWAN Regional Parameters v1.0.2rB
    date: Feb 2017

--- abstract

This document compiles the requirements for a lightweight authenticated key exchange protocol for OSCORE.

--- middle

# Introduction  {#intro}

OSCORE {{RFC8613}} is a lightweight communication security protocol providing end-to-end security on application layer for constrained IoT settings (cf. {{RFC7228}}). OSCORE lacks a matching authenticated key exchange protocol (AKE).

To ensure that the AKE is efficient for the expected applications of OSCORE, we list the relevant public specifications of technologies where OSCORE is included:

* The IETF 6TiSCH WG charter (-02) identifies the need to "secur\[e\] the join process and mak\[e\] that fit within the constraints of high latency, low throughput and small frame sizes that characterize IEEE802.15.4 TSCH". OSCORE protects the join protocol as described in 6TiSCH Minimal Security {{I-D.ietf-6tisch-minimal-security}}.

* The IETF LPWAN WG charter (-01) identifies the need to improve the transport capabilities of LPWA networks such as NB-IoT and LoRa whose "common traits include ... frame sizes ... \[on\] the order of tens of bytes transmitted a few times per day at ultra-low speeds". The application of OSCORE is described in {{I-D.ietf-lpwan-coap-static-context-hc}}.

* OMA Specworks LwM2M version 1.1 {{LwM2M}} defines bindings to two challenging radio technologies where OSCORE will be deployed: LoRaWAN and NB-IoT.

Other industry fora which plan to use OSCORE:

* Fairhair Alliance has defined an architecture {{Fairhair}} which adopts OSCORE for multicast, but it is not clear whether the architecture will support unicast OSCORE.

* Open Connectivity Foundation (OCF) has been actively involved in the OSCORE development for the purpose of deploying OSCORE, but no public reference is available since OCF only references RFCs. We believe that these OSCORE consumers reflect similar levels of constraints on the devices and networks in question.

This document compiles the requirements for the AKE for OSCORE.
It summarizes the security requirements that are expected from such an AKE, as well as the main characteristics of the environments where the solution is envisioned to be deployed.
The solution will presumably be useful in other scenarios as well since a low security overhead improves the overall performance.

# Problem description {#prob-desc}

## Credentials

IoT deployments differ in terms of what credentials can be supported. Currently many systems use pre-shared keys (PSK) provisioned out of band, for various reasons. PSK are often used in a first deployment because of its perceived simplicity. The use of PSK allows for protection of communication without major additional security processing, and also enables the use of symmetric crypto algorithms only, reducing the implementation and computational effort in the endpoints.

However, PSK-based provisioning has inherent weaknesses. There has been reports of massive breaches of PSK provisioning systems, and as many systems use PSK without perfect forward secrecy (PFS) they are vulnerable to passive pervasive monitoring. The security of these systems can be improved by adding PFS through an AKE authenticated by the provisioned PSK.

Shared keys can alternatively be established in the endpoints using an AKE protocol authenticated with asymmetric public keys instead of symmetric secret keys. Raw public keys (RPK) can be provisioned with the same scheme as PSKs, which allows a more relaxed trust model since RPKs need not be secret.

As a third option, by running the same asymmetric key AKE with public key certificates instead of RPK, key provisioning can be omitted, leading to a more automated bootstrapping procedure.

These steps provide an example of a migration path in limited scoped steps from simple to more robust security bootstrapping and provisioning schemes where each step improves the overall security and/or simplicity of deployment of the IoT system, although not all steps are necessarily feasible for the most constrained settings.

In order to allow for these different schemes, the AKE must support PSK, RPK and certificate-based authentication.

Bandwidth is a scarce resource in constrained-node networks.
To minimize the bandwidth consumption it is therefore desirable to support transporting the certificates by reference rather than by value.
Considering the wide variety of deployments the AKE must support different schemes for transporting and identifying credentials, see Section 2 of {{I-D.ietf-cose-x509}}.

The common lack of a user interface in constrained devices leads to various credential provisioning schemes.
The use of RPKs may be appropriate for the authentication of the AKE initiator but not for the AKE responder.
The AKE must support different credentials for authentication in different directions of the AKE run, e.g. certificate-based authentication for the initiating endpoint and RPK-based authentication for the responding endpoint.

## Identity Protection

Transporting identities as part of the AKE run is a necessity in order to provide strong mutual authentication. In the case of constrained devices, the identity may contain sensitive information on the manufacturer of the device, the batch, default firmware version, etc. Protecting the identities from passive and active attacks is important from the privacy point of view.

The AKE is required to support identity protection of one of the peers in the AKE run in the case of public key identities, or the protection of the PSK identifier in the case of PSK-based authentication.

## Crypto Agility

Motivated by long deployment lifetimes, the AKE is required to support crypto agility, including modularity of COSE crypto algorithms and negotiation of preferred crypto algorithms for OSCORE and the AKE. The AKE negotiation must be protected against downgrade attacks.

## AKE for OSCORE {#AKE-OSCORE}

In order to at all be suitable for OSCORE, at the end of the AKE protocol run the two parties must agree on (see Section 3.2 of {{RFC8613}}):

* a shared secret (OSCORE Master Secret) with PFS and a good amount of randomness. (The term "good amount of randomness" is borrowed from {{HKDF}} to signify not necessarily uniformly distributed randomness.)

* identifiers providing a hint to the receiver of what security context to use when decrypting the message (OSCORE Sender IDs of peer endpoints), arbitrarily short

* COSE algorithms to use with OSCORE

Moreover, the AKE must support the same transport as OSCORE, in particular any protocol where CoAP can be transported.


##Lightweight {#lw}
 
As motivated in {{AKE-OSCORE}} we target an AKE which is efficiently deployable in 6TiSCH multi-hop networks, LoRaWAN networks and NB-IoT networks. The desire is to optimize the AKE to be 'as lightweight as reasonably achievable' in these environments, where 'lightweight' refers to:

* resource consumption, measured by bytes on the wire, wall-clock time and number of round trips to complete, or power consumption
* the amount of new code required on end systems which already have an
OSCORE stack

These properties need to be considered in the context of the use of an existing CoAP/OSCORE stack in the targeted networks. However, some properties may be difficult to evaluate for a given protocol, for example, because they depend on the radio conditions or other simultaneous network traffic. Therefore these properties should be taken as input for identifying plausible protocol metrics that can be more easily measured and compared between protocols.

Per 'bytes on the wire', it is desirable for these AKE messages to fit into the MTU size of these protocols; and if not possible, within as few frames as possible, since using multiple MTUs can have significant costs in terms of time and power.

Per 'time', it is desirable for the AKE message exchange(s) to complete in a reasonable amount of time, both for a single uncongested exchange and when multiple exchanges are running in an interleaved fashion, like e.g. in a "network formation" setting when multiple devices connect for the first time. This latency may not be a linear function depending on congestion and the specific radio technology used. As these are relatively low data rate networks, the latency contribution due to computation is in general not expected to be dominant.

Per 'round-trips', it is desirable that the number of completed request/response message exchanges required before the initiating endpoint can start sending protected traffic data is as small as possible, since this reduces completion time. See {{disc}} for a discussion about the tradeoff between message size and number of messages.

Per 'power', it is desirable for the transmission of AKE messages and crypto to draw as little power as possible. The best mechanism for doing so differs across radio technologies.  For example, NB-IoT uses licensed spectrum and thus can transmit at higher power to improve coverage, making the transmitted byte count relatively more important than for other radio technologies.  In other cases, the radio transmitter will be active for a full MTU frame regardless of how much of the frame is occupied by message content, which makes the byte count less sensitive for the power consumption.  Increased power consumption is unavoidable in poor network conditions, such as most wide-area settings including LoRaWAN.

Per 'new code', it is desirable to introduce as little new code as possible onto OSCORE-enabled devices to support this new AKE. These devices have on the order of 10s of kB of memory and 100 kB of storage on which an embedded OS; a COAP stack; CORE and AKE libraries; and target applications would run. It is expected that the majority of this space is available for actual application logic, as opposed to the support libraries. In a typical OSCORE implementation COSE encrypt and signature structures will be available, as will support for COSE algorithms relevant for IoT enabling the same algorithms as is used for OSCORE (e.g. COSE algorithm no. 10 = CCM* used by 6TiSCH). The use of those, or CBOR or CoAP, would not add to the footprint.

While the large variety of settings and capabilities of the devices and networks makes it challenging to produce exact values of some these dimensions, there are some key benchmarks that are tractable for security protocol engineering and which have a significant impact.


### LoRaWAN

LoRaWAN employs unlicensed radio frequency bands in the 868MHz ISM band, in Europe regulated by ETSI EN 300 220. For LoRaWAN the most relevant metric is the Time-on-Air, which determines the back-off times and can be used an indicator to calculate energy consumption. LoRaWAN is legally required to use a 1% (or smaller) duty cycle, a payload split into two fragments instead of one increases the time to complete the sending of this payload by at least 10,000%. The use of an AKE for providing end-to-end security on application layer need to comply with the duty cycle. One relevant benchmark is performance in low coverage with Data Rates 0-2 corresponding to a packet size of 51 bytes {{LoRaWAN}}. While larger frame sizes are also defined, their use depend on good radio conditions. Some libraries/providers only support 51 bytes packet size.

#### Bytes on the wire


#### Time



#### Round trips and number of messages



#### Power



### 6TiSCH

6TiSCH operates in the 2.4 GHz unlicensed frequency band and uses hybrid Time Division/Frequency Division multiple access (TDMA/FDMA).
Nodes in a 6TiSCH network form a mesh.
The basic unit of communication, a cell, is uniquely defined by its time and frequency offset in the communication schedule matrix.
Cells can be assigned for communication to a pair of nodes in the mesh and so be collision-free, or shared by multiple nodes, for example during network formation.
In case of shared cells, some collision-resolution scheme such as slotted-Aloha is employed.
Nodes exchange frames which are at most 127-bytes long, including the link-layer headers.
To preserve energy, the schedule is typically computed in such a way that nodes switch on their radio below 1% of the time ("radio duty cycle").
A 6TiSCH mesh can be several hops deep.
In typical use cases considered by the 6TiSCH working group, a network that is 2-4 hops deep is commonplace; a network which is more than 8 hops deep is not common.

#### Bytes on the wire

Increasing the number of bytes on the wire in a protocol message has an important effect on the 6TiSCH network in case the fragmentation is triggered.
More fragments contribute to congestion of shared cells (and concomitant error rates) in a non-linear way.

The available size for key exchange messages depends on the topology of the network, whether the message is traveling uplink or downlink, and other stack parameters.
A key performance indicator for a 6TiSCH network is "network formation", i.e. the time it takes from switching on all devices, until the last device has executed the AKE and securely joined.
As an example, given the size limit on the frames and taking into account the different headers (including link-layer security), if a 6TiSCH network is 5 hops deep, the maximum CoAP payload size to avoid fragmentation is 47/45 bytes (uplink/downlink) {{AKE-for-6TiSCH}}.

#### Time

Given the slotted nature of 6TiSCH, the number of bytes in a frame has insignificant impact on latency, but the number of frames has.
The relevant metric for studying AKE is the network formation time, which implies parallel AKE runs among nodes that are attempting to join the network.
Network formation time directly affects the time installers need to spend on site at deployment time.

#### Round trips and number of messages

Given the mesh nature of the 6TiSCH network, and given that each message may travel several hops before reaching its destination, it is highly desirable to minimize the number of round trips to reduce latency.

#### Power

From the power consumption point of view, it is more favorable to send a small number of large frames than a larger number of short frames.


### NB-IoT {#nb-iot}

3GPP has specified Narrow-Band IoT (NB-IoT) for support of infrequent data transmission via user plane and via control plane. NB-IoT is built on cellular licensed spectrum at low data rates for the purpose of supporting:

* operations in extreme coverage conditions,
* device battery life of 10 years or more,
* low device complexity and cost, and
* a high system capacity of thousands of connected devices per square kilometer.

NB-IoT achieves these design objectives by:

* Reduced base band processing, memory and RF enabling low complexity device implementation. 
* A lightweight setup minimizing control signaling overhead to optimize power consumption.
* In-band, guard-band, and stand-alone deployment enabling efficient use of spectrum and network infrastructure.


#### Bytes on the wire {#nbiot-bytes} 

The number of bytes on the wire in a protocol message has a direct effect on the performance for NB-IoT. In contrast to LoRaWAN and 6TiSCH, the NB-IoT radio bearers are not characterized by a fixed sized PDU. Concatenation, segmentation and reassembly are part of the service provided by the NB-IoT radio layer. As a consequence, the byte count has a measurable impact on time and energy consumption for running the AKE.


#### Time {#nbiot-time} 

Coverage signficantly impacts the available bit rate and thereby the time for transmitting a message, and there is also a difference between downlink and uplink transmissions (see {{nbiot-power}}). The transmission time for the message is essentially proportional to the number of bytes.

Since NB-IoT is operating in licensed spectrum, in contrast to e.g. LoRaWAN, the packets on the radio interface can be transmitted back-to-back, so the time before sending OSCORE protected data is limited by the number of round trips/messages of the AKE.


#### Round trips and number of messages {#nbiot-rtt} 

As indicated in {{nbiot-time}}, the number of messages and round-trips is one limiting factor for protocol completion time.
 

#### Power {#nbiot-power} 

Since NB-IoT is operating in licensed spectrum, the device is allowed to transmit at a high power, which has a large impact on the energy consumption in particular in bad coverage.

The benchmark for NB-IoT energy consumption is based on the same computational model as was used by 3GPP in the design of this radio layer. The device power consumption is assumed to be 500mW for transmission and 80mW for reception. Power consumption for "light sleep" (~ 3mW) and ”deep sleep” (~ 0.015mW) are omitted. The bitrates (uplink/downlink) are assumed to be 28/170 kbps for good coverage and 0,37/2,5 kbps for bad coverage.

The energy consumption benchmark includes RRC Resume procedure for transition from RRC Inactive to RRC Connected, perform operation and returning RRC Inactive, see {{AKE-for-NB-IoT}}. The results show a high per-byte energy consumption for uplink transmissions in particular in bad coverage. Since the application decides about the device being initiator or responder in the AKE, the protocol cannot be tailored for a particular message being uplink or downlink. To perform well in both kind of applications the overall number of bytes of the protocol needs to be as low as possible. 




### Discussion {#disc}

While "as small protocol messages as possible" does not lend itself to a sharp boundary threshold, "as few protocol messages as possible" does and is relevant in all settings above.

The penalty is high for not fitting into the frame sizes of 6TiSCH and LoRaWAN networks. Fragmentation is not defined within these technologies so requires fragmentation scheme on a higher layer in the stack. With fragmentation increases the number of frames per message, each with its associated overhead in terms of power consumption and latency. Additionally the probability for errors increases, which leads to retransmissions of frames or entire messages that in turn increases the power consumption and latency.

There are trade-offs between "few messages" and "few frames"; if overhead is spread out over more messages such that each message fits into a particular frame this may reduce the overall power consumption. While it may be possible to engineer such a solution for a particular radio technology and signature algorithm, the benefits in terms of fewer messages/round trips in general and for NB-IoT in particular (see {{nb-iot}}) are considered more important than optimizing for a specific scenario. Hence an optimal AKE protocol has 3 messages and each message fits into as few frames as possible, ideally 1 frame per message.

The difference between uplink and downlink performance should not be engineered into the protocol since it cannot be assumed that a particular protocol message will be sent uplink or downlink.



# Requirements Summary

* The AKE must support PSK, RPK and certificate based authentication and crypto agility, be 3-pass and support the same transport as OSCORE. It is desirable to support different schemes for transporting and identifying credentials.

* After the AKE run, the peers must be mutually authenticated, agree on a shared secret with PFS and good amount of randomness, peer identifiers (potentially short), and COSE algorithms to use.

* The AKE must reuse CBOR, CoAP and COSE primitives and algorithms for low code complexity of a combined OSCORE and AKE implementation.

* The messages must be as small as reasonably achievable and fit into as few LoRaWAN packets and 6TiSCH frames as possible, optimally 1 for each message.


# Security Considerations  {#sec-cons}

This document compiles the requirements for an AKE and provides some related security considerations.

The AKE must provide the security properties expected of IETF protocols, e.g., providing confidentiality protection, integrity protection, and authentication with strong work factor.

# IANA Considerations  {#iana}

None.

--- back



