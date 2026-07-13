# Chapter 19: Algorithms — PBC & MD

## Core Idea
GROMACS uses periodic boundary conditions (PBC) with minimum image convention and triclinic boxes. The MD engine integrates Newton's equations using the leap-frog algorithm with buffered Verlet pair lists for neighbor searching.

## Periodic Boundary Conditions

### Box Types
| Type | Volume ratio | Best for |
|---|---|---|
| Cubic | 1.0 (d³) | Crystals, simple systems |
| Rhombic dodecahedron | 0.707 d³ (71% of cubic) | Spherical solutes — saves 29% solvent |
| Truncated octahedron | 0.770 d³ (77% of cubic) | Spherical solutes (alternative) |

Box vectors (a, b, c) must satisfy: a_y = a_z = b_z = 0, a_x > 0, b_y > 0, c_z > 0, and |b_x| ≤ ½a_x, |c_x| ≤ ½a_x, |c_y| ≤ ½b_y.

### Minimum Image Convention
Only the nearest image of each particle is considered for short-range interactions. Combined with PBC, this eliminates surface artifacts.

## Neighbor Searching

### Verlet Scheme
- **Pair list**: Buffer of all atom pairs within rlist + buffer
- **Update frequency**: Every nstlist steps (default 10)
- **Buffer**: Accounts for atom movement between updates
- **Grid search**: Atoms binned into cells; only neighboring cells searched
- `verlet-buffer-tolerance`: Controls buffer size (default 0.005 kJ/mol/ps)

## MD Algorithm

### Leap-frog Integrator (Default: `integrator=md`)
- Positions at integer steps: r(t + Δt) = r(t) + v(t + ½Δt)·Δt
- Velocities at half-integer steps: v(t + ½Δt) = v(t − ½Δt) + F(t)/m · Δt
- Symplectic, time-reversible, numerically stable
- Kinetic energy from half-step velocities (slightly low)

### Velocity Verlet (`integrator=md-vv`)
- Positions: r(t + Δt) = r(t) + v(t)·Δt + F(t)/(2m)·Δt²
- Velocities: v(t + Δt) = v(t) + [F(t) + F(t + Δt)]/(2m)·Δt
- Full-step velocity output, better for Nose-Hoover/PR coupling
- More expensive with constraints (extra computation and communication)

### Temperature Calculation
- Instantaneous temperature: T = 2·E_kin / (N_df · k_B)
- N_df = 3N − N_c − N_com (degrees of freedom, minus constraints and COM)

## The Group Concept
Atoms are organized into groups for:
- Temperature coupling (tc-grps)
- Freezing (freezegrps)
- Acceleration (accelerate)
- Energy monitoring (energygrps)

## Key Takeaways
1. Rhombic dodecahedron saves 29% solvent vs cubic for spherical solutes
2. Leap-frog is the default and recommended integrator for production
3. Verlet scheme with buffered pair lists is the only supported cutoff scheme
4. Grid search enables O(N) neighbor list construction
5. Use velocity Verlet only when you specifically need full-step velocities

