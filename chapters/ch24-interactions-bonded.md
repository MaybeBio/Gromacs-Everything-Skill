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

| 中文 | English | Notes |
|------|---------|-------|
| 成键相互作用 | Bonded interactions | 基于固定原子列表：键伸缩(2体)+键角(3体)+二面角(4体) |
| 键伸缩 | Bond stretching | 共价键i-j之间的简谐/非简谐势 |
| 简谐键势 | Harmonic bond | V=½k^b(r_ij−b_ij)²，大多数力场使用 |
| 四次幂键势 | GROMOS-96 quartic bond | V=¼k^b(r²−b²)²，不需开方，效率高 |
| Morse势 | Morse potential | V=D[1−exp(−β(r−b))]²，不对称势阱，无穷远处力为零 |
| 立方键势 | Cubic bond | V=½k^b(r−b)²+k^cub(r−b)³，OPLS/Ferguson柔性水 |
| FENE势 | FENE potential | V=−½k^b·b²·ln(1−r²/b²)，粗粒化聚合物，短距→简谐 |
| 键角弯曲 | Angle bending | i-j-k三联体的简谐/余弦势 |
| 简谐角势 | Harmonic angle | V=½k^θ(θ−θ₀)²，单位kJ/mol/rad²，拓扑输入用度 |
| GROMOS-96角势 | GROMOS cosine angle | V=½k^θ(cosθ−cosθ₀)²，力常数k^θ·sin²(θ₀)=k^(θ,harm) |
| 受限弯曲势 | Restricted bending (ReB) | V=½k·(cosθ−cosθ₀)²/sin²θ，防止θ→180° |
| Urey-Bradley势 | Urey-Bradley | V=½k^θ(θ−θ₀)²+½k^UB(r_ik−r⁰_ik)²，CHARMM特征 |
| 键-键交叉项 | Bond-bond cross-term | V=k_rr'·(|r_i−r_j|−r₁e)·(|r_k−r_j|−r₂e) |
| 键-角交叉项 | Bond-angle cross-term | V=k_rθ·(|r_i−r_k|−r₃e)·(|r_i−r_j|−r₁e+|r_k−r_j|−r₂e) |
| 四次键角势 | Quartic angle | V=Σ_{n=0}⁴ C_n(θ−θ₀)^n |
| 正常二面角 | Proper dihedral | 四原子i-j-k-l扭转，0°=顺式(cis) |
| 周期性二面角 | Periodic dihedral | V=k_φ[1+cos(nφ−φ_s)]，AMBER/CHARMM/OPLS |
| Ryckaert-Bellemans势 | RB dihedral | V=Σ_{n=0}⁵ C_n·cos^n(ψ), ψ=φ−180°，GROMOS/OPLS |
| 傅里叶二面角 | Fourier dihedral | V=½[C₁(1+cosφ)+C₂(1−cos2φ)+C₃(1+cos3φ)+C₄(1−cos4φ)] |
| OPLS→RB转换 | OPLS to RB conversion | C₀=F₂+½(F₁+F₃), C₁=½(−F₁+3F₃), C₂=−F₂+4F₄等 |
| 异常二面角/反常二面角 | Improper dihedral | 维持平面基团(芳环)或手性 |
| 受限扭转势 | Restricted torsion (ReT) | V=½k·(cosφ−cosφ₀)²/sin²φ，防止二面角跨区间 |
| 组合弯曲扭转势 | Combined Bending-Torsion (CBT) | V=k_φ·sin³θ_{i−1}·sin³θ_i·Σ a_n·cos^n φ_i，粗粒化 |
| CMAP修正 | CMAP correction | CHARMM专有，网格基φ/ψ二面角交叉项 |
| 表格式成键相互作用 | Tabulated bonded | 三次样条插值，键/角/二面角均可定制表格 |
| 排除 | Exclusions | 第一/第二相邻原子从LJ列表排除 |
| 1-4相互作用 | 1-4 interactions | 第三相邻原子，分开列表，参数可缩放 |
| 位置限制 | Position restraints | V=½k_pr|r_i−R_i|²，可三分量分别控制 |
| 平底位置限制 | Flat-bottom position restraints | V=½k_fb[d_g(r_i;R_i)−r_fb]²·H[d_g−r_fb] |
| 球/圆柱/层限制 | Sphere/Cylinder/Layer restraints | 平底势三种几何构造，可反转(r_fb负值) |
| 角限制 | Angle restraints | V=k_ar(1−cos(n(θ−θ₀)))，两对原子间或与z轴夹角 |
| 二面角限制 | Dihedral restraints | V=½k_dihr(φ'−Δφ)²，带平底区的惩罚势 |
| 距离限制 | Distance restraints | 多重边界势，用于NMR结构精修 |
| 时间平均距离限制 | Time-averaged distance restraints | ¯r_ij=⟨r_ij⁻³⟩^(−1/3)，衰减时间τ的指数运行平均 |
| 多对平均 | Multi-pair averaging | r_N(t)=[Σ_n ¯r_n(t)^(−6)]^(−1/6)，NMR NOE信号用 |
| 系综平均距离限制 | Ensemble-averaged distance restraints | 边界缩小r₁/=M^(−1/6)，M为分子数 |
| 取向限制 | Orientation restraints | δ_i=⅔tr(SD_i)，NMR残留偶极耦合/化学位移各向异性 |

**关键概念**:
1. **键伸缩势选择**: 简谐(多数力场)、GROMOS四次幂(效率高但非简谐)、Morse(非对称势阱)、立方(OPLS)、FENE(聚合物)。Morse简谐极限: exp(−βx)≈1−βx → V≈½k(r−b)²
2. **RB vs 周期性二面角**: RB使用cosⁿ(ψ), ψ=φ−180°; 周期性使用cos(nφ−φ_s)。OPLS肽保留1-4相互作用+RB; GROMOS烷烃排除1-4+RB
3. **受限弯曲势(ReB)**: 分母含sin²θ，防止θ→180°。平衡角θ₀不能太接近0°或180°(推荐至少10°差)。可与标准弯曲势组合使用
4. **CBT势**: 含sin³θ_{i−1}·sin³θ_i前因子，弯曲角→180°时扭转势自动抵消。在[−180°:180°]内只有一个最小值(非对称需用其他势)
5. **距离限制NMR**: r⁻³运行平均防止过于刚性; 多对r⁻⁶加和(物理基础); 守恒vs等力分配; 系综平均边界降M^(−1/6); 混合法防质子被拉得过近

**重要公式**:
- 简谐键: V_b=½k^b(r−b)², F=k^b(r−b)·r_ij/r_ij
- GROMOS四次幂: V=¼k^b(r²−b²)², F=k^b(r²−b²)·r_ij
- Morse: V=D[1−exp(−β(r−b))]², F=2Dβ·exp(−β(r−b))·[1−exp(−β(r−b))]
- FENE: V=−½k·b²·ln(1−r²/b²), F=−k·(1−r²/b²)^(−1)·r
- 余弦角势: V=½k^θ(cosθ−cosθ₀)², k^(θ,harm)=k^θ·sin²(θ₀)
- 周期性二面角: V=k_φ[1+cos(nφ−φ_s)]
- RB二面角: V=Σ_{n=0}⁵ C_n·cos^n(ψ), ψ=φ−180°
- 位置限制: V_pr=½k_pr|r_i−R_i|², F=−k_pr(r_i−R_i)

Sources: GROMACS 2019.6 中文译版, §5.5.2-5.5.3
