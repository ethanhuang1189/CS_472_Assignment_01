# Part 1: Problem-Solution Mapping Table

| Problem | Solution Proposed by Paper | How We See This Today |
|---------|----------------------------|-----------------------|
| Different Packet Sizes| The way to fix it was through gateway fragmentation which was a way for bigger packets that wouldn't "fit" in destination networks to break down into different packets | We see this today with online video games as there is lots of information to send and receive from servers.  One of the most common ways to see gateway fragmentation is packet loss spiking when your game is lagging |
| Reliability Across Multiple Networks | When a packet\packets are sent the sender also sends a "message" requesting a response of which packets were received.  When the receiver sends back what packets it's received and there are some missing, the sender will resend the missing ones | This is still seen in almost everything today, an exmple would be a youtube video, when the servers are transmitting video data there can be missing data occasionally leading to buffering. |
| Flow Control | To determine how much the receiver is able to receiv, the receiver will send a window telling the sender the size it can accept at a time.  This can change at anytime even while packets are being sent if the receiver is overwhelmed for any reason | This is seen today when you're watching youtube, if your device can't process the data as quickly as the server is sending it, your device will send a window telling the server to send less data in each packet |
| Network Performance At Scale | I think this issue wasn't fully addressed because it would've been impossible to think about how widespread computers would become.  People each having multiple different divices.  Their solutions were focues on 1 on 1 communication and not when many devices are communication to each other | We can see this today in multiplayer videogames when lobbies can hold many people all needing to send their location, health, and other data, and everyone in the game would need to have all the data as well |
| Security\Authentication | They didn't really mention anything on security as it was primarily focused connecting networks.  It would make sense as security would only come up as a problem if connecting different networks becomes mainstream | We can see this today with digital certificates like HTTPS |
| Process to Process Communication | The solution is to use ports to allow different computers to establish connections | We see this today on apps like Discord where hundreds of messages are being sent to different people even when they are on different networks |


# Part 2: AI-Assisted Investiation

## A. Investigation overview
I picked Row 4 Network Performance at Scale and specifically I ended up investigating how multiplayer games are able to transfer all that data between servers and all the players.  I wanted to learn how they are able to keep up with accurate data, like who shot first, and how they deal with out of sync packets.

## B. Key Questions You Asked
"I can watch a 4K video on YouTube with millions of other people. How does this work when the 1974 paper's routing was designed for much smaller networks?"
"Could you further explain border gateway protocol and how it relates back"
"relating it back to network performance at scale, can you explain how it works with multiplayer video games were there could be hundreds of people playing together?"
"So what would happen if the authoritative game server can't keep up with the demand? Packet loss? Do servers do anything to try preventing that?"
"The idea that more real time packets are more important than guaranteed delivery is very interesting.  What would happen is some of the players started becoming out of sync whether through out of date packets or not"
"How does this relate to what Cerf-Kahn proposed?"

## C. Connection to the 1974 Paper
Things like gateways, windows, and resending lost packets are still used today.  The biggest difference between the communication explained in the paper verses modern multiplayer games is how the priority works.  Video games today prioritize sending new information over recovering or resending old data, this is because in real time games prioritizing old data can lead to more problems with syncing everyones games.

As the internet grew, more problems started to appear that the original paper didn't anticipate.  Things like BGP or border gateway protocol were created to help manage different networks.  The original paper mentioned data traveling through 3 different networks from A to B to C, but modern day networks have data traveling through many more different networks by letting internet providers to exchange routing information.  Things like lag reduction and sync technologies weren't even thought of in the original paper and are common place now in games like VALORANT using prediction technology to correct differences between players.

## D. Suprise Findings
The biggest surprise was learning that multiplayer games don't really need to recover lost packets for the most part.  In the paper it seemed important to recover lost data through windows and communication between the sender and receiver on what packets were transfered, but in games resending lost packets could be more problematic then just skipping the resending part and receiving and using the newer data.  A good example that helped me was player locations, if there is packet loss at some point and the server loses track of where the player is, it's usually more helpful to get their newer location than to find and recover their old position from when the packet was lost.  This shows a slight change in priority from a priority in recovering lost data to speed.

## E. AI Conversation Documentation

ME:  I can watch a 4K video on YouTube with millions of other people. How does this work when the 1974 paper's routing was designed for much smaller networks?

AI: The key is that today’s Internet still uses the fundamental ideas in Cerf and Kahn’s 1974 paper, but it has added scalable routing, much faster infrastructure, and content delivery networks (CDNs). When millions of people watch a 4K YouTube video, they generally aren't receiving it from one central server. They receive copies distributed across many servers, often geographically close to them.
There are two separate scaling problems here: routing packets across billions of devices and delivering enormous amounts of video data to millions of viewers.
1. What the 1974 paper got right
In A Protocol for Packet Network Intercommunication, Cerf and Kahn describe how independent packet-switched networks can communicate through gateways.
Their crucial insight is that each network can operate independently. A gateway forwards packets toward the destination without needing to understand every detail of every other network.
In the paper's Figure 2 (page 3), a packet travels from network A through gateways M and N to network C. Each gateway uses the destination address to determine where to forward the packet next. cerf74.pdf


This basic principle survives in today's Internet, although gateways are now generally called routers.
However, the proposed addressing scheme had an important limitation: it allocated 8 bits to network identification, allowing only 256 networks. The authors considered this sufficient for the foreseeable future. cerf74.pdf


From the 1974 proposal to today's Internet
1974 proposal	Today's Internet
256 possible network identifiers	IPv4 uses 32-bit addresses; IPv6 uses 128-bit addresses
Gateways forward between networks	Routers forward between networks
Destination network identified by a fixed field	Variable-length network prefixes and hierarchical routing
Routing between networks left largely unspecified	BGP exchanges routing information between independently operated networks
End hosts handle sequencing and recovery	TCP retains these principles; QUIC implements analogous functions over UDP

