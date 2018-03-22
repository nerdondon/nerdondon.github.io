---
layout: post
title: "[Notes]: Distributed Sagas: A Protocol for Coordinating Microservices"
description: "Notes on Distributed Sagas: A Protocol for Coordinating Microservices"
tags: notes microservices distributed sagas protocol
---

## Prelude

Some notes from the Distributed Sagas: A Protocol for Coordinating Microservices talk given by
Caitie McAffery at .NET Fringe 2017. Watch on YouTube [here](https://www.youtube.com/watch?v=1H6tounpnG8).

## Actual Notes :D

### Pretext

- With the evolution of service architectures toward a microservices based architecture comes the
  need to maintain database integrity. One class of solutions for this is termed **feral concurrency
  control** by Peter Bailis in 2015 paper.
  - Side note: [feral](http://www.dictionary.com/browse/feral) is defined as wild and uncultivated.

- Feral concurrency control refers to "*application-level* mechanisms for mainting database
  integrity" (emphasis my own). This means that the database is no longer the source of truth that
  enforces consistency and concurrency guarantees. This code is now spread throughout the services
  that constitute the application.

- This is an issue for code maintenance and puts more burden on application developers that should
  really be abstracted away from individual services.

- Example given in talk is a trip planning application. Can book hotels, rental cars, and flights.
  There is also a payment system.

- Example flow with feral concurrency control. Front-end service sends request to hotels service
  which has code that determines that the next step is the payments service. The hotel service also
  determines what happens when a payment fails.

- The key here is that causal relationships--the chain of events that need to occur--are written
  into the services.

- You can imagine a trips service being added that utilizes all of the other services--the hotel,
  car rental, flight, and payments services. There would be a ton of feral concurrency mechanisms
  built into the service in order to deal with failures in its dependencies.

- As these systems evolve you can end up with a mesh of service relationships i.e. the death star
  architecture. These become hard to maintain because you don't know what caual relationships are
  encoded into which services. Therefore, you don't have observabiliity into the architecture.

### Solved problem?

- Frowns on Google's Spanner project because it promises all of both consistency and availibility
  in the CAP theorem, but this is only true because they modify the meaning of availibility to 5
  9's availibility.

- Strong consistency between all services may not be the boon it is made out to be because--as
  hypothesized by a Facebook paper--service latencies may increase. This latency increase is
  posited to be due to all services being bound to the slowest machine in a cluster.

- So the strongest form of consistency may not be desirable due to the latency it adds back into
  the system and users may be more disturbed by such latency increases than minor consistency
  issues.

- Two phase commit (2PC) is another way to keep atomicity across distributed transactions.

- A prepare phase where services reply to a coordinator if they can service the request. A commit
  phase where the coordinator tells services to commit/abort. Finally, services will reply with
  done message.

- 2PC isn't used in industry because it doesn't scale well. There may be O(N^2) messages in the
  worst case and the coordinator presents a single point of failure

### Distributed sagas

- Sagas were originally proposed in a 1987 paper for usage in a single database for long lived
  transactions

- "A Saga is a Long Lived Transaction that can be written as a sequence of transactions that can be
  interleaved. All transactions in the sequence complete successfully or compensating transactions
  are ran to amend a partial execution."

- A distributed saga is defined in the talk to be a "collection of requests and compensating
  requests that represent a single business level action"
  - The requests can be booking a flight or and the compensating transaction can be canceling that
    flight

- Invariants that we want in our requests:
  1. Can abort
  1. Idempotent

- Invariants that we want for our compensating requests:
  1. *Semantically* undoes the effect of a request. Doesn't need to rollback to the same exact
    state e.g. charge a credit and then issue a refund. May see two line items on a statement but
    the state is semantically the same.
  1. Cannot abort i.e. must always succeed
  1. Idempotent so we can re-run until success
  1. Commutative because timeouts may cause retries of the same request that we are trying to
    compensate for

- Distributed saga guarantee: "All requests were completed successfully or a subset of requests and
  the corresponding compensating requests were executed"

- No guarantee of either atomicity or isolation of requests

- Can model as a DAG and inherits nice properties of DAGs like concurrent execution of certain
  constituent sagas

- Need a *distributed saga log* in order to maintain fault tolerance and high availability

- Also need a *saga execution coordinator* (SEC) to execute the saga DAG. The SEC differs from the
  coordinator in 2PC in that there can be multiple SEC's.

- To do rollback, flip the edges of the DAG. This keeps causal relationships intact e.g if there is
  a payment for a flight then there was a flight booked.

- The SEC is a key component because all concurrency control is stored within it and not spread out
  the codebase i.e. not feral.

- Makes service composition easy. You can imagine having a saga for car rentals like book rental
  car -> pay for rental and analogous sagas for booking a hotel and booking a flight. A big trips
  saga can then just pull the independent sagas into a DAG of its own.

## Useful references

- [Link to talk on YouTube](https://www.youtube.com/watch?v=1H6tounpnG8)
- [Consensus Protocols: Two-Phase Commit](http://the-paper-trail.org/blog/consensus-protocols-two-phase-commit/)