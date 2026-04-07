# SIMD Directive Reference by Compiler

## GFortran Directives

| Directive | Purpose |
|-----------|---------|
| `!GCC$ ivdep` | Ignore vector dependencies |
| `!GCC$ unroll N` | Unroll loop N times |
| `-ftree-vectorize` (flag) | Enable auto-vectorization (on at -O2+) |
| `-fopt-info-vec` (flag) | Report vectorization decisions |

## Intel Fortran (ifort/ifx)

| Directive | Purpose |
|-----------|---------|
| `!dir$ simd` | Force SIMD vectorization |
| `!dir$ ivdep` | Ignore vector dependencies |
| `!dir$ vector always` | Override profitability heuristic |
| `!dir$ noinline` | Prevent inlining (for security-critical code) |
| `!dir$ unroll(N)` | Unroll N times |

## OpenMP (Portable)

| Directive | Purpose |
|-----------|---------|
| `!$omp simd` | Vectorize loop |
| `!$omp simd reduction(+:var)` | Vectorize with reduction |
| `!$omp simd aligned(x:32)` | Assert alignment |
| `!$omp declare simd` | Vectorize function for SIMD calls |
| `!$omp parallel do simd` | Parallelize + vectorize |
| `!$omp target teams distribute` | GPU offload |

## Vectorization Blocker Checklist

- [ ] No loop-carried dependencies (a(i) depends on a(i-1))
- [ ] No function calls without `!$omp declare simd`
- [ ] Array access is contiguous (stride-1 inner dimension)
- [ ] No conditional exits (EXIT/GOTO out of loop)
- [ ] No I/O in loop body
- [ ] No ALLOCATE/DEALLOCATE in loop body
- [ ] Trip count is known at compile time (or large enough)
