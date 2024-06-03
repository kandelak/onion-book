# P2P Protocol Design  

As already pointed out in [High Level Design](./high_level_design.md) section, P2P protocol is responsible for peer-to-peer connections of the same module. It could be RPS-to-RPS, CM/UI-To-CM/UI, Gossip-To-Gossip, or as in our case: **Onion-To-Onion**
```
            ____________________________________________________________________
            |                                 |                                 |
            |_________________________________|_________________________________|
```