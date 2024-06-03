# P2P Protocol Design  

As already pointed out in [High Level Design](./high_level_design.md) section, P2P protocol is responsible for peer-to-peer connections of the same module. It could be RPS-to-RPS, CM/UI-To-CM/UI, Gossip-To-Gossip, or as in our case: **Onion-To-Onion**

TODO:
 - Define Payload size
 - 

## Entities 
- Relay
- Channel
- Cell
- Circuit
- Exit Node
- Client Node
According to the specification it seems we need following Peer-To-Peer Messages. The namings where heavily inspired
by famous Onion Tunnel Implemntation called Tor. 

- `ONION KEY CREATE P2P`
    - Sends a request to another Onion Peer to create a key using Diffie-Hellman Like Protocol
- `ONION KEY RESPONSE P2P`
    - The Onion Peer now responds with its Diffie-Hellman Parameter and a session key has been created!
- `ONION REDIRECT P2P`
    - This Paylaod indicates that the message 
```
            0                               15                                 31 
            ____________________________________________________________________
            |                                 |                                 |
            |_________________________________|_________________________________|
```