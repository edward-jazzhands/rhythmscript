# RhythmScript - The Idea

RhythmScript is the world's first time-based programming language. A purely declarative language where time itself is not just a feature - the entire language is written around time and synchronization, built on top of the MIDI protocol. The time and syncing abilities of the language under the hood uses MIDI Clock Syncing, which is already the most popular syncing protocol in the world due to its ubiquity and extremely low overhead.
In RhythmScript, time and MIDI are both treated as first-class citizens. Everything in the language runs through the scheduler, which ensures that MIDI information is always output at precisely the right time.

In some ways, RhythmScript is philosophically similar to audio programming languages such as SuperCollider or Csound. However, their purpose is fundamentally different. RhythmScript does not deal with audio in any way - It is purely a conductor of MIDI information. It is intended to be used everywhere that MIDI is used, including fields such as lightshow programming or animatronics. Its not a language for crafting sound, its a language for crafting patterns in time.

The RhythmScript parser will most likely be coded in Rust. Also being declarative, it will also be completely static (no run-time evaluation). The architecture for this model is probably most similar to CSS, with a focus on hot reloading. This should make it much easier to built a working MVP. Like CSS, RhythmScript is not "run", nor compiled into bytecode. The entire file is parsed into memory, which will be very fast thanks to Rust (Its basically just doing mathematical calculations. The only limit is how well the algorithms themselves are optimized).

## What is Programming in RhythmScript?

At its core, RhythmScript is **not a tool for describing logic**, but a **language for expressing intention over time**. You think in terms of **patterns**, **variations**, and **transformations**.

Programming in RhythmScript is closer to composing a score than writing traditional code. You don’t manage state or control flow—you define **temporal behaviors**. The language is declarative, because timing, rhythm, and musical structure are inherently declarative: you say _what should happen_, _when_, and _how it evolves_ over time.

The goal isn’t to simulate a sequencer—it’s to replace it with something more expressive. RhythmScript lets you define **patterns**, apply **transformations**, and schedule **events** in musically meaningful ways. Timing is always quantized to the global clock unless explicitly offset. Every event is placed with intentionality in a beat grid that’s designed for live and compositional use alike.

Instead of looping through steps manually, you declare repeatable **rhythmic structures**. Instead of building logic trees, you write **reactive rules** that respond to bar numbers, loop phases, or even external triggers. For example, you might say “play this fill every 8 bars,” or “increase velocity slightly every 4 repetitions.” These are high-level musical ideas expressed as concise, readable code.

Patterns are first-class values. You can assign them, pass them into modifiers like `reverse`, `evolve`, `transpose`, or `humanize`, and layer them into tracks or instrument channels. You can apply mathematical transforms to them, shift their density, rotate steps, or apply probability curves—all while staying in sync.

Custom functions are supported, but they’re written declaratively too. They’re more like macros or pattern templates than full procedural logic. Most creative power comes from combining built-in operations rather than defining new behavior from scratch.

There’s no imperative control flow because there doesn’t need to be. RhythmScript is designed for deterministic, synchronized environments. The scheduler ensures that everything plays back at the right time, in perfect sync with the host clock, whether that's a DAW, a hardware sequencer, or a MIDI network.

This is a language built for time. MIDI is the protocol, but time is the model. You describe patterns, structures, and transformations—and the engine turns that into precise, low-latency event streams across whatever systems you’re connected to.
## Problems being solved

1. **TidalCycles and Sonic Pi are closed systems** in terms of integration. They’re fantastic creative tools, but they’re environments unto themselves. You can’t just "import tidal into your project" or embed it into a broader toolchain. They’re not languages so much as curated playgrounds.
2. **MIDI support is either hacked in or non-native.** TidalCycles can send MIDI with some setup (SuperDirt, custom configs), but it’s not core to the language—it’s more of a bolt-on. Sonic Pi has some support too, but again, it’s not fundamental. You’re proposing MIDI as a **first-class runtime** concept, which flips the paradigm: instead of music controlling MIDI, MIDI is _the fabric_ of the language.
3. **Neither can act as a plugin in a DAW.** They’re standalone tools, not embeddable runtimes. No VST/AU, no LV2. That completely limits how producers and composers can adopt them in hybrid setups. RhythmScript—written in Rust—_could_ be compiled into a shared library or hostable engine, opening up VST integration or even embedding into DAW controllers and external sequencers.
4. **Neither has a way to program song structure.** Sonic Pi _can_ receive MIDI input. You can write logic that reacts to incoming MIDI events using `live_loop` and `sync`. But there's a catch: the MIDI input is still event-based and very low-level. You're reacting imperatively to events, not using them to drive patterns or transform structured musical ideas. There's no real abstraction for song structure or arrangement.

