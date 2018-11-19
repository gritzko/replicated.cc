# Replicated Object Notation

![](Untitled_Artwork-7146ef74-e595-46d0-8f35-e24cf02a361c.jpg)

Replicated Object Notation is a format for *distributed live data*. RON is suitable for optimized storage, efficient transmission, reliable negotiation and queries. RON was concieved as a network-native and efficient alternative to JSON and other serialization formats.

Key RON principles are:

- **Immutability**. Once created RON operations travel through system unchanged.
- **Addressability**. Everything: changes, versions, every piece of data is uniquely identified and globally referenceable.
- **Source and transport agnosticism**. It doesn’t matter where RON data comes from and how. Any node can read RON from the network, filesystem, database, message bus and/or local cache, in any order, and merge them correctly.
- **Efficiency**. RON data is optimized to make metadata overhead tolerable and real-time collaborative applications possible. It’s optimized machine first, human readability second.
- **Causality**. Each RON operation explicitly references what version of an object it is based on. No matter how and when you get your data, you can always reconstruct the correct order events have happened in.

RON is an answer to new reality where different agents join and leave network all the time and communicate to each other without any order and reliability guarantees. In other words, RON is lingua franca for Generation Internet.

[RON UUIDs](./RON-UUIDs-276bf8e8-41f2-4bab-986a-0f63b9560673.md)

[RON Specification](./RON-Specification-43aef42f-7f5e-40cc-90b0-29e5c4ede9ef.md)

Details of RON protocol.

[Replicated data types](./Replicated-data-types-5ecda8ec-a73a-4475-8cab-259a36e9b434.md)

Data sctructures implemented on top of RON mathematically proved to support concurrent editing.

[Swarm DB](./Swarm-DB-aa6efe0b-75d2-4023-bb62-ce52e075248f.md)

An reference implementation of the RON protocol with persistent storage and websocket API.

[Swarm.js](./Swarm-js-54b3c045-8040-41b6-acf5-895fd41c8642.md)

An implementation of RON protocol in JavaScript, embeddable in a browser.

[Advanced concepts](./Advanced-concepts-525e946e-bfdf-4485-b55d-09c7ee4cc3fd.md)