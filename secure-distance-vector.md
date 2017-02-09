# Hocnet a payment based mesh network protocol built on Batman-Adv
## Work in progress

This paper is incomplete, feel free to provide feedback via Github issues

## Abstract

As the number of connected individuals and devices expands the `last mile` continues to be the greatest challenge both in the connected and developing worlds, representing a disproportional portion of the cost and difficulty of connecting the world.

Ad-Hoc networks are a type of network where there is no dedicated infrastructure such as switches or routers and instead these tasks are handled by the participating users in a distributed fashion. Mesh networks are a derivative of Ad-Hoc networks in that dedicated hardware for routing is used and encouraged but not ultimately required for the functionality of the network. 

In this paper we propose a plethora of additions to the existing mesh network protocol Batman-Adv with the purpose of creating a 'decentralized last mile' where each node competes to be paid to carry traffic via the most efficient route it can use. Furthermore we will encourage the use of so called `Ad-Hoc infrastructure` where users may implement dedicated hardware such as directional antennas, IR links, or simply mundane wires to produce better and more profitable routes for their nodes to advertise.

We hope that by lowering the barrier of entry to providing connectivity services we can both decrease the cost of connecting the developing world and provide an environment that fosters competition in otherwise monopolistic last mile infrastructure in developed countries.


## Batman-Adv

Batman-Adv is the layer 2 version of the original BATMAN protocol outlined in \cite{batman}. Conceptually the protocol is very simple. Every originator period every node participating in the network broadcasts an originator message containing a MAC address, which identifies the node that `originated` the message as well as the MAC address of the last node to forward it.  When an originator message is broadcast every node that receives the broadcast updates the expanded bandwidth field with it's own estimation of the bandwidth between itself and its neighbor if and only if the predicted bandwidth is lower than the value already in the originator \cite{batroam}.

As the originator messages spread through the network each node checks each originator message it receives against it's own local routing table, either to be added or updated if the throughput advertised by the received originator message is better or the sequence number of the message is higher. Since originator messages contain the last hop that forwarded the message this data combined with throughput and sequence number is used to maintain an up to date routing table with the best first hop along the path to any other node in the network.

To send data to any other node it is sent to the neighbor node listed in the local routing table, who will forward that data in turn based on it's own copy of the routing table. This process continues until the data is delivered to its destination.

Batman-Adv is implemented as an up-streamed Linux kernel module with several optimizations from the original concept, including network coding to batch the broadcast of originator messages and several optimizations focusing on allowing hybrid networks containing traditional Ethernet links as well as wireless links \cite{catwoman}. Perhaps the most promising feature of Batman-Adv is that is remains competitively performant when compared to other mesh protocols \cite{meshperf}


## Modifications to Batman-Adv protocol spec

### Weak black-hole attack protection

With the addition of profit incentive a so called 'black-hole' attack becomes very appealing to potential attackers. A malicious node can falsely advertise routes that it is unable or unwilling to complete, or can complete but only in a worse fashion than advertised \cite{routesec}. 

For our purposes we will define a black-hole attack as any node which increases the throughput metric or decreases the price on any originator message it receives and rebroadcasts.

When a connection between two nodes across the network is opened, both nodes regularly produce a signed "ack" message. This ack message contains:

- A timestamp.
- How much data has been recieved from the other node.

When each side of the connection receives the ack message, they compare the information about how much data was received with their records of how much data was sent. From this, they can estimate the throughput of their route to the other node. Other measures are also possible. If the ack message contains the number of packets received, they can estimate the packet loss rate by comparing this with the number of packets they sent.

They then forward the ack on along the route. Other nodes along the route can use the ack message to make the same estimate. They compare the traffic they have forwarded from the source to the traffic that the destination says it received from the source.

This way, all the nodes on the route can make an estimate about the true quality of the route.

This provides enough information for the network at large to take action against false routing metrics. Each node participating in an active connection will be able to estimate an accurate quality metric to at least one of the ends of the connection, once every ack period.

At the very least, this lets nodes build up accurate information on routes they are using. If a route's true quality is less than other available routes to that destination, another route will be switched to within one ack period.

Blacklisting could also be an option. If a neighbor is consistently advertising innacurate quality metrics, something is wrong. There could be a wide range of blacklisting policies, and a lot of details involved in tweaking them.

Another technique could be to allow nodes to build up trust with their neighbors, and select routes from trusted neighbors more often. Unknown nodes would not recieve much traffic until they demonstrated the accuracy of their routes. This might act to mitigate "sybil" attacks where nodes quickly assume new identities.

```
\bibitem{batman}
David Johnson, Ntsibane Ntlatlapa, and Corinna Aichele.
\textit{A simple pragmatic approach to mesh routing using BATMAN} (2008)

\bibitem{catwoman}
Cigno, R., and Daniele Furlan.
\textit{Improving BATMAN Routing Stability and Performance} (2011)

\bibitem{batroam}
Quartulli, Antonio, and Renato Lo Cigno.
\textit{Client announcement and Fast roaming in a Layer-2 mesh network} (2011)

\bibitem{meshperf}
Murray, David, Michael Dixon, and Terry Koziniec.
\textit{An experimental comparison of routing protocols in multi hop ad hoc networks} Telecommunication Networks and Applications Conference (ATNAC), 2010 Australasian. IEEE, 2010.

\bibitem{meshflip}
Britton, Matthew, and Andrew Coyle.
\textit{Performance analysis of the BATMAN wireless ad-hoc network routing protocol with mobility and directional antennas} Military Communications and Information Systems Conference (MilCIS), 2011. IEEE, 2011.

\bibitem{sead}
Hu, Yih-Chun, David B. Johnson, and Adrian Perrig. 
\textit{SEAD: Secure efficient distance vector routing for mobile wireless ad hoc networks} Ad hoc networks 1.1 (2003): 175-192.

\bibitem{spins}
Perrig, Adrian, et al. 
\textit{SPINS: Security protocols for sensor networks} Wireless networks 8.5 (2002): 521-534.

\bibitem{hash}
Hu, Yih-Chun, Markus Jakobsson, and Adrian Perrig. 
\textit{Efficient constructions for one-way hash chains} International Conference on Applied Cryptography and Network Security. Springer Berlin Heidelberg, 2005.

\bibitem {routesec}
Lundberg, Janne. 
\textit{Routing security in ad hoc networks} Helsinki University of Technology, http://citeseer. nj. nec. com/400961. html (2000).
```