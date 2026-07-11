# Chapter 22: Algorithms — Parallelization

## Core Idea
GROMACS achieves high performance through domain decomposition (spatial) + MPI (inter-node) + OpenMP (intra-node) + GPU offloading. The key tuning decisions are: PME rank count, GPU-resident mode, and MPI/OpenMP thread balance.

## Parallelization Schemes

### Domain Decomposition
- Simulation box divided into 3D grid of domains
- Each MPI rank owns one domain
- Atoms migrate between domains as they move
- Communication: positions and forces at domain boundaries
- Minimum: ~500 atoms per MPI rank for efficiency

### PME Parallelization
- Real-space: domain decomposition across PP ranks
- Reciprocal-space: slab decomposition across PME ranks
- PP and PME ranks communicate via MPI
- Optimal split: `gmx tune_pme -s topol.tpr`

### MPI + OpenMP Hybrid
- MPI for inter-node communication
- OpenMP for intra-node shared-memory parallelism
- Typically 1 OpenMP thread per MPI rank is optimal
- On many-core nodes: 2-4 threads per rank

## GPU Acceleration

### GPU Offloading
- Short-range non-bonded: GPU (most expensive part)
- PME: GPU or CPU (GPU faster for large systems)
- Bonded interactions: GPU
- Constraints: GPU
- Integration/update: GPU (GPU-resident mode)

### GPU-Resident Mode
- All data stays on GPU between steps
- Requires: `constraints=h-bonds`, Verlet scheme
- Command: `-nb gpu -pme gpu -bonded gpu -update gpu`
- Best single-node performance

### Multi-GPU
- Separate PP and PME GPU assignments
- `gmx mdrun -nb gpu -pme gpu -npme N`
- Each GPU handles different ranks

## Load Balancing
- Dynamic load balancing for heterogeneous systems
- Atoms redistributed when imbalance exceeds threshold
- `dlb = auto` (default): enables dynamic load balancing

## Communication Optimization
- Constraints: `constraints=h-bonds` avoids constraint communication for heavy atoms
- MTS: reduces PME communication frequency
- Larger nstlist: fewer pair list communications

## Key Takeaways
1. ~500 atoms/core minimum for efficient domain decomposition
2. GPU-resident mode: `-nb gpu -pme gpu -bonded gpu -update gpu`
3. `gmx tune_pme` for optimal PME rank count
4. 1 OpenMP thread per MPI rank is typically optimal
5. Dynamic load balancing helps with heterogeneous systems

## Connects To
- **Ch 11**: Performance tuning practical guide
- **Ch 2**: Build configuration for parallelization

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 区域分解 | Domain decomposition | 将模拟盒分为3D网格, 每个MPI进程负责一个域 |
| 动态负载均衡 | Dynamic load balancing (DLB) | 重分配原子以平衡计算负载 |
| 通讯坐标和力 | Communication of coordinates and forces | 域边界halo区的信息交换 |
| 多程序多数据 | MPMD (Multiple Program Multiple Data) | PME专用进程模式 |
| 隐式溶剂模型 | Implicit solvent | 用连续介质替代显式水 (GBSA方法) |
| 相互作用范围 | Interaction range | 决定了域分解中的通讯量 |
| 壳层分子动力学 | Shell MD | 用于极化力场的壳层粒子方法 |

Sources: GROMACS 5.0.2 中文手册 (李继存译) §3.17-3.18, CC-BY compatible.
