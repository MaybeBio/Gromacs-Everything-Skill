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
| 区域分解 | Domain decomposition (DD) | 将空间划分成平行六面体，每个MPI进程配一个域 |
| 第八壳层法 | Eighth shell method | GROMACS使用的力分配方法，比半壳层法通信更少 |
| 中点法 | Midpoint method | 某些系统更有优势的通信方法，将来可能实现 |
| 区域分解单元 | DD cell | 1D/2D/3D格点划分的最小并行单元 |
| 更新组 | Update group | 最小分配单位，含依赖关系的原子组(约束/虚拟位点) |
| 坐标通信 | Coordinate communication | 沿x,y,z负方向移动至相邻区域 |
| 力通信 | Force communication | 与坐标通信相反方向的力分配 |
| 动态负载均衡 | Dynamic load balancing (DLB) | 独立调整各单元体积，失衡≥2%时自动开启 |
| 交错格点 | Staggered grid | 2D/3D DD格点交错实现DLB，需双重距离检查 |
| 负载失衡 | Load imbalance | 粒子分布/计算时间/统计涨落/通信差异导致 |
| 最小缩放比例 | Minimum scaling | 每维度≥0.8，3D时体积可缩至0.5倍以补偿100%失衡 |
| P-LINCS | Parallel LINCS | 处理跨域边界的键约束，通信几何中心+迭代 |
| 相互作用范围 | Interaction range | r_c=max(rlist,rvdW,rcoul), 两体r_mb, 多体r_mb, 约束r_con |
| 多程序多数据 | MPMD (Multiple Program Multiple Data) | 独立PME进程分离计算长程静电 |
| PP进程 | Particle-Particle (PP) rank | 负责除PME外的所有计算 |
| PME进程 | PME rank | 只负责PME网格计算，PP:PME≈3:1(长方体)或2:1(菱形十二面体) |
| PME负载 | PME load | 理想占总计算负载25%-33% |
| 铅笔分解 | Pencil decomposition | PME高度并行2D分解，区域形状类似铅笔 |
| GPU重叠 | GPU overlap | DLB计时不包括MPI通信，CPU/GPU重叠工作 |
| 笛卡尔排序 | Cartesian ordering | 用于3D环形网络的最优进程分配 |

**关键概念**:
1. **区域分解核心**: 将三斜晶胞划分为1D/2D/3D平行六面体。坐标沿负方向脉冲通信，力沿相反方向。第八壳层法通信量始终少于半壳层法
2. **DLB自动调整**: 默认当力计算失衡导致性能损失≥2%时自动开启。测量不包括MPI通信。自2016.1起，若测量显示DLB导致性能下降则自动禁用
3. **MPMD原理**: PME需要全局All-to-All通信但只需1/4到1/3进程。分离PP和PME进程后，通信量降为~1/16，可扩展到4倍以上进程
4. **约束通信**: P-LINCS通信(lincs_order+1)根键远的坐标，应用到这些非局部键并进行迭代校正
5. **最小单元尺寸**: L_C ≥ max(r_mb, r_con)，菱形十二面体时沿x和y方向长度为√(3/2)

**重要公式**:
- DD单元最小尺寸: L_C ≥ max(r_mb, r_con)
- 菱形十二面体xy方向间距: 长度×√(3/2)
- 默认DLB性能阈值: 2% (
  性能损失在日志文件结尾报告)

Sources: GROMACS 2019.6 中文译版, §5.4.15-5.4.16
