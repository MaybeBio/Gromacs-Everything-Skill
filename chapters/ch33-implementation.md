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

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 分子动力学模拟 | Molecular Dynamics (MD) Simulation | 求解牛顿运动方程 |
| 能量最小化 | Energy Minimization | 局部能量最小化，包括最陡下降、共轭梯度、L-BFGS |
| 力场 | Force Field | 提供力的保守力场，成对累加 |
| 周期性边界条件 | Periodic Boundary Conditions (PBC) | 最小映像约定 |
| 截断半径 | Cutoff Radius | 截断LJ和库仑相互作用 |
| 粒子网格Ewald | Particle-Mesh Ewald (PME) | 长程静电相互作用算法 |
| 估算维里 | Virial | 压力计算：P = (2/V)(E_kin - Xi) |
| Born-Oppenheimer近似 | Born-Oppenheimer Approximation | 电子基态假设 |
| 经典力学 | Classical Mechanics | 量子效应对氢原子等有影响 |
| 全局极小值 | Global Minimum | 势能超曲面的最低点 |
| 局部极小值 | Local Minimum | 势能函数所有一阶导数为零 |
| 模拟退火 | Simulated Annealing | 高温→降温搜索全局极小值 |
| 最陡下降法 | Steepest Descent | 沿负梯度方向前进 |
| 共轭梯度法 | Conjugate Gradient | 利用前面步骤的梯度信息加快收敛 |
| L-BFGS | L-BFGS | 准牛顿最小化方法 |
| SIMD | SIMD | C内部函数编译为SIMD机器指令 |
| GPU加速 | GPU Acceleration | 基于CUDA/OpenCL/SYCL/HIP |
| 混合精度 | Mixed Precision | 状态向量单精度，关键变量双精度 |
| 双精度 | Double Precision | 所有变量双精度，比混合精度慢20%-100% |
| 自由软件 | Free Software | LGPL 2.1许可证 |
| 贡献 | Contribute | 通过Gerrit提交 |
| 自由能计算 | Free Energy Calculation | 相平衡、结合常数、溶解度 |
| 系综平均值 | Ensemble Average | 宏观性质 = 系综平均值 |
| 蒙特卡洛模拟 | Monte Carlo Simulation | 相比MD，不计算力 |
| 单点维里 | Single Sum Virial | 避免双重力计算 |
| 邻居列表 | Neighbor List | 分摊O(N^2)配对搜索成本 |
| 区域分解 | Domain Decomposition | 空间并行化方案 |
| 多重时间步长 | Multiple Time Stepping (MTS) | PME评估频率低于直接空间力 |

Sources: GROMACS 2019.6 中文译版 (第5章 参考手册 §5.1-5.3)
