# F77 → F2008 Migration Checklist

## Per-File Checklist

- [ ] Rename `.f` → `.f90`
- [ ] Convert to free format (remove column restrictions)
- [ ] `C`/`c`/`*` comments → `!` comments
- [ ] Add `implicit none` after every `module`/`subroutine`/`function`/`program`
- [ ] Add `intent(in/out/inout)` to all dummy arguments
- [ ] Replace `REAL*8` → `real(8)` or `real(c_double)`
- [ ] Replace `INTEGER*4` → `integer` or `integer(c_int)`
- [ ] Replace `CHARACTER*N` → `character(len=N)`
- [ ] Replace `COMMON` blocks → `use module`
- [ ] Replace `EQUIVALENCE` → explicit variables
- [ ] Replace `GOTO` → `if/then/else`, `do`, `select case`, `exit`, `cycle`
- [ ] Replace `SAVE` (implicit) → explicit `save` attribute or module scope
- [ ] Replace statement functions → internal functions or `contains` functions
- [ ] Remove sequence numbers (columns 73-80)
- [ ] Verify compilation with `-Wall -Wextra -std=f2008`

## Common GOTO Replacement Patterns

| F77 Pattern | F2008 Replacement |
|------------|-------------------|
| IF (..) GOTO label | IF (..) THEN ... END IF |
| GOTO (10,20,30), I | SELECT CASE (I) |
| DO 10 I=1,N ... 10 CONTINUE | DO I=1,N ... END DO |
| GOTO top_of_loop | CYCLE |
| GOTO end_of_loop | EXIT |

## Compiler Flags for Migration Validation

```bash
# GFortran
gfortran -std=f2008 -Wall -Wextra -fimplicit-none -pedantic

# Intel Fortran
ifort -stand f08 -warn all -check all

# Flang
flang -std=f2008 -Weverything
```
