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
| 蛙跳式算法 | Leap-frog algorithm | GROMACS默认积分器，对位置三阶精度，时间可逆 |
| 速度Verlet算法 | Velocity Verlet | 需要精确Nose-Hoover/P-R耦合时使用 |
| Trotter分解 | Trotter decomposition | 理解可逆积分方法的数学框架 |
| 周期性边界条件 | Periodic Boundary Conditions (PBC) | 消除表面效应 |
| 最小映像约定 | Minimum image convention | 每个粒子只考虑最近的映像 |
| 邻区列表 | Neighbor list (pair list) | 短程相互作用的粒子对列表 |
| 麦克斯韦-玻尔兹曼分布 | Maxwell-Boltzmann distribution | 初始速度生成 |
| 质心运动移除 | Center of mass (COM) motion removal | 防止系统整体漂移 |

**Leap-frog精度的中文表述**: 蛙跳式算法对位置 r 具有**三阶精度** (O(Δt³)), 时间可逆。公式 (3.25-3.26): v(t+Δt/2) = v(t−Δt/2) + (Δt/m)·F(t); r(t+Δt) = r(t) + Δt·v(t+Δt/2)

**初始速度生成**: 根据给定温度 T 从麦克斯韦-玻尔兹曼分布生成初始速度, 使用12个随机数产生正态分布, 移除质心运动后进行速度缩放以使总动能对应于目标温度。

Sources: GROMACS 5.0.2 中文手册 (李继存译) §3.4, CC-BY compatible.
