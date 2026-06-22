# Chapter 11: Performance Tuning

## Core Idea
GROMACS performance depends on correct compilation, optimal run setup, and efficient parallelization. A systematic checklist approach can yield 2x speedup: use single precision, GPU acceleration, rhombic dodecahedron box, constraints=h-bonds, MTS, and proper PME rank tuning.

## Performance Checklist

### Build Configuration
1. Use single precision unless double is required
2. Compile FFTW with correct x86 flags (usually auto-configured)
3. Use GCC (not ICC, PGI, Cray) on x86 and POWER
4. Use newest compiler version
5. OpenMPI usually gives best MPI performance
6. Ensure compiler supports OpenMP (some Clang versions do not)
7. Use newest GPU SDK and drivers
8. AVX2 preferred over AVX512 in GPU or highly parallel MPI runs

### Run Setup
1. Rhombic dodecahedron box for spherical solutes (saves ~29% solvent)
2. `constraints=h-bonds` (not `all-bonds`) — faster, required for GPU-resident mode
3. MTS (`mts=yes`) for PME every 2nd step on CPU
4. HMR (`mass-repartition-factor`) for 4 fs timestep
5. Tune PME ranks for massively parallel runs: `gmx tune_pme`
6. Larger coupling periods reduce global communication in massively parallel runs

## GPU Acceleration

### GPU Backends
| Backend | GPU | Recommended for |
|---|---|---|
| CUDA | NVIDIA | All NVIDIA GPUs |
| OpenCL | Cross-vendor | Legacy support |
| SYCL/DPC++ | Intel/AMD/NVIDIA | Intel GPUs, exotic configs |
| SYCL/ACPP | AMD/NVIDIA | AMD GPUs |
| HIP | AMD | AMD GPUs (native) |

### GPU-Resident Mode
- All force calculations and integration on GPU
- Requires: `constraints=h-bonds`, Verlet scheme, no energy groups
- Enable with: `-nb gpu -pme gpu -bonded gpu -update gpu`
- Maximum performance: GPU handles everything, CPU only orchestrates

### Multi-GPU
- Separate PP (particle-particle) and PME ranks
- Use `-npme N` to assign N ranks to PME
- PP ranks use GPUs for short-range, PME ranks for long-range
- `gmx tune_pme` automates optimal PME rank count

## Parallelization

### Domain Decomposition
- Spatial decomposition: box divided into domains
- Each domain assigned to one MPI rank
- Atoms migrate between domains during simulation
- ~500 atoms/rank minimum for efficiency

### MPI + OpenMP Hybrid
- MPI for inter-node, OpenMP for intra-node
- Typically 1 OpenMP thread per MPI rank is optimal
- On many-core nodes: try 2-4 OpenMP threads per rank

### Performance Metrics
Six metrics in .log file:
- **ns/day**: Most intuitive throughput metric
- **hours/ns**: Inverse of ns/day
- **Performance**: Aggregate throughput
- **Msteps/s**: Computational rate
- **Wall time**: Total elapsed

## Code Examples
```bash
# GPU-resident run (maximum performance)
gmx mdrun -deffnm md -nb gpu -pme gpu -bonded gpu -update gpu

# PME rank tuning
gmx tune_pme -s topol.tpr

# MPI with separate PME ranks
mpirun -np 16 gmx_mpi mdrun -s topol.tpr -npme 4

# Check log for performance
grep "Performance" md.log

# Check SIMD
gmx mdrun -version 2>&1 | grep SIMD
```

## Key Takeaways
1. GPU-resident mode gives best single-node performance
2. `constraints=h-bonds` is required for GPU-resident mode
3. Rhombic dodecahedron box saves 29% CPU vs cubic
4. ~500 atoms/core minimum for efficient parallelization
5. Use `gmx tune_pme` for optimal PME rank allocation
6. AVX2 > AVX512 in GPU or highly parallel MPI runs

## Connects To
- **Ch 2**: Build configuration
- **Ch 9**: MDP performance-relevant parameters
- **Ch 22**: Parallelization algorithms
