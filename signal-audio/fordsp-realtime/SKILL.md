---
name: "fordsp-realtime"
description: "Use when implementing Ring buffer design for audio, JACK/CoreAudio callbacks, lock-free inter-thread communication, sample rate conversion, buffer sizing in Zig."
---

# forDSP Real-Time

Ring buffer design for audio, JACK/CoreAudio callbacks, lock-free inter-thread communication, sample rate conversion, buffer sizing in Zig. Kernels in Fortran, I/O and orchestration in Zig.

---

## Overview

## Ring Buffer for Audio

```zig
const AudioRingBuffer = struct {
    buffer: []f64,
    write_pos: std.atomic.Value(usize),
    read_pos: std.atomic.Value(usize),
    capacity: usize,

    pub fn write(self: *AudioRingBuffer, data: []const f64) usize {
        const wp = self.write_pos.load(.acquire);
        const rp = self.read_pos.load(.acquire);
        const available = self.capacity - (wp - rp);
        const to_write = @min(data.len, available);
        for (0..to_write) |i| {
            self.buffer[(wp + i) % self.capacity] = data[i];
        }
        self.write_pos.store(wp + to_write, .release);
        return to_write;
    }
};
```

## JACK Client Pattern

```zig
// JACK callback — must complete within buffer period
fn processCallback(nframes: u32, arg: ?*anyopaque) callconv(.C) c_int {
    const client: *JackClient = @ptrCast(@alignCast(arg));
    const input = jack.port_get_buffer(client.input_port, nframes);
    const output = jack.port_get_buffer(client.output_port, nframes);
    
    // Process audio — must be lock-free, no allocation, no I/O
    fortran.fm_process_audio(input, output, @intCast(nframes));
    
    return 0;
}
```

## Buffer Size vs Latency

| Buffer Size | Latency @48kHz | Use Case |
|------------|----------------|----------|
| 64 samples | 1.3 ms | Live monitoring |
| 128 samples | 2.7 ms | Live performance |
| 256 samples | 5.3 ms | Low-latency recording |
| 512 samples | 10.7 ms | Standard recording |
| 1024 samples | 21.3 ms | Mixing/mastering |
| 2048 samples | 42.7 ms | Offline processing |

---

## Anti-Patterns

- **Allocating in audio callbacks** — lock-free only in real-time path
- **Using Python for DSP kernels** — Fortran for numerics, Zig for I/O
- **Ignoring latency budget** — buffer size determines max processing time

---

## Cross-References

- [fortran-kernels/](../../fortran-kernels/) — LAPACK for matrix ops, SIMD hints
- [zig-systems/](../../zig-systems/) — Allocator patterns for audio buffers