This idea breaks out of the sandbox and becomes _infrastructure_. It could be used for:

- Live performance in a REPL or piped into Sonic Pi / TidalCycles
- Composing entire arrangements as code
- Generating MIDI for other instruments
- Embedding rhythmic logic into DAWs, VSTs, or hardware
- Lighting rigs in sync with music (especially via MIDI-to-DMX bridges)
- Mechanical instruments or robots (MIDI triggers mapped to servos or relays)
- Animatronics (Disney used MIDI for syncing show scenes for decades)
- Custom synth hardware, Eurorack modules, or DIY electronics
- Even puppetry and **kinetic sculpture** installations

There’s also a whole quiet world of engineers and artists who _already_ use MIDI to control things like Lighting rigs or animatronics. But they’re mostly wiring it all up with low-level tools: C++, Teensy/Arduino sketches, Max/MSP patches, hardware sequencers, or custom MIDI controller scripts. All of that is fragile, verbose, and full of edge-case timing bugs.

RhythmScript, by contrast, offers:

- **Declarative MIDI timelines** without writing a single loop or conditional
- **Composability and reusability** with musical semantics
- A **shared abstraction layer** that works the same for music, light, robots, or animation
- Potentially exportable event graphs or interpretable MIDI streams

## Technology

MIDI is treated as a first-class citizen in all major desktop OSes due to its age and importance in computing history. All major desktop operating systems maintain registries of MIDI ports natively:

- Windows: Device Manager category specifically for MIDI devices
- macOS: Audio MIDI Setup utility with a dedicated MIDI Studio view
- Linux: ALSA MIDI device nodes and sequencer interfaces

This native support means that when you write RhythmScript, you won't need to install special drivers or middleware for basic MIDI functionality. The language can directly access the system's MIDI services through libraries like `midir` in Rust, which are essentially wrappers around these native OS APIs. 

The MIDI-only approach also aligns perfectly with the language's philosophical goals - it's about the algorithmic composition of music, not the sonic texture creation. This keeps the language focused on what makes it unique.

From an implementation perspective, this means the Rust code only needs to interface with these excellent MIDI libraries:

- `midir` for cross-platform MIDI I/O
- `midly` for MIDI message encoding/decoding
- `wmidi` for type-safe MIDI message handling

We can always add audio capabilities later as an optional extension if there's demand, but starting with MIDI-only gives us a cleaner, more focused project that will be much easier to complete successfully.
## Declarative thinking

The language provides a host of built-in functions that all work on the underlying MIDI information. 
For example, ideas for functions (in no particular order):

|            |            |          |            |
| ---------- | ---------- | -------- | ---------- |
| .transform | .reverse   | .shuffle | .velocity  |
| .mutate    | .intensify | .tremolo | .transpose |
| .octave    | .solo      | mute     | .filter    |
| .arpeggio  | .scale     | .send    | .return    |
| .modulate  | .delay     | .push    | .pull      |
| .syncopate | .chord     | .speed   | .bar       |
| .attack    | .decay     | .sustain | .release   |
| .beat      | .bpm       | .pitch   | .interval  |
| .key       | .meter     | .rest    | .fade      |
A RhythmScript coder can use these built-ins as well as define their own custom functions which string together these built-ins in any combination they want. But, the emphasis is on keeping these function definitions as declarative as possible. People should feel like they're writing a musical score, not doing math.

RhythmScript aims to formalize the idea that **functions are “musical operators”**, and they’re declarative recipes for evolving streams over time. That can let us define a really clean mental model early-on:

- **Patterns** are **time-mapped value streams** (notes, velocities, CC messages)
- **Tracks** are **aggregations** of patterns, routed to MIDI channels
- **Operators** are **transformations** on patterns or tracks
- **Triggers** are **reactive hooks** that listen to musical time or events
- **The Clock** is the **world’s heartbeat**

## Looping / Hot Reloading

Hot reloadability is practically essential for a language like this because it feeds directly into the "live instrument" feel. You don’t want to be restarting a session every time you tweak a drum pattern or a melody walk.
RhythmScript allows for very simple looping syntax - like everything else in the language, loops are set declaratively. These look like simple [loop] and [/loop] statements. Or loop the entire song, or individual components. Its all easy to do.

Any edits made to the code while looping are automatically reflected in the running program. In addition, there will also be an included REPL if people want to play around in it, or even prefer to use it for live performances just because it looks cooler.

