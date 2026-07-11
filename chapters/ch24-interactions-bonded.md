# Chapter 24: Bonded Interactions

## Core Idea
Bonded interactions are defined by the molecular topology and include bond stretching, angle bending, proper/improper dihedrals, and restraints. GROMACS supports multiple functional forms for each, matching different force field conventions.

## Bond Stretching
| Form | Potential | Used by |
|---|---|---|
| Harmonic | ½k_b(b − b₀)² | Most force fields |
| Fourth power | ¼k_b(b² − b₀²)² | GROMOS-96 |
| Morse | D_e[1 − exp(−β(b − b₀))]² | Special cases |
| Cubic | ½k_b(b − b₀)² + ½k_b·k_cub(b − b₀)³ | OPLS |
| FENE | −½k_b·b₀²·ln[1−(b/b₀)²] | Polymers |
| Restrained | Harmonic with flat bottom | ReaxFF |

## Angle Bending
| Form | Potential | Used by |
|---|---|---|
| Harmonic | ½k_θ(θ − θ₀)² | AMBER, CHARMM, OPLS |
| GROMOS-96 | ½k_θ(cos θ − cos θ₀)² | GROMOS |
| Cosine-based | ½k_θ(cos θ − cos θ₀)² | GROMOS variant |
| Urey-Bradley | Harmonic + 1-3 distance term | CHARMM |
| Linear | k_θ(1 − cos θ) | Special |
| Restrained | Harmonic with flat bottom | ReaxFF |

## Dihedrals

### Proper Dihedrals (4-atom torsion)
| Type | Form | Used by |
|---|---|---|
| Periodic | k_φ[1 + cos(nφ − φ_s)] | AMBER, CHARMM, OPLS |
| Ryckaert-Bellemans | Σ_n C_n·cosⁿ(ψ) with ψ = φ − 180° | GROMOS, OPLS |
| Fourier | ½[F₁(1+cos φ) + F₂(1−cos 2φ) + F₃(1+cos 3φ) + F₄(1−cos 4φ)] | OPLS |
| CMAP | Grid-based cross-term for φ/ψ backbone | CHARMM |
| Multiple periodic | Sum of periodic terms | Multiplicity |

### Improper Dihedrals (out-of-plane)
| Type | Purpose |
|---|---|
| Harmonic | Maintain planar groups (rings) |
| Periodic | Maintain chirality |

## 1-4 Interactions
- Non-bonded interactions between atoms separated by 3 bonds
- Scaled: `fudgeLJ` (default 1.0), `fudgeQQ` (default 1.0)
- Listed in `[pairs]` section of topology

## Restraints

### Position Restraints
- Harmonic potential: ½k_pr(r − R_ref)²
- Used during equilibration to keep solute near starting position
- `-DPOSRES` define activates them

### Distance Restraints
- Time-averaged or instantaneous
- Upper/lower bounds: flat-bottom harmonic well
- NMR-derived restraints

### Angle Restraints
- Similar to distance restraints for angles
- Used with NMR data

### Orientation Restraints
- Restrain orientation of vectors relative to reference
- NMR order parameter restraints (RDC)

### Dihedral Restraints
- Restrain dihedral angles to target values
- Used with NMR J-coupling data

## Key Takeaways
1. Harmonic bonds/angles are the default for most force fields
2. Ryckaert-Bellemans dihedrals are used by GROMOS and OPLS
3. CMAP correction is CHARMM-specific for backbone φ/ψ
4. Position restraints are essential during equilibration
5. 1-4 interactions are scaled separately from other non-bonded

## Connects To
- **Ch 23**: Non-bonded interactions
- **Ch 25**: Virtual sites and restraints implementation
- **Ch 26**: Topology file format

## 中文术语对照 (Chinese Terminology)

**键合相互作用框架** (来自中文手册 §4.2):
键合相互作用 = 键伸缩(双体) + 键角(三体) + 二面角(四体)。基于固定的原子列表计算。异常二面角用于保持平面或手性。

| 中文 | English | 力场/说明 |
|------|---------|-----------|
| 键伸缩 | Bond stretching | 共价键的简谐/非简谐势 |
| 简谐势 | Harmonic potential | V = ½k_b(b − b₀)², 大多数力场 |
| 四次幂势 | Fourth power / GROMOS-96 | V = ¼k_b(b² − b₀²)² |
| Morse势 | Morse potential | V = D_e[1 − exp(−β(b − b₀))]² |
| 立方键伸缩 | Cubic bond | OPLS, 含三次修正项 |
| FENE势 | FENE potential | 聚合物, ln形式 |
| 键角弯曲 | Angle bending | 三体相互作用 |
| 简谐角势 | Harmonic angle | AMBER/CHARMM/OPLS |
| GROMOS-96角势 | GROMOS angle | ½k_θ(cos θ − cos θ₀)² |
| 余弦键角势 | Cosine-based angle | GROMOS变体 |
| Urey-Bradley势 | Urey-Bradley | CHARMM, 含1-3距离项 |
| 受限弯曲势 | Restricted bending | 平底势, ReaxFF |
| 键键交叉项 | Bond-bond cross-term | 键-键耦合 |
| 键-角交叉项 | Bond-angle cross-term | 键-角耦合 |
| 正常二面角 | Proper dihedral | 四原子扭转 (i,j,k,l) |
| 异常二面角 | Improper dihedral | 维持平面/手性 |
| Ryckaert-Bellemans | RB dihedral | GROMOS/OPLS, C_n·cosⁿ(ψ) |
| 周期性二面角 | Periodic dihedral | AMBER/CHARMM/OPLS |
| CMAP修正 | CMAP correction | CHARMM专有, 网格基φ/ψ交叉项 |
| 表格键合相互作用 | Tabulated bonded | 数值形式的相互作用 |
| 1-4相互作用 | 1-4 interactions | 间隔3个键的非键作用, fudgeLJ/fudgeQQ缩放 |
| 位置限制 | Position restraints | ½k_pr(r − R_ref)², 平衡时使用 |
| 距离限制 | Distance restraints | 上下界平底势, NMR约束 |
| 角限制 | Angle restraints | 类似距离限制, 用于NMR |
| 取向限制 | Orientation restraints | NMR RDC约束 |
| 二面角限制 | Dihedral restraints | NMR J-耦合约束 |
| 极化 | Polarization | 壳层模型, Thole极化 |
| 自由能相互作用 | Free energy interactions | 软核势, λ耦合 |

Sources: GROMACS 5.0.2 中文手册 (李继存译) §4.2-4.4, CC-BY compatible.
