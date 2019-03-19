---
layout: default
title: Placements
parent: Concepts
nav_order: 2
---
## Placements
A placement in ScalaLoci defines the location where a function or a variable is located.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Functions
For a function without return values, this can be defined like this:
```scala
def peerfunc() = placed[Registry] {
    // code runs on registry
}
```
The type in the placed annotations defines, where the function is placed. In the case above, the function
would be available on all `Registry`-peers and can be remote called on different peers. 
If the function should return Value, additional type hints are necessary:
```scala
def peerfunc2(): Int on Registry = placed {
    // code runs on registry and returns an Int
}
```
Notice that in this case the additional type hint for the `placed`-Expression can be omitted.

## Variables
The syntax for variables is quite similar to the one for functions:
```scala
val chats: Array[Strings] on Registry = placed { new Array[Strings](10)}
```

## localOn
Variables and functions defined with `on` are visible and accessible by all peers. If only local access on the peer is needed,
the `localOn` directive can be used.
```scala
val local_chats: Array[Strings] localOn Registry = placed { new Array[Strings](5)}
```