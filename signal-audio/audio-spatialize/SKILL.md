---
name: "audio-spatialize"
description: "Use when implementing HRTF convolution, ambisonics (FOA/HOA) encoding/decoding, binaural rendering, distance attenuation, sound source localization for robotics."
---

# Audio Spatialize

HRTF convolution, ambisonics (FOA/HOA) encoding/decoding, binaural rendering, distance attenuation, sound source localization for robotics. Kernels in Fortran, I/O and orchestration in Zig.

---

## Overview

## HRTF Convolution

```fortran
! Apply HRTF to mono source → binaural output
subroutine fm_hrtf_apply(mono, n, hrtf_left, hrtf_right, hrtf_len, &
    out_left, out_right) bind(C, name="fm_hrtf_apply")
  integer(c_int), intent(in), value :: n, hrtf_len
  real(c_double), intent(in) :: mono(n), hrtf_left(hrtf_len), hrtf_right(hrtf_len)
  real(c_double), intent(out) :: out_left(n + hrtf_len - 1)
  real(c_double), intent(out) :: out_right(n + hrtf_len - 1)
  ! Convolution via FFT: out_left = IFFT(FFT(mono) * FFT(hrtf_left))
end subroutine
```

## Ambisonics (First-Order)

Encode source at (azimuth θ, elevation φ):
- W = signal (omnidirectional)
- X = signal · cos(θ)·cos(φ) (front-back)
- Y = signal · sin(θ)·cos(φ) (left-right)
- Z = signal · sin(φ) (up-down)

## Distance Attenuation Models

| Model | Formula | Use |
|-------|---------|-----|
| Inverse | 1/d | Simple, natural |
| Inverse square | 1/d² | Physical (free field) |
| Linear | max(0, 1 - d/d_max) | Games, predictable |
| Logarithmic | 1/(1 + a·log(d)) | Perceptual |

---

## Anti-Patterns

- **Allocating in audio callbacks** — lock-free only in real-time path
- **Using Python for DSP kernels** — Fortran for numerics, Zig for I/O
- **Ignoring latency budget** — buffer size determines max processing time

---

## Cross-References

- [fortran-kernels/](../../fortran-kernels/) — LAPACK for matrix ops, SIMD hints
- [zig-systems/](../../zig-systems/) — Allocator patterns for audio buffers
