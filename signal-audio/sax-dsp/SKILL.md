---
name: "sax-dsp"
description: "Use when implementing Reed instrument physical modeling synthesis, waveguide bore simulation, formant filters for saxophone timbre, breath pressure mapping."
---

# Sax DSP

Reed instrument physical modeling synthesis, waveguide bore simulation, formant filters for saxophone timbre, breath pressure mapping. Kernels in Fortran, I/O and orchestration in Zig.

---

## Overview

## Physical Modeling: Digital Waveguide

```
Mouthpiece (reed excitation)
    ↓
Bore (delay line pair — forward + backward traveling waves)
    ↓
Bell (radiation impedance + reflection)
    ↓
Output (sum of forward wave at bell)
```

```fortran
subroutine fm_waveguide_step(forward, backward, n, reflection_coeff, output) &
    bind(C, name="fm_waveguide_step")
  integer(c_int), intent(in), value :: n
  real(c_double), intent(inout) :: forward(n), backward(n)
  real(c_double), intent(in), value :: reflection_coeff
  real(c_double), intent(out) :: output
  ! Shift delay lines
  ! Apply reflection at bell end
  ! Apply reed excitation at mouthpiece
  output = forward(n)  ! Radiated wave
end subroutine
```

## Saxophone Formant Frequencies

| Formant | Frequency (Hz) | Bandwidth (Hz) |
|---------|----------------|----------------|
| F1 | 520 | 60 |
| F2 | 1480 | 90 |
| F3 | 2500 | 120 |
| F4 | 3500 | 150 |
| F5 | 4950 | 200 |

---

## Anti-Patterns

- **Allocating in audio callbacks** — lock-free only in real-time path
- **Using Python for DSP kernels** — Fortran for numerics, Zig for I/O
- **Ignoring latency budget** — buffer size determines max processing time

---

## Cross-References

- [fortran-kernels/](../../fortran-kernels/) — LAPACK for matrix ops, SIMD hints
- [zig-systems/](../../zig-systems/) — Allocator patterns for audio buffers
