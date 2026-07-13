# Chapter 25: Advanced Interactions

## Core Idea
Advanced interaction features include soft-core potentials for free energy, virtual interaction sites for larger timesteps, LJ-PME for long-range dispersion, and force field organization. Soft-core prevents endpoint singularities in alchemical transformations; virtual sites enable 5 fs timesteps.

## Soft-Core Potentials
Prevent singularities when atoms appear/disappear in alchemical free energy calculations:

V_sc(r) = (1−λ)·V_A(r_A) + λ·V_B(r_B)

with modified distance: r_A = (α·σ_A⁶·λ^p + r⁶)^(1/6)

- **sc-alpha = 0.5**: Soft-core α parameter
- **sc-power = 1**: Soft-core power p
- **sc-sigma = 0.3 nm**: Soft-core σ (or read from atom types)
- **sc-coul = no**: Apply soft-core to Coulomb (usually not needed)

### Lambda Dynamics
- Lambda treated as dynamical variable
- Expanded ensemble or Wang-Landau for lambda sampling

## Virtual Interaction Sites
Massless sites constructed from real atom positions:

| Type | Construction | Use |
|---|---|---|
| 2 | Linear combination of 2 atoms | CH₂ groups |
| 3 | Linear combination of 3 atoms | CH₃ groups, water |
| 3fd | Fixed distance from 3 atoms | Aromatic H |
| 3fad | Fixed angle + distance | Lone pairs |
| 3out | Out-of-plane from 3 atoms | Out-of-plane H |
| 4fdn | Fixed distance from 4 atoms | TIP5P water |
| N | Linear from N atoms | General |

- Enable 5 fs timestep with `constraints=all-bonds`
- Eliminate fastest C-H vibrations
- Required for TIP4P and TIP5P water models

## Long-Range Electrostatics

### PME Detail
1. Real-space sum: short-range Coulomb + correction
2. Reciprocal sum: charge grid → FFT → potential → gradient → forces
3. Self-energy correction
4. Dipole correction for charged systems

### Ewald Sum
- Direct: erfc(βr)/r
- Reciprocal: exp(−k²/4β²)/k²
- Optimal β balances direct/reciprocal cost

### LJ-PME
- Mesh-based long-range dispersion
- Analogous to PME for LJ interactions
- `vdwtype = PME` enables LJ-PME
- Better than simple cutoff + dispersion correction

## Force Field Organization
GROMACS force fields in `share/gromacs/top/`:
```
amber99sb-ildn.ff/
├── forcefield.itp       # [defaults], [atomtypes], combination rules
├── forcefield.doc       # Documentation
├── atomtypes.atp        # Atom type definitions
├── ffnonbonded.itp      # Non-bonded parameters
├── ffbonded.itp         # Bonded parameters
├── aminoacids.rtp       # Standard amino acids
├── aminoacids.c.tdb     # C-terminal patches
├── aminoacids.n.tdb     # N-terminal patches
├── ions.itp             # Ion parameters
├── spc.itp / tip3p.itp  # Water model
└── *.hdb                # Hydrogen database
```

## Key Takeaways
1. Soft-core is essential for alchemical free energy — prevents λ=0,1 singularities
2. Virtual sites enable 5 fs timesteps by eliminating C-H vibrations
3. LJ-PME provides better long-range dispersion than cutoff + correction
4. Force fields are self-contained .ff directories
5. sc-alpha=0.5, sc-power=1, sc-sigma=0.3 are standard soft-core values

