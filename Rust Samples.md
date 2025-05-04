# Architecture Chart

```
┌─────────────────────────────────────────────┐
│             Language Frontend               │
│ (Parsing, Type Checking, AST Construction)  │
└────────────────────┬────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────┐
│              Scheduler Core                 │
│   - Musical time management                 │
│   - Event queue with musical timestamps     │
│   - Pattern evaluation and expansion        │
└────────────────────┬────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────┐
│             MIDI Subsystem                  │
│   - MIDI event formatting                   │
│   - Output buffering and timing             │
│   - Device management                       │
└─────────────────────────────────────────────┘`
```

# Rust Code Samples

Note: The following is only a very rough idea of what some Rust code in the interpreter may look like. This is by no means indicative of the actual coding intentions.

```
struct MusicalEvent {
    time: MusicalTime,          // Musical position (beats, ticks)
    event_type: EventType,      // Note on, note off, CC change, etc.
    data: Vec<u8>,              // MIDI data bytes
    channel: u8,                // MIDI channel
    source: PatternReference,   // Reference to originating pattern
}

struct Scheduler {
    events: BinaryHeap<MusicalEvent>,  // Priority queue of future events
    clock: MusicalClock,               // Handles timing and tempo
    patterns: HashMap<String, Pattern>, // Active patterns
    // ...
}

impl Scheduler {
    fn schedule_event(&mut self, event: MusicalEvent) {
        self.events.push(event);
    }
    
    fn advance_to(&mut self, target_time: MusicalTime) {
        // Process all events up to target_time
        while let Some(event) = self.events.peek() {
            if event.time > target_time {
                break;
            }
            
            let event = self.events.pop().unwrap();
            self.process_event(event);
        }
        
        // Schedule more events from patterns
        self.expand_patterns(target_time);
    }
    
    fn process_event(&mut self, event: MusicalEvent) {
        // Convert to MIDI and send to output
        self.midi_system.send_event(event);
    }
    
    fn expand_patterns(&mut self, horizon: MusicalTime) {
        // Look at all active patterns and generate concrete events
        // up to the time horizon
        for pattern in self.patterns.values_mut() {
            let new_events = pattern.generate_events_until(horizon);
            for event in new_events {
                self.schedule_event(event);
            }
        }
    }
}
```

```
// Simplified Rust code for direct MIDI output
let mut midi_out = MidiOutput::new("RhythmScript")?;
let ports = midi_out.ports();
let port = &ports[port_index]; // Or find by name
let connection = midi_out.connect(port, "output")?;

// Later, when events fire:
connection.send(&[0x90, 60, 100]); // Note on, middle C, velocity 100
```

```
// A simplified architecture might look like:

struct RhythmScriptRuntime {
    scheduler: Scheduler,
    patterns: PatternManager,
    interpreter: Interpreter,
    // Common infrastructure
}

// Output interfaces
trait MidiOutput {
    fn send_event(&mut self, event: MidiEvent);
    fn start_clock(&mut self);
    fn stop_clock(&mut self);
}

struct DirectMidiBackend {
    connections: Vec<MidiConnection>,
    // Direct MIDI implementation
}

struct VstBackend {
    host: VstHost,
    // VST-specific implementation
}

// Both implement the same interface
impl MidiOutput for DirectMidiBackend { /* ... */ }
impl MidiOutput for VstBackend { /* ... */ }
```

```
// The core code might look like this:
fn main() {
    // Initialize the RhythmScript interpreter
    let mut runtime = RhythmScriptRuntime::new();
    
    // Load user script
    runtime.load_file("my_patterns.rs");
    
    // Create system MIDI output
    let midi_out = MidiOutput::new("RhythmScript")?;
    runtime.connect_to_midi_output(midi_out);
    
    // Run the scheduler loop
    runtime.start();
    
    // Event loop to handle timing
    loop {
        runtime.process_next_time_slice();
        std::thread::sleep(Duration::from_millis(1));
    }
}
```

```
// A simple VST wrapper might look like:

struct RhythmScriptVst {
    runtime: RhythmScriptRuntime,  // Reuse the same core runtime
    // VST-specific state
}

impl VstPlugin for RhythmScriptVst {
    fn process_events(&mut self, events: &Events) {
        // Convert VST events to RhythmScript events
        for event in events.events() {
            if let Event::Midi(midi_event) = event {
                self.runtime.process_midi_input(midi_event);
            }
        }
    }
    
    fn process(&mut self, buffer: &mut AudioBuffer<f32>) {
        // Process time slice and output MIDI events
        let midi_events = self.runtime.process_next_time_slice();
        
        // Send MIDI events to host
        for event in midi_events {
            self.send_midi_event_to_host(event);
        }
    }
    
    // Other VST interface methods...
}
```

```
// Example of creating virtual MIDI ports

use midir::{MidiOutput, MidiOutputPort};
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    // Get access to the system's MIDI services
    let midi_out = MidiOutput::new("RhythmScript")?;
    
    // List all available output ports
    let ports = midi_out.ports();
    for (i, port) in ports.iter().enumerate() {
        println!("Port {}: {}", i, midi_out.port_name(port)?);
    }
    
    // Connect to a specific port (or create virtual port on supported systems)
    let port = &ports[0]; // First available port
    let mut connection = midi_out.connect(port, "output-connection")?;
    
    // Send a MIDI message: Note On, C4 (60), velocity 100
    connection.send(&[0x90, 60, 100])?;
    
    // Later: Note Off
    connection.send(&[0x80, 60, 0])?;
    
    // Connection is closed when it goes out of scope
    Ok(())
}

// MacOS
#[cfg(target_os = "macos")]
fn create_virtual_port() -> Result<(), Box<dyn Error>> {
    use coremidi::{Client, Source, Destination};
    
    // Create a virtual source (outputs MIDI)
    let client = Client::new("RhythmScript")?;
    let source = Source::new(&client, "RhythmScript Output")?;
    
    // Now it's available for other applications
    // Send MIDI data through it
    let packet = coremidi::PacketBuffer::new(0, &[0x90, 60, 100]);
    source.received(&packet);
    
    Ok(())
}
```