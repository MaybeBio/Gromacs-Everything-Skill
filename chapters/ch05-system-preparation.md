# Chapter 5: System Preparation

## Core Idea
Proper system preparation is critical for simulation success. The key decisions are: force field selection, water model, box type/size, and ion composition. Each choice must be consistent with the others — mismatched force field/water model combinations are a common source of errors.

## Frameworks Introduced
- **Preparation checklist**: Force field → water model → box type → box size → ion placement → EM → equilibration
- **Consistency principle**: Water model must match the force field's parametrization conditions

## Key Concepts

### Force Field Selection
- **AMBER family** (ff99SB, ff99SB-ILDN, ff14SB, ff19SB): Best for proteins. TIP3P water.
- **CHARMM36** (charmm36, charmm36m): Best for proteins + lipids + nucleic acids. TIP3P water.
- **GROMOS** (53a6, 54a7): United-atom, computationally lighter. SPC water.
- **OPLS-AA/L**: Small molecules, organic solvents. TIP4P water.

### Water Models
- **SPC**: Simple 3-point, GROMOS-compatible
- **SPC/E**: Improved SPC, better dielectric
- **TIP3P**: 3-point, AMBER/CHARMM standard
- **TIP4P**: 4-point, OPLS standard
- **TIP4P/2005**: Modern 4-point, excellent phase behavior

### Box Types
- **Cubic** (`-bt cubic`): Simple, 100% volume. Use for crystals.
- **Rhombic dodecahedron** (`-bt dodecahedron`): 71% volume vs cube. **Recommended for spherical solutes** — saves 29% solvent.
- **Truncated octahedron** (`-bt octahedron`): 77% volume vs cube. Alternative to dodecahedron.

### Box Size
- `-d 1.0`: Minimum 1.0 nm between solute and box edge (standard for proteins)
- Larger for: membrane proteins, systems with significant conformational changes
- `-c`: Center the molecule in the box

### Ion Placement
- **Neutralize**: `-neutral` flag with genion — replace solvent with counterions
- **Physiological**: ~0.15 M NaCl. Calculate: `gmx genion -conc 0.15`
- **Ion naming**: Use `-pname NA -nname CL` (AMBER/CHARMM) or `-pname NA+ -nname CL-` (GROMOS/OPLS)

## Code Examples
```bash
# Generate topology
gmx pdb2gmx -f protein.pdb -o processed.gro -water tip3p -ff amber99sb-ildn -ignh

# Box definition
gmx editconf -f processed.gro -o box.gro -c -d 1.0 -bt dodecahedron

# Solvation
gmx solvate -cp box.gro -cs spc216.gro -o solv.gro -p topol.top

# Ion addition (neutral + 0.15 M NaCl)
gmx grompp -f ions.mdp -c solv.gro -p topol.top -o ions.tpr
gmx genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -neutral -conc 0.15
```

