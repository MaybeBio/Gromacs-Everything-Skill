# Chapter 28: Special Topics — Free Energy & Pulling

## Core Idea
The pull code applies external forces to defined groups, enabling umbrella sampling, steered MD, and AFM pulling. Combined with the free energy code, it computes potential of mean force (PMF) along reaction coordinates.

## Free Energy Implementation
- `free-energy = yes` enables alchemical transformations
- Couple parameters: `couple-moltype`, `couple-lambda0`, `couple-lambda1`
- `couple-intramol = yes` for intramolecular interactions
- Lambda schedule: `init-lambda` + `delta-lambda` or explicit `init-lambda-state`
- `calc-lambda-neighbors = 1` for BAR analysis
- `nstdhdl = 10` for dH/dλ output frequency
- Separate VdW and Coulomb coupling: `couple-lambda0 = vdw-q`, `couple-lambda1 = none`

## Potential of Mean Force

### Pull Code Setup
```ini
pull                    = yes
pull-ncoords            = 1
pull-ngroups            = 1
pull-group1-name        = Chain_A
pull-group2-name        = Chain_B
pull-coord1-type        = umbrella      ; or constant-force, constant-rate
pull-coord1-geometry    = distance      ; direction, direction-periodic, cylinder
pull-coord1-dim         = N N Y
pull-coord1-rate        = 0.0
pull-coord1-k           = 1000          ; force constant (kJ/mol/nm²)
pull-coord1-start       = yes
```

### Pull Types
| Type | Purpose | Key parameters |
|---|---|---|
| `umbrella` | Harmonic restraint along coordinate | `pull-coord1-k` |
| `constraint` | Fixed distance (SHAKE-like) | No force constant needed |
| `constant-force` | Constant pulling force | `pull-coord1-k` as force magnitude |
| `flat-bottom` | Harmonic only outside range | `pull-coord1-tol` |
| `flat-bottom-high` | Harmonic only above range | Upper wall restraint |

### Pull Geometries
| Geometry | Description |
|---|---|
| `distance` | Center-of-mass distance |
| `direction` | Projection onto a vector |
| `direction-periodic` | Direction with periodic component |
| `cylinder` | Radial distance from axis |
| `angle` | Angle between groups |
| `dihedral` | Dihedral angle |
| `position` | Absolute position |

## Non-Equilibrium Pulling
- Constant velocity: `pull-coord1-rate > 0`
- Steered MD: time-dependent pulling
- Jarzynski equality for free energy from non-equilibrium work
- Multiple trajectories needed for convergence

## Workflow
```bash
# Generate umbrella windows
for i in $(seq 0 0.1 3.0); do
    # Set pull-coord1-init = $i in each window
    gmx grompp -f umbrella.mdp -c npt.gro -p topol.top -o window_${i}.tpr
done

# Run all windows
gmx mdrun -multidir window_*

# Analyze with WHAM
gmx wham -it tpr_files.dat -if pullf_files.dat -o profile.xvg -hist histo.xvg
```

## Key Takeaways
1. Umbrella sampling is the standard method for 1D PMF calculations
2. WHAM or umbrella integration for post-processing
3. Pull force constant ~1000 kJ/mol/nm² is typical
4. Use 20-40 windows for well-converged PMF
5. Non-equilibrium pulling needs many trajectories (Jarzynski)

## Connects To
- **Ch 21**: Free energy theory
- **Ch 25**: Soft-core potentials
- **Ch 29**: AWH (automatic alternative to umbrella sampling)

## Practical Umbrella Sampling Workflow (Lemkul Tutorial)

Based on peptide dissociation from Aβ42 protofibril (Lemkul & Bevan, 2010). CC-BY 4.0.

### Step 1: Generate Configurations via Pulling
```ini
pull                    = yes
pull-ncoords            = 1
pull-ngroups            = 2
pull-group1-name        = Chain_A      ; the molecule being pulled
pull-group2-name        = Chain_B      ; immobile reference
pull-coord1-type        = umbrella
pull-coord1-geometry    = distance
pull-coord1-groups      = 1 2
pull-coord1-rate        = 0.01         ; nm/ps — SLOW for near-equilibrium
pull-coord1-k           = 1000         ; kJ/mol/nm²
```
```bash
gmx grompp -f pull.mdp -c npt.gro -p topol.top -n index.ndx -t npt.cpt -o pull.tpr
gmx mdrun -deffnm pull -pf pullf.xvg -px pullx.xvg
```

