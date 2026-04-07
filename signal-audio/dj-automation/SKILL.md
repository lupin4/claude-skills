---
name: "dj-automation"
description: "Use when implementing Autocorrelation BPM detection, onset detection, beatgrid structure, cue point types, energy computation, key detection (Camelot wheel)."
---

# DJ Automation

Autocorrelation BPM detection, onset detection, beatgrid structure, cue point types, energy computation, key detection (Camelot wheel). Kernels in Fortran, I/O and orchestration in Zig.

---

## Overview

## BPM Detection via Autocorrelation

```fortran
subroutine fm_bpm_detect(audio, n, sample_rate, bpm, confidence, status) &
    bind(C, name="fm_bpm_detect")
  integer(c_int), intent(in), value :: n, sample_rate
  real(c_double), intent(in) :: audio(n)
  real(c_double), intent(out) :: bpm, confidence
  integer(c_int), intent(out) :: status
  ! 1. Compute onset strength envelope
  ! 2. Autocorrelation of onset envelope
  ! 3. Find peak in BPM range [60, 200]
  ! 4. bpm = 60 * sample_rate / peak_lag
  ! 5. confidence = peak_height / mean_height
end subroutine
```

## Beatgrid Structure

```zig
const Beatgrid = struct {
    first_beat_sample: u64,   // Sample position of first downbeat
    bpm: f64,                  // Beats per minute
    beats_per_bar: u8 = 4,     // Time signature numerator
    
    fn sampleAtBeat(self: Beatgrid, beat: f64, sample_rate: u32) u64 {
        const samples_per_beat = @as(f64, @floatFromInt(sample_rate)) * 60.0 / self.bpm;
        return self.first_beat_sample + @intFromFloat(beat * samples_per_beat);
    }
};
```

## Camelot Wheel

| Key | Camelot | Compatible |
|-----|---------|-----------|
| C major | 8B | 7B, 9B, 8A |
| A minor | 8A | 7A, 9A, 8B |
| G major | 9B | 8B, 10B, 9A |
| E minor | 9A | 8A, 10A, 9B |

---

## Anti-Patterns

- **Allocating in audio callbacks** — lock-free only in real-time path
- **Using Python for DSP kernels** — Fortran for numerics, Zig for I/O
- **Ignoring latency budget** — buffer size determines max processing time

---

## Cross-References

- [fortran-kernels/](../../fortran-kernels/) — LAPACK for matrix ops, SIMD hints
- [zig-systems/](../../zig-systems/) — Allocator patterns for audio buffers