## Tips and Tricks
1. Always remove crystallographic waters and unwanted ligands from PDB before pdb2gmx
2. Use `-ignh` with pdb2gmx to ignore H atoms in PDB (they're added by the force field)
3. Check protonation states match target pH (pdb2gmx uses pH 7 by default; use `-ph` to change)
4. For membranes: use `-bt rectangular` or `-bt cubic`, not dodecahedron
5. Check `topol.top` after ion addition — verify correct number of molecules
6. The `ions.mdp` file can be minimal: just needs `integrator = steep`, `nsteps = -1`, basic cutoffs

## Key Takeaways
1. Force field and water model are a package deal — never mix
2. Rhombic dodecahedron saves 29% CPU for spherical solutes
3. Always neutralize the system — non-neutral systems cause PME artifacts
4. Check topology files after each preparation step
5. The `GMXLIB` environment variable controls where force field files are found

## Connects To
- **Ch 4**: Getting started workflow overview
- **Ch 8**: Force field details
- **Ch 16**: Advanced preparation (membranes, non-water solvents)

## Practical Workflow: Lysozyme in Water (Lemkul Tutorial)

This 12-step pipeline is the canonical GROMACS workflow. Each step is explained in detail in the original tutorial by [Justin Lemkul](http://www.mdtutorials.com/gmx/lysozyme/index.html) (CC-BY 4.0).

### Complete Command Sequence
```bash
# 1. Clean PDB: remove crystal waters, check for MISSING residues
grep -v HOH 1aki.pdb > 1AKI_clean.pdb

# 2. Generate topology (CHARMM36 + TIP3P)
gmx pdb2gmx -f 1AKI_clean.pdb -o 1AKI_processed.gro -water tip3p
# Choose force field from menu, verify terminal protonation states

# 3. Inspect topology: check [ moleculetype ] name, #include order, [ molecules ]
# KEY: The order of molecules in [ molecules ] MUST match the coordinate file order

# 4. Define unit cell
gmx editconf -f 1AKI_processed.gro -o 1AKI_newbox.gro -c -d 1.2 -bt cubic
# For production: use -bt dodecahedron to save ~29% solvent

# 5. Solvate
gmx solvate -cp 1AKI_newbox.gro -cs spc216.gro -o 1AKI_solv.gro -p topol.top
# spc216.gro works for all 3-point water models (SPC, SPC/E, TIP3P)

# 6. Add ions: first build .tpr with minimal .mdp, then neutralize
gmx grompp -f ions.mdp -c 1AKI_solv.gro -p topol.top -o ions.tpr
gmx genion -s ions.tpr -o 1AKI_solv_ions.gro -p topol.top -pname NA -nname CL -neutral
# Select group "SOL" (13) when prompted — NEVER replace protein atoms

# 7. Energy minimization (steepest descent)
gmx grompp -f minim.mdp -c 1AKI_solv_ions.gro -p topol.top -o em.tpr
gmx mdrun -v -deffnm em

# 8. Verify EM: Epot should be negative (~10⁵-10⁶), Fmax < emtol (1000)
gmx energy -f em.edr -o potential.xvg
# Select "Potential" — should show steady convergence

# 9. NVT equilibration (100 ps, position restraints ON)
gmx grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -o nvt.tpr
gmx mdrun -deffnm nvt
# Verify temperature plateau at target (298 K): gmx energy -f nvt.edr

# 10. NPT equilibration (500 ps, position restraints ON)
gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -t nvt.cpt -p topol.top -o npt.tpr
gmx mdrun -deffnm npt
# Verify density plateau: gmx energy -f npt.edr → select Density

# 11. Production MD (remove restraints)
gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_10.tpr
gmx mdrun -deffnm md_0_10

# 12. Post-process trajectory for analysis
gmx trjconv -s md_0_10.tpr -f md_0_10.xtc -o md_0_10_noPBC.xtc -pbc mol -center
# Select Protein (1) to center, System (0) for output
```

### Key Decision Points in the Workflow

**After pdb2gmx — inspect the topology:**
- `[ moleculetype ]` name is what you reference in `[ molecules ]`
- `#include` statements define parameter sources in order
- The `[ atoms ]` section shows atom types, charges, and the running total charge (`qtot`)

**After genion — verify topology was updated:**
- `[ molecules ]` should now list SOL count + counterions
- Ion names must match the force field's `ions.itp` moleculetype names
- Use elemental symbols (`NA`, `CL`) — NOT atom or residue names

**After EM — two success criteria:**
- Epot < 0 and large (10⁵–10⁶ range for typical solvated protein)
- Fmax < emtol — if Fmax > emtol but Epot is fine, reduce emstep or switch to CG
- A converged EM with Fmax still too high = likely bad starting geometry

**After NVT — check temperature:**
- Should plateau within ~20 ps at target temperature
- Fluctuations ~±2 K are normal

**After NPT — check density and pressure:**
- Density should stabilize (not necessarily match pure solvent — protein + ions change it)
- Pressure fluctuates widely (± 100 bar) — look at the running average, not raw data

### Common Mistakes
- **Forgetting `-r em.gro` in NVT grompp**: Without the reference, position restraints have no zero-point
- **Not passing `-t nvt.cpt` to NPT grompp**: Velocities and coupling state are lost; you'd need `gen_vel=yes`
- **Using `-bt dodecahedron` then wondering why the box looks triclinic**: This is normal — GROMACS uses the most efficient triclinic representation internally
