# Midterm Report

>Note: Following is a printed version of our Midterm report. For better visualization, please visit our documentation website

[Onion Project Documentation Website](https://kandelak.github.io/onion-book/reports/midterm_report.html) 


## Changes To  the Initial Report
There were no changes made.
## Architecture of our Module

### High Level Overview 

## API vs P2P
As described in the specification, there are 2 types of communications: **API** and **P2P** communication. API communication is used to facilitate information exchange between different modules of the same node instance, whereas P2P communication enables different nodes from the P2P netweork to communicate. The following graph visualizes the above-mentioned concept.

```mermaid
---
title: API vs P2P
---
flowchart LR
    o_1 -.P2P.- o_2
    g_3 -.P2P.- g_2
    o_1 -.P2P.- o_3
    d_3 -.P2P.- d_2
    subgraph Peer1
    g_1[Gossip] 
    d_1[DHT]
    o_1[Onion]
    r_1[RPS]
    u_1[CM/UI]

    u_1 -.API.- o_1
    g_1 & r_1 & d_1 -.API.- o_1
    g_1 & r_1 & d_1 -.API.- u_1
    g_1 -.API.- r_1
    end
    subgraph Peer2
    g_2[Gossip]
    d_2[DHT] 
    o_2[Onion]
    r_2[RPS]
    u_2[CM/UI]
    end
    subgraph Peer3 
    g_3[Gossip]
    d_3[DHT] 
    o_3[Onion]
    r_3[RPS]
    u_3[CM/UI]
    end
```
### Logical Structure (Classes/Specs)

---
|Name|Functionality| Fields/Methods |
| --- | --- | --- |  
| Onion Module (OM) | Construct tunnels and execute main functionality | run(), traffic, connection, config
| Tunnel | Onion Tunnel | Peers (Hops), round, hopCount, config, build(), extend()
|Traffic| Data transmitted trough tunnels, cover or real| isTraffic, packetSize, createCoverTraffic|
| Connection | Represents an Open Connection using a Tunnel. Changes Tunnels From Round to Round | peerFrom, peerTo switchTunnel(), ephemeralKeys, socket
|Peer| Represents an Onion Node (Peer) | networkAddress, hostkey, tunnel, round, connections,  
|Hostkey| Represents a Hoskey, i.e Public-private Key Pair | pubKey, privKey
|Ephemeral Key| Represents an ephemeral key created during connection initialization| generate()
|Round| Represents a Round | duration
|Bootstraping Service| Represents a service which is responsible for bootstraping the "genesis" Peer-To-Peer Network (Maybe for testing purposes)
|Config| Config File | addrToHostkeys, roundDuration, packetSize, api_address (where to listen API connections), load()
|P2P_Protocol|Describes P2P Message Formats| |
|API_Protocol|Describes API Message Formats| ONION_TUNNEL_BUILD, etc.|
|Serializer| Wrapper for serializer (Protobuf)| serialize() |

### Graphical Representation of The Logical Structure
```mermaid
---
title: Logical Structure
---
classDiagram
    direction TB
    Tunnel o-- "2 (hopCount)" Peer
    Peer o-- "*" HostKey
    Connection o-- Tunnel
    Connection o-- EphemeralKey
    Tunnel -- Round
    OnionModule --> Connection
    OnionModule o-- Traffic
    OnionModule .. Config
    OnionModule ..ProtocolP2P
    OnionModule ..ProtocolAPI
    class Tunnel{
        +[Peer] hops    
        +int round
        +int hopCount
        +build() Tunnel
    }

    class OnionModule{
        -Config config
    }

    class HostKey{
        +String pubKey
        +String privKey
    }
    class Connection{
        -Peer src
        -Peer dest
        +switchTunnel() Tunnel
        +Tunnel tunnel
        +socket
    }

    class Peer{
        +String networkAddress
        -Hostkey hostkey
    }

    class EphemeralKey{
        +generate() 
    }

    class Config{
        +load()
    }

    class Round{
        +int duration
    }

    class ProtocolP2P{
    }
    class ProtocolAPI{
    }

    class Traffic{
        +String traffic
        +bool isCover
    }

```

### Process Architecture

Our module consists of primarily three threads namely:
- **Round Thread:** Responsible for managing rounds (e.g, building and switching tunnels).
- **API Thread:** Listens for API connections.
- **P2P Thread:** Listens for P2P connections.

In addition to the threads above, the **API Thread** and the **P2P Thread** both will spawn arbitrary threads to handle the established connections.

#### Round Thread

```mermaid
---
title: Round Thread
---
flowchart TD
    A{Time To Build Tunnel?} --> |Yes| B(Build Tunnel)
    A --> |No| AN(Wait 0.25 sec)
    AN --> A
    B --> C{Time To Switch Tunnel?}
    C --> |No| CN(Wait 0.1 sec)
    CN --> C
    C --> |Yes| D(Switch Tunnel)
    D --> A
```

##### Time To Build Tunnel

The keys provided in the following table is to be utilized to interpret the flowchart below.

|Key|Description|
|-|-|
|T<sub>N</sub>|**Time Now:** The current time.|
|T<sub>S</sub>|**Time Start:** The timestamp of the starting of the current round.|
|R<sub>T</sub>|**Round Time:** The duration of a round in seconds. Defined in the config file.|
|T<sub>B</sub>|**Time Build:** The estimated max time elapsed while building a tunnel. Defined in the config file.| 
|T<sub>SW</sub>|**Time Switch:** The estimated max time elapsed while switching a tunnel. Defined in the config file.|

```mermaid
---
title: Time To Build Tunnel
---
flowchart TD
    A{Is 1st Round?} --> |Yes| AY(Return True)
    A --> |No| B{"If (TS + RT - TB) <= TN"}
    B --> |Yes| BY(Return True)
    B --> |No| BN(Return False)
```

##### Build Tunnel

```mermaid
---
title: Build Tunnel
---
flowchart TD
    A("Get 3 nodes from RPS (the last one from global state if possible)") --> B("Key Exchange Basic (Node 1)")
    B --> C("Key Exchange Adv (Node 2)")
    C --> D("Key Exchange Adv (Node 3)")
```

###### Key Exchange

The keys provided in the following table is to be utilized to interpret the sequence diagram below.

|Key|Description|Example|
|-|-|-|
|P<sub>X</sub>|Public key of node `X`.|-|
|[C]K|Content `C` encrypted by the key `K`.|[X]P<sub>A</sub>: Content `X` encrypted by the public key of node `A` |
|K<sub>AB</sub>|Ephemeral key for nodes `A` and `B`|-|

```mermaid
---
title: Key Exchange Basic
---
sequenceDiagram
    A->>B: [X]PB
    B->>A: [Y]PA
    A->B: K(AB)
```

```mermaid
---
title: Key Exchange Adv
---
sequenceDiagram
    A->>B: [[X]PC]K(AB)
    B->>C: [X]PC
    C->>B: [[Y]PA]PB
    B->>A: [Y]PA
```

> Note: The `Key Exchange Adv` process is extended with the same logic for additional nodes.

##### Time To Switch Tunnel
```mermaid
---
title: Time To Switch Tunnel
---
flowchart TD
    A{Is 1st Round?} --> |Yes| AY(Return True)
    A --> |No| B{"If (TS + RT - TSW) <= TN"}
    B --> |Yes| BY(Return True)
    B --> |No| BN(Wait 0.1 sec)
    BN --> B
```

##### Switch Tunnel

```mermaid
---
title: Switch Tunnel
---
flowchart TD
    A(Destroy Current Tunnel) --> B("Destroy Tunnel Message (Node 3)")
    B --> BD{"Tunnel Destroyed Ack (Node 3)?"}
    BD --> |No| BDN(Wait 0.5 sec)
    BDN --> B
    BD --> |Yes| C("Destroy Tunnel Message (Node 2)")
    C --> CD{"Tunnel Destroyed Ack (Node 2)?"}
    CD --> |No| CDN(Wait 0.5 sec)
    CDN --> C
    CD --> |Yes| D("Destroy Tunnel Message (Node 1)")
    D --> DD{"Tunnel Destroyed Ack (Node 1)?"}
    DD --> |No| DDN(Wait 0.5 sec)
    DDN --> D
    DD --> |Yes| E(Free Resources & Reset State)
```

## Security Measures

Whenever designing a Peer-To-Peer architecture, modelling adversaries inside the network is a major part of a
good decision-making process. We would like to make strict adversary assumption to ensure high security. However, there is always trade-Off between usability and security. We keep that in mind as well and make our implementation generic so that the users can decide what they prefer. In order to underline what we think facilitates high security standards, we 
introduce couple of terms as described in the [RFC2119](https://link-url-here.orghttps://datatracker.ietf.org/doc/html/rfc2119): **MUST**, **MUST NOT**, **SHOULD**. 

These are the security measures we want to take in our Project: 

---
|Goal|Why|Measure|
|---|---|---|
|Encryption, Authentication & Integrity|Without Encryption data is transmitted as a plaintext. If Alice is talking to Bob, any other party is able to eavesdrop the communication. Without Authentication malicious peers could impersonate others. In our context: If Malory is able to impersonate Bob and call Alice, Alice will think that she is talking with Bob but in reality she is talking with Malory. Without integrity, messages can be be tampered without noticing. | We'll use [tls package](https://pkg.go.dev/crypto/tls) from the standard Crypto Library of Golang that acomplishes encryption, authentication and integrity. We'll define TLS Version 1.3 as our default choice to mitigate securiy risks related to older versions such as weak Ellitic Curves. However if weak algorithms and Cipher Suites known for TLS Version 1.2 is avoided, TLS 1.2 works as well. Therefore we'll give our users option to choose between those 2. Our recommention: Users **SHOULD** go for the default version, i.e TLS 1.3. We may have to only partly use the tls package in combination with our P2P Protocol but we'll finalize our thoughts about that in our final report.|
|Forward Secrecy|Without forward secrecy leaked secrets could lead to leaked information. For instance, if ephemeral keys are transmitted through channels and that channel is compromised, an attacker could use ephemeral keys to decrypt data encrypted with those. |Diffie-Hellman (Elliptic Curve) Key Exchange for Ephemeral Key Formation which is part of the TLS. In the case of Diffie-Hellman, secrets aren't transmitted through channels but only paramters for computing secrets. However, computing the key using those parameters depends on the discrete logarithm problem which is computationally infeasible to solve. Since Elliptic Curve based schemes have stronger security per bit, this means that we can achieve same security level as in the case of RSA with much less data transmitted over the network. In our context, this will enhance call quality, since formation of the secure channel over which Alice and Bob will communicate, will be much faster.|
|Client & Server Authentication| In the case of standard telephony, identities are mapped with phone numbers. In our case, i.e IP-Telephony, phone numbers are replaced with IP addresses. However, IP addresses can be spoofed. Therefore authentication in both directions is detrimental. |Whereas server authentication is a stardard, client Authentication is almost always regarded optional nowadays. However, we think that in Peer-To-Peer architecture and especially in our context, this should be encouraged to ensure higher security standards. Therefore our default configuration will authenticate Clients as well (Source Onion Module). Our users **SHOULD** use the option for client authentication. Again, the trade-off would be clients not to authenticate themselves and for that the call will happen much faster.|
|Certificates and Secure Bootstraping |Proving identity is of major importance in any network infrustructure. TLS uses PKI (Public Key Infrustructure) to map identities to public keys. In our case, these are *Hostkeys*. However, we want to discourage PKI in our network since PKIs are realized through trusted third parties. Despite the fact that PKIs are considered secure through utilizing CT (Certificate Transparency) and many more mature mechanisms, we still think that avoiding trusted parties should always be the goal of a Peer-To-Peer.| We would like to issue certificates using Peer-To-Peer network only. For simplicity purposes, we would define some peers as trusted and they will be able to issue certificates for new joiners. However, we discuss more sophisticated approaches in the [Future Work](/src/reports/midterm_report.md#future-work) section.|
|Minimize Space for Traffic Analysis|[Latest Research](https://arxiv.org/abs/1808.07285) shows that modern learning algorithms can learn correlation function for Tor's network, which is an implementaion of Onion Routing just like ours. Their approach provides 96% flow correlation accuracy which reuslts in deanonymized connections.|  One method to minimize the attack surface is to make data flow as homogenous and non-differantiable as possible. This can be done through choosing fixed package sizes. However, to ensure higher degree of security, users **SHOULD** use VPN connections but in a [right order](https://old.reddit.com/r/TOR/comments/rzxuex/in_what_circumstances_would_using_tor_over_vpn_be/hryfo3k/). |
|Limit Sybill Attacks|Sybil Attacks mostly occur when fake identities are being created to gain large influence on the network. This could result in degraded network performance. In our context: If Alice wants to call Bob and chooses relays (one of them or all) from a sybill sub-network, the call quality can be affected since sybill group memebers can decide not to act as hops or not to redirect the traffic to other hops. The probability of this vulnerability to happen increases with increased number of fake identities. |Proof of Work: Creating new identities will require solving hard puzzles resulting in infeasibility of issueing multiple fake identities. In our implementation, the original identities (hostkey) that are present in the very first network are considered valid. If new peers want to join the network, they have to register themselves to one of the peers that are present in the network. Registering means solving a puzzle acquired from trusted peers in the network and signing them with the new identity. |
|DDoS Protection|A malicios actor in our network could call multiple Onion Modules to congest the network and execute a DDoS attack. For instance, Malory (Malicios Actor) calls every participant in the network. Alice wants to call Bob, but Bob is talking with Malory and the communication between Alice and Bob can't happen anymore | We'll apply rate limiting, hence a peer is able to call only limited amount of participants (or only one) in the network at the same time. Monitoring if every peer adheres to this rule can be done through Gossip Protocol. |

## Specification

### API Specification

> Note 1: All object types listed in the APIs below are defined in the original specification document.
> Note 2: The specified `Response Object Type` is for success cases, in the case of failure `ONION ERROR` is sent.

#### Incoming API Requests
|#|Name|Request Object Type|Response Object Type|Description|
|-|-|-|-|-|
|1.1|Build Onion Tunnel|`ONION TUNNEL BUILD`|`ONION TUNNEL READY`|Sets the `DEST NODE` in the global state to the address/port specified in the request. If the tunnel is built successfully in the subsequent round, the specifeid response is sent, otherwise an error message is sent.|
|1.2|Destroy Onion Tunnel|`ONION TUNNEL DESTROY`|-|If the specified tunnel exists in the global state, then it is destoryed.|
|1.3|Send Onion Cover|`ONION COVER`|-|The specified random amounts of bytes are sent on the established tunnel, if `DEST NODE` in the global state is set.|

#### Outgoing API Requests
|#|Name|Request Object Type|Response Object Type|Description|
|-|-|-|-|-|
|2.1|Incoming Onion Tunnel|`ONION TUNNEL INCOMING`|-|Once a new tunnel is registerd in the global state this message is sent to the CM/UI module.|
|2.2|Incoming Onion Data|`ONION TUNNEL DATA`|-|Once a data packet is recieved from an established tunnel, this message is sent to the CM/UI module.|

### P2P Specification

Note: This is just a draft of our final P2P Specification and each specification is subject to change for our final report.

## Creating Tunnel

### CREATE_SEC_CHANNEL (Request)
|Field|Description|Size|
|---|---|---|
|DEST_IP_ADDRESS|IP adress of the destination (IPv4/IPv6)|Depends on the IP version|
|DEST_HOST_KEY|Host key of the destination|Host key length in bytes
|HAND_DATA_LENGTH|Initiator Handshake Data Length|2 bytes|
|HAND_DATA|Initiator Handshake Data|HAND_DATA_LENGTH|

### CREATED_SEC_CHANNEL (Response)
|Field|Description|Size|
|---|---|---|
|DEST_IP_ADDRESS|IP adress of the destination (IPv4/IPv6)|Depends on the IP version|
|DEST_HOST_KEY|Host key of the destination|Host key length in bytes
|HAND_DATA_LENGTH|Responder Handshake Data Length|2 bytes|
|HAND_DATA|Responder Handshake Data|HAND_DATA_LENGTH|

## Extending Tunnel (With variable number of Hops)

After the tunnel is created, last hop to which the tunnel exists is used to extend the tunnel. Message formats are similar to CREATE_SEC_CHANNEL/CREATED_SEC_CHANNEL but we weill rename them to:
- EXTEND_SEC_CHANNEL
- EXTENDED_SEC_CHANNEL



> We are still designing how the handshake data will look like but after the handshake is done, we have a secure channel between initiator and responder



We are still working on other parts of the P2P specification and will include it in the final report. This is partly because we want to try out different approaches and would adjust specification during the development. 

The types of P2P Messages we would like to employ are:
- TRAFFIC_DATA - Datagram for exchanging data through tunnel using P2P Protocoll. This will incorporate the cover traffic data as well
- DESTROY_TUNNEL - Datagram for signalling that tunnel should be deestroyed.
## Destroy Tunnel




## Future Work

### Group Certificates Through Ring Signatures 
As opposed to signing the identity of a peer with a single trusted peer, we would like to introduce group signing mechanism through [Ring Signatures](https://en.wikipedia.org/wiki/Ring_signature#:~:text=In%20cryptography%2C%20a%20ring%20signature,a%20particular%20set%20of%20people.). The advantage of this approach is that instead of a single trusted peer, a set of trusted peers identities (public keys) are used to sign a message. 

## Workload Distribution - Who Did What? 

|Aleksandre Kandelaki|Hammad Nasir|
|---|---|
|Designed Class Diagram and did use-case analysis of the project|Contributed to analyzing the Process Architecture, threading and Networking|
|Contributed to the Security Measures Section|Contributed to the API & P2P Specification|
|Contributed to P2P Specification|Worked on analyzing Security Measures that will be addressed in our project|


## Effort Spent for the Project

Aleksandre Kandelaki - Spent around 5-7 hours a week on average for this project

Hammad Nasir - Spent around 5-7 hours a week on average
