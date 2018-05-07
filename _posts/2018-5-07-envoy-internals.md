---
layout: post
title: "[Notes]: Envoy Internals Deep Dive by Matt Klein"
description: "Notes on Envoy Internals Deep Dive by Matt Klein"
tags: notes envoy internals lyft matt klein
---

## Prelude

Some notes from the **Envoy Internals Deep Dive** talk by Matt Klein. The talk was given at KubeCon
EU 2018. Watch on YouTube [here](https://www.youtube.com/watch?v=gQF23Vw0keg).

## Actual Notes :D

### Architecture

- Envoy is designed with an out of process architecture specifically to address the polyglot nature
  of microservices

- At its core, Envoy is a byte proxy. This gives it extensibility to supoort a multitude of
  protocols.

- Create with versatility from the start so that Envoy can act as service, middle, and edge proxy

- Hot restart is key feature

- One part of Envoy is built as a pipeline with _filters_ as building blocks. Filters are extension
  points where users can add functionality.

- The other part of the architecture is where the central management functionality lives. Things
  like the cluster/service managers or stats engine.

- The prevailing school of thought for proxies before the 2000s was to have serve a single
  connection per thread. Envoy is takes a different approach where it serves multiple connections
  per thread via an event loop.

- The "main" thread of Envoy handles the management functionality and worker threads handle the
  connections and filter pipeline. The key to keeping Envoy simple is the latter part of the
  previous statement. It means that worker threads do not have to communicate with each other and
  only have to acquire locks in a small set of cases.

- Read-Copy-Update (RCU) is used to govern parallelization within Envoy. "Designed for read heavy,
  write infrequent workloads". Helps reduce the need for locking by making the read path lock free.

- Envoy keeps a vector pointers in thread local storage (TLS) that is mirrored across the main and
  worker threads. RCU is used to update slots in the vector and posts messages to worker
  threads to update information e.g. route tables.

### Hot Restarts

- "Full binary reload without dropping any connections"

- Allow two processes of Envoy to run at the same time and utilize a shared memory region. Stats
  and locks are stored here.

- The two proceses communicate through an RPC protocol over Unix domain sockets (UDS) and perform
  socket passing. After the second Envoy process spins up, it will communicate that it is good to go
  and the first process will begin to drain connections.

### Stats

- There's a 2 level caching mechanism for stats. There is a TLS cache, a central cache on the main
  thread, and the actual entries either in shared memory or in process memory.

## Useful references

- [Link to talk on YouTube](https://www.youtube.com/watch?v=gQF23Vw0keg)
- [Slides from the talk](https://speakerdeck.com/mattklein123/kubecon-eu-2018)
- [Link to Matt Klein's Medium](https://medium.com/@mattklein123) - ALL HIS ARTICLES ARE HIGHLY
  RECOMMENDED READING
  - Especially this one on [modern load balancing](https://medium.com/@mattklein123)