The original paper therefore provided the architectural foundation, not the complete routing system used today.
2. How modern routing scales
Imagine that every router on Earth had to know the exact location of every phone, laptop, television, and server. Updating all those routers whenever a device connected or disconnected would be enormously expensive.
Modern Internet routing avoids this through hierarchy and route aggregation.
An Internet service provider (ISP), for example, can advertise that it can reach an entire block of IP addresses. Other networks don't have to maintain a separate global route for every device within that block.
How hierarchical routing works
Global Internet routing (BGP)
Networks advertise reachable IP address prefixes.

ISP A
Routes traffic within its own network

Local router

Your laptop



ISP B
Routes traffic within its own network

Local router

Connected servers





Conceptual illustration: global routing identifies networks and address prefixes, while each ISP handles routing internally.
The Border Gateway Protocol (BGP) helps independently operated networks exchange reachability information. Internal routing protocols handle routing within individual networks.
This preserves an important principle from 1974: one network doesn't need complete knowledge of the internal structure of every other network.
3. How YouTube serves millions of viewers
Routing is only half the story. Even if the Internet can direct millions of users to YouTube, a single server cannot necessarily transmit a 4K video to all of them simultaneously.
YouTube addresses this problem through Google's global infrastructure and distributed content delivery, including Google Global Cache.
One video, millions of viewers
Original YouTube video
Uploaded, processed and encoded

Google's distributed delivery infrastructure
Video data can be replicated and cached in different regions.

New York
Cache server

Local viewers


London
Cache server

Local viewers


Tokyo
Cache server

Local viewers




Simplified example, not a depiction of Google's actual server locations or network topology.
When you request a video, YouTube can direct you to a suitable serving location, potentially a cache inside or near your ISP.
This has several benefits. Your video doesn't always have to travel from a distant data center. Popular videos can be stored in multiple locations, and traffic can be spread across numerous servers and network links.
It also greatly reduces the amount of traffic that must cross the global Internet backbone.
4. How 4K streaming works without downloading everything at once
YouTube also doesn't typically send you an entire 4K video as one enormous file before playback begins.
Instead, it uses adaptive bitrate streaming. The video is encoded into different resolutions and quality levels, and your player requests successive chunks based on your available bandwidth and playback buffer.
Adaptive streaming
Illustrative example of how a video player might react to changing network conditions.

Fast connectionModerateSlow

Selected video quality
4K · 2160p










Your player requests high-quality chunks and builds a playback buffer.


Millions of people can watch the same video at different resolutions, served by different machines, over different network paths.
5. What about the enormous bandwidth required?
Suppose a 4K stream averages 20 megabits per second. If one million people watch simultaneously, their combined video traffic is 20 terabits per second.
That traffic doesn't normally flow through one router or one physical link. It is distributed across data centers, caches, ISPs, backbone connections, and local access networks.
The 1974 paper anticipated the need for independent networks and packet fragmentation, but it did not specify the global routing algorithms or distributed video delivery systems needed for this scale. Its proposed addressing limits were also superseded as the Internet grew.
The important distinction is that the Internet's architecture scales by distributing responsibility. BGP helps networks discover routes to one another, local routing handles individual networks, CDNs move popular content nearer to users, and adaptive streaming adjusts delivery to each viewer's connection.
That combination—not a single giant router or a single server transmitting to everyone—is what makes millions of simultaneous 4K streams possible.

ME: Could you further explain border gateway protocol and how it relates back

AI: BGP (Border Gateway Protocol) is one of the major developments that allowed Cerf and Kahn's 1974 idea of connecting independent networks to scale into today's global Internet. Their paper explains how gateways should forward packets between networks, but largely leaves open the question of how those gateways discover which paths to use. BGP addresses that question at the level of independently administered networks.
The important distinction is that BGP does not carry your YouTube video or individually route its packets. Instead, it distributes routing information that helps routers decide where those packets should go.
1. The routing problem in the 1974 paper
On page 2, Cerf and Kahn introduce three networks, A, B and C, interconnected by two gateways, M and N. A packet traveling from A to C passes through both gateways.
The paper's original model
Adapted from the gateway example in Figure 2, page 3 of your paper.
A gateway reads a packet's destination network identifier and determines the next gateway. The authors specifically acknowledge that multiple gateway paths might exist between networks. cerf74.pdf


But consider what happens as the system grows. If there are thousands of interconnected networks, how does gateway M know which other gateway leads toward a particular destination? And if a network connection fails, how does M learn about an alternative route?
The paper establishes the need for interconnecting networks but does not specify a complete, scalable routing-information exchange protocol. BGP is a later solution to that problem.
2. How BGP works
BGP is an inter-domain routing protocol. It exchanges information between autonomous systems (ASes), which are networks or groups of networks operated under a common routing administration, such as an ISP, a large company or a cloud provider. 

RFC Editor




Each autonomous system has an identifying number called an ASN.
Instead of maintaining detailed knowledge of all the devices inside other networks, an autonomous system can advertise the IP address ranges it can reach and the path information associated with those ranges.
Example: BGP advertises a route
Suppose AS 65003 owns the illustrative IP address range 203.0.113.0/24. How could AS 65001 discover a path to it?
The advertisements
AS 65003 tells AS 65002: “I can reach 203.0.113.0/24.”

AS 65002 tells AS 65001: “I can reach that prefix through AS 65003.”

AS 65001 now knows a route toward that destination through AS 65002.

All AS numbers and addresses in this illustration are examples, not actual YouTube routes.
BGP communicates these routes through UPDATE messages. One important part of a route advertisement is its AS_PATH: the sequence of autonomous systems through which the routing information has passed. BGP uses this information to detect routing loops and support routing decisions. 

RFC Editor




