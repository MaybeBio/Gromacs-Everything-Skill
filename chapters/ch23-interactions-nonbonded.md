# Chapter 23: Non-bonded Interactions

## Core Idea
Non-bonded interactions (Lennard-Jones + Coulomb) are pair-additive and computed using buffered neighbor lists. The Verlet cutoff scheme applies a potential shift at the cutoff. PME is the standard method for long-range electrostatics, using FFT on a grid.

## Interaction Forms

### Lennard-Jones Potential
V_LJ(r) = C₁₂/r¹² − C₆/r⁶ = 4ε[(σ/r)¹² − (σ/r)⁶]

- **C₁₂**: Repulsion (Pauli exclusion)
- **C₆**: Dispersion (induced dipole)
- **σ**: Distance where V_LJ = 0
- **ε**: Well depth

### Combination Rules
| Type | σ_ij | ε_ij | Used by |
|---|---|---|---|
| 1 | — | — | GROMOS (geometric C₆, C₁₂) |
| 2 | (σ_ii + σ_jj)/2 | √(ε_ii·ε_jj) | AMBER, CHARMM |
| 3 | √(σ_ii·σ_jj) | √(ε_ii·ε_jj) | OPLS |

### Buckingham Potential
V_bh(r) = A·exp(−Br) − C/r⁶

- More flexible repulsion, more expensive
- **No longer supported by gmx mdrun** — use LJ

### Coulomb Interaction
V_c(r) = f · q_i·q_j / (ε_r · r)

- **f** = 138.935458 kJ mol⁻¹ nm e⁻²
- **ε_r**: Relative dielectric constant (1.0 in vacuum)

## Cutoff Schemes

### Verlet Scheme (Only Option)
- Pair list with buffer zone
- Potential shifted to zero at cutoff
- `rlist ≥ rcoulomb = rvdw`
- Shift function: smooth transition to zero

### Reaction Field
- Continuum dielectric beyond cutoff
- V_rf = f·q_i·q_j/(ε_r·r) · [1 + (ε_rf − ε_r)/(2ε_rf + ε_r) · r³/r_c³]

## Long-Range Electrostatics

### PME (Particle-Mesh Ewald)
1. Assign charges to FFT grid (B-spline interpolation)
2. Solve Poisson equation in reciprocal space (FFT)
3. Interpolate forces back to particles
4. `fourierspacing = 0.12 nm` (grid spacing)
5. `pme-order = 4` (interpolation order)
6. `ewald-rtol = 1e-5` (direct space tolerance)

### Ewald Summation
- Direct space: short-range real-space sum
- Reciprocal space: long-range Fourier sum
- Self-energy correction
- PME is more efficient for large systems

### PPPM
- Alternative to PME
- Not recommended for new simulations

## Long-Range VdW
- **Dispersion correction** (`DispCorr = EnerPres`): Analytical correction for missing LJ tail
- **LJ-PME**: Mesh-based long-range dispersion (like PME for LJ)
- Usually not needed: cutoff + dispersion correction is sufficient

## Key Takeaways
1. PME with fourierspacing=0.12 nm is the production standard
2. Geometric combination rules (type 1) for GROMOS, Lorentz-Berthelot (type 2) for AMBER/CHARMM
3. Buckingham potential is no longer supported
4. Verlet cutoff scheme is the only option in modern GROMACS
5. Dispersion correction (EnerPres) accounts for LJ tail

## Connects To
- **Ch 9**: MDP electrostatics parameters
- **Ch 24**: Bonded interactions
- **Ch 25**: Advanced interactions

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 非键相互作用 | Non-bonded interactions | 对势累加 (pair-additive), 满足中心对称 |
| 色散项 | Dispersion term | C₆/r⁶ 项, 诱导偶极相互作用 |
| 排斥项 | Repulsion term | C₁₂/r¹² 项, Pauli排斥 |
| 组合规则 | Combination rules | 异种原子间LJ参数的推导方式 |
| 截断距离 | Cutoff distance | 短程相互作用的计算范围 |
| 反应场 | Reaction field | 截断外的连续介质近似 |
| 库仑相互作用 | Coulomb interaction | 带电原子间的静电作用 |
| 粒子网格Ewald | PME (Particle-Mesh Ewald) | FFT加速的长程静电方法 |
| 自能校正 | Self-energy correction | Ewald求和中点电荷自身的相互作用 |
| 色散校正 | Dispersion correction | 截断外LJ尾的解析校正 |

**LJ势的两种等价表述**:
- C-参数形式: V_LJ(r) = C^(12)/r^12 − C^(6)/r^6 (公式4.3)
- σ/ε形式: V_LJ(r) = 4ε[(σ/r)^12 − (σ/r)^6] (公式4.5)
- Verlet截断方案中, 通过将势能移动一个恒量使其在截断处为零

**势能函数三部分** (来自§4):
1. **非键项**: LJ或Buckingham + Coulomb或修正Coulomb
2. **键合项**: 共价键伸缩, 键角弯曲, 反常二面角, 正常二面角 (基于固定列表)
3. **约束项**: 位置约束, 角度约束, 距离约束, 方向约束, 二面角约束

**力场组合规则**:
| GROMOS | 几何平均 (C₆和C₁₂) |
| AMBER/CHARMM | Lorentz-Berthelot (σ算术平均, ε几何平均) |
| OPLS | 几何平均 (σ和ε都用几何平均) |

Sources: GROMACS 5.0.2 中文手册 (李继存译) §4.1, CC-BY compatible.
