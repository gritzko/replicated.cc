# The tiered network architecture

When classifying decentralized databases of various kinds, the most natural approach is to focus on their *network architecture*.
Very roughly, the usual suspects are:

1. Gossip - everyone talks to everyone.
2. DHT - peers split the work, forming a distributed hash table.
3. Blockchain - all peers replicate the same data, where consensus is the tricky part.
4. Centralized - this model may not be a contestant, but it is definitely our yardstick.

All these architectures are widely used and their key tradeoffs are well known. 

One good example is BitCoin. It proves a cryptographical ledger as a very reliable technology. Unfortunately, it also proves that blockchain databases have difficulties scaling, not to mention the energy footprint.

Another example is the BitTorrent network using a combo of DHT and gossip (PEX) for peer discovery. Actually, the BitTorrent DHT is the main source of misconception that DHT works well in the wild. 
In fact, the BitTorrent DHT works in very forgiving conditions; finding one working address in 1 minute is good enough. Once that first connection is made, PEX kicks in, supplying all the other available addresses in no time.

In the general case, DHT is plagued by slow response times and low reliability. In common-sense terms, a DHT client has to repeatedly "call" complete strangers to obtain information on other strangers they never met; that information was received from yet other strangers. Quite often, phone numbers are either dated or inaccessible. Sounds tough, right?

Those DHT issues are in a sharp contrast to the PEX gossip, which piggybacks an existing connection to a known peer to deliver information on immediate proven peers. Unfortunately, PEX does not scale beyond that, hence the DHT+PEX combo.

Even if your life's crusade is battling centralization, you can't deny that centralized systems deliver adequate response in deterministic time. DNS is a good example. The downside is having strategic chokepoints. Those chokepoints might be seen as a vulnerability or as a value extraction opportunity, depending on your perspective.  

Limiting the focus to network performance, I want to highlight two key network architecture properties: *scale* and *locality*.

Scale, as Adam Smith noted, allows for economies of scale. Because of scale, specialization and division of labour allow for overall increases in productivity. In our case, a centralized architecture allows for specialized facilities, cheaper hardware, dedicated network links, high reliability.

Locality is the ability to fetch information from immediate peers, without costly multi-hop retrieval.
PEX is a good example to that. Locality allows to solve problems fast, before they require mitigation at scale.

Scale and locality are mutually-exclusive to some degree, as scaled-up facilities are too expensive to be broadly distributed (and the other way around).

I believe that there is a network architecture that is both (a) universally used and (b) relatively obscure, that allows for scale and locality at the same time.

## Power-law/lognormal

