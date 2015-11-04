---
layout: post
title: EVENTUATE 0.4 RELEASED
author: mkrasser
tag: eventuate
---

We, the Eventuate committers, are pleased to announce the availability of [Eventuate 0.4](https://github.com/RBMHTechnology/eventuate/tree/v-0.4). Eventuate is a toolkit for building distributed, highly-available and partition-tolerant event-sourced applications. It is written in [Scala](http://www.scala-lang.org/) and built on top of [Akka](http://akka.io), a toolkit for building highly concurrent, distributed, and resilient message-driven applications on the JVM. Eventuate

- derives current application state from logged events (event sourcing)
- replicates application state by replicating events across multiple locations
- allows updates to replicated state at multiple locations concurrently (multi-master)
- allows individual locations to continue writing even if they are partitioned from other locations
- provides means to detect, track and resolve conflicting updates (interactive and automated)
- enables reliable event collaboration between event-sourced microservices
- enables applications to implement a causal consistency model
- preserves causal ordering of replicated events
- provides implementations of operation-based CRDTs
- supports distribution up to global scale

A more detailed introduction is given in the following documentation sections:

- [Introduction](http://rbmhtechnology.github.io/eventuate/introduction.html)
- [Architecture](http://rbmhtechnology.github.io/eventuate/architecture.html)

For [getting started](http://rbmhtechnology.github.io/eventuate/getting-started.html) with Eventuate, take a look at:

- [User guide](http://rbmhtechnology.github.io/eventuate/user-guide.html)
- [Example application](http://rbmhtechnology.github.io/eventuate/example-application.html)

New features and enhancements in Eventuate 0.4 include:

- [Support for cyclic replication networks](https://github.com/RBMHTechnology/eventuate/issues/116) (based on a complete re-write of the event replication mechanism)
- [Disaster recovery of replication endpoints](http://rbmhtechnology.github.io/eventuate/reference/event-log.html#disaster-recovery)
- [Disaster recovery example implementation](http://rbmhtechnology.github.io/eventuate/example-application.html#disaster-recovery)
- [Event replay backpressure](http://rbmhtechnology.github.io/eventuate/reference/event-sourcing.html#backpressure)
- [CRDT instance snapshots](https://github.com/RBMHTechnology/eventuate/issues/132)
- [Lazy load and recovery of CRDT instances](https://github.com/RBMHTechnology/eventuate/issues/47)
- [LWW-Register CRDT](https://github.com/RBMHTechnology/eventuate/pull/113)
- [Isolation of Cassandra log and index reads](https://github.com/RBMHTechnology/eventuate/issues/134)
- [Reference documentation TOC expanded](http://rbmhtechnology.github.io/eventuate/reference.html)
- Example application startup scripts enhancements
  - [bug fix for running on Linux](https://github.com/RBMHTechnology/eventuate/pull/120)
  - [automated classpath generation](https://github.com/RBMHTechnology/eventuate/pull/121)

You can find further details in the [release notes](https://github.com/RBMHTechnology/eventuate/releases/tag/v-0.4). If you have any questions or comments, please [let us know](https://gitter.im/RBMHTechnology/eventuate).