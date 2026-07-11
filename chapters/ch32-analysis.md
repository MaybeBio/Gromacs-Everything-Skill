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

| 中文 | English | 说明 |
|------|---------|------|
| 分析/轨迹分析 | Analysis / Trajectory analysis | 对MD轨迹进行计算 |
| 均方根偏差 | RMSD | 与参考结构的偏差 |
| 均方根涨落 | RMSF | 每个原子/残基的柔性 |
| 回旋半径 | Radius of gyration (Rg) | 蛋白质紧致度的度量 |
| 径向分布函数 | Radial distribution function (RDF/g(r)) | 溶剂/离子结构 |
| 均方位移 | Mean square displacement (MSD) | 扩散系数 |
| 氢键 | Hydrogen bonds | 供体-受体距离+角度判据 |
| 聚类分析 | Clustering | GROMOS方法, RMSD基 |
| 二级结构 | Secondary structure | DSSP算法 |
| 主成分分析 | PCA / Essential dynamics | 原子涨落的协方差分析 |
| 能量分析 | Energy analysis | `gmx energy` 提取各能量项 |
| 二面角分析 | Dihedral analysis | Ramachandran图, χ角 |
| 相关性函数 | Correlation functions | 自相关和交叉相关 |
| 曲线拟合 | Curve fitting | `gmx analyze -f data.xvg -fit` |

Sources: GROMACS 5.0.2 中文手册 (李继存译) §8 (分析), CC-BY compatible.
