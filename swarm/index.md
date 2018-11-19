---
layout: default
title: Swarm
section: swarm
---

# Swarm

Swarm is a distributed, offline-ready reactive KV storage with eventual consistency and automatic conflict resolution. Think Git but for your entire database.

<img class="fig" src="diagram.jpeg">

## What does Swarm do?

Swarm solves for you the hard problem of reliable and efficient data synchronization between multiple parties in a complex topologies. You hand your data to local Swarm instance and Swarm ensures it gets propagated to all interested parties. Swarm easily scales up to multitude of replicas and sessions, spanning to browsers, mobile devices and multi-region server deployments.

Swarm also spends extra effort to guarantee data conflicts are resolved in a predictable, controllabe way. Even in a presence of concurrent writers and intermittent/offline network connections. Your data is always safe and where in needs to be, delivered just in time.

## What is Swarm good for?

Many things:

- reactive database,
- collaborative text editing,
- distributed version control,
- decentralized Web,
- geo-distributed eventually consistent data store,
- shared database run by multiple parties (e.g. a [two-sided market](http://lexicon.ft.com/Term?term=two_sided-markets) data exchange),
- [super-peer network](http://ilpubs.stanford.edu:8090/594/1/2003-33.pdf).

## Documentation

- [Storage](storage/)
- [Application layer](app/)
- [API](api/)
- [Tutorials](tutorials/)