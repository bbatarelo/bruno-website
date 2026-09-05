---
title: "Projects"
permalink: /projects/
layout: single
toc: true
toc_label: "Projects"
---

A selection of software projects — professional, open-source, and personal
explorations. The focus is on projects that involved real technical depth rather
than a list of technologies used.

---

## Current Focus

### Rust Exploration

Working through systems-level problems in Rust: async runtimes, ownership
patterns, building small services with axum/Tokio, and exploring the type system
as a design tool rather than just a safety net. [onset-matcher](#onset-matcher)
below is the most complete thing to come out of that so far.

---

## Past & Ongoing

### E-MU USB Audio Interface Driver for Apple Silicon

*macOS audio drivers, USB protocol work, real-time systems — ongoing*

E-MU dropped macOS support in 2011, and their last driver was a kernel extension
— which will not load on Apple Silicon at all. The hardware has been silent on
modern Macs for years. This started as a way to get one Tracker Pre making sound
again and turned into a userspace driver for the wider CA0189 family. The
Tracker Pre and the 0404 USB are both verified on real hardware: playback and
capture at all six sample rates from 44.1 up to 192 kHz, volume and mute,
several interfaces published and streamed independently, and the 0404's DIN MIDI
ports exposed as ordinary CoreMIDI endpoints. No kernel extension, no system
extension, no disabling SIP.

It is a Core Audio HAL plug-in that talks to the hardware over USB through
IOKit, handling isochronous streaming, clock recovery and format conversion
itself. The central design constraint is that the *device's* clock, not the
computer's, decides how fast audio moves — so capture runs even when only
playback is wanted, because the capture stream is how the driver measures what
the hardware is actually doing. A lock-free ring buffer is the join between the
two clock domains. Most of the work was protocol archaeology: USB descriptor
and packet captures for each sample rate are committed alongside the code.

The recurring lesson is that "the USB transfer succeeded" and "the audio is
correct" are entirely different statements, which is why the project grew an
analogue loopback test bench before it grew features. It also stopped being a
solo project early: David Nadlinger arrived with an 0404, better diagnostics and
the MIDI work, and pushed it well past the original goal.

[Read the full write-up →]({{ '/projects/emu-apple-silicon/' | relative_url }})

[github.com/bbatarelo/emu-apple-silicon](https://github.com/bbatarelo/emu-apple-silicon)

### Motorcycle Traction-Control Research Platform

*Embedded systems, real-time signal handling, automotive electronics — 2017*

An inline electronics platform that sits between a motorcycle and several of its
own signals — injector drive, ABS wheel-speed sensors and throttle position —
observes them in real time, and can selectively suppress future injection events
without replacing the OEM ECU. The goal was not a finished traction-control
product but an instrumented platform on which fuel-cut strategies could actually
be measured rather than simulated.

The interesting work was electrical before it was algorithmic: injectors are
inductive loads with substantial flyback transients, ABS sensors are not
logic-level devices, and the throttle signal sits in a path the ECU expects to
remain trustworthy. Everything was characterized on a scope first — including a
commercial piggyback controller as a reference — and transparent pass-through
was treated as the first milestone, ahead of any deliberate intervention.

[Read the full write-up →]({{ '/projects/motorcycle-traction-control-platform/' | relative_url }})

---

## GitHub

Most of my public code lives at
[github.com/bbatarelo](https://github.com/bbatarelo) — including the E-MU
driver above. This is the other one worth reading.

### onset-matcher

*Rust, DSP, audio analysis, test tooling*

A MIDI-guided audio alignment tool that solves an awkward problem in music
software testing: when the expected output of your code is *audio*, automated
tests are hard to write, because raw waveform comparison is hopelessly fragile.
The same musical pattern played through a different drum kit produces a
completely different waveform, and a diff that says "sample 44103 differs by
0.002" tells you nothing.

`onset-matcher` sidesteps this by comparing MIDI event sequences at beat
positions instead — a representation that is instrument-agnostic. It detects
onsets in a reference recording, then searches jointly over tempo and per-file
beat offsets using Gaussian-bump cross-correlation to infer which MIDI patterns
were playing, where, and with what overlap. The result is exported as a golden
test fixture that any application can compare against without needing the
original audio. Clock jitter between a drum machine and a DAW is treated as
measurement noise to reason through rather than data to encode.

The part I like most is a small type-level detail: audio time and musical time
are separate Rust newtypes (`Seconds(f64)` and `Beat(f64)`), so mixing the two
coordinate systems is a compile error rather than a debugging session.

[github.com/bbatarelo/onset-matcher](https://github.com/bbatarelo/onset-matcher)
