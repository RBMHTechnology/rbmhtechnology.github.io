---
layout: post
title: EVENTUATE 0.2 RELEASED
author: mkrasser
tag: eventuate
---

We, the Eventuate committers, are pleased to announce the availability of [Eventuate 0.2](https://github.com/RBMHTechnology/eventuate/tree/v-0.2). Eventuate is a toolkit for building distributed, highly-available and partition-tolerant event-sourced applications. It is written in [Scala](http://www.scala-lang.org/) and built on top of [Akka](http://akka.io), a toolkit for building highly concurrent, distributed, and resilient message-driven applications on the JVM. Eventuate

- derives current application state from logged events (event sourcing)
- replicates application state by replicating events across multiple locations
- allows updates to replicated state at multiple locations concurrently (multi-master)
- allows individual locations to continue writing even if they are partitioned from other locations
- provides means to detect, track and resolve conflicting updates (automated and interactive)
- enables applications to implement a causal consistency model
- preserves causal ordering of replicated events
- provides implementations of operation-based CRDTs
- supports distribution up to global scale.

A more detailed introduction is given in the following documentation sections:

- [Introduction](http://rbmhtechnology.github.io/eventuate/introduction.html)
- [Architecture](http://rbmhtechnology.github.io/eventuate/architecture.html)

For [getting started](http://rbmhtechnology.github.io/eventuate/getting-started.html) with Eventuate, take a look at:

- [User guide](http://rbmhtechnology.github.io/eventuate/user-guide.html)
- [Example application](http://rbmhtechnology.github.io/eventuate/example-application.html)

New features in Eventuate 0.2 include:

- [Cassandra storage backend](http://rbmhtechnology.github.io/eventuate/reference/event-log.html#cassandra-storage-backend) for event logs.
- [Snapshot capturing and storage](http://rbmhtechnology.github.io/eventuate/reference/event-sourcing.html#snapshots) for event-sourced actors and views.
- [Custom snapshot serialization](http://rbmhtechnology.github.io/eventuate/reference/event-sourcing.html#custom-snapshot-serialization).
- [Example application](http://rbmhtechnology.github.io/eventuate/example-application.html) can now save and recover from order snapshots.

You can find further details in the [release notes](https://github.com/RBMHTechnology/eventuate/wiki/Eventuate-0.2-release-notes). If you have any questions or comments, please [let us know](https://groups.google.com/forum/#!forum/eventuate).