## Connects To
- **Ch 9**: MDP parameters for integrator selection
- **Ch 20**: Temperature/pressure coupling algorithms
- **Ch 22**: Parallelization of neighbor search

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 周期性边界条件 | Periodic Boundary Conditions (PBC) | 消除表面效应，系统被自身平移副本包围 |
| 最小映像约定 | Minimum image convention | 每个粒子只考虑最近的一个映像 |
| 三斜晶胞/三斜盒子 | Triclinic box | 最一般的空间填充形状，所有盒子都是三斜晶胞的特例 |
| 菱形十二面体 | Rhombic dodecahedron | 体积为0.707d³，比立方体省29%溶剂 |
| 截角八面体 | Truncated octahedron | 体积为0.770d³，比立方体省23%溶剂 |
| 盒向量 | Box vectors (a, b, c) | 定义三斜盒子，需满足 a_y=a_z=b_z=0 及不等式约束 |
| 截断半径 | Cutoff radius R_c | < ½ min(‖a‖,‖b‖,‖c‖), 最小映像约定要求 |
| 晶格加和 | Lattice sum methods | Ewald, PME, PPPM等长程静电方法 |
| 组/原子组 | Group / Atom group | 最多256组，每原子最多属于6类不同组 |
| 温度耦合组 | Temperature coupling group | 每组可单独定义温度耦合参数 |
| 冻结组 | Freeze group | 原子在模拟中保持静止，可部分冻结(1或2个方向) |
| 加速组 | Accelerate group | 施加加速度a^g，用于非平衡MD |
| 能量监测组 | Energy monitoring group | 分别计算各组间LJ和Coulomb交叉相互作用 |
| 质心组 | COM group | 移除特定组的质心运动 |
| 压缩位置输出组 | Compressed position output group | 只保存指定组轨迹到压缩文件(xtc/tng) |
| 蛙跳式算法 | Leap-frog algorithm | 默认积分器，v(t+½Δt)=v(t−½Δt)+(Δt/m)·F(t); r(t+Δt)=r(t)+Δt·v(t+½Δt) |
| 速度Verlet算法 | Velocity Verlet | v(t+Δt)=v(t+½Δt)+(Δt/2m)·F(t+Δt), 全步速度输出 |
| Verlet算法 | Verlet algorithm | r(t+Δt)=2r(t)−r(t−Δt)+F(t)Δt²/m + O(Δt⁴) |
| Trotter分解 | Trotter decomposition | 可逆积分方法的理论框架，Liouville算符分解 |
| 邻区搜索/配对搜索 | Neighbor search (NS) / Pair search | 找截断范围内所有粒子对，O(N)格点搜索 |
| 配对列表 | Pair list / Neighbor list | 含所有需计算非键力的粒子对，每nstlist步更新 |
| Verlet截断方案 | Verlet cutoff scheme | 使用缓冲配对列表，自动确定缓冲大小 |
| 组截断方案 | Group cutoff scheme | 已弃用，基于电荷组的原始方案 |
| 电荷组 | Charge group | 净电荷为零的原子组，用于组截断方案 |
| 格点搜索 | Grid search | 搜索的相邻单元≤125(5³)个，O(N)线性标度 |
| 动能张量 | Kinetic energy tensor | E_kin = ½ Σ_i m_i v_i⊗v_i, 用于三斜或剪切系统 |
| 维里 | Virial Ξ | Ξ = −½ Σ_{i<j} r_ij⊗F_ij, 压力=2(E_kin−Ξ)/V |
| 多重时间步 | Multiple Time Stepping (MTS) | GROMACS不实现，用约束和虚拟位点替代 |
| 麦克斯韦-玻尔兹曼分布 | Maxwell-Boltzmann distribution | 初始速度生成，12个随机数生成正态分布 |
| 质心运动移除 | Center of mass (COM) motion removal | 每步设质心速度为零，防止整体漂移 |

**关键概念**:
1. **三斜盒子条件**: a_y=a_z=b_z=0, a_x>0,b_y>0,c_z>0, |b_x|≤½a_x, |c_x|≤½a_x, |c_y|≤½b_y。即使使用三斜盒子，粒子始终置于长方体空间内以提高效率
2. **格点搜索效率**: 对菱形十二面体盒子同样快，因此是蛋白质/多肽模拟的首选盒形。粒子~几百时格点搜索即远优于O(N²)
3. **Verlet方案缓冲**: 默认verlet-buffer-tolerance=0.005 kJ/mol/ps。总能量漂移通常比容差小一个数量级以上。约束算法贡献常占主导
4. **蛙跳vs速度Verlet**: Leap-frog对位置三阶精度，时间可逆；半步平均动能vs全步动能计算方法不同导致温度差异，对NVE模拟差异不明显但对NVT模拟可能有影响
5. **六个组类型**: 温度耦合组、冻结组、加速组、能量监测组、质心组、压缩位置输出组 — 每原子最多分别属于六类中的各一个

**重要公式**:
- 蛙跳: v(t+½Δt)=v(t−½Δt)+(Δt/m)·F(t); r(t+Δt)=r(t)+Δt·v(t+½Δt)
- 速度Verlet: v(t+Δt)=v(t+½Δt)+(Δt/2m)·F(t+Δt)
- 温度(瞬时): ½N_df·kT = E_kin = ½Σ_i m_i v_i²
- 自由度: N_df = 3N − N_c − N_com (约束数+COM)
- 压力张量: P = (2/V)(E_kin − Ξ), 维里 Ξ = −½ Σ_{i<j} r_ij⊗F_ij
- 速度分布: p(v_i) = √(m_i/2πkT)·exp(−m_i v_i²/2kT)

Sources: GROMACS 2019.6 中文译版, §5.4.1-5.4.3
