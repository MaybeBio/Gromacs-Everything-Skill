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

**自由能相互作用** (来自中文手册 §4.5):
| 中文 | English | 说明 |
|------|---------|------|
| 自由能相互作用 | Free energy interactions | λ依赖的势能插值 |
| 线性插值 | Linear interpolation | 键合参数在A/B状态间线性变化 |
| 软核相互作用 | Soft-core interactions | 防止λ中间值奇点 |
| λ向量 | Lambda vector | 不同分量以不同速率转变 |
| 状态A/状态B | State A/State B | λ=0/λ=1 对应的物理状态 |
| 键伸缩插值 | Bond stretch interpolation | V_b = ½[(1-λ)k^A + λk^B][b − (1-λ)b₀^A − λb₀^B]² |
| 二面角插值 | Dihedral interpolation | V_d = [(1-λ)k^A + λk^B](1 + cos[nφ − (1-λ)φ_s^A − λφ_s^B]) |

**虚拟相互作用位点** (来自中文手册 §4.7):
| 中文 | English | 说明 |
|------|---------|------|
| 虚拟相互作用位点 | Virtual interaction sites | 以前称"哑原子"(dummy atoms) |
| 构建原子 | Constructing atoms (parents) | 决定虚拟位点位置的真实原子 |
| 力的重新分配 | Force redistribution | 虚拟位点上的力→分配到构建原子 |
| 张量形式 | Tensor form | F_i' = (∂r_s/∂r_i)ᵀ·F_s |
| 合力守恒 | Total force conservation | 合力和总力矩均守恒 |
| 维里修正 | Virial correction | 使用虚拟位点时必须先重新分配力再计算维里 |
| 六种构建类型 | Six construction types | 按构建原子数目分类 |

Sources: GROMACS 5.0.2 中文手册 (李继存译) §4.5, §4.7, CC-BY compatible.

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
