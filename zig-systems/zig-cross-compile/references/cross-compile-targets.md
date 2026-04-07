# Cross-Compilation Target Reference

## Zig Target Triples

```bash
# List all available targets
zig targets

# Common forKernels targets
zig build -Dtarget=x86_64-linux-gnu
zig build -Dtarget=aarch64-linux-gnu
zig build -Dtarget=x86_64-macos
zig build -Dtarget=aarch64-macos
```

## Fortran Cross-Compiler Matrix

| Host | Target | Compiler | Package |
|------|--------|----------|---------|
| x86_64-linux | aarch64-linux | aarch64-linux-gnu-gfortran | gcc-aarch64-linux-gnu |
| x86_64-linux | x86_64-macos | osxcross + gfortran | Custom build |
| aarch64-linux | x86_64-linux | x86_64-linux-gnu-gfortran | gcc-x86-64-linux-gnu |
| Any | Any | flang --target=TRIPLE | LLVM flang |

## Platform-Specific Linker Flags

| Platform | libgfortran | Math | Notes |
|----------|-------------|------|-------|
| Linux | -lgfortran | -lm | Standard |
| macOS | -lgfortran (Homebrew) | -lm | /opt/homebrew/lib |
| macOS (Accelerate) | -lgfortran | -framework Accelerate | Replaces BLAS/LAPACK |
