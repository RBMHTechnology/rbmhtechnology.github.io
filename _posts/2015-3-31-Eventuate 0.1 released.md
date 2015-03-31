---
layout: post
title: EVENTUATE 0.1 RELEASED
author: mkrasser
tag: eventuate
---

We, the Eventuate committers, are pleased to announce the availability of [Eventuate 0.1](https://github.com/RBMHTechnology/eventuate/tree/v-0.1). This is the first release of Eventuate and an **early-access release**. Eventuate is a toolkit for building distributed, highly-available and partition-tolerant event-sourced applications. It is written in [Scala](http://www.scala-lang.org/) and built on top of [Akka](http://akka.io), a toolkit for building highly concurrent, distributed, and resilient message-driven applications on the JVM. Eventuate

- derives current application state from logged events (event sourcing)
- replicates application state by replicating events across multiple locations
- allows updates to replicated state at multiple locations concurrently (multi-master)
- allows individual locations to continue writing even if they are partitioned from other locations
- provides means to detect, track and resolve conflicting updates (automated and interactive)
- enables applications to implement a causal consistency model
- preserves causal ordering of replicated events
- provides implementations of operation-based CRDTs
- supports replication at any scale e.g. from single node to multi-datacenter

A more detailed introduction is given in the following documentation sections:

- [Introduction](http://rbmhtechnology.github.io/eventuate/introduction.html)
- [Architecture](http://rbmhtechnology.github.io/eventuate/architecture.html)

For [getting started]() with Eventuate, take a look at:

- [User guide](http://rbmhtechnology.github.io/eventuate/user-guide.html)
- [Example application](http://rbmhtechnology.github.io/eventuate/example-application.html)

If you have any questions or comments, please [let us know](https://groups.google.com/forum/#!forum/eventuate).