The hot reloading ability will be achieved because RhythmScript is architecturally more similar to CSS than an interpreted langauge (one might say a parsed language). Building RhythmScript as a parser lets us copy lessons from CSS and gives a much easier route to MVP.

## Timing

RhythmScript will endeavor to support advanced timing settings - sub-millisecond delays should likely be built-in (many composers will expect this). In addition, it would be great to eventually support other clock/sync protocols alongside MIDI clock, such as OSC Timestamping.

## A note on Reactivity

Reactivity should be a core part of the language and treated as a first class citizen. It will provide a very easy and declarative syntax for composers to set up triggers that can make complex MIDI-effect chains

## A note on MPE

The MIDI Polyphonic Expression protocol should eventually be supported in full. It may not be in the first version of the language but at the moment I have no reason to believe it will not eventually make its way in.

## A note on editors

RhythmScript does not come with an editor or require a special one. Your editor is your coding app. Or notepad if you're that type of person. Naturally I will plan to release extensions for all major code editors. You should be able to just download a plugin for your favorite editor and get syntax highlighting, AST parsing, and code completion.

In terms of usage, most people will likely want to use it to route input into Super Pi or TidalCycles. But it can be used in any program that processes MIDI information, including DAWs or simpler audio environments such as Cantabile.

Some kind of visual editing environment would certainly be a cool project down the road, but its not part of the language fundamentals. It would be a separate project unto itself.

## How RhythmScript and Sonic Pi Can Work Together

RhythmScript can drive Sonic Pi just like a MIDI controller—except instead of triggering notes with your hands, you write structured, time-based code to generate those events. This opens up a powerful hybrid workflow where **RhythmScript handles structure and timing**, while **Sonic Pi handles sound design and synthesis**.

Let’s say you’ve written a drum pattern and lead melody in RhythmScript:

```
clock.bpm = 120

pattern kick = every(1.beat)
pattern snare = every(2.beats, offset: 1.beat)
pattern melody = scale("C minor").walk().every(0.25.beat)

track drums = [kick, snare].mix().toChannel(1)
track lead = melody.toChannel(2)

output.to("IAC Bus 1")
clock.start()
```

Then in Sonic Pi:

```
live_loop :drums do
  use_real_time
  note, velocity = sync "/midi:iac_bus_1:1/note_on"
  sample :drum_heavy_kick if note == 36
  sample :drum_snare_soft if note == 38
end

live_loop :melody do
  use_real_time
  note, velocity = sync "/midi:iac_bus_1:2/note_on"
  synth :pluck, note: note, amp: velocity / 127.0
end
```

You’ve now split the responsibilities:

- RhythmScript composes and schedules the music.
- Sonic Pi plays the sound with your own synth and sample design.

RhythmScript acts as the **external composer**, generating structured, reactive, clock-accurate MIDI output. Sonic Pi acts as the **sound engine**, reacting to that MIDI input in real time. It’s no different than plugging in a MIDI keyboard, except now that "keyboard" is a fully programmable, time-aware language that can:

- Trigger sequences and melodies at exact musical times
- Evolve patterns over time without manual input
- Follow a formal song structure (verse, chorus, drop, etc.)
- Sync to an external DAW or hardware clock
- Trigger not just notes, but velocity curves, CC values, or any kind of expressive MIDI stream

With RhythmScript driving the MIDI port, Sonic Pi sees that input _as if it were a live instrument_—except the “instrument” is writing its own score and adapting it on the fly.

This means a live coder in Sonic Pi doesn’t have to worry about micromanaging timing anymore. They can focus entirely on timbre, synthesis, and FX chains, while RhythmScript takes care of the _when_ and _what_.

## Plugins

- **VST plugin**: For users who need to integrate directly with a DAW, generating **MIDI output** and controlling audio instruments.
- **OSC adapter**: For users who want to integrate RhythmScript with **visuals, lighting, interactive systems**, or **performance setups**.

### 1. **VST Plugin for Music Producers**

A **VST plugin** would be ideal for people who want to use RhythmScript directly in a **digital audio workstation (DAW)** or music production environment. This would allow **music producers** to incorporate the language into their existing workflow, interacting with MIDI, audio tracks, and DAW-specific features. The VST could:

- **Generate MIDI**: RhythmScript could create MIDI events that trigger instruments, drum machines, or synthesizers within the DAW.
- **Trigger audio events**: RhythmScript could send MIDI notes or CC values to trigger sounds in virtual instruments or external hardware.
- **Time Sync**: The plugin could sync the music generation with the DAW’s transport (play, stop, record) so that the rhythms, patterns, and musical sequences align with the rest of the production.

