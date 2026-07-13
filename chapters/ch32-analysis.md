# Chapter 32: Analysis

## Core Idea
GROMACS provides comprehensive trajectory analysis tools. The most common analyses — RMSD, RMSF, Rg, RDF, MSD, hydrogen bonds, and clustering — form the standard post-simulation workflow for understanding protein stability, dynamics, and interactions.

## Using Groups
- Analysis tools operate on atom groups (index groups)
- Default groups: System, Protein, Backbone, C-alpha, Water, etc.
- Create custom groups: `gmx make_ndx -f structure.gro`
- Most tools accept `-n index.ndx`

## Standard Analysis Tools

### RMSD (Root Mean Square Deviation)
```bash
gmx rms -s topol.tpr -f traj.xtc -o rmsd.xvg -tu ns
```
- Measures structural deviation from reference
- Fit to reference before calculation (typically backbone)
- Cα RMSD: stable protein ~0.1-0.3 nm

### RMSF (Root Mean Square Fluctuation)
```bash
gmx rmsf -s topol.tpr -f traj.xtc -o rmsf.xvg -res
```
- Per-atom or per-residue flexibility
- High RMSF: flexible loops, termini; Low RMSF: core, active site
- Correlate with B-factors from crystallography

### Radius of Gyration
```bash
gmx gyrate -s topol.tpr -f traj.xtc -o gyrate.xvg
```
- Measure of compactness: Rg² = Σ m_i·r_i² / Σ m_i
- Unfolding: Rg increases; compaction: Rg decreases

### Radial Distribution Function
```bash
gmx rdf -s topol.tpr -f traj.xtc -o rdf.xvg -ref "group A" -sel "group B"
```
- Pair correlation function g(r)
- Solvent structure, ion coordination, binding site analysis

### Mean Square Displacement
```bash
gmx msd -s topol.tpr -f traj.xtc -o msd.xvg -beginfit -1 -endfit -1
```
- Diffusion coefficient from Einstein relation: D = MSD/(6t)
- Fit linear region of MSD vs time

### Hydrogen Bonds
```bash
gmx hbond -s topol.tpr -f traj.xtc -num hbnum.xvg -dist hbdist.xvg -ang hbang.xvg
```
- Donor-Acceptor cutoff: 0.35 nm, Angle cutoff: 30°
- Secondary structure stabilization, protein-ligand interactions

### Clustering
```bash
gmx cluster -s topol.tpr -f traj.xtc -method gromos -cutoff 0.2 -cl clusters.pdb
```
- GROMOS method: RMSD-based, deterministic
- `-cutoff 0.2 nm`: similar conformations
- `-cl`: output cluster representatives as PDB

### Secondary Structure (DSSP)
```bash
gmx dssp -f traj.xtc -s topol.tpr -o ss.xpm -sc scount.xvg
```
- Per-residue secondary structure over time
- Requires DSSP binary installed

## Other Analysis Methods

### General Properties
```bash
gmx energy -f ener.edr -o potential.xvg
```
- Extract: potential, kinetic, total energy, temperature, pressure, volume, density
- Check equilibration: stable energy, temperature, density

### Covariance Analysis
```bash
gmx covar -s topol.tpr -f traj.xtc -o eigenvec.trr -v eigenval.xvg
gmx anaeig -s topol.tpr -f traj.xtc -v eigenvec.trr -2d proj.xvg
```
- Principal Component Analysis of atomic fluctuations
- Essential dynamics: dominant collective motions
- First 2-3 PCs capture >70% of variance

### Correlation Functions
- Auto- and cross-correlation of time series
- Hydrogen bond lifetime, dipole relaxation

### Curve Fitting
- Exponential, polynomial, user-defined functions
- `gmx analyze -f data.xvg -fit`

### Dihedral Analysis
```bash
gmx rama -s topol.tpr -f traj.xtc -o rama.xvg
gmx chi -s topol.tpr -f traj.xtc -g chi.log
```
- Ramachandran plots: φ/ψ backbone dihedrals
- Chi angles: side-chain rotamer analysis

## Key Takeaways
1. Cα RMSD < 0.3 nm indicates stable protein; RMSD is a degenerate metric — cannot assess convergence alone
2. RMSF identifies flexible regions (loops > helices > sheets)
3. Rg monitors folding/compaction state
4. RDF reveals solvent/ion structure around solute
5. MSD linear region gives diffusion coefficient
6. Hydrogen bonds defined by D-A distance < 0.35 nm, angle > 150°

## Practical Analysis Workflow (Lemkul Tutorial)

Standard post-production analysis sequence. CC-BY 4.0.

### Step 0: Correct Periodicity (ALWAYS FIRST)
```bash
gmx trjconv -s md.tpr -f md.xtc -o md_noPBC.xtc -pbc mol -center
# Select Protein (1) to center, System (0) for output
```
Without this, the protein may appear broken across periodic boundaries.

### Step 1: Structural Stability
```bash
# RMSD vs starting structure (after equilibration)
gmx rms -s md.tpr -f md_noPBC.xtc -o rmsd.xvg -tu ns
# Select Backbone (4) for both fit and calculation

# RMSD vs crystal structure
gmx rms -s em.tpr -f md_noPBC.xtc -o rmsd_xtal.xvg -tu ns
# em.tpr still has the original crystal coordinates (pre-EM)

# RMSF per residue
gmx rmsf -s md.tpr -f md_noPBC.xtc -o rmsf.xvg -res
```
**Interpreting RMSD**: For a stable ~15 kDa protein, RMSD ~0.09 ± 0.01 nm. Larger complexes naturally have larger RMSD. RMSD cannot assess convergence — it's a degenerate metric that accumulates many small deviations.

