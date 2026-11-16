---
title: "Measuring the Wabeco, part 1: a three-axis USB scale reader"
excerpt: "Three glass scales, an STM32 board, a printed enclosure and a USB updater: the first working stage of measuring and calibrating my Wabeco mill."
description: "Three glass scales, an STM32 board, a printed enclosure and a USB updater: the first working stage of measuring and calibrating my Wabeco mill."
permalink: /projects/wabeco-usb-scale-reader/
layout: single
classes: wabeco-article
toc: true
toc_label: "On this page"
toc_sticky: true
header:
  teaser: /assets/images/projects/wabeco-scale-reader/finished-reader.jpg
---
Before I start asking my Wabeco milling machine to be more accurate, I would like to know how accurately it moves.

The machine is a **Wabeco F1410 LF-C**, running a **Centroid Acorn** controller and Teknic servos with integrated drives. There is already plenty of electronics involved in getting an axis from one position to another. What I wanted was an independent measurement of the resulting linear movement, available to software on my computer.

That became the first part of a larger calibration project: three glass scales, a small microcontroller board, a USB connection, and eventually a printed enclosure with its own display.

The reader now works with **all three scales connected concurrently**. It streams XYZ positions to the computer, captures the scales' reference marks, displays coordinates locally, and accepts firmware updates over the same USB cable. The scales are still on the bench. Mounting them on the mill is the next job, and the next article.

<figure>
  <img src="{{ '/assets/images/projects/wabeco-scale-reader/finished-reader.jpg' | relative_url }}" alt="The finished black scale reader showing X, Y and Z coordinates, with three DB9 scale cables connected." width="2400" height="1800" loading="eager">
  <figcaption>The finished reader. Three scale connections, one USB connection, and a screen that was not in the original plan.</figcaption>
</figure>

## First, find scales I can actually buy

The early discussion was about the ideal scales: suitable lengths for the Wabeco, a slim body where space is limited, perhaps 1 µm resolution, and an output format that would leave options open for a future DRO or controller upgrade.

Then I had to find something I could actually obtain locally.