For example, a producer could set up **RhythmScript patterns** that generate beats and rhythms, then use the VST plugin to route the MIDI to virtual instruments (like drums, synthesizers, etc.) inside the DAW.

### Why VST?

- **Seamless DAW Integration**: Music production tools are built around the VST format, so making a VST plugin means the language can be used as part of an established toolchain.
- **MIDI Support**: VST plugins typically send and receive MIDI, making it easier for RhythmScript to interact with other instruments, plugins, and external MIDI devices.

---

### 2. **OSC Adapter for Broader Integration**

For users who want to interact with **live performance systems** or **interactive installations**, the **OSC adapter** would be more suitable. It allows you to integrate with non-audio-centric software and devices, such as:

- **Resolume Arena** (for video and visual performance)
- **Lighting control software** (e.g., for synchronized lighting effects in a live show)
- **Visual arts platforms** (like **TouchDesigner** or **Max/MSP**)
- **Robotics systems** (if you're controlling external hardware or interactive devices)

With an **OSC adapter**, RhythmScript can control the visuals, effects, and other aspects of live performances, giving **visual artists, VJs, or event planners** the ability to sync visual elements with the music and rhythms generated by RhythmScript. The OSC adapter would:

- **Send OSC messages** to control video clips, layers, effects, and transitions in software like Resolume.
- **Control lighting** and other multimedia aspects of a live performance.
- **Integrate with interactive installations**, where music or rhythm events trigger physical actions, lighting changes, or generative visuals.

### Why OSC?

- **Widely Supported**: OSC is supported by a broad range of multimedia software, including Resolume, TouchDesigner, and Max/MSP.
- **Low Latency**: OSC can operate in real-time, making it ideal for live performance control.
- **Flexible**: It can be adapted to many different types of programs or hardware.

---

# Comparison to SuperCollider

**Similarities:**

1. **Algorithmic Approach:** Both RhythmScript (as conceived) and SuperCollider allow you to define musical or sonic processes algorithmically, generating complex results from relatively simple rules or patterns.
2. **Time-Based:** Both are fundamentally concerned with events happening over time.
3. **Live Performance Potential:** Both are designed with real-time manipulation and live performance in mind (SuperCollider via its real-time synthesis, RhythmScript via hot-reloading, REPL, and MIDI focus).
4. **Pattern Generation:** Both offer powerful ways to generate and manipulate patterns.

**Key Differences (Where RhythmScript Stands Apart):**

1. **Core Domain: MIDI vs. Audio:**
    
    - **SuperCollider:** Its absolute core is **real-time audio synthesis and processing (DSP)**. `scsynth` (the server) is a powerful audio engine. `sclang` (the language) is primarily used to control this engine, generate audio structures, and perform DSP tasks. While it _can_ handle MIDI and OSC for control, its soul is in _generating sound_.
    - **RhythmScript:** Its absolute core is **MIDI timing, sequencing, transformation, and synchronization**. It treats _MIDI_ as the fundamental layer and _musical time_ (based on MIDI Clock) as the primary dimension. It's explicitly _not_ about generating audio itself, but about generating the _instructions_ (MIDI data) that tell other things (synths, drum machines, VSTs, lights, robots) what to do and precisely _when_.
    
2. **Language Philosophy:**
    
    - **SuperCollider (`sclang`):** A powerful, flexible, object-oriented language. It allows for various programming styles (imperative, functional, object-oriented). It's a more general-purpose language tailored for audio.
    - **RhythmScript:** Intentionally designed as **purely declarative**. The goal is to express _what_ should happen over time, not _how_ to execute it step-by-step. It focuses on patterns, transformations, and reactive rules within a constrained, time-centric model ("musical algebra").
    
3. **Architecture & Integration:**
    
    - **SuperCollider:** Typically runs as a client-server architecture (sclang controlling scsynth, often via OSC). While embedding is possible (e.g., libpd-like efforts, some VST experiments), it's often treated as a more self-contained environment.
    - **RhythmScript:** Designed from the ground up with **embeddability and integration** as a primary goal (Rust implementation, VST/library focus). The vision is for it to become _infrastructure_ that plugs into other systems (DAWs, visual software, hardware).
    
4. **Focus:**
    
    - **SuperCollider:** Sonic exploration, synthesis techniques, complex DSP, generative soundscapes.
    - **RhythmScript:** Rhythmic structure, polyrhythms, complex sequencing, MIDI manipulation, temporal logic, synchronizing diverse MIDI-controlled systems.