## Connects To
- **Ch 21**: Free energy calculations
- **Ch 23**: Non-bonded interactions
- **Ch 26**: Topology structure
- **Ch 30**: QM/MM, NNP

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 极化 | Polarization | 壳层(Drude)粒子附着到原子/虚拟位点 |
| 简单极化 | Simple polarization | 平衡距离为0的简谐势，k_cs = q_s²/α |
| 极化率 | Polarizability α | 拓扑输入，GROMACS单位nm³ (1 Å³=0.001 nm³) |
| 过极化 | Over-polarization | 非简谐极化修正: 四阶项k_hyp(r_cs−δ)⁴ |
| Thole极化 | Thole polarization | 壳层粒子间相互作用屏蔽，V_thole=(q_i·q_j/r)[1−(1+¯r/2)e^(−¯r)] |
| Thole魔数a | Thole magic number a | 通常取2.6，无量纲参数 |
| 自由能相互作用 | Free energy interactions | λ从A→B的(非)线性插值 |
| λ向量 | Lambda vector | 自4.6起独立控制Coulomb/LJ/成键/约束分量的λ转变 |
| 简谐键λ插值 | Harmonic bond λ interpolation | V=½[(1−λ)k^A+λk^B][b−(1−λ)b₀^A−λb₀^B]² |
| 二面角λ插值 | Dihedral λ interpolation | V=[(1−λ)k^A+λk^B](1+cos[nφ−(1−λ)φ_s^A−λφ_s^B]) |
| 库仑λ插值 | Coulomb λ interpolation | V_c=(f/ε_r·r)[(1−λ)q_i^A·q_j^A+λq_i^B·q_j^B] |
| LJ λ插值 | LJ λ interpolation | V_LJ=[(1−λ)C₁₂^A+λC₁₂^B]/r^12 − [(1−λ)C₆^A+λC₆^B]/r^6 |
| 动能λ贡献 | Kinetic λ contribution | ∂E_k/∂λ=−½v²(m^B−m^A)，质量变化时有贡献 |
| 约束λ贡献 | Constraint λ contribution | ∂G/∂λ=−Σλ_k(d_k^B−d_k^A)，LINCS; SHAKE乘2因子 |
| 软核相互作用 | Soft-core interactions | r_A=(α·σ_A⁶·λ^p+r⁶)^(1/6)，r_B=(α·σ_B⁶·(1−λ)^p+r⁶)^(1/6) |
| 标准软核 | Standard soft-core (6-exp) | sc-alpha≈0.5, sc-power=1或2, p=1得更平滑∂H/∂λ |
| 新型软核(1-1-48) | New soft-core 1-1-48 protocol | 48指数替代6，sc-alpha=0.001-0.003，统计方差更低 |
| 虚拟相互作用位点 | Virtual interaction sites | 曾称"哑原子"(dummy atoms)，无质量 |
| 构建原子 | Constructing atoms (parents) | 决定虚拟位点位置的真实原子 |
| 力的重新分配 | Force redistribution | F_i'=F_i^(direct)+(∂r_s/∂r_i)ᵀ·F_s，张量形式 |
| 2原子线性(类型2) | 2-atom linear (type 2) | r_s=w_i·r_i+w_j·r_j, w_i=1−a, w_j=a |
| 3原子线性(类型3) | 3-atom linear (type 3) | r_s=w_i·r_i+w_j·r_j+w_k·r_k, w_i=1−a−b |
| 3fd(固定距离平面内) | 3fd (fixed dist in-plane) | r_s=r_i+b·(r_ij+a·r_jk)/|r_ij+a·r_jk| |
| 3fad(固定角度距离) | 3fad (fixed angle+dist) | r_s=r_i+dcosθ·r_ij/|r_ij|+dsinθ·r_⊥/|r_⊥| |
| 3out(平面外) | 3out (out-of-plane) | r_s=r_i+a·r_ij+b·r_ik+c·(r_ij×r_ik) |
| 4fdn(四原子固定距离) | 4fdn (new 4-atom fixed dist) | 拓扑type值为2，替代不稳定旧4fd(type=1) |
| N原子线性(几何/质量/权重中心) | N-atom linear (geometric/COM/COW) | w_i=a_i/(Σa_j)，三种权重选择 |
| 维里修正 | Virial correction | 使用虚拟位点时必须**先**分配力再计算维里 |
| Ewald加和 | Ewald summation | 直接空间(erfc)+倒易空间(exp)+自能校正 |
| Ewald参数β | Ewald β | 决定直接/倒易空间划分，平衡两者计算成本 |
| 直接空间加和 | Direct space sum | V_dir=f/2·Σ q_i·q_j·erfc(βr)/r |
| 倒易空间加和 | Reciprocal space sum | V_rec=f/(2πV)·Σ q_i·q_j·Σ exp(−(πm/β)²+2πi·m·r_ij)/m² |
| 自能校正 | Self-energy correction | V₀=−(fβ/√π)·Σ q_i² |
| 色散长程校正 | Long-range dispersion correction | 假定r>r_c后g(r)=1，解析积分 |
| 能量长程校正 | Energy LR correction | V_lr=−⅔πNρ·C₆·r_c⁻³ (简单截断) |
| 压力长程校正 | Pressure LR correction | P_lr=−(4/3)π⟨C₆⟩·ρ²·r_c⁻³，S PC水~−280 bar |
| 平均色散常数 | ⟨C₆⟩ average | ⟨C₆⟩=2/[N(N−1)]·Σ_i Σ_{j>i} C₆(i,j) |
| LJ-PME | Lennard-Jones PME | 网格基长程色散，类PME但7次FFT(L-B规则需优化) |
| 几何组合近似 | Geometric combination approximation | LJ-PME用几何组合≈L-B组合，误差<0.5%但显著提速 |
| LJ直接空间修正 | LJ direct space correction | V_dir=C₆^dir·r^(−6)−C₆^recip[1−g(βr)]·r^(−6) |
| 力场 | Force field | 势函数(方程集) + 参数(同类函数多套参数) |
| 电荷组 | Charge group | 保持净电荷为零，使用PME后不再需要但保留 |

