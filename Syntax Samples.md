
# RhythmScript Samples

Note: the following is only very rough sketches of what the syntax might look like. 

```
// Set global tempo and time signature
clock.bpm = 120
clock.timeSignature = 4/4

// Define a rhythmic pattern (returns notes at proper timings)
pattern kick = every(1.beat)
pattern snare = every(2.beats, offset: 1.beat)
pattern hihat = every(0.5.beats).withVelocity([100, 80, 90, 70])

// Mathematical transformations that stay in sync
pattern melody = scale("C minor").randomWalk()
                .transform(t => sin(t * π) * 12).quantize("C minor")
                .every(0.25.beats)
                
// All patterns automatically stay synchronized to the clock
track drums = [kick, snare, hihat].mix().toChannel(1)
track lead = melody.toChannel(2)

// Start playback - everything aligned to beats automatically
clock.start()
```

```
// Create a melody pattern
pattern melody = cMajor.walk(stepSize: [-2, 1, 2], length: 8)
                       .rhythm([1/8, 1/8, 1/4])
                       .velocity(mf, var: 10)

// Apply transformations
melody.transpose(5).loop(4).send(channel: 1)
```

```
// Define global song parameters
song "Algorithmic Dreams" {
  tempo = 124.bpm
  key = "F minor"
  
  // Define patterns as reusable components
  pattern bassline = arpeggio(chord("Fm7"), direction: "up")
                    .rhythm([1/8, 1/8, 1/4, 1/8, 1/4])
                    .velocity(dynamic: mf, groove: "swing")
  
  pattern lead = scale(key).seedRandom(42).walk([-2, 1, 3])
                .filter(note => isInChord(note, currentChord()))
  
  pattern drums = {
    kick: every(1.beat),
    snare: every(2.beats, offset: 1.beat),
    hihat: every(0.25.beats).velocityPattern([100, 70, 90, 60])
  }
  
  // Define song structure with chord progressions
  section intro(bars: 4) {
    chords = ["Fm7", "Bbm7"] * 2
    play drums.sparse(0.5)
    play bassline.octave(-1)
  }
  
  section verse(bars: 8) {
    chords = ["Fm7", "Bbm7", "Eb7", "Ab"] * 2
    play drums
    play bassline.octave(-1)
    play lead.withEffect(delay(0.5.beat, feedback: 0.3))
  }
  
  section chorus(bars: 8) {
    chords = ["Ab", "Bb7", "Gm7", "Fm7"] * 2
    play drums.intensify(1.2)
    play bassline.octave(-1).density(2)
    play lead.transpose(12).withEffect(reverb(size: 0.8))
  }
  
  // Arrange the full song
  arrangement = [
    intro,
    verse,
    chorus,
    verse, 
    chorus * 2,
    intro.fade(out: 4.bars)
  ]
}

// Export the song to MIDI and/or play it
export("algorithmic_dreams.mid")
play()
```

```
// Basic step patterns with bit arrays
pattern kick   = [1000 1000 1000 1000]  // Quarter notes
pattern snare  = [0000 1000 0000 1000]  // Beats 2 and 4
pattern hihat  = [1010 1010 1010 1010]  // Eighth notes
pattern shaker = [1111 1111 1111 1111]  // Sixteenth notes

// Extended syntax with velocity values (0-9)
pattern kick_groove   = [9000 7000 8000 5030]  // With velocity accents
pattern hihat_groove  = [8040 6040 7040 5040]  // With dynamic variation

// Pattern manipulation
pattern kick_fill = kick.last(1.bar).densify([0010 0100 1010 1111])
pattern snare_roll = [0000 0000 0000 1234 5678 9999]

// Combining with other language features
if (currentBar % 8 == 7) {
  play kick_fill
} else {
  play kick
}

// Conditional patterns based on step position
pattern complex = (step) => {
  if (step % 16 < 8) return [1010 1010]
  else return [1111 0000]
}
```

```
// 2D grid pattern (rows = instruments, columns = steps)
patterns drums = [
  kick:  [9000 8000 9000 8000],
  snare: [0000 8000 0000 8000],
  hihat: [8080 8080 8080 8080],
  tom:   [0000 0000 0030 0407]
]

// Operations on the entire grid
drums.density(2)  // Double the density of all patterns
drums.shuffle(0.2)  // Add 20% shuffle/swing to all patterns
drums.every(4.bars, transform: (p) => p.reverse())  // Every 4 bars, reverse all patterns

// Manipulate a pattern with code
pattern base = [1010 1010 1010 1010]
pattern evolved = base.evolve(iterations: 8, mutation: 0.2)
pattern complex = base.convolve([0.5, 1.0, 0.7, 0.3])

// Grooves evolving over time
pattern bassline = [1000 0000].rotate(currentBar % 8).swing(60).humanize(3)
```

```
// In RhythmScript VST:
pattern kick = [1000 1000 1000 1000]  // Basic kick pattern
pattern snare = [0000 1000 0000 1000]  // Snare on 2nd and 4th beats

// MIDI Clock Sync
sync_to(DAW)

// Define pattern and sequence
play kick for 4.bars at 0.bars
play snare for 4.bars at 1.bar

// Trigger external event (lighting, robot, etc.)
when (currentBar % 4 == 0) {
  trigger "strobe_light" with intensity=0.9
}
```

```
// Connect to system MIDI ports by name or index
midi.connect("Arturia KeyLab")
midi.connect(0)  // First available port

// Or list available ports
available_ports = midi.list_ports()

// Send patterns to specific ports
pattern bassline = [1000 0010 1000 0010].notes("C2", "G2")
bassline.send(port: "Arturia KeyLab", channel: 3)

// Send to all connected MIDI devices
midi.broadcast_enable()
pattern drums = [kickPattern, snarePattern, hiHatPattern].mix()
drums.play()  // Automatically sent to all connected devices

// Create a virtual MIDI output that other apps can see
midi.create_virtual_port("RhythmScript Output")

// Then route patterns to it
myPattern.send(port: "RhythmScript Output")

// Script can detect environment
if runtime.is_vst {
    // VST-specific functionality
    sync_to_host()
} else {
    // Standalone functionality
    clock.bpm = 120
}

// Most code remains identical regardless of mode
```

```
// Define a pattern once
pattern mainBeat = [9000 0300 8000 0400].swing(0.2)

// Send to multiple destinations with perfect synchronization
mainBeat.toMIDI(port: "Sonic Pi", channel: 1)  // For audio
mainBeat.toOSC("/resolume/layer1/opacity", mapping: {
  1: 1.0,  // Full opacity on beats
  0: 0.3   // Lower opacity between beats
})
mainBeat.toDMX(universe: 1, channel: 5, mapping: {
  velocity: "brightness"  // Map note velocity to light brightness
})
```

```
// Complex polyrythm example

pattern rhythm1 = [1000 1000 1000 1000]  // 4/4 pattern
pattern rhythm2 = [100 100 100]           // 3/4 pattern

// The same pattern driving multiple outputs
rhythm1.evolve(mutation: 0.2)
       .toMIDI(channel: 1)                // Music
       .toOSC("/visuals/layer1")          // Visuals 
       .toDMX(channels: [1, 2, 3])        // Lights

rhythm2.evolve(mutation: 0.1)
       .toMIDI(channel: 2)                // Second musical element
       .toOSC("/visuals/layer2")          // Second visual element
       .toDMX(channels: [4, 5, 6])        // More lights
```