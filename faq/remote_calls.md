---
layout: default
title: Remote Access
parent: How Tos
nav_order: 1
---
# Remote Access

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Functions
To access remote functions **with no return values**, they have to be called using the `remote` directive:
```scala
var counter = placed[Client] {0}
def increment() = placed[Client] {counter += 1}

placed[Client].main {
    increment() // executed locally
    remote call increment() // executed an all Client instances, except on this one
}

placed[Server].main {
    // increment() // does not compile
    remote call increment() // executed an all Client instances
}
```

If a function should be executed on a specific peer or the return type is needed, a `remote.on` block has to be used. Please see the [remote.on section](#remoteon) for more information on this topic.
<span style="color:red;">**This is actually not true. It is possible to perform a remote call, then call `asLocal` and resolve the resulting future. See the following code:**</span>
```scala
// register on registry
val fut = (remote call register(peer.listeningPort)).asLocal
fut.foreach{others => println(s"The others: $others")}
```

## Variables
Accessing variables works by using the `asLocal` directive:
```scala
val counter = placed[Client] {0}
val geigercounter = placed[Server] {0}

placed[Client].main {
    print(counter) // 0, the initial local value
    print(counter.asLocalFromAll) // retrieve all connected counter values
}
```
### asLocal
Depending on the `tie` type, a different access method has to be used.

| Tie Type        | Access Method | Return Type | 
|:-------------|:------------------|:---------------|
| `Single`     | `asLocal`          | `Future[T]` |
| `Optional`   | `asLocal`          | `Option[Future[T]]` |
| `Multiple`     | `asLocalfromAll`          | `Map[Remote[R], Future[T]]` |

### asLocal Variants
There are two variants of the `asLocal` function, which allow accessing the underlying object in an easier fashion:

`asLocal_? (timeout:Duration)` "removes" the `Future` part of the return type by waiting the specified time, e.g. `Future[T]` becomes `T`

`asLocal_!` is similar to the previous variant, but sets the duration to infinity, meaning it waits infinitely long for an result.


## remote.on
Code written inside a `remote.on` expression is executed on a different peer.
```scala
placed[Client].main {
    remote[Server] {
        // this gets called on all connected Server instances
    }
    val mainServer = serverList[0] // a list with remote instances
    remote.on(mainServer) {
        // this gets executed on the mainServer only
    }
}
```
Accessing and returning values can be done by using `capture` and `asLocal`:
```scala
var counter = placed[Server] {0}
def increment(a: Int) = placed[Server] {counter += a; counter}
placed[Client].main {
    val i = 15
    val new_counter_value = remote[Server].capture(i) {
        increment(i)
    }.asLocal
}

```