# AI/ML Kernels — Claude Code Guidance

Machine learning primitive skills implemented in Fortran+Zig — tensor operations, quantization, training loops, Bayesian optimization, and Hopfield-attention kernels. NOT Python/PyTorch wrappers.

## Skills Overview (5)

| Skill | Focus |
|-------|-------|
| **fortorch-tensor-model** | View-based refcounted tensor memory, RSSM, VQ patterns |
| **forml-turboquant** | PolarQuant + QJL extreme quantization (arxiv 2504.19874) |
| **forai-training-loop** | Training infrastructure, RL, KV cache in Fortran+Zig |
| **forbayes-gp** | Gaussian process surrogates, Pareto dominance, MCMC |
| **hopfield-attention** | Energy-minimizing retrieval, Hopfield↔attention equivalence |

## Key Conventions

- **Implementation language:** Fortran kernels + Zig orchestration (not Python)
- **Memory layout:** Column-major (Fortran-native) for kernels, row-major adapters for external interop
- **Precision:** Double (real(8)) default, single (real(4)) for quantized paths
- **BLAS dependency:** All matrix ops route through BLAS/LAPACK (see fortran-kernels/fortran-lapack-blas)
- **Tensor model:** View-based with reference counting — no implicit copies

## Cross-References

- fortran-kernels/ — LAPACK/BLAS, module design patterns
- zortran/ — Fortran↔Zig interop for kernel wrapping
- physics-sim/forsim-differentiable — AD integration with forTorch