**关键概念**:
1. **软核原理**: 防止λ=0/1处粒子消失/出现时因距离→0导致奇点。标准公式r_A=(α·σ_A⁶·λ^p+r⁶)^(1/6)，p=1比p=2给出更平滑的∂H/∂λ。新型1-1-48使用48指数替代6，α取0.001-0.003
2. **虚拟位点构造**: 六种类型按构建原子数分。线性组合最简单(力也用相同权重分配)。3fd/3fad/3out需要更复杂的力分配。使用虚位点时可做到dt=5 fs
3. **Ewald→PME**: Ewald倒易空间O(N²)或O(N^(3/2))，PME用3D FFT降至O(N·logN)。PME默认pme-order=4(立方)，fourierspacing=0.12nm，ewald-rtol=1e-5
4. **色散校正重要性**: 能量校正通常小但在自由能计算中很重要。压力校正非常大约−280 bar(SPC水，r_c=0.9nm)，任何NPT模拟不可忽略
5. **LJ-PME**: 用几何组合近似Lorentz-Berthelot规则(仅倒易空间部分)，误差<0.5%总色散能但性能显著提升。倒易组合规则由lj-pme-comb-rule决定

**重要公式**:
- 简单极化: k_cs = q_s²/α
- Thole屏蔽: V_thole = (q_i·q_j/r)·[1−(1+¯r/2)e^(−¯r)], ¯r = a·r_ij/(α_i·α_j)^(1/6)
- 软核r_A: r_A = (α·σ_A⁶·λ^p + r⁶)^(1/6)
- 软核力: F_sc = (1−λ)·F^A(r_A)·(r/r_A)⁵ + λ·F^B(r_B)·(r/r_B)⁵
- Ewald: V=V_dir+V_rec+V₀, V_dir=f/2·Σq_i·q_j·erfc(βr)/r
- 能量LR校正(简单截断): V_lr = −⅔πNρ·C₆·r_c⁻³
- 压力LR校正: P_lr = −(4/3)π⟨C₆⟩·ρ²·r_c⁻³
- 虚拟位点力分配: F_i' = F_i^(direct) + (∂r_s/∂r_i)ᵀ·F_s
- 虚拟位点线性组合: r_s = Σ_i w_i·r_i, F_i = w_i·F_s

Sources: GROMACS 2019.6 中文译版, §5.5.4-5.5.9

## Practical Virtual Site Construction (Lemkul Tutorial)

Manual construction of a linear triatomic molecule (CO₂) with virtual sites. CC-BY 4.0.

### Concept
Replace two real O atoms with massless virtual sites (νO) constructed from the central C atom. This removes the C=O bond vibration (the fastest degree of freedom), enabling dt = 5 fs.

### Mass Redistribution
| Atom | Original mass | Mass with vsites |
|---|---|---|
| C (parent) | 12.011 | **43.010** (12.011 + 15.999 + 15.999) |
| νO1 (virtual) | 15.999 | **0.000** |
| νO2 (virtual) | 15.999 | **0.000** |

### Topology Format
```
[ moleculetype ]
; Name  nrexcl
CO2     3

[ atoms ]
; nr  type   resnr  residue  atom  cgnr  charge    mass
  1   C      1      CO2      C     1     0.70      43.010
  2   MW     1      CO2     vO1    1    -0.35       0.000   ; ν-site
  3   MW     1      CO2     vO2    1    -0.35       0.000   ; ν-site

[ virtual_sitesn ]
; Site  funct  a       parents
   2     2     1.0     1       ; vO1 at distance from C (a=1.0 forward)
   3     2    -1.0     1       ; vO2 opposite direction (a=-1.0)

[ constraints ]
; ai  aj  funct  b0
   1   2   1      0.116       ; C=O bond length
   1   3   1      0.116
```
- Function type 2 = linear construction: r_ν = (1−a)·r_parent1 + a·r_parent2
- Use `constraints = all-bonds` in .mdp, dt = 5 fs

### HMR vs Virtual Sites
| Approach | Max dt | Constraints | Setup effort |
|---|---|---|---|
| Standard | 2 fs | `h-bonds` | None |
| HMR | 4 fs | `h-bonds` | Low (MDP flag) |
| Virtual sites | 5 fs | `all-bonds` | High (manual topology) |

HMR is the pragmatic choice for most users; virtual sites give maximum performance.

### Key Rules
- Virtual sites **cannot be constrained** — they have no mass to apply forces to
- Position restraints go on **parent atoms**, never on virtual sites
- The coordinate file **must list virtual site atoms** in the same order as topology
- ν/ν prefix in atom names is a visual convention, not a format requirement