A router can receive several possible routes to the same destination. It selects a route according to configured policies and BGP's decision process, not necessarily the route with the fewest network hops. Commercial relationships, routing preferences and other attributes can influence its choice.
3. How BGP solves the scaling problem
One particularly important development is CIDR (Classless Inter-Domain Routing), which allows networks to advertise ranges of IP addresses using prefixes instead of advertising each individual address.
For example, the prefix 203.0.113.0/24 covers 256 IPv4 addresses. One route advertisement can describe reachability to that entire block.
Larger networks can often aggregate several adjacent address blocks into a single advertisement, reducing how much information other networks must store. This substantially improves routing scalability, although multihoming and other requirements limit how much aggregation is possible. 

RFC Editor




There is a direct conceptual connection to page 5 of the 1974 paper. Cerf and Kahn propose an 8-bit network identifier and a 16-bit host identifier. A gateway examines the network identifier to choose the next gateway. cerf74.pdf


Modern routers still use the destination's network address to forward packets, but IP prefixes provide much more flexible addressing. BGP distributes reachability information for those prefixes rather than relying on a small, fixed set of network identifiers.
4. What happens when you open a YouTube video?
Imagine you are watching a video from home. Your device establishes communication with a YouTube serving location selected through Google's delivery infrastructure.
Your ISP has already learned routes to various destination prefixes, potentially through BGP. It can then forward your video requests and the returning video packets using its routing and forwarding tables.
The relationship between BGP and a video stream
1. BGP exchanges reachability information.
   Your ISP learns which neighboring networks can provide routes to the destination IP prefix.
2. Routers install usable forwarding entries.
   When you request video data, routers examine the packet's destination address and forward it toward the appropriate next hop.
3. The destination serves the video.
   YouTube sends video chunks back to you. The return packets can traverse different networks, and the return route need not match the outgoing one.
4. Routing can adapt to network changes.
   If an inter-network connection fails, BGP can withdraw affected routes and advertise alternatives. Routers may then use a different available path, although the transition can temporarily interrupt your stream.

Importantly, BGP does not select the YouTube server, control your video quality or perform retransmissions. Content-delivery mechanisms help choose where the video comes from, streaming protocols control video delivery, and transport protocols handle relevant transmission functions.
BGP's contribution is to make communication between the independently operated networks possible at global scale.
5. Bringing it back to Cerf and Kahn
The 1974 architecture and BGP address complementary problems.
Cerf and Kahn's 1974 design	What BGP adds
Independent networks communicate through gateways.	Autonomous systems exchange routing information.
Gateways use destination network identifiers.	BGP advertises reachable IP address prefixes.
Multiple gateway paths may exist.	BGP distributes information about alternative AS paths.
Internetwork routing is recognized as a problem, without a complete routing protocol specification.	BGP specifies how routers exchange inter-domain reachability information and apply routing policies.
BGP preserves the independence that was central to the original design. An ISP doesn't have to reveal every internal router or cable to the rest of the world. It advertises which destinations it can reach, allowing other networks to decide whether to send traffic through it.
The central connection: Cerf and Kahn described an architecture in which independent networks could form a larger internetwork. BGP provides a decentralized way for those networks to exchange the routing information needed to communicate, even when the Internet contains a vast number of networks and possible paths.
For further technical detail, the formal specifications are RFC 4271 (BGP-4)
 and RFC 4632 (CIDR and route aggregation)

ME: relating it back to network performance at scale, can you explain how it works with multiplayer video games were there could be hundreds of people playing together?

AI: Multiplayer games are an especially interesting example of how the Internet has scaled beyond the networks Cerf and Kahn envisioned in 1974. Unlike YouTube, where a server primarily sends video to viewers, a multiplayer game has to receive information from hundreds of players, process their actions, and send updated game information back to them in near real time.
The underlying Internet architecture is largely the same, but the performance requirements are very different. Routing, latency, server architecture, and the amount of information exchanged all become important.
1. How 100 players can share the same game
Imagine a battle royale game with 100 players. Each player needs to know where the others are, who's shooting, and what's happening in the environment.
A common architecture is to use an authoritative game server. Rather than every player communicating directly with every other player, each player communicates with a central server responsible for maintaining the game state.
How a 100-player game works
Authoritative game server
Receives actions, updates the game world and distributes relevant information.

Player 1
Your computer


Players 2–50
49 computers


Players 51–100
50 computers




Conceptual model of a server-authoritative multiplayer game. Arrows represent two-way communication, not direct physical connections.
For example, when you fire a weapon, your computer sends an action to the server. The server checks the action against its authoritative game state, calculates the result, and sends relevant updates to other players.
This differs from YouTube because a game server has to process incoming information from everyone continuously. Simply distributing copies of the same content from caches is not enough.
2. Where Cerf and Kahn's 1974 paper comes in
Three ideas from the original paper are particularly relevant to multiplayer games.
First, independent networks can communicate through gateways. A player in France and a player in Canada can reach the same game server even when their ISPs use different equipment, routing protocols, and network infrastructures. The paper's gateway architecture allows packets to travel between these independent networks. cerf74.pdf


Second, communication is organized into packets. The paper explains how messages can be divided into smaller units and how packets can be forwarded between networks with different transmission requirements. cerf74.pdf


Third, packet delivery is not guaranteed. Packets can be lost, delayed or arrive out of order. Cerf and Kahn describe sequence numbers, acknowledgments and retransmission mechanisms for recovering from these situations. cerf74.pdf


However, a crucial complication arises in modern gaming: sometimes receiving an old packet is worse than never receiving it.
Suppose a packet describes a player's position 200 milliseconds ago. By the time a retransmitted copy arrives, the server may already know the player's newer position. Sending the outdated information again could be pointless.
That is one reason many real-time multiplayer games use UDP with their own application-level reliability mechanisms, rather than relying exclusively on TCP.
3. How BGP connects all the players
Suppose you live in New York, while other players live in London, Tokyo and Los Angeles. You're all connecting to a game server in Virginia.
Your packets must travel through your local network, your ISP and potentially other independently operated networks before reaching the game server's network.
BGP helps the networks involved exchange information about which IP address ranges they can reach.
Players around the world connecting to one server
New York
Player → local ISP


