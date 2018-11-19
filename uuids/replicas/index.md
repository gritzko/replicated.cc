---
layout: default
title: Replica IDs
in_section: uuids
---

# Replica IDs

Base 64x64 replica identifiers assume a hierarchic naming scheme consisting of up to four components (chunks), denoted by chunk lengths. Components are:

1. primus id (0-2 chars),
2. peer id,
3. client id,
4. session id.

Some possible schemes:

- `0280` (0 for primus id, 2 chars for a peer id chunk, 8 for client id chunk)
- `0262` (2 for peers, 6 for clients, 2 for a client's sessions)
- `1261` (1 char for primus id, 2 for peer, 6 for client, 1 for session)

Rules for encoding are:

- replica ids are hierarchical (peer id includes its primus id, client includes peer and primus, session includes client, peer and primus. For example, in the `0163` scheme, replica with id `Xgritzk0_D` would belong to client with id `Xgritzk` which, in turn, would originate from peer with id `X`),
- tailing chunks can be unfilled (zero) and thus omitted,
- a filled id chunk is never zero,
- no chunk can start with `~`.

## Read next

Calendar-aware [timestamp encoding](../timestamps/).