For the academic underpinnings of the approach, I have to cite a relatively obscure topic of [complex networks](https://en.wikipedia.org/wiki/Complex_network) and their features - in particular, the small-world phaenomenon.
One common-sense explanation of the idea is "Six degrees of separation":
> Six degrees of separation is the idea that all people are six, or fewer, social connections away from each other. As a result, a chain of "a friend of a friend" statements can be made to connect any two people in a maximum of six steps. It was originally set out by Frigyes Karinthy in 1929 and popularized in an eponymous 1990 play written by John Guare. // [Wikipedia](https://en.wikipedia.org/wiki/Six_degrees_of_separation)

Technically, a network of this kind is dominated by "hubs", exceptionally well-connected nodes. There were plenty of studies proving that online social networks tend to follow this kind of topology. Many other complex networks follow that too. Like, the real-world social network. 

The "power-law" (also [scale-free](https://en.wikipedia.org/wiki/Scale-free_network)) network is a term for complex networks where the degree distribution follows a [power-law](https://en.wikipedia.org/wiki/Power_law). Although, I personally side with [Drs B.Huberman and L.Adamic](https://www.hpl.hp.com/research/idl/papers/webgrowth/nature9sept99.pdf) claiming the power-law is an artifact created by a log-normal distribution in exponentially growing environments. In other words, we deal with the normal (bell-curve) distribution of fitness in the case of multiplicative growth. 
My personal experience with [Wikipedia](https://no-gritzko-here.livejournal.com/22900.html) and [LiveJournal](https://no-gritzko-here.livejournal.com/23712.html) datasets confirms the lognormal hypothesis.

This topology has some very relevant properties:

* it naturally occurs in an exponentially growing network,
* it is made not by central planning, but by local incremental decisions of participants,
* it is very efficient (the diameter is effectively constant),
* it allows for specialization and economies of scale (hubs),
* it is not "centralized" per se (it has many mutually-replaceable "centers").

## Tiered architecture: a proposal

Further on, I will address the following as a *tiered* architecture:

* there is a clear "switching core" in the network (top-level hubs),
* nodes tend to have *upstream* links, *downstream* links and *peering/shortcut* links, although here borders may be blurry,
* the network grows organically, new nodes attach themselves to existing nodes,
* queries are resolved by forwarding them through existing connections.

I assume, that the resulting degree distribution will likely be power-law/lognormal.

Good things about this architecture:

* constant network diameter, hence deterministic response times (likely three hops deep, six hops end-to-end),
* well-provisioned "professional" hub nodes (here we have scale),
* the tiered structure allows for reasonably provisioned regional/on-premise caches/exchanges (here we have locality),
* the network is not centrally managed, has no single gatekeeper (the status of hub/uplink is very fluid),
* no central planning necessary, organic growth is OK,
* no "leader elections" either (no synchronous decisions).

Bad things:
* the network degrades fast when hubs get killed,
* the network assumes some stable topology and always-on servers.

Yet another way to describe it: a tiered network [commoditizes](https://en.wikipedia.org/wiki/Commoditization) centralization. Its "centers" are competing and mutually-replaceable.

For example, a tiered name system will be able to function like DNS, in terms of response times.
Still, given everything is content-addressable and the names are cryptographic, there is no central gatekeeper and the content is perfectly cacheable. There are straightforward ways to cross-check upstreams to ensure their proper function. (Actually, there are numerous ways to borrow from the Apache Cassandra book to ensure the system functions well at scale, with reasonable redundancy.)

Last but not least, this network structure assumes persistent connections, hence it is possible to introduce business models to keep the network afloat. For the lack of strategic chokepoints, those business models will be (really) far from monopolistic.

## Open questions and tunable parameters

Technically, a tiered network may have a brilliant fully deterministic performance.
Assuming the worst case is a query meeting the data in one of the hubs, and a network of "six handshakes", the response time is likely limited by three round-trips (query goes to the hubs, the response returns back).
In case intermediate nodes are properly located, the combined RTT will be close to a single client-hub-client RTT.

On the impementation side, assuming lightweight payloads (name system and/or peer discovery), each participant maintains a key-value (hash)table.
Query processing is basically a read-lookup-forward cycle.

The interesting question though is the dynamic equilibrium shaped by the selfish behavior of participants.
(Note that there is no selfish behavior in DHT networks. Whatsoever.)

Possible interests that shape the equlibrium:

* publishers push their data upwards to make it available,
* clients pull the data down to consume it,
* hubs avoid repetitive queries and unwanted data,
* clients pursue fast response times,
* on-premise caches work for their clients.

My purely scientific gut feeling is that node interests could be aligned if we treat top hub access as a priviledge granted for long exemplary service.
(Well, if root hubs can handle the load themselves, this problem does not even exist.)

Another interesting question is the timescale of network-building.
Do we assume persistent client-server relations between nodes (like with DNS) or the network topology forms dynamically, from one run to another?
Quite likely, both.

Overall, content-addressibility allows for nice separation of storage and ownership.
Hence, the fact that the network is not entirely peer-to-peer is of secondary importance.
Top-level hubs guarantee correctness as record keepers/ data sources of last resort.
Essentially, the rest of the system optimizes response times and prevents repetitive querying.

