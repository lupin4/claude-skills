# Error Mapping Patterns

## Standard Status Code Ranges

| Range | Meaning | Zig Handling |
|-------|---------|-------------|
| 0 | Success | Return void/{} |
| -1 to -99 | Input validation errors | Specific error enum |
| -100 to -199 | Numerical errors (singular, not converged) | Specific error enum |
| -200+ | System errors (memory, I/O) | Specific error enum |
| 1+ | Warnings (non-fatal) | Log and continue |

## Per-Domain Error Sets

```zig
// Physics simulation
pub const PhysicsError = error{ InvalidTimestep, PenetrationDetected, SolverDiverged };

// FFT/DSP
pub const DSPError = error{ InvalidLength, NotPowerOfTwo, NyquistViolation };

// ML/Optimization
pub const OptError = error{ NotConverged, GradientExplosion, NaNDetected };
```