London
Player → local ISP


Tokyo
Player → local ISP


Los Angeles
Player → local ISP



Interconnected Internet networks
BGP exchanges reachability information between autonomous systems. Routers forward the actual packets.

Game server in Virginia
Maintains the shared game world


Illustrative topology; actual routes may differ in either direction.
BGP is particularly useful because no single ISP needs to maintain complete information about every individual player. Instead, autonomous systems advertise reachability to address prefixes, and routers forward packets according to their routing tables.
But there's an important distinction: BGP provides reachability, not a guarantee of low latency. BGP route selection also considers policies and network relationships, so the path chosen is not necessarily the geographically shortest or fastest one.
Game companies therefore often use geographically distributed servers, network measurements, and carefully selected connectivity to reduce latency.
4. The biggest challenge: latency rather than bandwidth
YouTube needs considerable bandwidth because 4K video consists of large amounts of data. A multiplayer game usually sends relatively small updates, but those updates are time-sensitive.
Consider these illustrative round-trip times between a player and a game server:
Why latency matters in gaming
Example round-trip latency values, not measurements of particular cities or providers.
Your ping
30 ms

10 ms
130 ms
250 ms





Relatively low latency
Actions can be confirmed quickly, and the player usually experiences responsive gameplay.
Illustrative network round trip
30 ms

Half the round trip
15 ms

Half the ping is a rough one-way estimate only when the routes and delays are symmetric. It excludes game processing and rendering.

In an online game, the delay you experience has several components: the time your input takes to reach the server, the time the server takes to process it, the time the response takes to return, and the time your computer takes to display the result.
BGP can influence the network portion by determining inter-network routes, but it cannot eliminate propagation delay, network congestion, or server processing time.
5. How one server handles hundreds of players
Now suppose 100 players are fighting in the same area. Does the server have to tell every player everything that every other player does?
Not necessarily. Games use several techniques to reduce traffic and computation.





Interest management
The server decides which information each player needs. You may need frequent updates about nearby opponents but far fewer details about someone on the opposite side of the map.







Client-side prediction
Your computer immediately displays an estimated result of your movement instead of waiting for the server. When the authoritative update arrives, the client corrects any differences.







Interpolation
Your computer smooths movement between received position updates. Other players can appear to move continuously even though the server sends updates at discrete intervals.


Interest management is especially important at scale. Without it, a game might need to send updates about every player to every other player.
For example, imagine 100 players, each producing 20 position updates per second, with each update occupying 100 bytes. Sending every update to the other 99 players would require approximately 19.8 megabytes per second of server outgoing traffic, before protocol overhead.
If the server instead sends each update to just nine relevant players on average, that component falls to approximately 1.8 megabytes per second.
Illustrative server outgoing traffic
100 players; 20 updates per second; 100 bytes per update. Excludes overhead and other game events.


0MB/s0.55MB/s1.1MB/s1.65MB/s2.2MB/sAll 99 others9 relevant players










     



These figures illustrate the scaling advantage rather than representing actual traffic measurements of a particular game.
6. Connecting everything back to the 1974 architecture
The biggest achievement of the original paper was separating the responsibilities of the networks from those of the communicating hosts.
That separation is still visible in a modern multiplayer game:
Layer or system	Responsibility
Internet architecture from 1974	Provides the foundation for communication across independent packet-switched networks.
BGP	Exchanges reachability information between autonomous systems.
IP and routers	Forward individual packets toward their destinations.
UDP or other transport protocols	Carry game data between clients and servers.
Game networking software	Handles relevant reliability, prediction, interpolation and synchronization.
Game server	Maintains the authoritative state and decides which updates each player receives.
Cerf and Kahn's architecture enables communication between networks without requiring all networks to behave identically. BGP helps this architecture scale globally. But keeping hundreds of players synchronized in real time is primarily an additional application-level engineering problem.
The key difference between YouTube and multiplayer gaming is how the traffic scales. YouTube can distribute copies of existing video across thousands of caching servers. A multiplayer game must continually combine fresh information from its players into a shared state, then distribute relevant updates within tight latency constraints.
The Internet provides the connectivity; the game's networking architecture makes that connectivity feel like a single, shared world.

ME:  So what would happen if the authoritative game server can't keep up with the demand? Packet loss? Do servers do anything to try preventing that?

AI: Yes. If an authoritative game server cannot keep up with hundreds of players, packet loss is one possible consequence, but the first symptom is often increased latency, even when no packets are being lost. The server might receive every player's inputs successfully but take too long to process them.
Modern multiplayer games use several techniques to prevent this, including limiting how much work the server performs, prioritizing important updates, adjusting update rates and distributing workloads across multiple machines.
The interesting connection to Cerf and Kahn's 1974 paper is that they anticipated the problem of limited processing and buffering resources, although modern games require additional application-level solutions.
1. What happens when a server gets overloaded?
Imagine a game with 100 players. The server is designed to update the game world 30 times per second, giving it approximately 33.3 milliseconds to complete each simulation cycle, or tick.
During each tick, it must process player inputs, calculate movement and collisions, resolve combat, update the world, and prepare messages for players.
What happens when the server falls behind?
An illustrative 30-tick-per-second server. Adjust the processing time to see how it affects the server's ability to maintain its tick rate.
Processing time per tick
25 ms

10 ms
33 ms
55 ms
80 ms





Tick processing budget
33.3 ms




The server finishes each tick with 8.3 ms to spare.

Server keeps up
The server has enough processing time for the simulated workload, although network congestion or other delays can still occur.


