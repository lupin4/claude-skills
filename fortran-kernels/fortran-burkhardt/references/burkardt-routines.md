# Burkardt Routine Categories Reference

## Quadrature Routines

| Original Name | Modernized | Purpose |
|--------------|------------|---------|
| `legendre_compute` | `fm_gauss_legendre` | Gauss-Legendre nodes & weights |
| `hermite_compute` | `fm_gauss_hermite` | Gauss-Hermite for exp(-x²) integrals |
| `clenshaw_curtis_compute` | `fm_clenshaw_curtis` | Nested quadrature (good for adaptivity) |
| `patterson_set` | `fm_gauss_patterson` | Kronrod extension nodes |
| `simplex_unit_sample` | `fm_simplex_quadrature` | Integration over simplices |

## Interpolation Routines

| Original | Modernized | Purpose |
|----------|------------|---------|
| `lagrange_interp_1d` | `fm_lagrange_1d` | Lagrange polynomial interpolation |
| `spline_cubic_set/val` | `fm_spline_cubic` | Natural/clamped cubic spline |
| `rbf_interp_nd` | `fm_rbf_interp` | Radial basis function interpolation |
| `shepard_interp_nd` | `fm_shepard_interp` | Shepard's method (inverse distance) |
| `barycentric_interp_1d` | `fm_barycentric_1d` | Numerically stable polynomial interp |

## Random Number Routines

| Original | Modernized | Purpose |
|----------|------------|---------|
| `halton` | `fm_halton` | Halton quasi-random sequence |
| `sobol` | `fm_sobol` | Sobol low-discrepancy sequence |
| `r8_uniform_01` | `fm_uniform` | Uniform [0,1) |
| `r8_normal_01` | `fm_normal` | Standard normal via Box-Muller |
| `latin_center` | `fm_lhs_center` | Latin hypercube sampling |

## Modernization Checklist

- [ ] Remove all COMMON blocks → module variables or parameters
- [ ] Remove all EQUIVALENCE statements
- [ ] Add `implicit none` to every program unit
- [ ] Convert `REAL*8` → `real(8)` or `real(c_double)`
- [ ] Convert `GOTO` → structured `if/do/select case`
- [ ] Convert fixed-format → free-format (.f90)
- [ ] Add `intent(in/out/inout)` to all arguments
- [ ] Add `bind(C, name="fm_...")` wrapper for public routines
- [ ] Write test case with known-answer validation
