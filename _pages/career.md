---
title: "Career"
permalink: /career/
layout: single
author_profile: false
toc: true
toc_label: "Timeline"
---

This is the longer version — the parts a resume has to leave out. For the
condensed, dated view, see the [resume](/resume/).

---

## The Through-Line

I've spent my career building backend systems that stay in production for a long
time, under real load, with real consequences when they break. That pushes you
toward a few things: strong typing, explicit domain modeling, understanding the
runtime you're standing on, and a preference for systems whose failure modes are
visible rather than hidden.

It's what drew me into Scala early, and it's what's drawing me toward Rust now —
the same instincts, different trade-offs, less runtime in the way.

The other constant is that I've never been only a software person. There has
always been a parallel track of hardware, measurement and mechanical work, and
the two inform each other more than they look like they should.

---

## Beginnings: Long-Lived Software (2001 – 2008)

My first serious work was contract development at **Kogram**, and it was
unglamorous in the most useful possible way. A document management system for
the Ministry of Environmental Protection and Physical Planning. Library
management software built on **ISO 2709** — a bibliographic record format
standardized in the 1970s — for school and special-purpose libraries.

Seven years on the same software teaches something that greenfield work cannot:
what your decisions feel like years later, when someone else depends on them and
you no longer remember why you did it that way.

---

## Teaching and Writing (2008 – 2012)

I taught at the **University of Split** as an external cooperative — *Introduction
to Computers and Programming* and *Programming for Unix* at FESB, and
*Programming Methods and Abstractions* at the Department of Professional
Studies. Alongside that I maintained parts of the university's information
system.

I also wrote monthly for **PC Chip Magazine**: contemporary technology,
relational databases, a multi-part Linux series covering installation, routing,
firewalls, virtualization and administration, and a piece on common security
mistakes in web development.

Teaching a thing is still the fastest way I know to find out whether I actually
understand it. It's a habit I never really dropped — it shows up now in code
review and in how I write documentation.

---

## Mobile, Scale, and Product (2009 – 2012)

**Where, Inc.** was my first exposure to consumer software at scale. I started as
a developer and became lead on the flagship *Where* application on J2ME, back
when "device porting" was a real and miserable engineering discipline. That
misery produced **BrewPort** — an internal porting framework and code generator
that filled in what Qualcomm's BREW SDK didn't provide across devices. Building a
tool to make a bad problem tractable, rather than grinding through it by hand, is
a pattern I've repeated ever since.

When Where was acquired into **PayPal**, the work changed shape. I was on the
core services team around the Offers subsystem, then became product lead for the
backend of **eBay's Lifestyle Deals** on the PayPal Media Network side. That role
was as much negotiation as engineering — agreeing data-exchange strategies with
partners, running spikes, integrating third parties.

It was the first time I was accountable for a system's direction rather than only
its implementation, and the first time I learned how much of architecture is
actually about other people's constraints.

---

## Founding and CTO Years (2012 – 2018)

A stretch of small companies, where the job was to turn an idea into something
that actually ran. These are grouped by when they started — one of them, Earplay,
kept running until 2025.

**BRIGHTdriver** — in-car entertainment across Rails, iOS and Android, taking it
from concept to a working product as Director of Engineering.

**Star Code** — a distributed video distribution platform: aggregate content from
many sources, distribute it across an array of sites, support multiple payment
gateways. Scala and Play on the backend with RabbitMQ for messaging. This is
where Scala stopped being something I was evaluating and became the language I
reached for.

**Reactive Studios**, later **Earplay** — cofounder, co-owner and CTO, and the
longest-running thing on this page. We built an iOS/Android interactive-audio
title, then a B2B authoring and publishing platform letting people write and
publish interactive voice experiences for Alexa and Google Home, on a Scala/Play
backend.

It ran commercially for roughly a decade. We offboarded the last customers in the
final quarter of 2025. The company still formally exists and I'm still formally
CTO, but it isn't trading — and a decade of running a real platform for real
customers taught me considerably more about backend engineering than the title
does.

