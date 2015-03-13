---
layout: post
title: Event Sourcing at Global Scale
author: mkrasser
tag: eventuate
---

We recently started to explore several options how to globally distribute an application that is based on <a href="http://martinfowler.com/eaaDev/EventSourcing.html" target="_blank">event sourcing</a>. The main driver behind this initiative is the requirement that geographically distinct locations (called sites) shall have low-latency access to the application: each site shall run the application in a near data center and application data shall be replicated across all sites. A site shall also remain available for writes if there are inter-site network partitions. When a partition heals, updates from different sites must be merged and conflicts (if any) resolved.

In this blog post, I’ll briefly summarize our approach. We also validated our approach with a prototype that we <a href="https://github.com/RBMHTechnology/eventuate" target="_blank">recently open-sourced</a>. During this year, we’ll develop this prototype into a production-ready toolkit for event sourcing at global scale.

As a starting point for our prototype, we used <a href="http://doc.akka.io/docs/akka/2.3.8/scala/persistence.html" target="_blank">akka-persistence</a> but soon found that the conceptual and technical extensions we needed were quite hard to implement on top of the current version of akka-persistence (2.3.8). We therefore decided for a lightweight re-implementation of the akka-persistence API (with some modifications) together with our extensions for geo-replication. Of course, we are happy to contribute our work back to akka-persistence later, should there be broader interest in our approach.

The extensions we wrote are not only useful in context of geo-replication but can also be used to overcome some of the current limitations in akka-persistence. For example, in akka-persistence, event-sourced actors must be cluster-wide singletons. With our approach, we allow several instances of an event-sourced actor to be updated concurrently on multiple nodes and conflicts (if any) to be detected and resolved. Also, we support event aggregation from several (even globally distributed) producers in a scalable way together with a deterministic replay of these events.

The following sections give an overview of the system model we developed. It is assumed that the reader is already familiar with the basics of <a href="http://martinfowler.com/eaaDev/EventSourcing.html" target="_blank">event sourcing</a>, <a href="http://martinfowler.com/bliki/CQRS.html">CQRS</a> and <a href="http://doc.akka.io/docs/akka/2.3.8/scala/persistence.html">akka-persistence</a>.

<h3>Sites</h3>
In our model, a geo-replicated event-sourced application is distributed across sites where each site is operated by a separate data center. For low-latency access, users interact with a site that is geographically close.

Application events generated on one site are asynchronously replicated to other sites so that application state can be reconstructed on all sites from a site-local event log. A site remains available for writes even if it is partitioned from other sites.