### Step 2: Compactness
```bash
gmx gyrate -s md.tpr -f md_noPBC.xtc -o gyrate.xvg -sel Protein -tu ns
```
Stable Rg ≈ stable fold. Drifting Rg → unfolding or compaction.

### Step 3: Secondary Structure
```bash
gmx dssp -s md.tpr -f md_noPBC.xtc -tu ns -o dssp.dat -num dssp_num.xvg
```
Requires `dssp` binary. Tracks α-helix/β-sheet/coil persistence per residue over time.

### Step 4: Hydrogen Bond Analysis
GROMACS uses two criteria (NOT the conventional donor-H-acceptor angle):
- Donor-H distance ≤ 0.35 nm (`-hbr` flag)
- Donor-acceptor-H angle ≤ 30° (so donor-H-acceptor = 150°–180°)

```bash
# Backbone H-bonds (use MainChain+H, NOT Backbone — Backbone has no H!)
gmx hbond -s md.tpr -f md_noPBC.xtc -tu ns -num hbnum_mainchain.xvg
# Select MainChain+H (7) for both groups

# Sidechain H-bonds
gmx hbond -s md.tpr -f md_noPBC.xtc -tu ns -num hbnum_sidechain.xvg
# Select SideChain (8) for both groups

# Protein-water H-bonds
gmx hbond -s md.tpr -f md_noPBC.xtc -tu ns -num hbnum_prot_wat.xvg
# Select Protein (1) and Water/SOL (12/13)
```

### Step 5: Smoothing with Running Averages
In XmGrace: Data → Transformations → Running Averages → select dataset → set window (e.g., 500 ps = 50 frames at 10 ps/frame). Smoothing reduces noise while preserving trends.

## Connects To
- **Ch 13**: Command reference for all analysis tools
- **Ch 19**: PBC, minimum image, trjconv periodicity correction
- **Ch 33**: Implementation details

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 轨迹分析 | Trajectory analysis | 对MD轨迹的各类计算分析 |
| 索引组 | Index groups | 分析工具操作的对象原子集 |
| 均方根偏差 | RMSD | 拟合(通常骨架)后与参考结构的偏差 |
| 均方根涨落 | RMSF | 每个原子/残基的时间平均涨落 |
| 回旋半径 | Radius of gyration (Rg) | Rg² = Σ mi·ri² / Σ mi，度量紧致度 |
| 径向分布函数 | Radial distribution function (RDF, g(r)) | 对相关函数，表征溶剂/离子结构 |
| 均方位移 | Mean square displacement (MSD) | D = MSD/(6t) 由Einstein关系求扩散系数 |
| 氢键 | Hydrogen bonds | 供体-受体距离≤0.35nm，供体-受体-H角≤30° |
| 聚类分析 | Clustering | GROMOS方法(RMSD基，确定性聚类) |
| 二级结构 | Secondary structure | DSSP算法，需dssp二进制 |
| 主成分分析/本征动力学 | PCA / Essential dynamics | 协方差对角化，前2-3个PC捕获>70%方差 |
| 能量分析 | Energy analysis | 提取势能/动能/总能量/温度/压力/体积/密度 |
| 拉氏图 | Ramachandran plot | φ/ψ骨架二面角分布 |
| χ角 | Chi angles | 侧链旋转异构体分析 |
| 相关性函数 | Correlation functions | 自相关(氢键寿命/偶极弛豫)和交叉相关 |
| 曲线拟合 | Curve fitting | 指数/多项式/用户自定义函数拟合 |
| PBC校正 | PBC correction | trjconv -pbc mol -center 使分子完整化 |
| 平滑处理 | Running averages | XmGrace/数据分析中减少噪声保持趋势 |
| 距离 | Distance analysis | `gmx distance` 计算两原子/组间距离 |
| 角度 | Angle analysis | `gmx angle` 计算键角分布 |
| SASA | Solvent accessible surface | 溶剂可及表面积 |
| 序参数 | Order parameter | 脂质尾链序参数 -Scd |
| 密度分布 | Density profile | 沿某轴的数密度/质量密度剖面 |
| 介电常数 | Dielectric constant | 由偶极矩涨落计算 |
| 自由能形貌图 | Free energy landscape | 沿2个CV的PMF形貌 |
| 回转张量 | Gyration tensor | 蛋白质形状的各向异性 |
| 键长/键角/二面角分布 | Bond/angle/dihedral distributions | 键合结构参数的统计分布 |

**关键概念**:
- 分析前必须PBC校正: `gmx trjconv -pbc mol -center`，否则蛋白质可能跨边界断裂
- RMSD是退化度量(degenerate metric)，不能单独评估收敛性；稳定蛋白Cα RMSD ~0.09±0.01nm
- 氢键判据: GROMACS使用donor-H距离(≤0.35nm)和donor-acceptor-H角(≤30°)，非传统DHA角
- `gmx hbond` 使用MainChain+H组而非Backbone(Backbone不含H!)
- MSD线性拟合需选择合适区域，通常舍弃开头和末尾
- GROMOS聚类: RMSD截止0.2nm，确定性算法按RMSD排序选取代表结构
- PCA分析: covar计算协方差+对角化，anaeig投影轨迹到主成分

Sources: GROMACS 2019.6 中文译版
