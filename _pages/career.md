---
title: "Career"
permalink: /career/
layout: single
author_profile: false
toc: true
toc_label: "Timeline"
---

This is the longer version. For the condensed view, see the [resume](/resume/).

---

## The Through-Line

I've spent my career building backend systems that live in production for a long
time, under real load, with real consequences when they break. That tends to
push you toward a few things: strong typing, explicit domain modeling,
understanding the runtime, and a preference for systems where the failure modes
are visible rather than hidden.

That's what drew me into Scala early on — the combination of a powerful type
system, JVM ecosystem access, and functional programming discipline. And it's
what's drawing me toward Rust now: similar principles, different trade-offs,
closer to the metal.

---

## Scala & Distributed Systems

The bulk of my career has been in Scala, working on distributed systems:
event-driven architectures, CQRS/Event Sourcing patterns, Akka/Pekko-based
actor systems, streaming pipelines, and service-oriented backends.

I've built and maintained systems where:
- Data consistency across distributed nodes is a hard requirement, not a
  best-effort
- Domain models are the source of truth, not database schemas
- Operational concerns (observability, deployment, failure recovery) are part of
  the design from the start

---

## Domain Modeling

Over time I've developed a strong interest in domain-driven design — not the
cargo-cult version, but the practice of building software whose structure
reflects the actual problem domain. This tends to produce systems that are
easier to reason about, easier to change, and less likely to accrete bugs that
nobody understands.

---

## Transition to Rust

The Rust transition is deliberate and ongoing. I've been building side projects
and deepening my knowledge of ownership, lifetimes, async Rust, and the
ecosystem around Tokio, axum, and similar tooling.

The goal isn't to abandon backend development — it's to bring a systems-level
perspective back to it, and to be capable of working in contexts where the JVM's
overhead is a real constraint: embedded-adjacent systems, high-performance
services, or anything where you want to reason precisely about memory and
scheduling.

---

## Other Roles

Beyond pure engineering, I've worked in roles that included technical leadership
— defining architecture, reviewing designs, mentoring engineers, and being the
person who makes the call when there's no obvious right answer.

I don't lead by committee, but I also don't lead by dictating. The goal is
always to get to the right answer faster by having the right people thinking
clearly about the right things.

---

*For a structured, dated view: [Resume](/resume/)*
