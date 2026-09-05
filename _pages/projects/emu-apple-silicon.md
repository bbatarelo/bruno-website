---
title: "Reviving an E-MU Tracker Pre on Apple Silicon — with a lot of help from AI"
excerpt: "How a side project to revive one old E-MU audio interface turned into an Apple Silicon driver, an open-source collaboration, and a growing pile of test hardware."
description: "How a side project to revive one old E-MU audio interface turned into an Apple Silicon driver, an open-source collaboration, and a growing pile of test hardware."
permalink: /projects/emu-apple-silicon/
layout: single
toc: true
toc_label: "On this page"
toc_sticky: true
---
I have a weakness for good hardware that becomes "obsolete" only because the software around it stops being maintained.

The **E-MU Tracker Pre** is a good example. It is an old USB audio interface, but there is nothing particularly obsolete about the part that matters: it still sounds good, it has useful I/O, and the hardware itself works perfectly well. The problem is that E-MU's official macOS support ended a long time ago, and the old kernel-extension driver belongs to a Mac era that Apple Silicon has left behind.

So the interface was sitting there, still perfectly capable electrically, but effectively disconnected from modern Macs.

Naturally, I started wondering whether it could be brought back.

## A project I probably would never have started without AI

Writing an audio driver is not exactly a convenient evening project. It combines USB, asynchronous isochronous transfers, Core Audio, device clocks, real-time constraints, old driver archaeology, undocumented behaviour, and a lot of opportunities to produce perfectly correct-looking counters while listening to complete garbage from the speakers.

I am a software developer, but this is far enough outside my normal work that, realistically, I probably would never have allocated enough of my life to attack it from scratch in the traditional way.

This is where AI changed the equation for me.

I used AI heavily from the beginning: first to help turn the problem into a development plan, compare possible architectures, identify the risky unknowns, and design small experiments that could answer one question at a time. Later it became part of the implementation and debugging loop as well.

The important distinction is that this was never a "prompt: write me a driver" exercise. The useful pattern was much more like working with a very fast technical collaborator: form a hypothesis, write a probe, run it against real hardware, inspect the result, discover that the hardware does something nobody expected, update the model, and try again.

And the hardware always got the final vote.

The original plan actually started with DriverKit and a hybrid architecture: keep the macOS-facing layer small and put the device-specific protocol and timing logic into a testable Rust core. That experiment was useful even though the production driver eventually took a more pragmatic route: a **Core Audio HAL plug-in talking directly to the E-MU over USB from userspace**. No kernel extension, no reduced system security, no special DriverKit entitlement.

The Rust core survived that architectural change. It now contains the parts that are about the E-MU rather than about macOS: descriptor parsing, protocol details, clock estimation, feedback handling and, later, MIDI packet handling. A large part of it can be tested without an interface even being connected.

The first development history is slightly surreal to look back at. The serious work started on August 20, and by August 21 the Tracker Pre was playing and recording through ordinary macOS applications.

That was the point where this stopped being a feasibility experiment and became a real project.

## Then the Internet did what the Internet occasionally does well

I published the work, partly because keeping something like this private would make very little sense. There are still people using these E-MU interfaces, and some of them have kept old Macs around specifically because their perfectly good audio hardware no longer worked on current machines.

Very quickly, **David Nadlinger (@dnadlinger)** appeared and started contributing serious work.

This changed the project substantially. David had an **E-MU 0404 USB**, which is from the same family but not identical to the Tracker Pre. His work helped bring up the 0404, improve timing and buffer handling, build better diagnostic and loopback tooling, investigate the difficult high-sample-rate behaviour, and eventually add the 0404's DIN MIDI ports as normal CoreMIDI endpoints.

There is another fun detail here: David was also using AI heavily. So the project became a rather modern kind of open-source collaboration: two humans with real hardware, experience and judgement, each using LLMs as force multipliers, exchanging traces, code, measurements and increasingly annoying edge cases.

At some point I realised where this was heading.

I bought a used **E-MU 0404 USB** myself.

Of course I did.

## This is how projects escalate

At first I had my reference interface, a **MOTU M4**, the Tracker Pre, some cables and the usual temporary-test-bench chaos.

<figure>
  <img src="{{ '/assets/images/projects/emu-apple-silicon/test-bench-early.jpg' | relative_url }}" alt="A cluttered desk holding an E-MU Tracker Pre, a MOTU M4 audio interface, a studio monitor, a BeatBuddy pedal and a tangle of patch cables">
  <figcaption>The "temporary" test setup, before the project started occupying the desk.</figcaption>
</figure>

Once the 0404 arrived, having more than one real E-MU device on my desk became extremely useful. It meant I could test whether we were accidentally writing a Tracker-Pre-specific driver or whether the common CA0189 family behaviour had really been understood.

It also meant my desk became ridiculous.

My OCD eventually won, so I designed and 3D-printed a little rack for the three interfaces: the MOTU M4 as the known-good reference, the Tracker Pre and the 0404.

