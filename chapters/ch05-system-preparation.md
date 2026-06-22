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
