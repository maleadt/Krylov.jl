---
title: 'Krylov.jl: A Julia basket of hand-picked Krylov methods'
tags:
  - Julia
  - linear algebra
  - Krylov methods
  - sparse linear systems
authors:
  - name: Alexis Montoison^[corresponding author]
    orcid: 0000-0002-3403-5450
    affiliation: 1
  - name: Dominique Orban
    orcid: 0000-0002-8017-7687
    affiliation: 1
affiliations:
 - name: GERAD and Department of Mathematics and Industrial Engineering, Polytechnique Montréal, QC, Canada.
   index: 1
date: 29 July 2022
bibliography: paper.bib
header-includes: |
  \usepackage{booktabs}
  \usepackage{fontspec}
  \setmonofont[Path = ./, Scale=0.7]{JuliaMono-Regular.ttf}
---

# Summary

[Krylov.jl](https://github.com/JuliaSmoothOptimizers/Krylov.jl) is a Julia [@bezanson-edelman-karpinski-shah-2017] package that implements a collection of Krylov processes and methods for solving a variety of linear problems:

|  Square systems | Linear least-squares problems | Linear least-norm problems              |
|:---------------:|:-----------------------------:| :--------------------------------------:|
| $Ax = b$        | $\min \|b - Ax\|$             | $\min \|x\|~~\text{subject to}~~Ax = b$ |

\vspace{-0.85cm}

| Adjoint systems | Saddle-point and Hermitian quasi-definite systems | Generalized saddle-point and non-Hermitian partitioned systems |
|:---------------:|:-------------------------------------------------:|:--------------------------------------------------------------:|
|$\begin{matrix} Ax = b \\ A^{H\!} y = c \end{matrix}$ | $\begin{bmatrix} M & \!\phantom{-}A \\ A^{
  H\!} & \!-N \end{bmatrix} \begin{bmatrix} x \\ y \end{bmatrix} = \begin{bmatrix} b \\ c \end{bmatrix}$ | $\begin{bmatrix} M & A \\ B & N \end{bmatrix} \begin{bmatrix} x \\ y \end{bmatrix} = \begin{bmatrix} b \\ c \end{bmatrix}$ |

$A^{H\!}$ denotes the conjugate transpose of $A$.
It coincides with $A^{T\!}$, the transpose of $A$, for real matrices.
Krylov methods are iterative methods based on [@krylov-1931] subspaces.
They are an alternative to direct methods such as Gaussian elimination or QR decomposition when storage requirements or computational costs become prohibitive, which is often the case for large and sparse linear systems.
Contrary to direct methods, which require storing $A$ explicitly, Krylov methods support linear operators to model operator-vector products $u \leftarrow Av$, and in some instances $u \leftarrow A^{H\!}w$ because Krylov processes only require these operations to build Krylov subspaces.
This specific feature works with preconditioners as well, i.e., transformations that modify a linear system into an equivalent form that may yield faster convergence in finite-precision arithmetic.
<!-- Preconditioning can be used to reduce the condition number of the problem or cluster its eigenvalues or singular values for instance. -->
We refer interested readers to [@ipsen-meyer-1998] for an introduction to Krylov methods along with [@greenbaum-1997] and [@saad-2003] for more details.

# Features and Functionalities

##  Largest collection of Krylov processes and methods

Krylov.jl aims to provide a unified interface for the largest collection of Krylov processes and methods, all programming languages taken together, with six and thirty-three implementations, respectively:

- \textbf{Krylov processes}: \textsc{Arnoldi}, \textsc{Golub-Kahan}, \textsc{Hermitian Lanczos}, \textsc{Montoison-Orban}, \textsc{Non-Hermitian Lanczos},  \textsc{Saunders-Simon-Yip};
- \textbf{Krylov methods}: \textsc{Bicgstab}, \textsc{Bilq}, \textsc{Bilqr}, \textsc{Cg}, \textsc{Cg-lanczos}, \textsc{Cg-lanczos-shift}, \textsc{Cgls}, \textsc{Cgne}, \textsc{Cgs}, \textsc{Cr}, \textsc{Craig}, \textsc{Craigmr}, \textsc{Crls}, \textsc{Crmr}, \textsc{Diom}, \textsc{Dqgmres}, \textsc{Fgmres}, \textsc{Fom}, \textsc{Gmres}, \textsc{Gpmr}, \textsc{Lnlq}, \textsc{Lslq}, \textsc{Lsmr}, \textsc{Lsqr}, \textsc{Minres}, \textsc{Minres-qlp}, \textsc{Qmr}, \textsc{Symmlq}, \textsc{Tricg}, \textsc{Trilqr}, \textsc{Trimr}, \textsc{Usymlq}, \textsc{Usymqr}.

Some processes and methods are not available elsewhere.
<!-- mettre en avant que c'est le fruit de notre recherche -->
References for each process and method are available in the [documentation](https://juliasmoothoptimizers.github.io/Krylov.jl/stable/).
<!-- placement de produit pour JSO et utilisation dans LinearSolve.jl? -->

## Support for any floating-point system supported by Julia

Krylov.jl works in any floating-point system supported by Julia, including complex numbers, which means that Krylov.jl handles any precision `T` and `Complex{T}` where `T <: AbstractFloat`.
Although most personal computers offer IEEE 754 single and double precision computations, new architectures implement native computations in other floating-point systems.
In addition, software libraries such as the GNU MPFR, shipped with Julia, let users experiment with computations in variable, extended precision at the software level with the `BigFloat` data type.
Working in high precision has obvious benefits in terms of accuracy.
<!-- We can solve linear systems within the common half, single, double and arbitrary precision (`Float16`, `Float32`, `Float64`, `BigFloat` and their complex counterparts) but shipped with Julia or additional precision implemented through packages. -->
<!-- The alternative half precision format `BFloat16` provided by [BFloat16s.jl](https://github.com/JuliaMath/BFloat16s.jl) or the quadruple precision `Float128` implemented in [Quadmath.jl](https://github.com/JuliaMath/Quadmath.jl) are supported by Krylov.jl for instance. -->
<!-- DoubleFloats.jl, AbNumerics.jl, DecFP.jl, MultiFloats.jl... -->

## Support for Nvidia, AMD and Intel GPUs

Krylov methods are well suited for GPU computations because they only require operator-vector products ($u \leftarrow Av$, $u \leftarrow A^{H\!}w$) and vector operations ($\|v\|$, $u^H v$, $v \leftarrow \alpha u + \beta v$), which are highly parallelizable.
The implementations in Krylov.jl are generic so as to take advantage of the multiple dispatch and broadcast features of Julia.
Those allow the implementations to be specialized automatically by the compiler for both CPU and GPU.
Thus, Krylov.jl works with GPU backends that build on [GPUArrays.jl](https://github.com/JuliaGPU/GPUArrays.jl).
It includes [CUDA.jl](https://github.com/JuliaGPU/CUDA.jl), [AMDGPU.jl](https://github.com/JuliaGPU/AMDGPU.jl) and [oneAPI.jl](https://github.com/JuliaGPU/oneAPI.jl), the Julia interfaces to Nvidia, AMD and Intel GPUs.
<!-- Our implementations target the CUDA, ROCm or OneAPI libraries for efficient operator-vector products and vector operations on Nvidia, AMD and Intel GPUs. -->

## Support for linear operators

The input arguments of all Krylov.jl solvers that model $A$, $B$, $M$, $N$ or the preconditioners can be any object that represents a linear operator.
Krylov methods combined with linear operators allow to reduce computation time and memory requirements considerably by avoiding building and storing the system matrix.
In the field of nonlinear optimization, finding critical points of a continuous function frequently involves linear systems where $A$ is a Hessian or a Jacobian.
Materializing such operators as matrices is expensive in terms of operations and memory consumption and is unreasonable for high-dimensional problems.
However, it is often possible to implement efficient Hessian-vector and Jacobian-vector products, for example with the help of automatic differentiation tools.
<!-- For a preconditioned linear system $P^{-1} A x = P^{-1} b$ where $P$ is an incomplete LU decomposition $A$, we can just create a linear operator that perform the forward and backward sweeps with the factors of $P$ to model $P^{-1}$. -->

## In-place methods

All solvers in Krylov.jl have an in-place variant that allows to solve multiple linear systems with the same dimensions, precision and architecture.
Optimization methods such as the Newton and the Gauss-Newton methods can take advantage of this functionality.
The in-place variants only require a Julia structure that contains all the storage needed by a Krylov method as additional argument.
In-place methods limit allocation and deallocation of memory, which are particularly expensive on GPUs.
<!-- C'est le moment de placer un mot sur JSO -->

## Performance optimizations and storage requirements

To perform the most expensive operations in Krylov.jl, which are the operator-vector products and vector operations, we rely on BLAS routines as much as possible.
By default, Julia ships with OpenBLAS and provides multithreaded routines.
Since Julia 1.6, users can also switch dynamically to other BLAS backends, such as Intel MKL or BLIS, thanks to the BLAS demuxing library `libblastrampoline`, if a more optimized BLAS is available.
<!-- une petite transition ne serait pas du luxe -->
A ``storage requirements'' section is available in the documentation to provide the theoretical number of bits required by each method.
We also implemented functions that return the number of bits allocated by our implementations.
They are a guarantee that we match the theoretical results.
<!-- recover / match -->

# Exemples

```julia
using SparseArrays     # Sparse library of Julia
using Krylov           # Krylov methods and processes
using ForwardDiff      # Automatic differentiation
using LinearOperators  # Linear operators
using Quadmath         # Quadruple precision
using MKL              # Intel BLAS
using CUDA             # Interface to Nvidia GPUs
using CUDA.CUSPARSE    # Nvidia CUSPARSE library
```

<!-- At each iteration of Newton's method applied to a $\mathcal{C}^2$ strictly convex function $f : \mathbb{R}^n \rightarrow \mathbb{R}$, a descent direction direction is determined by minimizing the quadratic Taylor model of $f$:
$$\min_{d \in \mathbb{R}^n}~~f(x_k) + \nabla f(x_k)^T d + \tfrac{1}{2}~d^T \nabla^2 f(x_k) d$$
which is equivalent to solving the symmetric and positive-definite system
$$\nabla^2 f(x_k) d  = -\nabla f(x_k).$$
The system above can be solved with the conjugate gradient method. -->

```julia
"The Newton method for convex optimization"
function newton(∇f, ∇²f, x₀; itmax = 200, tol = 1e-8)
    n = length(x₀)
    x = copy(x₀)
    gx = ∇f(x)
    iter = 0
    S = typeof(x)               # precision and architecture
    solver = CgSolver(n, n, S)  # structure that contains the workspace of CG
    Δx = solver.x
    solved = false
    tired = false
    while !(solved || tired)
        Hx = ∇²f(x)           # Compute ∇²f(xₖ)
        cg!(solver, Hx, -gx)  # Solve ∇²f(xₖ)Δx = -∇f(xₖ)
        x = x + Δx            # Update xₖ₊₁ = xₖ + Δx
        gx = ∇f(x)            # ∇f(xₖ₊₁)
        iter += 1
        solved = norm(gx) ≤ tol
        tired = iter ≥ itmax
    end
    return x
end

T = Float16  # IEEE half precision
x₀ = -ones(T, 4)
f(x) = (x[1] - 1)^2 + (x[2] - 2)^2 + (x[3] - 3)^2 + (x[4] - 4)^2    # f(x)
∇f(x) = ForwardDiff.gradient(f, x)                                  # ∇f(x)
H(y, x, v) = ForwardDiff.derivative!(y, t -> ∇f(x + t * v), 0)      # y ← ∇²f(x)v
symmetric = hermitian = true
∇²f(x) = LinearOperator(T, 4, 4, symmetric, hermitian, (y, v) -> H(y, x, v))  # ∇²f(x)
newton(∇f, ∇²f, x₀)
```

<!-- At each iteration of the Gauss-Newton method applied to a nonlinear least-squares objective $f(x) = \tfrac{1}{2}\| F(x)\|^2$ where $F : \mathbb{R}^n \rightarrow \mathbb{R}^m$ is $\mathcal{C}^1$, we solve the subproblem:
$$\min_{d \in \mathbb{R}^n}~~\tfrac{1}{2}~\|J(x_k) d + F(x_k)\|^2,$$
where $J(x)$ is the Jacobian of $F$ at $x$.
An appropriate iterative method to solve the above linear least-squares problems is LSMR. -->

```julia
"The Gauss-Newton method for Nonlinear Least Squares"
function gauss_newton(F, JF, x₀; itmax = 200, tol = 1e-8)
    n = length(x₀)
    x = copy(x₀)
    Fx = F(x)
    m = length(Fx)
    iter = 0
    S = typeof(x)                 # precision and architecture
    solver = LsmrSolver(m, n, S)  # structure that contains the workspace of LSMR
    Δx = solver.x
    solved = false
    tired = false
    while !(solved || tired)
        Jx = JF(x)              # Compute J(xₖ)
        lsmr!(solver, Jx, -Fx)  # Minimize ‖J(xₖ)Δx + F(xₖ)‖
        x = x + Δx              # Update xₖ₊₁ = xₖ + Δx
        Fx_old = Fx             # F(xₖ)
        Fx = F(x)               # F(xₖ₊₁)
        iter += 1
        solved = norm(Fx - Fx_old) / norm(Fx) ≤ tol
        tired = iter ≥ itmax
    end
    return x
end

T = Float128  # IEEE quadruple precision
x₀ = ones(T, 2)
F(x) = [x[1]^4 - 3; exp(x[2]) - 2; log(x[1]) - x[2]^2]         # F(x)
J(y, x, v) = ForwardDiff.derivative!(y, t -> F(x + t * v), 0)  # y ← JF(x)v
Jᵀ(y, x, w) = ForwardDiff.gradient!(y, x -> dot(F(x), w), x)   # y ← JFᵀ(x)w
symmetric = hermitian = false
JF(x) = LinearOperator(T, 3, 2, symmetric, hermitian, (y, v) -> J(y, x, v),   # non-transpose
                                                      (y, w) -> Jᵀ(y, x, w),  # transpose
                                                      (y, w) -> Jᵀ(y, x, w))  # conjugate transpose
gauss_newton(F, JF, x₀)
```

```julia
rows = [1, 1, 1, 2, 2, 2, 3, 3, 3]
cols = [1, 2, 3, 1, 2, 3, 1, 2, 3]
vals = [7.0, im, -5im, -im, 8.0, 5.0, 5im, 5, 10.0]
A_cpu = sparse(rows, cols, vals, 3, 3)
b_cpu = [11.0-6im, 32.0+12im, 35.0+20im]

# Transfer the linear system from the CPU to the GPU
A_gpu = CuSparseMatrixCSR(A_cpu)
b_gpu = CuVector(b_cpu)

# Incomplete Cholesky decomposition LLᴴ ≈ A with zero fill-in
P = ic02(A_gpu, 'O')

# Solve Py = x
function ldiv_ic0!(y, P, x)
  copyto!(y, x)
  ldiv!(LowerTriangular(P), y)   # Forward substitution with L
  ldiv!(LowerTriangular(P'), y)  # Backward substitution with Lᴴ
  return y
end

# Linear operator that model the preconditioner P⁻¹
n = length(b_gpu)
T = eltype(b_gpu)
symmetric = false
hermitian = true
P⁻¹ = LinearOperator(T, n, n, symmetric, hermitian, (y, x) -> ldiv_ic0!(y, P, x))

# Solve a Hermitian positive definite system with an incomplete Cholesky preconditioner on GPU
x, stats = minres(A_gpu, b_gpu, M=P⁻¹)
```

# Acknowledgements

Alexis Montoison is supported by a FRQNT grant and an excellence scholarship of the IVADO institute,
and Dominique Orban is partially supported by an NSERC Discovery Grant.

# References

<!--
Livre de Greenbaum
Book Series Name:Frontiers in Applied Mathematics
Book Code:FR17

Livre de Saad:
Book Series Name:Other Titles in Applied Mathematics
Book Code:OT82
-->