<figure>
  <img src="{{ '/assets/images/projects/emu-apple-silicon/three-interface-rack.jpg' | relative_url }}" alt="Three audio interfaces stacked in a black 3D-printed rack: E-MU 0404 USB on top, E-MU Tracker Pre in the middle, MOTU M4 at the bottom">
  <figcaption>The three-interface tower: MOTU M4, E-MU Tracker Pre and E-MU 0404 USB.</figcaption>
</figure>

That solved the physical clutter for approximately five minutes.

Then testing became more ambitious.

A driver that reports that it moved the right number of USB bytes is not necessarily a driver that moved the right **audio**. We hit several bugs where all the counters looked healthy and the result was silence, noise or channel corruption. So proper loopback testing became increasingly important: generate a known signal, send it through an E-MU, capture the analogue result on a separate trusted interface, and compare what came back.

Doing that by constantly rearranging cables got old very quickly.

So, naturally, a small mixer joined the setup.

<figure>
  <img src="{{ '/assets/images/projects/emu-apple-silicon/test-bench-with-mixer.jpg' | relative_url }}" alt="The interface rack standing behind a small mixer, coloured patch cables running from each interface into mixer channels labelled MOTU M4, Tracker Pre and 0404">
  <figcaption>The current test bench. By this point the "small driver experiment" had acquired a rack, three interfaces, a mixer and dedicated loopback wiring.</figcaption>
</figure>

This is a pattern in my life: if I become interested enough in a project, it tends to acquire tools, fixtures, cables and test equipment until the original problem looks suspiciously like a small laboratory.

## What works now

What began as "can I make my Tracker Pre produce sound on an M-series Mac?" has turned into a proper multi-device driver.

The **Tracker Pre and 0404 USB are both verified on real hardware**. They play and record at all six sample rates the devices support, from 44.1 kHz through 192 kHz. Volume and mute work. The driver can publish multiple E-MU interfaces at the same time and stream them independently. The 0404's DIN MIDI input and output are also exposed through CoreMIDI.

There is now an increasingly serious test suite around all this: hardware probes, capture checks, timing diagnostics, analogue loopback tests, latency measurements, glitch detection, MIDI loopback, fault injection and recovery tests. This matters because one of the recurring lessons of the project has been that **"USB transfer succeeded" and "the audio is correct" are very different statements**.

The hardware also contained some wonderful little archaeological surprises. For example, at 176.4 and 192 kHz the capture packets contain a four-byte length word that is absent at lower rates. The interface can occasionally acknowledge a sample-rate change without actually applying it. At stream startup it can temporarily adjust packet sizes while aligning its internal clock. Some incorrect configurations do not fail cleanly at all — they wedge the device until it is physically replugged.

These are exactly the kinds of behaviours that make this work slow if every experiment takes an evening, and much more approachable when an AI-assisted workflow lets you build the next probe while the previous result is still fresh in your head.

The **0202 USB** is the remaining obvious sibling we have not yet verified on real hardware. It uses the same CA0189 family and is expected to be close, but "expected" is not the same thing as tested. If somebody has one and an Apple Silicon Mac, that is a particularly useful piece of hardware for the project right now.

## What I find most interesting about the whole thing

Yes, I am happy that my old Tracker Pre works again.

But the more interesting part is what this project says about the kinds of things that have suddenly become realistic for individual developers.

A few years ago, I would have looked at this problem, estimated the amount of unfamiliar Apple audio-driver code, USB protocol work, reverse engineering, instrumentation and testing involved, and decided that it was a fascinating way to lose several months of evenings.

With modern AI tools, the economics of curiosity are different.

They do not remove the need to understand what you are doing. In a hardware project they very quickly run into reality: the interface either plays clean audio or it does not. A clock is either stable or it drifts. A packet contains the bytes you predicted or it contains something else. No amount of confident text can negotiate with an oscilloscope, a USB trace or a loopback recording.

What AI did for me was lower the cost of exploring each branch far enough to find out whether it was real. It helped me cross areas where I did not have years of accumulated domain knowledge, while leaving the engineering process grounded in measurements, source material, tests and actual devices on the desk.

And then open source added the other important ingredient: another human showed up, brought different hardware and expertise, found things I had not found, improved things I had already built, and pushed the project much further than the original "make my Tracker Pre work" goal.

So this is now a small but very satisfying example of **old hardware, modern AI and traditional open-source collaboration meeting in the middle**.

If this story ever travels beyond the small E-MU corner of the Internet, the one-line version is probably this: **a developer in Croatia used AI to help revive a discontinued E-MU audio interface on Apple Silicon, published the work, and an open-source collaborator helped turn the experiment into something much bigger.**

A perfectly good audio interface gets another life. A small group of E-MU owners gets to keep using hardware they like. And I somehow ended up with three audio interfaces, a mixer, a 3D-printed rack and a lot more USB knowledge than I had planned to acquire.

As usual, things escalated.

---

*The driver and development documentation are available in the public [emu-apple-silicon](https://github.com/bbatarelo/emu-apple-silicon) repository. At the time of writing, the project is still young and intentionally developer-oriented, but the supported hardware already works as normal macOS audio devices. Reports, testing and contributions — especially from 0202 USB owners — are very welcome.*