The practical option was the TTL glass-scale range available to me from Metal Kovis in Croatia, with listings for [270 mm](https://metal-kovis.hr/shop/cijena/digitalna-mjerna-letva-270-mm-ttl-signal), [320 mm](https://metal-kovis.hr/shop/cijena/digitalna-mjerna-letva-320-mm-ttl-signal) and [520 mm](https://metal-kovis.hr/shop/cijena/digitalna-mjerna-letva-520-mm-ttl-signal). These were the parts around which the project could actually proceed. An ideal specification is useful until it becomes a reason never to build anything.

The scales I ended up with have **5 V TTL outputs and 5 µm resolution**. The supplied inspection paper identified a SINO KA-300. Opening a connector and comparing its wiring with the documentation established what was actually present: power, ground, the two quadrature signals, and a reference signal.

The A and B signals are square waves with a phase offset. Their order tells the reader which way the head is moving; counting their transitions gives displacement. With these scales and x4 decoding, one count represents a nominal 0.005 mm.

The connector happens to be a nine-pin D-sub. That does not make it a serial port, and it certainly does not make every scale with the same connector electrically interchangeable. Checking the actual pinout was a much better starting point than trusting the shape of the plug.

I also considered future use with an AcornSix or a conventional DRO. That remains a separate compatibility question. For this stage, the useful target was an independent USB instrument with a signal format I understood.

## Shopping in my own collection

The first controller candidates were a Raspberry Pi Pico and an Arduino Mega I already had. The Mega's 5 V logic made it an obvious possibility; a Pico would need an appropriate interface for these 5 V signals.

Then I remembered the collection of MikroElektronika boards I had accumulated over the years. There were UNI Clicker MCU cards, several Clicker 2 variants, smaller clickers, shields and a Shuttle system. Apparently I had been preparing for this project without knowing it.

The useful candidate was a [Clicker 2 for STM32](https://www.mikroe.com/clicker-2-stm32f4) with an **STM32F407VGT6**. It offered native USB and hardware timers that could decode three quadrature inputs independently. There was no need to buy three dedicated counter Click boards to get started.

The STM32 is a 3.3 V device, so this still required checking individual pins. The selected inputs support these 5 V signals in the digital configuration we use; that is a property of those pins under the documented conditions, not permission to apply 5 V anywhere on the board. The [STM32F407 datasheet](https://www.st.com/resource/en/datasheet/stm32f407vg.pdf) and the board routing mattered more here than the processor's headline clock speed.

## Make one scale work first

The first prototype looked like most of my first prototypes: a development board, several colours of jumper wire, WAGO connectors, a programmer, and an optimistic amount of clear desk space.

<figure>
  <img src="{{ '/assets/images/projects/wabeco-scale-reader/first-bench-prototype.jpg' | relative_url }}" alt="The first prototype: a Clicker 2 board surrounded by jumper wires, WAGO connectors and a scale cable on the workbench." width="1500" height="2000" loading="lazy" style="width: min(100%, 33rem); height: auto; display: block; margin-inline: auto;">
  <figcaption>The early bench setup. At this stage, being able to change a connection easily was more useful than having tidy wiring.</figcaption>
</figure>

I had an ST-LINK V2 clone, so the first firmware experiments went through SWD. We started with small tests, checked the inputs and supply, and connected one scale. I moved its readhead by hand while the computer recorded what the board saw.

This was also where I pushed back on expanding the test too early. There were three scales waiting, but getting one complete path working was the useful milestone: scale, wiring, hardware counter, USB, and a readable position on the Mac. Once that worked, adding the other two axes was a much more contained exercise.

The firmware uses a hardware encoder timer for each axis. Those timers keep counting while the processor deals with USB and the screen. Software samples and extends the counters into signed 64-bit positions, including across the smaller timers' wraparound points. Otherwise a perfectly ordinary amount of travel could eventually turn into a very surprising coordinate.

The firmware is written in C using [libopencm3](https://github.com/libopencm3/libopencm3), with Python tools on the computer for reading, logging and updating it. Tests cover things such as counter wrapping, packet checks and reference-event handling. Physical movement of a real scale supplied the other half of the evidence.

## USB is the main feature

A display is convenient, but the original reason for building this was to get measurements into software.

The board presents a USB CDC interface and sends a binary position stream at a nominal **100 updates per second**. Each position record contains all three axes, together with timing, sequence and health information. The host tool turns that into a live millimetre readout and can save it to CSV for later analysis.

This is more useful for calibration than simply printing three numbers repeatedly. Sequence numbers make missing samples visible. A boot identifier distinguishes a restarted device from uninterrupted movement. Integrity checks help the receiver reject damaged or incomplete records. The hardware counters continue running between USB reports, so the reporting rate is not the quadrature counting rate.

The console can also choose a convenient local zero without resetting the device's raw counts. That keeps presentation separate from the underlying measurement record.

The reader currently has no machine-control interface. It observes the scales. Later, software can correlate those observations with CNC positions and measurement steps, but that part of the project has not been built yet.

## A box makes it a usable instrument

Once the electronics worked, I wanted to stop treating every cable movement as an experiment.

The scales came with male DB9 plugs. I bought three female panel connectors, soldered short internal leads, and kept the factory scale cables intact. The enclosure was designed around the Clicker board, those connectors, and access to the USB port and power switch.

<figure>
  <img src="{{ '/assets/images/projects/wabeco-scale-reader/plain-enclosure.jpg' | relative_url }}" alt="A black two-piece printed enclosure with a plain lid and openings for the board connections." width="1500" height="2000" loading="lazy" style="width: min(100%, 33rem); height: auto; display: block; margin-inline: auto;">
  <figcaption>The first enclosure had a plain lid. At the time, that was all it needed.</figcaption>
</figure>

The design was made in [CadQuery](https://cadquery.readthedocs.io/), with printable STL files and editable STEP models. A small connector-fit piece helped check the openings before committing to the complete case. The base and lid are separate parts, which turned out to be useful almost immediately.

<figure>
  <img src="{{ '/assets/images/projects/wabeco-scale-reader/enclosure-assembly.jpg' | relative_url }}" alt="The Clicker 2 mounted inside the printed base, with three panel DB9 connectors and internal wiring being assembled." width="1500" height="2000" loading="lazy" style="width: min(100%, 33rem); height: auto; display: block; margin-inline: auto;">
  <figcaption>Moving the working prototype into the enclosure. The three panel connectors keep the scale cables out of the board&#x27;s headers.</figcaption>
</figure>

This is still a bench-built instrument with hand wiring inside. The enclosure gives it a useful physical form; installation near chips, coolant and moving machine parts will need its own mechanical decisions.

## I found a screen

Somewhere during all this, a small TFT module I already owned became part of the discussion.

It was a **KMRTM28028-SPI**, a 2.8-inch 240 × 320 display. There was enough room, the STM32 had the necessary interface, and the plain lid was a separate printed part.

So it acquired a screen.

<figure>
  <img src="{{ '/assets/images/projects/wabeco-scale-reader/display-lid-rear.jpg' | relative_url }}" alt="The underside of the revised printed lid, showing the red TFT module mounted behind the screen opening." width="1500" height="2000" loading="lazy" style="width: min(100%, 33rem); height: auto; display: block; margin-inline: auto;">
  <figcaption>The replacement lid holds the display behind the opening. The existing base could stay.</figcaption>
</figure>

The working driver uses the ILI9341 command set. Display updates are kept small so that drawing numbers does not monopolise the processor while the counters and USB stream are doing their jobs.

The screen now shows X, Y and Z directly on the box, without needing the console open. Its coordinates are relative to the board's startup; the host console can have its own zero. Different displayed origins are therefore possible without any disagreement in the underlying movement.

The module is touch-capable, which immediately suggested three zero buttons. I have deliberately left that for later. The current screen is a readout, and the useful part was already working.

<figure>
  <img src="{{ '/assets/images/projects/wabeco-scale-reader/finished-enclosure.jpg' | relative_url }}" alt="The completed enclosure with its display fitted and three exposed female DB9 connectors along one edge." width="2400" height="1800" loading="lazy">
  <figcaption>The finished enclosure, before connecting the scale cables. A new lid was considerably easier than redesigning the whole box.</figcaption>
</figure>

## The programmer cable had done its job

ST-LINK was useful while bringing up the board. Once it was inside a case, attaching a programmer for every small change became much less appealing.

We added a small, separate **USB bootloader** and installed it through ST-LINK. The scale-reader application was then programmed through that new USB path. Routine updates now use the same cable as the measurement stream.

The update process checks the package and target, writes the application, reads it back, and commits it only after verification. The updater occupies its own flash area, which ordinary application updates leave alone. This is a custom updater for the reader; it does not rely on the application always being healthy to validate an image at startup.

Getting that right involved a useful reminder that one successful upload is not enough. An early version had a USB receive/transmit handoff race. Repeated requests exposed it, and fixing it was followed by repeated uploads and byte-for-byte verification.

One of the first practical uses was wonderfully mundane. I wanted the DB9 cables to leave the enclosure downwards, which made the screen upside down in the original orientation. With ST-LINK already disconnected, we changed the display rotation by 180 degrees and installed the new firmware over USB.

That was the moment the updater stopped being another feature on a list and became the normal way to work on the box.

## The extra wire: reference marks

Each scale also has a reference output. In the scale documentation it is called Z, which is slightly unfortunate when the instrument already has a machine axis called Z. Every scale has one.

After the position reader was working, we connected and implemented all three reference inputs. The timer hardware captures the encoder count at each edge, and the firmware sends those events separately over USB. A brief reference pulse therefore does not have to coincide with one of the regular 100 Hz position reports to be seen.

Bench passes over successive marks produced the expected spacing of about **50 mm**, or **10,000 nominal counts**. X and Z showed matching-edge intervals of 50.000 mm in the recorded passes; Y showed 50.000–50.005 mm. These are distances calculated from the scale's own counts, useful for checking consistency, not an independent calibration of its accuracy.

There was also a very physical debugging episode: Z initially reported reference edges while the readhead was stationary. After powering down, checking and reseating the reference connection, and reconnecting, that chatter stopped and real movement produced the expected marks. We did not isolate the precise original cause. The diagnostic stream made the symptom visible, which was the important part.

The reference signals currently **report events without changing the coordinates**. They do not automatically zero the display or resynchronise it. Identical marks repeated every 50 mm also cannot tell us which mark we have crossed without some other positional context. Keeping the raw counts continuous leaves that future policy separate from capturing the signal correctly.

## What is finished, and what comes next

The completed reader handles three scales concurrently, provides a live USB position stream and logging, shows XYZ locally, captures reference events on every axis, and can update its application over USB. It has also made the journey from loose wires to a box I can pick up without reconstructing the experiment afterwards.

<figure>
  <img src="{{ '/assets/images/projects/wabeco-scale-reader/three-scale-bench.jpg' | relative_url }}" alt="Three glass scales laid out beside the powered reader, with all three scale cables connected and XYZ positions visible." width="2400" height="1800" loading="lazy">
  <figcaption>All three scales and the reader working together on the bench. The next photograph in this story should involve the milling machine.</figcaption>
</figure>

As with my [E-MU driver project]({{ '/projects/emu-apple-silicon/' | relative_url }}), AI was part of the development process throughout. Codex helped work through board documentation, pin choices, firmware, host tools, enclosure revisions and test results. I supplied the hardware, soldering, measurements and the occasional reminder that we should finish the current feature before inventing another one.

The next stage is to mount the scales on the Wabeco, protect and route their cables, and check how the complete installation behaves. Then I want to compare CNC positions with measured travel at settled points, approach them from both directions, and look at repeatability and position-dependent error.

[Centroid's TB-58](https://centroidcnc.com/kb/view_bulletin.php?id=51) describes using measured positions, including measurements from digital scales, to build and verify axis compensation tables. That gives this little reader a concrete future purpose. Automating data collection will need the additional work of coordinating motion, settling and measurement, followed by reviewing any proposed correction.

The **5 µm figure is the scale resolution**, not a claim that the installed mill is accurate to 5 µm. Mounting, alignment, temperature, repeatability and the scales' own errors still have to be understood. No machine compensation has been applied as part of this build.

For now, I have the measurement instrument I wanted. The brackets are next.