### Step 2: Extract Frames at Target Spacings
Read `pullx.xvg` to find the frame where COM distance ≈ target for each window. Extract with `trjconv`. Typical spacing: 0.1–0.2 nm, ensuring ~1–2 Å overlap in position distributions.

### Step 3: Run Umbrella Windows
Each window: fixed restraint (`pull-coord1-rate = 0.0`), 5–20 ns production, independent directory.
```ini
pull-coord1-init        = <target_distance>
pull-coord1-rate        = 0.0          ; fixed position
pull-coord1-k           = 1000
```

**Force constant tuning:**
- Too weak (k < 500): system drifts, poor localization
- Too strong (k > 3000): sampling too narrow, poor overlap
- **Rule of thumb**: k should produce σ ≈ 0.05–0.1 nm in the position distribution

### Step 4: WHAM Analysis
```bash
gmx wham -it tpr-files.dat -if pullf-files.dat -o profile.xvg -hist histo.xvg
```
- **Check histogram overlap before WHAM**: visualize position distributions from each window
- Gaps = windows too far apart or force constant too weak → WHAM fails
- ΔG_bind = PMF_plateau − PMF_minimum

### Common Pitfalls
| Issue | Cause | Fix |
|---|---|---|
| WHAM fails to converge | No histogram overlap | Decrease window spacing or increase k |
| PMF not smooth | Insufficient sampling | Longer production per window (> 10 ns) |
| Unphysical PMF shape | Pull rate too high in Step 1 | Reduce `pull-coord1-rate` to ≤ 0.005 |
| Reference drifts | Chain_B too small | Apply position restraints to reference group |

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 自由能计算 | free energy calculation | 热力学积分或 BAR 方法 |
| 终止状态 | end states | λ=0 (初态) 和 λ=1 (终态) |
| λ 向量 | lambda vector | 库仑/VdW/成键/约束/质量分量独立 |
| 热力学积分 | thermodynamic integration (TI) | 积分 ∂H/∂λ |
| 平均力势 | potential of mean force (PMF) | 沿反应坐标的自由能剖面 |
| 熵效应 | entropy contribution | Vpmf(r) = -(nc-1)kBT log(r) |
| 非平衡牵引 | non-equilibrium pulling | 恒速拉动 |
| Jarzynski 关系 | Jarzynski equality | 非平衡功计算自由能 |
| 牵引代码 | pull code | 对原子组质心施加力/约束 |
| 伞形牵引 | umbrella pulling | 简谐势维持距离 |
| 约束牵引 | constraint pulling | SHAKE 约束固定距离 |
| 恒力牵引 | constant-force pulling | 恒力拉动 |
| 平底牵引 | flat-bottom pulling | 区间内无力 |
| 牵引几何 | pull geometry | distance/direction/cylinder/angle/dihedral |
| 方向相对几何 | direction-relative | 相对参考组的方向牵引 |
| 圆柱几何 | cylinder geometry | 适用于脂质双层 PMF |
| 余弦权重剖面 | cosine weighting | 周期性系统的参考位置定义 |
| WHAM 分析 | WHAM analysis | 加权直方图分析法 |

**关键概念**: GROMACS 中自由能计算通过 λ 向量耦合实现。从 4.6 版本开始，库仑、范德华、成键、约束和质量分量可独立调整，可实现先关闭库仑项再关闭范德华项的渐进自由能路径。PMF 有多种计算方法：牵引代码（分子/分子组质心之间）、AWH（自适应偏置）、自由能代码配合简谐键/约束。Jarzynski 关系可以从非平衡牵引的功计算出平衡自由能变化。牵引代码支持多种几何类型，其中圆柱几何特别适用于脂质双层 PMF。注意熵效应校正：300 K 时距离减半贡献约 3.5 kJ/mol。约束牵引严格仅适用于不含约束的整个分子/组，但大组加小力时误差可忽略。

Sources: GROMACS 2019.6 中文译版 (§5.8.1-5.8.4)
