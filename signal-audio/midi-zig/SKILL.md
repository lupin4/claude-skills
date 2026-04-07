---
name: "midi-zig"
description: "Use when implementing MIDI message parsing (status/data bytes, running status), CC mapping, MIDI clock (24 PPQ), SysEx, MIDI file reading in Zig."
---

# MIDI-Zig

MIDI message parsing (status/data bytes, running status), CC mapping, MIDI clock (24 PPQ), SysEx, MIDI file reading in Zig. Kernels in Fortran, I/O and orchestration in Zig.

---

## Overview

## MIDI Message Parsing

```zig
const MidiMessage = union(enum) {
    note_on: struct { channel: u4, note: u7, velocity: u7 },
    note_off: struct { channel: u4, note: u7, velocity: u7 },
    cc: struct { channel: u4, controller: u7, value: u7 },
    pitch_bend: struct { channel: u4, value: u14 },
    program_change: struct { channel: u4, program: u7 },
    clock: void,
    start: void,
    stop: void,
    sysex: []const u8,
};

fn parseMidi(byte: u8, state: *ParserState) ?MidiMessage {
    if (byte & 0x80 != 0) {
        // Status byte
        state.status = byte;
        state.data_index = 0;
        if (byte == 0xF8) return .clock;
        if (byte == 0xFA) return .start;
        if (byte == 0xFC) return .stop;
        return null;  // Wait for data bytes
    }
    // Data byte — use running status
    state.data[state.data_index] = byte;
    state.data_index += 1;
    // Check if message is complete
    return buildMessage(state);
}
```

## CC Number Catalog

| CC# | Name | Use |
|-----|------|-----|
| 1 | Mod Wheel | Vibrato, filter sweep |
| 7 | Volume | Channel volume |
| 10 | Pan | Stereo position |
| 11 | Expression | Dynamic volume |
| 64 | Sustain | Pedal on/off |
| 74 | Cutoff | Filter frequency (MPE) |

---

## Anti-Patterns

- **Allocating in audio callbacks** — lock-free only in real-time path
- **Using Python for DSP kernels** — Fortran for numerics, Zig for I/O
- **Ignoring latency budget** — buffer size determines max processing time

---

## Cross-References

- [fortran-kernels/](../../fortran-kernels/) — LAPACK for matrix ops, SIMD hints
- [zig-systems/](../../zig-systems/) — Allocator patterns for audio buffers
