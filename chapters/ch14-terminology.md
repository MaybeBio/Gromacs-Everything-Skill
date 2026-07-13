# Chapter 14: Terminology

## Core Idea
GROMACS has specific terminology for its components, algorithms, and file formats. Understanding these terms is essential for reading the manual, interpreting error messages, and configuring simulations correctly.

## Key Terms

### Algorithm Terms
- **Verlet scheme**: Modern cutoff scheme using buffered pair lists; required for GPU acceleration
- **Group scheme**: Legacy cutoff scheme (removed in recent versions)
- **PME (Particle-Mesh Ewald)**: FFT-based method for long-range electrostatics
- **PPPM**: Alternative to PME (not recommended for new simulations)
- **LINCS**: Linear Constraint Solver; default bond constraint algorithm
- **SHAKE**: Iterative constraint algorithm; alternative to LINCS
- **SETTLE**: Analytical constraint for rigid 3-atom water molecules
- **Domain decomposition**: Spatial parallelization dividing box into subdomains
- **Neighbor list (pair list)**: Buffer of nearby atoms updated every nstlist steps
- **MTS (Multiple Time Stepping)**: Evaluating slow forces less frequently than fast forces
- **HMR (Hydrogen Mass Repartitioning)**: Transferring mass from heavy atoms to hydrogens for larger timestep
- **Virtual sites**: Massless interaction sites constructed from real atoms; enable 5 fs timestep

### Simulation Terms
- **EM (Energy Minimization)**: Removing steric clashes before dynamics
- **NVT**: Constant particle number, volume, temperature (canonical ensemble)
- **NPT**: Constant particle number, pressure, temperature (isothermal-isobaric)
- **NVE**: Constant particle number, volume, energy (microcanonical ensemble)
- **Equilibration**: Phase where system relaxes to target conditions before data collection
- **Production**: Data collection phase after equilibration

### Force Field Terms
- **Atom type**: Numeric code mapping chemical environment to parameters
- **Combination rule**: Method to compute pair parameters from pure atom types
- **Non-bonded**: Interactions between atoms not connected by bonds (LJ + Coulomb)
- **Bonded**: Interactions defined by topology (bonds, angles, dihedrals)
- **1-4 interactions**: Non-bonded interactions between atoms separated by 3 bonds; scaled by fudgeLJ/fudgeQQ
- **Proper dihedral**: Torsion around a bond defined by 4 atoms
- **Improper dihedral**: Out-of-plane bending, often used to maintain chirality
- **CMAP**: Grid-based correction to backbone dihedral potentials (CHARMM)
- **Restraints**: External biases (position, distance, angle, orientation) not part of the force field

### Enhanced Sampling Terms
- **FEP (Free Energy Perturbation)**: Computing ΔG from ensemble averages
- **TI (Thermodynamic Integration)**: Computing ΔG by integrating ∂H/∂λ
- **BAR (Bennett's Acceptance Ratio)**: Optimal method combining forward/reverse FEP
- **MBAR**: Multistate BAR; combines data from all lambda windows
- **Soft-core**: Modified LJ/Coulomb potentials preventing singularities at λ=0 or 1
- **Lambda**: Coupling parameter (0 = initial state, 1 = final state)
- **Replica exchange**: Swapping configurations between parallel simulations
- **AWH (Accelerated Weight Histogram)**: Adaptive biasing method for PMF
- **Umbrella sampling**: Restraining system along reaction coordinate with harmonic bias
- **Expanded ensemble**: Sampling multiple states in a single simulation

## Key Takeaways
1. Verlet scheme is the only supported cutoff scheme in modern GROMACS
2. LINCS is the default and recommended constraint algorithm
3. HMR enables 4 fs timestep by repartitioning hydrogen masses
4. MTS evaluates PME every 2nd step for performance
5. PME is the standard for long-range electrostatics

## Connects To
- **Ch 9**: MDP parameters using these terms
- **Ch 20**: Algorithm details
- **Ch 21**: Free energy methods

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 周期性边界条件 | periodic boundary conditions (PBC) | 消除表面效应 |
| 压强 | pressure | 波动性质 |
| 控温器/恒温器 | thermostat | 维持温度恒定 |
| 控压器/恒压器 | barostat | 维持压力恒定 |
| 能量守恒 | energy conservation | NVE 系综 |
| 平均结构 | average structure | 轨迹平均 |
| 爆破 | system blow-up | 模拟崩溃 |
| 力场 | force field | 势能函数和参数 |
| 约束 | constraints | 固定键长 |
| 限制 | restraints | 限制位移 |
| 环境变量 | environment variables | 控制运行时行为 |
| 浮点运算 | floating-point arithmetic | 精度限制 |
| 粒子类型 | particle type | 原子类型/电荷组 |
| 分子类型 | moleculetype | 拓扑中的定义 |
| 质心/质量中心 | center of mass (COM) | 加权平均位置 |
| 默认组 | default groups | System, Protein, Water 等 |
| 联合原子 | united-atom | 无显式脂肪族 H |
| 虚拟位点 | virtual sites | 无质量相互作用位点 |

**关键概念**: 中文手册的术语部分涵盖了 GROMACS 中所有核心概念的中英文对照。压强在模拟中是一个波动量（而非精确常数），周期性边界条件是最常用边界条件模拟体相系统。控温器种类包括 Berendsen, V-rescale, Nose-Hoover 等，控压器包括 Berendsen, Parrinello-Rahman 等。约束（constraints，如固定键长为精确值）和限制（restraints，如用简谐势惩罚偏离）是两个不同的概念。系统爆破通常由初始结构不良、时间步长过大或力场参数错误引起。环境变量如 GMX_MAXBACKUP 控制文件备份数量，GMX_GPU_ID 选择 GPU 设备。

Sources: GROMACS 2019.6 中文译版 (§3.12-3.16)
