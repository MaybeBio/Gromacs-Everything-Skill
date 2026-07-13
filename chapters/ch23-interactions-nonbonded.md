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
| 非键相互作用 | Non-bonded interactions | 配对累加Σ_{i<j}V_ij(r_ij)，中心对称，F_j=−F_i |
| 排斥项 | Repulsion term | C₁₂/r¹² (Pauli排斥) |
| 色散项 | Dispersion term | −C₆/r⁶ (诱导偶极) |
| C-参数形式 | C-parameter form | V_LJ = C^(12)/r^12 − C^(6)/r^6 |
| σ/ε形式 | σ/ε form | V_LJ = 4ε[(σ/r)^12 − (σ/r)^6] |
| 力常数 | Force constant | F_i = (12C^(12)/r^13 − 6C^(6)/r^7)·r_ij/r_ij |
| Buckingham势 | Buckingham (exp-6) potential | V = A·exp(−Br) − C/r^6，排斥更柔性但更昂贵 |
| 库仑相互作用 | Coulomb interaction | V_c = f·q_i·q_j/(ε_r·r_ij), f=138.935458 |
| 相对介电常数 | Relative dielectric constant ε_r | 可在grompp输入中设置 |
| 反应场 | Reaction field | 截断r_c外为介电常数ε_rf的连续介质 |
| 反应场校正 | RF correction | V_crf= f·q_i·q_j/(ε_r·r)[1+(ε_rf−ε_r)/(2ε_rf+ε_r)·r³/r_c³]−常数 |
| 广义反应场 | Generalized RF | 含Debye屏蔽长度κ，对应离子强度I |
| 组合规则类型1 | Combination rule type 1 | C^(6)_ij=√(C^(6)_ii·C^(6)_jj), C^(12)_ij=√(C^(12)_ii·C^(12)_jj) (GROMOS) |
| 组合规则类型2 | Combination rule type 2 | σ_ij=½(σ_ii+σ_jj), ε_ij=√(ε_ii·ε_jj) (Lorentz-Berthelot, AMBER/CHARMM) |
| 组合规则类型3 | Combination rule type 3 | σ_ij=√(σ_ii·σ_jj), ε_ij=√(ε_ii·ε_jj) (OPLS) |
| 移位函数 | Shift function | 将势能或力移动特定函数，使截断处光滑过渡 |
| 力切换函数 | Force switch function | S_F(r)=A(r−r₁)²+B(r−r₁)³，三次多项式 |
| 势能切换函数 | Potential switch function | 五次多项式S_V(r)，使势和力在边界连续 |
| 移位vs切换 | Shift vs Switch | 无本质区别，切换是移位的特例 |
| Ewald直接空间修正 | Ewald modified short-range | V(r)=f·erfc(βr_ij)/r_ij·q_i·q_j |
| 余误差函数 | erfc(βr) | 控制直接/倒易空间划分 |
| PME | Particle-Mesh Ewald | Cardinal B样条插值(SPME)，3D FFT加速 |
| SPME | Smooth PME | GROMACS的PME实现，用样条插值 |
| P3M-AD | P3M with analytical differentiation | 优化Green影响函数，比PME精度略高(<10%) |
| 傅里叶格点间距 | Fourier spacing | 默认0.12 nm，控制FFT格点密度 |
| 插值阶数 | PME order | 默认4阶(立方)，控制B样条插值精度 |
| 容差 | ewald-rtol | 默认1e-5，直接空间静电相对强度 |
| 色散校正 | Dispersion correction (DispCorr) | 能量+压力校正，解析积分均匀分布假设 |
| 平均色散常数 | Average dispersion constant ⟨C₆⟩ | 均匀混合物N(N−1)/2对平均 |

**关键概念**:
1. **势能函数三部分**: (a) 非键项—LJ/Buckingham+Coulomb/修正Coulomb(动态原子对列表); (b) 键合项—键伸缩+键角+二面角(固定原子列表); (c) 约束项—位置/角度/距离/方向/二面角约束
2. **Verlet截断方案**: 将势能移动一常数使截断处为零，势能=力的精确积分。组方案中势能截断为零才能保证相等
3. **切换函数vs移位函数**: 切换是乘函数，移位是加函数，无本质区别。切换势能可导致虚高力，不推荐用于Coulomb；用于LJ可接受
4. **电荷组起源**: 引入电荷组是为了减少Coulomb截断假象，将效应从电荷-电荷降至偶极-偶极水平。使用PME后不再需要但保留用于性能优化
5. **PME参数**: fourierspacing=0.12nm, pme-order=4 → 静电能精度约5×10⁻³。能量误差通常小于LJ误差，可略微增大间距

**重要公式**:
- LJ力: F_i(r_ij) = (12C^(12)/r^13 − 6C^(6)/r^7)·r_ij/r_ij
- 反应场常数: k_rf = r_c⁻³·(ε_rf−ε_r)/(2ε_rf+ε_r), c_rf = r_c⁻¹·3ε_rf/(2ε_rf+ε_r)
- 切换力: F_s(r) = α/r^(α+1) + A(r−r₁)² + B(r−r₁)³
- 切换势: V_s(r) = 1/r^α − (A/3)(r−r₁)³ − (B/4)(r−r₁)⁴ − C
- Ewald短程: V(r)=f·erfc(βr_ij)/r_ij·q_i·q_j
- 广义RF常数: k_rf = r_c⁻³·[(ε_rf−ε_r)(1+κr_c)+½ε_rf(κr_c)²]/[(2ε_rf+ε_r)(1+κr_c)+ε_rf(κr_c)²]

Sources: GROMACS 2019.6 中文译版, §5.5.1
