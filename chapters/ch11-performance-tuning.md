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

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 性能检查清单 | performance checklist | 构建+运行优化 |
| 区域分解 | domain decomposition | 空间并行化 |
| 动态负载均衡 | dynamic load balancing | 根据挂钟时间重新分配粒子 |
| GPU 驻留模式 | GPU-resident mode | 全部计算在 GPU 上 |
| 多 GPU | multi-GPU | PP 和 PME 秩分离 |
| SIMD 优化 | SIMD optimization | 核内指令级并行 |
| PME 调优 | PME tuning | gmx tune_pme |
| OpenMP 线程 | OpenMP threading | 节点内并行 |
| 标度极限 | scaling limit | 硬件性能上限 |
| 节点级并行 | node-level parallelization | GPU+线程MPI |
| 多节点运行 | multi-node execution | MPI 跨节点 |
| 基准测试 | benchmark | 标准测试体系 |
| 性能回归 | performance regression | 性能下降 |
| AVX2优于AVX512 | AVX2 preferred over AVX512 | GPU 或高并行 MPI |

**关键概念**: GROMACS 的性能优化分为构建配置和运行设置两个层面。构建层面：使用单精度，选择最佳 SIMD（AVX2 通常优于 AVX512 在 GPU 运行中），用最新编译器版本。运行层面：使用菱形十二面体盒子节省 29% 溶剂，constraints=h-bonds 是 GPU 驻留模式的前提条件，使用 gmx tune_pme 优化 PME 进程数。GPU 驻留模式将所有力计算和积分放到 GPU 上实现最高性能。多节点运行时，区域分解将盒子分成子区域，每个 MPI 秩负责一个区域，至少需要 ~500 原子/核才能高效并行。中文手册提供了完整的性能检查清单和优化指南。

Sources: GROMACS 2019.6 中文译版 (§2.3.6, §3.10)
