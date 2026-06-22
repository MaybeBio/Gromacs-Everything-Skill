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
1. Cα RMSD < 0.3 nm indicates stable protein
2. RMSF identifies flexible regions (loops > helices > sheets)
3. Rg monitors folding/compaction state
4. RDF reveals solvent/ion structure around solute
5. MSD linear region gives diffusion coefficient
6. Hydrogen bonds defined by D-A distance < 0.35 nm, angle > 150°

## Connects To
- **Ch 13**: Analysis command reference
- **Ch 19**: PBC and minimum image
- **Ch 33**: Implementation details
