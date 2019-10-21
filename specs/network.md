---
layout: default
title: Network protocol
in_section: specs
---

# Network protocol

Network protocol is how clients query Swarm DB and get replies and server pushes.

Network protocol is RON-based. Client sends frames to query data or perform an operation, server replies with one or more frames in response.

## Frame syntax

- One or more operations.
- Ops end with question mark `?`.

## Network protocol restrictions

- Swarm does not track individual frame ids.
- Responses might be sent out-of-order.
- Main wait to match response to request is by object id.
- Swarm has a limitation that only one subscription could be active per object at any given moment. That includes subscriptions through raw reducer or any mappers.

## Subscribe to object’s ops from its creation

To subscribe to object changes, use:

- reducer or mapper as `*` reducer id,
- object id as `#` object id,
- object id as `@` operation id,
- reducer or mapper as `:` reference id.

In closed RON:

```
*txt #123+origin @123+origin :txt ? .
```

In open RON:

```
@123+origin :txt ? .
```

Server will return current object state first and will start sending patches in real-time as they arrive:

```
@123+origin :txt !
@124+origin :123+origin 'x' 1 ,
@125+origin :124+origin 'y' 2 , .

@123+origin :txt !
@126+origin :125+origin 'x' 3 , .

@123+origin :txt !
@127+origin :126+origin 'x' 4 , .
```

## Subscribe to object’s ops since particular version

If you’re reconnecting and already have partial object state, you’re only interested in operations that happened after particular known point in time. In that case, pass operation id that you last saw instead of object id:

If you’re reconnecting and already have partial object state, you’re only interested in operations that happened after particular known point in time. In that case, use:

- reducer or mapper as `*` reducer id,
- object id as `#` object id,
- operation id as `@` operation id,
- reducer or mapper as `:` reference id.

In closed RON:

```
*txt #123+origin @125+origin :txt ? .
```

In open RON:

```
@125+origin :txt ? .
```

Server will send operations that have happened logically after op `125+origin`:

```
@123+origin :txt !
@126+origin :125+origin 'x' 3 , .

@123+origin :txt !
@127+origin :126+origin 'x' 4 , .
```

## Unsubscribe

To unsubscribe, use:

- reducer or mapper as `*` reducer id,
- object id as `#` object id,
- object id (not operation id!) as `@` operation id,
- Special value `~` as `:` reference id.

In closed RON:

```
*txt #123+origin @123+origin :~ ? .
```

In open RON:

```
@123+origin :~ ? .
```

There’s no expected response from the server.

## Get object’s state

To get object’s state without live updates about its changes in the future, subscribe and immediately unsubscribe from it. You can send both requests in a single frame.

In closed RON:

```
*txt #123+origin @123+origin :txt ?
*txt #123+origin @123+origin :~ ?
.
```

In open RON:

```
@123+origin :txt ?
@123+origin :~ ?
.
```

Since repeated ids can be skipped in subsequent ops, it can be written as:

```
@123+origin :txt ?
:~ ?
.
```

## Create new object

To create an object, send raw frame with its reducer as reference:

- reducer as `*` reducer id,
- operation id as `#` object id,
- operation id as `@` operation id,
- reducer as `:` reference id.

In closed RON:

```
*txt #123+origin @123+origin :rga ; .
```

In open RON:

```
@123+origin :rga ; .
```

There’s no expected response from the server.