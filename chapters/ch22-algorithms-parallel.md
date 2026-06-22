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
