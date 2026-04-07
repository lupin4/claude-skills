# Signal Processing / Music — Claude Code Guidance

DSP and audio skills — FFT kernels, real-time audio, MIDI, DJ automation, saxophone synthesis, and spatial audio. Kernels in Fortran, I/O and orchestration in Zig.

## Skills Overview (6)

| Skill | Focus |
|-------|-------|
| **fordsp-kernel** | FFT (Cooley-Tukey, radix-3), filter banks, windowing |
| **fordsp-realtime** | Ring buffers, JACK/CoreAudio, lock-free audio I/O |
| **midi-zig** | MIDI parsing, CC mapping, clock sync in Zig |
| **dj-automation** | BPM detection, beatgrid, cue points, Camelot key detection |
| **sax-dsp** | Reed instrument physical modeling, waveguide synthesis |
| **audio-spatialize** | HRTF, ambisonics (FOA/HOA), binaural rendering |

## Key Conventions

- **Kernels:** Fortran for numerical DSP (FFT, filters, synthesis)
- **I/O:** Zig for audio device I/O, MIDI, real-time scheduling
- **Sample rate:** 44100/48000 Hz default, 96000 Hz for spatial audio
- **Buffer size:** 256 samples for low-latency, 1024 for batch processing
- **Precision:** real(8) for FFT/filter design, real(4) for real-time paths

## Cross-References

- fortran-kernels/ — LAPACK for matrix ops in spatial audio, SIMD for DSP
- zig-systems/ — Allocator patterns for audio buffers, real-time constraints
- synthetic-data/ — Audio rendering for robotics simulation environments