When the server falls behind, several problems can occur.
- Server lag: Actions take longer to be processed, so players may see delayed movement, hit registration or interactions.
- Rubber-banding: A player's computer predicts their movement, but the server later corrects their position because its authoritative state disagrees.
- Packet loss: If incoming or outgoing packet queues fill up, packets may be discarded. Network congestion can cause additional losses.
- Disconnections: If the server cannot respond for long enough, clients may time out.
These are related but distinct problems. A server can be overloaded without losing packets, and packets can be lost even when the game server has plenty of spare processing capacity.
2. Does the server discard packets to prevent overload?
Sometimes. Servers and operating systems use receive and transmit buffers to accommodate bursts of network traffic. When packets arrive faster than the application can process them, they may accumulate in a queue.
How incoming packets can be lost
100 players continuously sending inputs

Server's incoming packet queue

























10 queued
2 dropped

Illustrative queue with a capacity of 10 packets. In practice, there may be several separate queues in network hardware, the operating system and the game application.
Game server processes received inputs



This connects directly to the 1974 paper. On page 8, Cerf and Kahn explain that an incoming packet can be discarded when no receive buffers remain, relying on retransmission of unacknowledged packets to recover. cerf74.pdf


However, modern real-time games often treat information differently depending on its importance.
If a position update is lost, the client or server may simply use the next, more recent update. Retransmitting the old position could introduce unnecessary delay.
If an important event is lost, such as a player picking up an item, the networking system may retransmit it or use another reliability mechanism to ensure that the event is recorded.
The objective is not necessarily to deliver every packet. It's to deliver the information needed to maintain a consistent, responsive game world.
3. How game servers try to prevent overload

Interest management
The server avoids sending every player every update. For example, it may send frequent position updates for opponents nearby and much less information about distant players.
This reduces outgoing bandwidth, packet processing and sometimes the amount of game-state computation required.




Adaptive update rates
Some games reduce the frequency or detail of less important network updates under load. A nearby opponent might receive more frequent updates than a distant object that is barely visible.
This can save bandwidth and processing capacity, although it cannot fix an underlying simulation that is consistently taking too long.




Prioritizing and combining packets
Instead of transmitting every intermediate position, a game can prioritize the newest relevant state and combine several small updates into one packet. Critical events can be sent reliably while rapidly changing state is allowed to supersede outdated information.



Scaling server infrastructure
A game company can add servers to host more matches simultaneously. For very large shared worlds, developers can also partition the world across different server processes or machines, although synchronizing those partitions introduces additional complexity.


An important distinction is that adding more machines does not automatically increase the number of players one match can support. If the game simulation runs as a single process that must calculate interactions between every player, that process may remain the bottleneck.
4. Why simply adding more bandwidth isn't enough
There are at least three different capacity limits.
Bottleneck	What happens	Possible mitigation
CPU or simulation	Game ticks take too long	Optimize simulation, distribute work, or reduce simulation complexity
Network bandwidth	Outgoing traffic exceeds available capacity	Reduce update sizes and frequency, prioritize relevant updates
Packet processing and buffers	Packets accumulate faster than they can be handled	Batch processing, larger or better-managed buffers, prioritize or discard obsolete updates
Consider a server with a fast 10 Gbps network connection. It can still struggle if the processor cannot calculate the interactions between hundreds of players quickly enough.
Conversely, a powerful processor might simulate the entire match easily, but players can still experience lag if the server's Internet connection becomes congested.
A larger packet buffer isn't necessarily a solution either. It may reduce packet drops but increase the time packets spend waiting in queues. For a real-time game, delivering an old packet hundreds of milliseconds late can be less useful than dropping it.
This is one reason game developers measure both server tick performance and network performance.
5. What happens when the server is severely overloaded?
At that point, different games make different trade-offs. Some may reduce the frequency of network updates; others may allow the simulation to run slower, temporarily limit incoming players, or terminate an unhealthy match.
Large multiplayer games may implement load balancing or server migration, but transferring a live authoritative game state to another server is substantially more complicated than assigning a new match to an idle machine.
For players already in the same match, the server must preserve the shared world, player actions and ongoing interactions. Splitting a match between machines can require frequent communication between those machines, which creates another source of latency.
In other words, the hardest scalability problem is often not supporting 100,000 players across 1,000 separate matches. It's supporting hundreds or thousands of players who must all interact within the same shared simulation.
The connection to the 1974 paper
Cerf and Kahn's design anticipated that packet networks and destination hosts would have limited resources. Their mechanisms for buffering, flow control and retransmission help communication continue despite temporary resource shortages. The paper also discusses how receiving hosts can discard packets when resources are exhausted. cerf74.pdf


But multiplayer games introduce an additional requirement: freshness can be more important than guaranteed delivery.
The original paper emphasizes reliable, ordered communication between processes. Modern real-time games often use UDP and build application-specific mechanisms that selectively provide reliability, prioritize current information and tolerate some loss.
That distinction explains why an overloaded game server may deliberately discard obsolete updates rather than attempt to retransmit everything. The goal is to keep all players experiencing a sufficiently consistent version of the same game world without letting stale information accumulate.

ME: The idea that more real time packets are more important than guaranteed delivery is very interesting.  What would happen is some of the players started becoming out of sync whether through out of date packets or not

AI: This is one of the most interesting problems in multiplayer game networking: what happens when two players see different versions of the same game world?
Imagine you're playing a first-person shooter. You run behind a wall, but another player shoots you just before you reach cover. On your screen, you're already safely behind the wall. On their screen, you were still running toward it.
Both players may have acted correctly based on what they saw. The problem is that their computers were showing different versions of the game at slightly different times.
Games use several mechanisms to manage this, including client-side prediction, server reconciliation, interpolation and lag compensation. They don't completely eliminate disagreement, but they make multiplayer games playable despite network delays.
1. Why players become out of sync
The authoritative server maintains the official version of the game world. However, each player has a local copy of the world that is updated as new information arrives.
Because players have different network connections, their copies can become temporarily inconsistent.
Three players, three slightly different views
Imagine the server has just calculated the positions of everyone in the game. Its updates reach the players at different times.
Authoritative state
Server: tick 1,000
Official state of the game world

