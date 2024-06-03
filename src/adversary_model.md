# Modelling Aversary Behavior 

This is an excerpt from the specification which we decided to analyze thoroughly:

> While building the onion tunnels there is a chance that attacker peers are picked at
random. Building a tunnel through them has the disadvantage that if all of the peers in
the tunnel are attacker peers or co-operate among themselves, our communication can be
de-anonymised. Thus, the more peers participate in the tunnel the smaller the risk, as a
single honest peer in the tunnel is enough to scramble your communication. On the other
hand, having more peers increases the latency of the communication and may effect the call
quality. For this reason, we restrict to a hop count of 2 hops in a tunnel (the 3rd hop being
the destination). Combined with cover traffic, small duration of rounds, and end-to-end
encryption between the caller and the callee, even with attacker peers in the tunnel, the
attacker still has to overcome the dilemma whether the traffic observed is cover traffic or
real traffic. The actual hop count may be parameterised via configuration.

> Since we are building an application employing anonymity, it is important to defend
against certain types of traffic analysis attacks which make use of packet sizes to analyze
communication, even though it is encrypted. For this reason, the Onion to Onion communication should employ packets of fixed sizes. The size of the packet is up to the module
developer to decide. It is wise to choose neither of a too big nor too small value. This also
means that packets should be padded accordingly at each hop to maintain their fixed size.
The packet size should be strictly enforced by the protocol — any peer deviating from this
should be disconnected immediately.


