# Chapter 33: Implementation Details

## Core Idea
GROMACS implements the single sum virial for pressure calculation and various computational optimizations. Understanding these implementation choices helps interpret results and diagnose issues.

## Single Sum Virial
- Pressure calculated from virial tensor: P = (2/V)(E_kin − Ξ)
- Single sum virial: Ξ = −½ Σ_i<j r_ij ⊗ F_ij
- Avoids double force calculation
- Consistent with PME electrostatic forces

## Key Optimizations
- **SIMD**: Single Instruction Multiple Data for vectorized force kernels
- **GPU offloading**: Most expensive calculations (non-bonded) on GPU
- **Neighbor list buffering**: Amortize list construction over nstlist steps
- **Grid search**: O(N) scaling for neighbor list construction
- **Dynamic load balancing**: Redistribute atoms when domains become imbalanced
- **MTS**: Evaluate PME less frequently than direct-space forces
- **Domain decomposition**: Spatial parallelization minimizing communication

## Computational Flow (per MD step)
1. Build neighbor list (every nstlist steps)
2. Compute short-range non-bonded forces (LJ + direct Coulomb)
3. PME: spread charges to grid → FFT → solve Poisson → inverse FFT → interpolate forces
4. Compute bonded forces
5. Apply constraints (LINCS/SHAKE/SETTLE)
6. Update positions and velocities (integrator)
7. Apply temperature/pressure coupling
8. Write output if at output interval

## Key Takeaways
1. Single sum virial is the standard pressure calculation method
2. SIMD acceleration is critical for CPU performance
3. Neighbor list buffering amortizes O(N²) pair search cost
4. PME is the computational bottleneck for large systems

## Connects To
- **Ch 19-22**: Algorithm details
- **Ch 11**: Performance optimization
- **Ch 34**: Averages and fluctuations