Player A
Tick 1,000
Recent update received


Player B
Tick 998
Updates delayed


Player C
Tick 992
Several updates missed



Illustrative tick numbers. Real clients often interpolate between past snapshots while predicting their own movement, rather than displaying a single server tick verbatim.
There are several reasons this can happen. Packets can arrive late, be dropped entirely, arrive out of order or be delayed by congestion. A player's own computer can also struggle to process updates or maintain its frame rate.
An important distinction is that the authoritative server doesn't necessarily become out of sync with every player. Instead, individual players' local representations diverge from the authoritative state.
2. What happens if a player misses several position updates?
Suppose you're watching another player run across an open field. Their position is updated by the server 20 times per second, or every 50 milliseconds.
Now imagine your connection loses three consecutive position updates.
Your game might go 200 milliseconds between received updates rather than the expected 50 milliseconds.
What happens when position updates are lost?
Select a network condition to see how your client might display another player's movement.

All updates arrive

Three consecutive updates are lost

A delayed update arrives after a newer one

Received updates

Received


Lost



Smooth movement
Your computer can interpolate between the regularly received snapshots.


For a short interruption, the game may estimate where the other player is going based on their previous position and velocity. This technique is called extrapolation or, in some networking contexts, dead reckoning.
If the other player continues running in a straight line, the estimate might be reasonably accurate. But if they suddenly change direction, jump or stop, the prediction becomes incorrect.
When fresh information finally arrives, your game has to correct its estimate. This is one major cause of rubber-banding.
3. How the server corrects a player who is out of sync
Now imagine your own character is affected.
You press the forward key, and your computer immediately moves your character on screen. It doesn't wait for the server to confirm each movement, because doing so would make the controls feel sluggish.
Meanwhile, the server is independently processing your inputs.
When it sends its authoritative position back, your computer compares the server's result with its prediction.
Client-side prediction and reconciliation
Example: moving toward a wall
Your client predicts you've reached a position nearer the wall, but the server's authoritative simulation places you farther back.

1. Your client predicts movement. It immediately displays the result of your input and sends that input to the server.
2. The server processes the input. It calculates the authoritative result, including collisions and other game rules.
3. The server sends a correction. It returns the authoritative state and identifies which player inputs have been processed.
4. Your client reconciles the difference. It restores the authoritative state and replays inputs that the server hasn't yet processed. Depending on the discrepancy, the game may smooth the visual correction or immediately snap the player to the correct position.

This is called server reconciliation. It is particularly useful because it corrects the player's simulation without necessarily discarding all their most recent actions.
However, if the discrepancy is large, the correction may be visible. You might suddenly teleport backward, particularly when the server rejects movement because of a collision or when substantial packet loss has occurred.
4. What if two players disagree about whether a shot hit?
This is where synchronization becomes especially complicated.
Consider a player with a 150 ms round-trip latency shooting at someone with a 20 ms round-trip latency.
By the time the shooter's input arrives, the target may have moved significantly.
If the server evaluates the shot using only the target's current position, the shooter may miss even though their aim was accurate according to what they saw on their screen.
Many shooters address this with server-side lag compensation, often involving historical hitboxes.
The same shot, two different perspectives
Shooter's view


Target's view



Simplified example of differing perceptions of the same event.

The server can maintain a short history of player positions and hitboxes. When it receives a shooting action, it can evaluate the shot against an appropriate historical state, accounting for the shooter's latency and the game's rendering delay.
This can make shooting feel more consistent for players with different connection speeds. But it introduces a trade-off: the target might be registered as hit after they've already reached cover on their own screen.
Games therefore commonly impose limits on how far back the server will rewind. The precise rules vary between games.
5. What if someone gets extremely far out of sync?
Small discrepancies are normal. A player whose connection suffers sustained packet loss, extreme latency or long interruptions presents a more serious problem.
The server can take different actions depending on the game's design.
Situation	Possible response
A few missed position updates	Interpolate, predict or wait for the next update
A late, outdated packet arrives	Ignore it if a newer state has superseded it
Client's predicted position disagrees with the server	Reconcile and correct the client's state
Several seconds without usable updates	Freeze or extrapolate remote movement briefly, then stop predicting
Connection becomes unusable	Time out or disconnect the affected player
Client misses an important reliable event	Retransmit it or restore consistency through a new state snapshot
For example, a game may occasionally send a more complete snapshot of the world. Such snapshots can provide an opportunity for clients to recover from accumulated inconsistencies, although a complete snapshot can be relatively expensive.
Not every game uses this exact approach. Some use baseline snapshots with incremental updates; others have specialized reliability and synchronization systems.
6. How this connects to the original 1974 paper
There is a direct conceptual relationship between these game networking techniques and Cerf and Kahn's discussion of packet sequencing, acknowledgments and retransmissions.
On page 6, they describe assigning sequence numbers to transmitted data so the receiver can identify where arriving data belongs, even when some packets are missing. cerf74.pdf


On page 7, they discuss retransmission and duplicate detection, allowing a receiver to recognize data that has already arrived. cerf74.pdf


Modern game protocols can use analogous mechanisms to determine which game-state updates are new, which are missing and which have become obsolete.
The major difference is what happens after detecting missing information.
In the reliable, ordered communication model emphasized in the 1974 paper, the receiver generally needs the missing data to reconstruct the intended message. In a multiplayer game, a new position update may make an older missing position update unnecessary.
For example, if a game client receives positions from ticks 100, 101, 104 and 105, it may be perfectly reasonable to discard ticks 102 and 103 rather than request retransmission. But if those missing packets contained a critical event, such as the player's inventory changing, the game needs another way to ensure that the change is correctly applied.
This is the fundamental trade-off in real-time multiplayer networking: consistency versus responsiveness. Games try to make the local experience immediately responsive while preserving an authoritative shared world. Prediction and interpolation hide short-term network imperfections; reconciliation and reliable delivery of important events restore consistency when needed.
As a result, players are often temporarily seeing slightly different versions of the same world. The engineering challenge is keeping those differences small enough that the game remains playable and fair.

