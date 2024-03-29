# Table of Content

[Overview](#overview)

[Distance Vector](#distance-vector-routing-protocol)

[Basic Distance Vector](#basic-distance-verctor)

[Looping](#looping)

[Loop-free Distance Vector](#loop-free-distance-vector)

[Basic Vs Loop-free](#basic-vs-loop-free)

[Link State Routing Protocol](#link-state-routing-protocol)

[Link State: In-Band Hop-by-Hop](#link-state-protocol-in-band-hop-by-hop-dissemination)

[Link State Advertisement and Flooding](#link-state-advertisement-and-flooding)

[Link State: In-Band End-to-End](#link-state-in-band-end-to-end)

[Path Vector Routing Protocol](#path-vector-routing-protocol)

# Overview

Routing protocols are mechansims by which routing information is exchanged between routers so that routing decisions can be made

A major role of a routing protocol is to facilitate this exchange of information in a standardized manner, sometimes also coupled with routing computations.
distance vector, link state, and path vector


in-band, in our context, means that the communication network that carries user traffic also carries a routing information exchange
Out-of-band, as we use here, means that a completely separate network or mechanism is used for exchanging routing information

A routing table at a node provides a lookup entry for each destination by identifying an outgoing link/interface or a path

# Distance Vector Routing Protocol

### Basic distance verctor

(describe it out loud then you will understand)

- 来自 k 的 message 是什么: 是 k 到**所有** j 的距离
- 这个 message 会影响我什么: 
  - 如果我(i) 到达某个 j 的路径直接经过 k, 那么直接更新
  - 如果我(i) 到达某个 j 的路径原先不经过 k, 现在经过 k 能更短的话，那么更新

![distance verctor protocol basic framework](image.png)


timeline of activities between two nodes
![timeline of activities](image-1.png)

routing environment encounters a transient period during which different nodes may have different views of the network; this is the **root cause** of many undesirable problems


convergence refers to the same view arrived at by all nodes in a network from an inconsistent view


A major problem with a distance vector protocol is that it can cause routing loops


In this situation, user traffic will go in a circular manner 
e.g
node 2, node 3 both have path to node 6, and 3-6 directly connected
then 3-6 failed, node 3 update destination(6) to be super big, **but node 3 failed to update to node 2**
while later node 2 update to node 3 with some old data related to node 6 (**update with old data distribute later and is treated as newer data**)
then node 3 update destination(6) with old data
as a reult, node 2 route to node 3 for destination 6, node 3 also route to node 2 for destination 6


The basic idea of split horizon is quite simple: when transmitting a distance vector update on an outgoing link, send updates only for nodes for which this link is not on its routing table as the outgoing link. **solve the count to infinity problem in some instances**.


split horizon with poisoned reverse, where news about all nodes is provided; in this case, the ones accessible on the outgoing link are marked as ∞. 
Note that this **does not help solve the looping problem**; it simply helps to identify a possible loop and to stop doing a mistaken shortest path computation and from forwarding the user traffic


the following are good steps to take in a distance vector protocol:

•  Once the shortest path is computed by a node, it immediately updates the routing table—there is no reason to inject a time gap.

•  When an outgoing link is found to be down, the routing table is immediately updated with an infinite cost for the destinations that would have used this link as the outgoing link, and a distance vector is generated to other outgoing links to communicate explicitly about nodes that are not reachable.

•  As part of normal operations, it is a good idea to periodically send a distance vector to its neighbors, even if the cost has not changed. This helps the neighbor to recognize/realize that its neighbor is not down.

•  If a routing environment uses an unreliable delivery mechanism for dissemination of the distance vector information, then, besides the periodic update timer (“Keep-alive” timer), an additional timer called a Holddown Timer is also used. **Typically, the hold timer has a value several times the value of the periodic update timer**. This way, even if a periodic update is sent and a neighboring node does not hear it, for example, due to packet corruption or packet loss, the node would not immediately assume that the node is unreachable; it would instead wait till the holddown timer expires before updating the routing table (for more discussion, see Section 5.3 about Routing Information Protocol (RIP)—a protocol that uses an unreliable delivery mechanism for routing information).

•  If a routing environment uses a reliable delivery mechanism for dissemination of the distance vector information, the holddown timer does not appear to be necessary (in addition to the periodic update timer). However, the holddown timer still plays a critical role, for example, when a node's CPU is busy and cannot generate the periodic update messages within its timer window. Instead of assuming that it did not receive the periodic update because its neighbor is down, it can wait until the holddown timer expires. Thus, for such situations, the holddown timer helps to avoid unnecessary destabilization.

•  The count to infinity situation was aggravated partly because one of the critical links had a much higher cost then the other links. Thus, in a routing environment running a distance vector protocol, it is often recommended that link costs be of comparable value and certainly should not be different in orders of magnitude.

•  From the illustrations, it is clear that while the periodic update is a good idea, certain updates should be communicated as soon as possible; for example, when a node is activated, when a link goes down, or when a link comes up. In general, if the cost of a link changes significantly, it is a good idea to generate a distance vector update immediately, often referred to as the **triggered update**. This would then lead to a faster convergence; furthermore, the count to infinity problem can be minimized (although it cannot be completely ruled out).

•  If the cost on a link changes and then it changes back again very quickly, this would result in two triggered updates that can lead to other nodes updating their routing tables and then reverting back to the old tables. This type of oscillatory behavior is not desirable. Thus, to avoid such frequent oscillations, it is often recommended that there be a **minimum time interval** (holddown timer) between two consecutive updates. This certainly stops it from updating new information as quickly as possible and dampens convergence; but, at the same time, this also helps in stopping the spread of bad information too quickly.

•  There is another possible effect with a distance vector protocol. Nodes are set up to send distance vector updates on a periodic basis, as mentioned earlier. Now, consider a node that is directly connected to 10 other nodes. Then, this node will be sending a distance vector on 10 outgoing links and at the same time it will be receiving from all of them. This situation can lead to congestion at the node including CPU overload. Thus, it is preferable that periodic updates from different nodes in a network are asynchronous. To **avoid synchronous operations**, instead of updating on the expiration of an exact update time, a node computes the update time as a random amount around a mean value. For example, suppose that the average update time is 1 min; thus, an update time can be chosen randomly from a uniform distribution with 1 min as the average ± 10 sec. This way, the likelihood of advertising at the same time by different routers can be reduced.


it is also important to recognize that while a routing protocol needs a variety of timers, the actual value of the timers should not be rigidly defined as a part of the protocol description.
we have learned enough to know that it is important to leave the rigid values out of the protocol; 
instead, include the threshold values and range, and let the operational environment determine what are the best values to use in practice. 


### Looping

we could say that looping is the most serious problem since user packets will bounce back and forth between two (or more) nodes.

 the **source of looping**.
 On close scrutiny, you will note that the looping is induced by the Bellman–Ford computation in a distributed environment and it occurs when a link fails. In fact, looping can occur when the link cost also increases; incidentally, link failure can be thought of as a special case of increases in link cost (when link cost is set to ∞). You might wonder, what about a link cost decrease? This case is actually not a problem since Bellman–Ford can be applied as before.
 
 在 distributed 方式中，对于 node i, 只有 d_ik 是自己确信的，其他信息都来自别人，
 如果别人的信息是错误的，node i 的 routing table 就不会准确，
 如果有一些 neighbor 对于环境变化 learn 比较晚，就会告诉 node i 一些错误的信息，
 此时如果按照 根据错误 cost 计算出来的 path 去做 routing, 就会陷入一个 loop，
 我以为这条路可以通，但其实已经不通了，
 我以为这条路可以通，是因为你告诉我可以通，但其实已经不通了，
 routing table 还在更新，但是此时这些数值已经不能反映事实了，所以真正去 routing 的话就会进入 loop


if the distance vector announcement contains certain additional information beyond just the distance, and additionally, route computation is performed through inter-nodal coordination between a node and its neighbors, then looping can possibly be avoided.



 ### Loop-free distance vector

 It is important to note that in a loop-free distance vector protocol, distance vector updates are not periodic and also need not contain the distance for all destinations

 In the case of the loop-free distance vector protocol, route computation is a bit different depending on the situation; furthermore, there are three additional aspects that need to be addressed: building the neighbor table, node discovery and creating entry in the network node table, and coordination activity when the link cost changes or the link fails. 



It is important to note that in a loop-free distance vector protocol, distance vector updates are not periodic and also need not contain the distance for all destinations; furthermore, through the message type, it can be indicated whether a message is an update message or otherwise.
![distance vector message for loop-free distance vector](image-11.png)



a node is required to maintain two states: passive (0) and active (1).

When it is passive, it can receive or send normal distance vector updates.

When it is in an active state, node table entries are frozen. Note that when it is in an active state, a node generates the request message instead of the update message and keeps track of which requests are outstanding; it moves to the passive state when responses to all outstanding requests are received.

 ![node activity](image-2.png)



Loop-free distance vector protocol based on diffusing computation with coordinated update
 ![loop-free distance vector algorithm](image-4.png)


![might have a typo](image-12.png)



### Basic Vs Loop-free

difference:
- distance ij: only option -> a feasible set
- update message: all(might include old ones) -> latest updated ones
- if distance missing(or unavailable): leave it -> active solicit from neighbors 


To summarize, it is possible to use a distance vector protocol framework and extend it for loop-free routing by using diffusing computations with coordinated updates. Like any protocols, it has limitations under certain conditions.


因为 node 5 在 5-6 went down 时会主动询问它的所有邻点
所以如果有一个 loop 的话，一定会被这种询问打破

![loop-free distance vector when a node goes down](image-13.png)



# Link State Routing Protocol

has its roots in Dijkstra's shortest path first algorithm.
it requires a node to have topological information to compute the shortest paths.
record whether this link is up or down—generally referred to as the state of the link. This then gives rise to the name **link state**.


DCR: dynamically controlled routing

RTNR: real-time network routing


out-of-band:
- any pair of nodes can talk to each other through this mechanism irrespective of their location
- all nodes communicate to a central system through a dedicated channel, which then communicates back to all nodes. 


in-band:
- on a hop-by-hop basis
- on a connection/session basis.


The rest of the discussion in this section mostly centers around the exchange of routing information using in-band communication on a hop-by-hop basis. At the end of this section, we will also discuss in-band communication on a session basis.



## Link State Protocol: In-Band Hop-by-Hop Dissemination



### Link State Advertisement and Flooding

A link state message, often referred to as a link state advertisement (LSA), 
is generated by a node for **each of its outgoing links**, 
and each LSA needs to contain at least

![link state advertisement](image-14.png)


 we have learned one important thing: reliable delivery of routing information is important. 
 
 You will find out that almost all routing protocols, since the early days of the basic distance vector protocol, use reliable delivery of routing information. Henceforth, we will assume reliable flooding with the link state protocol.


figure 3.11
 ![figure 3.11](image-15.png)

 the link that connects from node 1 to node 2 in Figure 3.11: this LSA will be generated by node 1; 
 however, in the reverse direction, LSA for the same link from node 2 to node 1 will be generated by node 2.

![link 1->2](image-16.png)

also

![Alt text](image-27.png)

 this message is forwarded to both nodes 2 and 4.


If the cost value of both the LSAs for the same link is the same, then it is not difficult to resolve. However, if the value is different, then a receiving node needs to worry about which LSA for a particular link was generated more recently.

 ![link state advertisement with time](image-17.png)

 There are two possibilities: 
 - either all nodes are clock-synchronized through some geosynchronous timing system, 
 - or a clock-independent mechanism is used. 


 Most link state routing protocols use a clock-independent mechanism called the sequence number to indicate the notion of a time stamp

 ![link state advertisement with sequence number](image-18.png)

 also

 ![Alt text](image-28.png)

![Alt text](image-29.png)


应该是只有 source node 会改变 sequence number, flooding nodes 不会
- (yes)


 It is important that each node maintains a different sequence number counter for **each outgoing link**, 
 and that other nodes maintain their own sequence number counters for their outgoing links

the concept of a source-initiated, link-based sequence number. 

a key issue to consider: the size of the sequence number space.
When a node receives two LSAs for the same link-id from two different neighbors, one with sequence number 7 and the other with sequence number 2, the receiving node has no way of knowing if the sequence number 2 is after the number is wrapped or before.
This tells us that the size of the sequence number space should not be small

However, there is still some ambiguity, for example, when a node goes down and then comes back up with the sequence number set to one

Essentially, what this means is that some additional safeguard is required to ensure that a receiving node is not confused. A possible way to provide this safeguard is to use an additional field in LSA that tells the age of the LSA. Taking this information into account, the LSA takes the form:

![link state advertisement with age](image-19.png)

Now we describe how to handle the age field at different nodes. The originating node sets the starting age field at a maximum value; the receiving node **decrements this counter periodically while storing the link state information in its memory** (decreased by time instead of hop). When the age field reaches zero for a particular link, the link state information for this link is considered to be too old or stale.


in many protocol implementations, the sequence number space is considered as a lollypop sequence number space; 

consider a 32-bit signed sequence number space. The sequence number is varied from the negative number to the positive number while the ends are not used. 

The sequence number begins in the negative space and continues to increment; once it reaches the positive space, it continues to the maximum value, but cycles back to 0 instead of going back to negative; that is, it is linear in the negative space and circular in the positive space giving the shape of a lollypop and thus the name.

R1 announces the sequence number Image to its neighbor R2. The neighbor R2 immediately knows that R1 must have restarted and sends a message to R1 announcing where R1 left off as the last sequence number before the failure. On hearing this sequence number, R1 now increments the counter and starts from the next sequence number in the next link state advertisement.


It is important to understand and distinguish LSA from LSU. An LSA is an announcement generated by a node for its outgoing links; a node receiving LSAs from multiple nodes may combine them in an LSU message to forward to other nodes

When a node is restarted, the sequence number space may be reinitialized to 1 again; this again leaves a receiving node wondering whether it has received a new or old LSA generated from the node that just recovered from a failure. 

While in some cases such an exception can be handled through additional attributes in an LSA, it is usually done through additional mini-protocol mechanisms along with the proper logic control within the framework of the link state routing protocol. For example, there are several aspects to address here: 

1) the clock rate for aging needs to be about the same at all nodes; 
2) receiving, storing, and forwarding rules for an LSA need to take into account the age information; 
3) the maximum-age field should be large enough (for example, an hour); and 
4) if the sequence number is the same for a specific link that is received from two incoming links at a receiving node, then the age field should be checked to determine any anomaly. Thus, typically a link state routing protocol consists of three sub-protocol mechanisms:

•  Hello protocol

used for initialization when a node is first activated

•  Re-synchronization protocol

used after recovery from a link or a node failure.

Since the link state may have been updated several cycles during the failure, resynchronization is merely a robust mechanism to bring the network to the most up-to-date state at the nodes involved so that the link state advertisement can be triggered. The resynchronization step includes a link state database exchange between neighboring nodes, and thus involves both information pull and push. 

an example
![3.13](image-25.png)

![table 3.3](image-26.png)

**a original network might be separated into two isolated networks and they differ in some links state** This is partly why resynchronization is important.

When a failed link or a node is recovered, the best thing to do is to exchange **the entire list of link numbers along with the current sequence number** between the neighbor nodes through a database description message. This allows the node on each side to determine where they differ, and then requests **the cost information for the ones where they differ** in terms of the sequence number

•  Link state advertisement (normal).

The normal link state advertisement by an originating node is information push. 



The entire logic control for a link state protocol is

![figure 3.12](image-30.png)



Now we will bring age into the picture. If two parts have been isolated for a long time, the age field of the LSAs received from the other side will decrement all the way to zero. This will then trigger exception advertisements on both sides for the appropriate set of links. Through this process, links will be deleted from the local copy at the nodes. For example, nodes 1, 2, and 4 from the left side will not have any information about links on the right side. In this case, when link 4-5 recovers, nodes 4 and 5 would do the database description exchange and find out about the existence of new links that will be synchronized and then flooded to the rest of the network through a normal link state update.




## Link State: In-Band End-to-End


if the link state database is primarily what a node needs, is in-band hop-by-hop flooding the only mechanism that can accomplish it?

This would mean that in an N node network, each node would need to set up Image virtual connection sessions with the rest of the nodes.


There are, however, scalability issues with the notion of a virtual connection between any two nodes. 

 A possible alternative is to have a pair of primary and secondary specialized link state servers in a network; this way, each node would need to have just two connections to this pair of servers to retrieve all the link state information.


# Path Vector Routing Protocol

A path vector routing protocol is a more recent concept compared to both a distance vector protocol and the link state routing protocol.

In fact, the entire idea about the path vector protocol is often tightly coupled to Border Gateway Protocol (BGP).

a node receives the distance as well as **the entire path** to the destination from its neighbor. The path information is then helpful in detecting loops. 

A node is thus required to maintain two tables: the path table for storing the current path to a destination and the routing table to identify the next hop for a destination for user traffic forwarding.


![3.14](image-20.png)


protocol message
![protocol message](image-21.png)

e.g.
![Alt text](image-22.png)

destination: j=3
distance: d=0
number of nodes: 1
path known to 2: (3)


Inclusion of the path vector allows a receiving node to catch any looping immediately;
thus, rules such as split horizon that are used in a distance vector protocol to resolve similar issues are not necessary.


On receiving the path vector message from node 2, the receiving nodes (nodes 1, 2, and 4, in this case) can check for any looping problem based on the path list and discard any path that can cause looping and update their path table that has **the least cost**. It is important to note that the path vector protocol inherently uses **Bellman–Ford** for computing the shortest path.

On link failure, node 3 will send an “unreachable” message to its neighbors 2, 4, and 5 stating that path Image is not available.

At about the same time, on receiving the “unreachable” message from node 3, node 4 will take the same action for destination node 6.

However, node 5, on receiving the “unreachable” message from node 3, will realize that it can reach node 6, and thus will inform node 3 of its path vector to node 6.

In turn, node 3, on learning about the new path vector, will send a follow-up path vector message to node 2 and node 4. On receiving this message, node 2 will update its path table to node 6.

We can see that a path vector protocol requires more than the exchange of just path vector messages; specifically, a path vector protocol needs to provide the **ability to coordinate with neighbors**, which is helpful in discovering a new route, especially after a failure.



a node may learn about multiple nonlooping paths to a destination from its different neighbors, and it may choose to cache multiple path entries in its path table

while the path table may have multiple entries for each destination, the routing table points only to the next hop for the best or single preferred path for each destination unless there is a tie; in case of a tie, the routing table will enter the next hop for both paths.


Questions:
- it seems when a link failed, node 3 only update available paths to neighbors, it does not tell neighbors that a previous path is unavailable now?

(This announcement will be understood by the receiving nodes as an **implicit withdrawal**, meaning its previous one does not work, and it is replaced by a new one.)

(also, there is coordination, it sends withdrawl messages of a link when it fails)

- when a node receive, say, path (3,5,6), how does it exactly update stored paths that contains (3...6), what's the rules to replace?

(by P(known to i)kj, because in path vector protocol, a node i only receives path vector from its directly connected neighbors, so P(known to i)kj is distinct, we update this variable before compute the shortest path using Bellman-Ford)



![Path vector protocol with path caching (node i's view)](image-23.png)


a failure case with

![Alt text](image-24.png)

convergence can take quite a bit of time in case of a node failure when the failing node is connected to multiple nodes. 


Finally, here is an important comment. A path vector protocol tries to avoid looping by not accepting a path from a neighboring node if this path already contains itself. However, in the presence of path caching, a node can switch to a secondary path, which can in fact lead to an unusual oscillatory problem during the transient period; thus, an important problem in the operation of a path vector protocol is the stable paths problem [323]. This says that instead of determining the shortest path, a solution that reaches an equilibrium point is desirable where each node has only a local minimum; note that it is possible to have multiple such solutions at an equilibrium point.


Certainly, this implicitly assumes that all costs are hop-based instead of the link costs shown in Figure 3.3. In fact, BGP uses the notion of implicit hop-based cost. BGP has another difference; the node as described here is a model of a super-node that is called an Autonomous system on the Internet. This also results in a difference, instead of the simplifying assumption we have made that the node-ID is what is contained in a path vector message. Furthermore, **two supernodes may be connected by multiple links** where there may be a preference in regard to selecting one link over another. Thus, BGP is quite a bit more complicated than what we have described and illustrated here in regard to a path vector protocol; BGP will be discussed in detail in Chapter 9.






# Exercises

3.1  Review questions:

(a)  How is split horizon with poisoned reverse different from split horizon?

- poisoned reverse anounces when a link fails, while split horizon does not.

(b)  What are the sub-protocols of a link state protocol?

- hello protocol
- re-synchronization protocol
- normal link state advertisement

(c)  List three differences between a distance vector protocol and a link state protocol.

- distance vector every node maintains links between neighbors and itself, link state maintains an overall picture
- distance vector sends vector only to its neighbors, link state floods links to all the network
- basic distance vector has loop issue, link state can avoid this issue easily since it knows overall picture of the network


(d)  Compare and contrast announcements used by a basic distance vector protocol and the enhanced distance vector protocol based on the diffusing computation with coordinated update.

- loop-free anouncements contain a set of feasible distance vector instead of just one
-
-

3.2  Identify issues faced in a distance vector protocol that are addressed by a path vector protocol.

- looping
- count to infinity

3.3  Consider a link state protocol. Now, consider the following scenario: a node must not accept an LSA with age 0 if no LSA from the same node is already stored. Why is this condition needed?

3.4  Study the ARPANET vulnerability discussed in RFC 789 [716].

3.5  Consider the network given in Figure 3.13. Write the link state database at different nodes (similar to Table 3.3) before and after failure of link 4-5.

![3.13](image-25.png)

![table 3.3](image-26.png)


3.6  Consider a seven-node ring network.

(a)  If a distance vector protocol is used, determine how long it will take for all nodes to have the same routing information if updates are done every 10 sec.

(b)  If a link state protocol is used, how long will it take before every node has the identical link state database if flooding is done every 5 sec? Determine how many link state messages in total are flooded till the time when all nodes have the identical database.

3.7  Solve Exercise 3.6, now for a fully-connected 7-node network.

3.8  Investigate how to resolve a stuck in active (SIA) situation that can occur in the distance vector protocol that is based on the diffusing computation with coordinated update (DUAL).

3.9  Compare and contrast the distance vector protocol that is based on the diffusing computation with coordinated update (DUAL) and the Babel routing protocol, also based on the distance vector framework.

3.10  Consider a fully-mesh N node network that is running a link state protocol. Suppose one of the nodes goes down. Estimate how many total link state messages will be generated.

3.11  Implement a distance vector protocol using socket programming where the different “nodes” may be identified using port numbers. For this implementation project, define your own protocol format and the fields it must constitute.

3.12  Implement a link state protocol using socket programming. You may do this implementation over TCP using different port numbers to identify different nodes. For this implementation project, define your own protocol format and the fields it must constitute.