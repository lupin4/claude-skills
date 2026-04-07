---
name: "fordsp-kernel"
description: "Use when implementing FFT (Cooley-Tukey, radix-3 ternary), mixed-radix, filter bank design, windowing functions, real-valued FFT optimizations in Fortran."
---

# forDSP Kernel

FFT (Cooley-Tukey, radix-3 ternary), mixed-radix, filter bank design, windowing functions, real-valued FFT optimizations in Fortran. Kernels in Fortran, I/O and orchestration in Zig.

---

## Overview

## Cooley-Tukey FFT

Radix-2 DIT (Decimation in Time):

X[k] = Σ_{n=0}^{N-1} x[n] · W_N^{nk},  where W_N = e^{-j2π/N}

```fortran
! In-place radix-2 FFT
subroutine fm_fft_radix2(x_re, x_im, n, inverse, status) &
    bind(C, name="fm_fft_radix2")
  integer(c_int), intent(in), value :: n, inverse
  real(c_double), intent(inout) :: x_re(n), x_im(n)
  integer(c_int), intent(out) :: status
  real(c_double) :: angle, wr, wi, tr, ti
  integer :: m, mmax, j, istep, i
  
  ! Bit-reversal permutation
  j = 1
  do i = 1, n - 1
    if (i < j) then
      tr = x_re(j); ti = x_im(j)
      x_re(j) = x_re(i); x_im(j) = x_im(i)
      x_re(i) = tr; x_im(i) = ti
    end if
    m = n / 2
    do while (m >= 2 .and. j > m)
      j = j - m; m = m / 2
    end do
    j = j + m
  end do
  
  ! Butterfly stages
  mmax = 1
  do while (mmax < n)
    istep = 2 * mmax
    angle = -3.141592653589793d0 / dble(mmax)
    if (inverse /= 0) angle = -angle
    ! ... butterfly computation ...
    mmax = istep
  end do
  
  if (inverse /= 0) then
    x_re = x_re / dble(n)
    x_im = x_im / dble(n)
  end if
  status = 0
end subroutine
```

## Radix-3 Ternary FFT

For lengths N = 3^k. Twiddle factors: W_3 = e^{-j2π/3}

```fortran
! Radix-3 butterfly: uses ternary decomposition
! 17x speedup over radix-2 for 3^k lengths
subroutine fm_fft_radix3(x_re, x_im, n, status) &
    bind(C, name="fm_fft_radix3")
  ! Specialized for n = 3^k
end subroutine
```

## Windowing Functions

| Window | Main Lobe | Side Lobe | Use |
|--------|----------|----------|-----|
| Hann | 4 bins | -31 dB | General analysis |
| Hamming | 4 bins | -43 dB | Speech processing |
| Blackman | 6 bins | -58 dB | High dynamic range |
| Kaiser (β=8) | 5 bins | -65 dB | Configurable |
| Flat-top | 10 bins | -93 dB | Amplitude accuracy |

---

## Anti-Patterns

- **Allocating in audio callbacks** — lock-free only in real-time path
- **Using Python for DSP kernels** — Fortran for numerics, Zig for I/O
- **Ignoring latency budget** — buffer size determines max processing time

---

## Cross-References

- [fortran-kernels/](../../fortran-kernels/) — LAPACK for matrix ops, SIMD hints
- [zig-systems/](../../zig-systems/) — Allocator patterns for audio buffers