ME: Could you give me a game that does this really well compared to a game that doesn't or didn't?

AI: Two games make an interesting comparison: VALORANT and Battlefield 4 at its 2013 launch. Both use authoritative servers, but their developers have documented very different experiences with synchronization, server performance and hit registration.
VALORANT is an example of a game engineered around minimizing synchronization errors from the outset. Battlefield 4's early networking problems show what players experience when latency compensation, update frequency and server performance aren't working together effectively.





Designed around precise synchronization
VALORANT
Riot Games · Released 2020
Designed with 128-tick authoritative servers, client-side prediction, server reconciliation and historical hit registration. Riot has published detailed explanations of how these systems work. 

Riot Games
+1












Documented launch-era synchronization problems
Battlefield 4
DICE · Released 2013
Its early multiplayer experience included rubber-banding, inconsistent hit registration and delayed death notifications. DICE acknowledged these issues and introduced substantial networking improvements during 2014. 

EA
+1







1. VALORANT: keeping the client and server synchronized
Imagine you're holding an angle, waiting for an opponent to emerge from behind a wall.
Your opponent suddenly appears and fires. Both computers have slightly different representations of what's happening, and both players expect their shots to register accurately.
Riot specifically designed VALORANT's networking architecture to minimize the advantages caused by those differences.
How VALORANT handles synchronization
1. 128-tick servers. The simulation updates 128 times per second, approximately every 7.8 milliseconds. This gives the server frequent opportunities to process movement and combat.
2. Client-side prediction and reconciliation. Your computer predicts the results of your movement immediately. When the server calculates the authoritative result, your computer can correct discrepancies.
3. Short input buffers. Incoming movement inputs are buffered briefly to accommodate irregular packet arrival. Riot targets minimal buffering because excessive buffering adds latency.
4. Historical hit registration. The server keeps a history of player positions and animations. When it receives a shot, it can reconstruct the world at the time relevant to that shot, within configured limits.
These mechanisms are described in Riot's technical explanation of VALORANT's netcode
.
The reconciliation mechanism is particularly relevant to your question about players becoming out of sync.
Riot uses fixed simulation steps so that clients and servers can compare the results of corresponding movement updates. If a player's input doesn't arrive in time, the server may temporarily predict that the player continued holding their previous movement keys.
If that prediction turns out to be incorrect, the server's authoritative result allows the client's simulation to be corrected. 

Riot Games




Riot also publishes an interesting performance detail: during development, its server simulation initially took about 50 milliseconds per frame. Through optimization, engineers reduced this to under 2 milliseconds, making it economically feasible to host many high-frequency matches on the same physical machine. 

Riot Games




This illustrates how network synchronization and server computational performance are closely related.
2. Battlefield 4: when synchronization problems become visible
Battlefield 4 provides a different example, particularly because of its much larger multiplayer matches, vehicles, destruction and complex game environments.
During its early months, DICE acknowledged several networking and simulation problems.
Problems documented by DICE in 2014
Problem	What players experienced
Rubber-banding	Players being moved backward or having their positions abruptly corrected.
Hit registration	Shots that appeared to hit opponents but didn't register damage.
Death synchronization	Kill cameras activating before the client displayed the final damage, creating the impression of premature or instantaneous death.
Latency compensation	Discrepancies in how movement and combat were represented on different clients.
Based on DICE's March 2014 netcode report
. Some of these problems had already been partially addressed when the report was published. 

EA




One of the important fixes was introduced in June 2014: the High Frequency Bubble.
Instead of substantially increasing the frequency of every update sent to every player, Battlefield 4 began supporting more frequent updates for events occurring within a certain radius of each player.
This meant nearby movement, weapon fire and combat could be communicated more frequently without imposing the same bandwidth and processing costs across the entire game world. 

EA




Battlefield 4's High Frequency Bubble

 Your player

 Nearby players receive higher-frequency updates

 Distant players need less frequent information

DICE's producer later explained that increasing every update globally would have been too expensive for the server hardware handling 64-player matches. The selective approach was a way to improve network responsiveness while managing server load. 

GameSpot




3. The important differences
	VALORANT	Battlefield 4 at launch
Typical match	10 players	Up to 64 players
Server architecture	128-tick simulation engineered for competitive combat	Larger matches with significant server processing demands
Main networking challenge illustrated	Minimizing prediction errors and peeker's advantage	Balancing update frequency, bandwidth and server performance
Documented approach	Fixed-step prediction, reconciliation, low buffering and historical hit registration	Subsequent fixes to latency compensation, packet-loss handling and higher-frequency local updates
It's important not to conclude that VALORANT's techniques would automatically work better for a 64-player Battlefield match. VALORANT processes far fewer players and has a different simulation workload. Conversely, Battlefield 4 underwent substantial improvements after launch, so its initial problems shouldn't be mistaken for a description of its entire lifespan.
4. What this tells us about your original question
The comparison illustrates two ways of handling the tension between fresh information, accurate simulation and finite server resources.
VALORANT's architecture emphasizes keeping simulation steps consistent and correcting mismatches quickly. Battlefield 4's later High Frequency Bubble illustrates a complementary technique: allocate additional network resources to the information most relevant to each player rather than transmitting everything at maximum frequency.
Neither approach can eliminate network delay or packet loss. Both must decide which information to predict, which to retransmit and which can safely be replaced by newer information.
This brings us directly back to Cerf and Kahn's 1974 paper. Their sequencing and retransmission mechanisms let endpoints recognize missing information and recover it. Modern multiplayer games extend that basic idea with an additional question: does recovering this particular piece of information still matter, or has a newer game state made it obsolete?

