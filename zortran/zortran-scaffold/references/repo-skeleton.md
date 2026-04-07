# Zortran Repo Skeleton Reference

## Complete Directory Template

```
for{Name}/
├── build.zig
├── DEPENDENCY_RULES.md
├── README.md
├── .gitignore
├── src/
│   ├── fortran/
│   │   ├── fm_{name}_core.f90      # Core module
│   │   ├── fm_{name}_utils.f90     # Utility functions
│   │   └── ft_{name}_types.f90     # Type definitions
│   ├── zig/
│   │   ├── main.zig                # Executable entry (optional)
│   │   ├── fortran.zig             # extern fn declarations
│   │   ├── lib.zig                 # Public Zig API
│   │   └── types.zig               # Zig type wrappers
│   └── c/
│       └── .gitkeep                # For third-party C headers
├── tests/
│   ├── test_kernels.zig
│   └── test_data/
│       └── .gitkeep
├── lib/
│   └── .gitkeep                    # Pre-built dependency archives
└── docs/
    └── DESIGN.md
```

## Naming Convention Quick Reference

| Element | Convention | Example |
|---------|-----------|---------|
| Repo name | `for{Domain}` | `forSim`, `forDSP`, `forSec` |
| Fortran module | `fm_{domain}_{op}` | `fm_rigid_body_step` |
| Fortran type | `ft_{domain}_{name}_t` | `ft_tensor_t` |
| Fortran ML func | `fml_{operation}` | `fml_forward_pass` |
| Fortran subroutine | `fs_{operation}` | `fs_init_scene` |
| Zig test file | `test_{domain}.zig` | `test_rigid_body.zig` |
| Archive | `libfor{domain}.a` | `libforsim.a` |
| Fortran source | `fm_*.f90` or `ft_*.f90` | `fm_fft.f90` |

## build.zig Boilerplate Checklist

- [ ] `standardTargetOptions` and `standardOptimizeOption` configured
- [ ] Fortran compile step with `-fPIC -O2 -J zig-out/mod`
- [ ] Archive creation step with `ar rcs`
- [ ] Zig library linking Fortran archive + libgfortran
- [ ] Test step with same library linkage
- [ ] Step dependencies correctly chained (fortran → ar → zig)

## .gitignore Template

```
zig-out/
zig-cache/
.zig-cache/
*.o
*.mod
*.a
*.so
*.dylib
```

## Fortran Module Template

```fortran
module fm_{name}_core
  use iso_c_binding
  implicit none
  private

  public :: fm_{name}_init, fm_{name}_compute

contains

  subroutine fm_{name}_init(n, status) bind(C, name="fm_{name}_init")
    integer(c_int), intent(in), value :: n
    integer(c_int), intent(out) :: status
    ! Implementation
    status = 0
  end subroutine

  function fm_{name}_compute(x, n) result(y) bind(C, name="fm_{name}_compute")
    integer(c_int), intent(in), value :: n
    real(c_double), intent(in) :: x(n)
    real(c_double) :: y
    ! Implementation
    y = sum(x)
  end function

end module fm_{name}_core
```

## Zig fortran.zig Template

```zig
// Auto-generated extern declarations for Fortran bind(C) exports
// Keep in sync — see zortran-symbol-sync

pub extern fn fm_{name}_init(n: c_int, status: *c_int) void;
pub extern fn fm_{name}_compute(x: [*]const f64, n: c_int) f64;
```