**Quantum bit** — my own company, founded in 2014 and still the vehicle for my
independent and contract work. Contract engineering largely for the US market,
plus internal R&D in electronics and some collaboration with academic
institutions.

Founding things taught me the difference between an architecture that is correct
and one that a small team can actually finish.

---

## Scala at Depth (2014 – present)

The through-line of the last decade.

**Major League Gaming** is still one of the systems I'm most attached to. A
proprietary commerce platform selling limited merchandise through custom checkout
flows, and the load pattern was brutal in a specific way: not high average
traffic, but enormous synchronized spikes at announced drop times. Everyone
arrives in the same second. There is no graceful degradation story that the
audience will accept, and failure is immediately, publicly visible. Building for
throughput and resilience there was not an abstract exercise.

**McKinsey & Company** was architecture work — an internal travel-data
aggregation system, processing and exposing results over REST to web and mobile
clients built alongside it. I owned the design, not just the implementation.

**Moneyfarm** put me in regulated fintech: wealth management and life insurance,
a broad estate of internal microservices, Scala with Akka, Play, Cats and MySQL.
Regulated domains are good for a certain kind of engineer. Correctness is not
negotiable and "we'll fix it in the next release" is not always an available
sentence.

**Flaminem** I've now worked with twice. The first engagement was their flagship
KYC product; the current one, contracting through Quantum bit, covers backend
services more broadly. The stack is functional Scala — Akka, Cats Effect — over
**Neo4j**, and the graph database is the interesting part: identity and ownership
structures are genuinely graph-shaped, and modeling them as a graph instead of
forcing them into relational tables changes what questions are cheap to ask.

Somewhere in this stretch my Scala shifted from "Scala as a better Java" toward
the effect-system end of the language. Cats Effect and typed domain modeling
changed what I think a program should look like — pushing errors and effects into
types where the compiler has to acknowledge them, instead of leaving them in
comments and hope.

---

## Leadership

I've held Director of Engineering and CTO titles at several points, most recently
at **Blyott**, running the team behind a real-time IoT platform aggregating
asset and location data in healthcare.

Blyott is also a useful example of a thing I think is underrated: most
engineering leadership is not greenfield. It's inheriting a system, understanding
why it is the way it is, keeping it running for the people who depend on it, and
making it better without pretending you can start over.

---

## Toward Rust

The Rust transition is deliberate and ongoing, and it is not a rejection of the
JVM. It's that the things I liked about Scala — a type system worth arguing with,
explicit modeling, errors you can't ignore — exist in Rust with different
trade-offs and much less machinery between the program and the hardware.

[onset-matcher](/projects/#onset-matcher) is the most complete thing to come out
of it so far: a Rust tool that aligns reference audio against MIDI to generate
golden test fixtures for music software. The part that felt like the point is
that audio time and musical time are separate newtypes, so mixing up the two
coordinate systems is a compile error rather than an afternoon with a debugger.

The goal isn't to leave backend work. It's to bring a systems-level perspective
back to it, and to be useful in contexts where the JVM's overhead is a real
constraint.

---

## The Hardware Track

Running underneath all of this is a parallel engineering life: firmware on PIC
and STM32, circuit design, instrumentation, CNC machining, motorcycle suspension
work.

The clearest example is the
[motorcycle traction-control research platform](/projects/motorcycle-traction-control-platform/)
— an inline device sitting between a motorcycle's ECU and its injectors and ABS
sensors, built around two microcontrollers with the safety-critical paths
deliberately quarantined from everything else.

That project is the best illustration I have of a principle that transfers
directly to software: measure the real system before designing against it, and
make "do nothing correctly" the first milestone. Most of the work was
characterizing what the hardware actually did with an oscilloscope, rather than
trusting what the datasheets implied. Backend systems deserve the same
skepticism, and observability is the same instinct wearing different clothes.

More of that work lives in the [workshop](/workshop/).

---

*For the structured, dated view: [Resume](/resume/)*