ME: How does this relate to what Cerf-Kahn proposed?

AI: The connection is that Cerf and Kahn's 1974 paper established how computers could communicate reliably across different packet-switched networks, while modern multiplayer games adapt those principles to situations where delivering information quickly can matter more than delivering every packet.
VALORANT and Battlefield 4 illustrate different ways of building on that original architecture. There are four particularly important connections to your paper.
1. Packet loss and retransmission: when should missing data be recovered?
In the section Retransmission and Duplicate Detection (page 7), Cerf and Kahn recognize that packet transmission cannot be perfectly reliable. They propose using sequence numbers, positive acknowledgments and timeouts to detect missing data and retransmit it. cerf74.pdf


For example, suppose a computer sends packets numbered 1, 2 and 3. If packet 2 goes missing, the receiver needs a mechanism for recovering the missing information.
But consider what happens in a multiplayer game.
Reliable communication versus real-time game updates
Cerf–Kahn's reliable delivery approach
The receiver needs missing bytes to reconstruct the original data correctly.


Modern game position updates
If update 3 contains a sufficiently complete, newer position, update 2 may no longer be useful.




This is a fundamental difference. Cerf and Kahn's proposed retransmission mechanism focuses on reconstructing the transmitted data. A real-time game may instead use an unreliable channel for frequently changing information, while delivering important events reliably.
For instance, losing a position update may be acceptable, but losing an inventory transaction could leave a player with an incorrect view of the game.
2. Sequence numbers are still essential
On page 6, Cerf and Kahn describe assigning sequence numbers to bytes so the receiving computer can determine their correct positions, even if packets arrive out of order. cerf74.pdf


This same underlying idea appears in game networking, although the sequence numbers may identify application-level updates or simulation ticks rather than bytes in a reliable stream.
Imagine receiving the following updates about another player's position:
Arrival order	Simulation tick	Player position
First	100	10 metres
Second	102	12 metres
Third	101	11 metres
The packet describing tick 101 arrived after tick 102.
Without a sequence number or timestamp, the game might mistakenly move the player backward from 12 metres to 11 metres.
With sequencing information, the client can recognize that tick 101 describes an older state. Depending on its interpolation system, it can discard that update or use it appropriately when reconstructing past movement.
The game is therefore using the same fundamental concept from the 1974 paper—identifying the ordering of transmitted information—but applying different rules to what should happen when packets arrive out of order.
3. Flow control and managing server overload
A particularly relevant section of your paper is Flow Control on page 8.
Cerf and Kahn describe a sliding-window mechanism in which the receiver advertises how much additional data it can accept. When its available buffering capacity decreases, it can reduce the advertised window. cerf74.pdf


The objective is to prevent a sender from overwhelming a receiver with more data than it can accommodate.
In a multiplayer game, however, the server may also be limited by how quickly it can process player inputs and calculate the shared game world.
Two different overload problems
1974 paper
Receiver cannot accept data quickly enough
The receiver advertises its available window. The sender adjusts how much unacknowledged data it transmits.


Modern game adaptation
The simulation or network cannot process every update efficiently
Game networking software can prioritize important events, reduce unnecessary updates and allocate more resources to players who are interacting directly.


Battlefield 4's High Frequency Bubble is a useful illustration. By increasing update frequency primarily for nearby players, DICE could allocate more networking resources to combat interactions without increasing every update throughout a large match.
That is conceptually related to managing finite network resources, although the High Frequency Bubble is an application-level interest-management technique, not Cerf and Kahn's sliding-window flow control.
4. End-to-end responsibility and the authoritative game server
The final connection concerns where errors should be handled.
Cerf and Kahn argued that gateways should remain relatively simple and that much of the responsibility for reliable communication should reside with the communicating hosts. Their design allows independent networks to forward packets without needing to understand the applications using them. cerf74.pdf


A modern multiplayer game builds on this separation.
From gateways to multiplayer games
Player's computer
Predicts movement, sends inputs and interprets incoming updates.

Internet routers
Forward packets without needing to understand the game.

Authoritative game server
Validates player actions and maintains the official game state.



Routers don't need to know whether a particular packet represents a player's position, a gunshot or an inventory update. They simply forward it according to the network protocol.
The game's endpoints decide which information must be delivered reliably, which can be predicted and which can safely be replaced by newer information.
One important qualification: the 1974 paper does not propose authoritative game servers, UDP-based game protocols, lag compensation or client-side prediction. Those are later application-level solutions built on the broader internetwork architecture.
Bringing the comparison together
Cerf–Kahn proposal	Modern multiplayer equivalent
Packet sequence numbers identify where data belongs.	Sequence numbers and simulation ticks identify the order and age of updates.
Acknowledgments and retransmission recover missing data.	Games selectively recover critical events while sometimes allowing obsolete updates to be lost.
Sliding windows manage the receiver's available capacity.	Games employ additional techniques to control update volume and avoid overwhelming clients or servers.
Gateways forward packets between independent networks.	Internet routers deliver game packets without interpreting the game state.
Communicating hosts are responsible for important end-to-end functions.	Clients and authoritative servers implement prediction, reconciliation and application-specific reliability.
The central insight is that multiplayer games don't abandon Cerf and Kahn's principles; they build on them. The 1974 paper provides the mechanisms and architecture needed to communicate across heterogeneous, imperfect networks. Modern multiplayer games use that foundation while making different decisions about reliability and timing.
A file transfer must recover every missing byte. A multiplayer game often needs the newest possible information about the shared world. By placing these decisions at the endpoints instead of requiring the underlying network to understand the application, the Internet can support both kinds of communication using the same basic architecture.


# Part 3: