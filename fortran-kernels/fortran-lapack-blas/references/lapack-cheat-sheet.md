# LAPACK/BLAS Cheat Sheet

## Naming Convention: XYYZZZ

- X: precision (S=single, D=double, C=complex, Z=double-complex)
- YY: matrix type (GE=general, SY=symmetric, TR=triangular, PO=positive-definite)
- ZZZ: operation (SV=solve, EV=eigenvalue, SVD=svd, TRF=factorize)

## Most-Used LAPACK Routines

| Routine | Purpose | Input | Output |
|---------|---------|-------|--------|
| DGESV | Solve Ax=b (general) | A(n,n), B(n,nrhs) | X in B, LU in A |
| DPOSV | Solve Ax=b (SPD) | A(n,n), B(n,nrhs) | X in B, Chol in A |
| DSYEV | Eigenvalues (symmetric) | A(n,n) | eigenvalues W, eigenvectors in A |
| DGEEV | Eigenvalues (general) | A(n,n) | WR, WI (real/imag parts), VL, VR |
| DGESVD | SVD | A(m,n) | U, S, VT |
| DGETRF | LU factorization | A(m,n) | LU in A, IPIV |
| DPOTRF | Cholesky factorization | A(n,n) | L or U in A |
| DGEQRF | QR factorization | A(m,n) | R in upper, reflectors in lower |
| DGELS | Least squares (overdetermined) | A(m,n), B(m,nrhs) | Solution in B |

## BLAS Quick Reference

| Routine | Signature | Operation |
|---------|-----------|-----------|
| DSCAL | (n, alpha, x, incx) | x = alpha * x |
| DCOPY | (n, x, incx, y, incy) | y = x |
| DAXPY | (n, alpha, x, incx, y, incy) | y = alpha*x + y |
| DDOT | (n, x, incx, y, incy) → result | dot(x, y) |
| DNRM2 | (n, x, incx) → result | ||x||₂ |
| DGEMV | (trans, m, n, alpha, A, lda, x, incx, beta, y, incy) | y = αAx + βy |
| DGEMM | (trA, trB, m, n, k, α, A, lda, B, ldb, β, C, ldc) | C = αAB + βC |

## Linking Flags by Platform

```bash
# Reference (slow, for testing only)
-llapack -lblas

# OpenBLAS (good default)
-lopenblas

# Intel MKL (ILP64)
-lmkl_gf_ilp64 -lmkl_sequential -lmkl_core -lpthread -lm

# macOS Accelerate
-framework Accelerate

# pkg-config (if available)
$(pkg-config --libs lapack blas)
```
