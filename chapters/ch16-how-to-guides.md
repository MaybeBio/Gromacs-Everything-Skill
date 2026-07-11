# Chapter 16: How-To Guides

## Core Idea
Step-by-step guides for common advanced tasks: parameterizing novel molecules, setting up membranes, creating mixed solvents, and running specialized simulation types.

## Adding a Residue to a Force Field
1. Create .rtp entry in force field directory with `[atoms]`, `[bonds]`, `[angles]`, `[dihedrals]`
2. Add atom types to `[atomtypes]` section
3. Add bonded parameters to `[bondtypes]`, `[angletypes]`, etc.
4. Use `pdb2gmx -ff <your_ff>` to test

## Parameterization of Novel Molecules
1. Use external tools: ACPYPE (AMBER GAFF), CGenFF (CHARMM), ATB (GROMOS)
2. Generate .itp file for the molecule
3. `#include` the .itp in your system topology
4. Validate with QM calculations or experimental data

## Membrane Simulations
1. Use CHARMM36 force field (best lipid parameters)
2. Align protein to membrane normal with editconf
3. Use `gmx membed` or INSert membrane with `gmx insert-molecules`
4. Semisotropic pressure coupling (`pcoupltype = semiisotropic`)
5. Longer equilibration (100+ ns) for proper lipid packing
6. Check area per lipid and thickness during equilibration

## Water Solvation
```bash
gmx editconf -f protein.gro -o box.gro -c -d 1.0 -bt dodecahedron
gmx solvate -cp box.gro -cs spc216.gro -o solv.gro -p topol.top
```

## Non-Water Solvent
```bash
# Create solvent box first
gmx insert-molecules -ci solvent.pdb -nmol 1000 -box 5 5 5 -o solvent_box.gro
# Then solvate as usual
gmx solvate -cp box.gro -cs solvent_box.gro -o solv.gro -p topol.top
```

## Mixed Solvent
1. Create topology entries for each solvent species
2. Specify numbers of each molecule in .top
3. Use `gmx solvate` multiple times with different `-cs` files

## Disulfide Bonds
- Use `gmx pdb2gmx -ss` to auto-detect from CYS SG distances
- Or manually specify in `specbond.dat`

## PMF Calculations
- Pull code: constant velocity or umbrella sampling
- Alchemical: free energy code with soft-core potentials
- AWH: automatic adaptive method

## Key Takeaways
1. CHARMM36 is the recommended force field for membrane simulations
2. Semisotropic pressure coupling is required for membranes
3. External tools (ACPYPE, CGenFF) are needed for novel molecule parameters
4. AWH is the easiest method for 1D PMF calculations

## Connects To
- **Ch 5**: Basic system preparation
- **Ch 28**: Pull code details
- **Ch 29**: AWH details

## Membrane Protein Construction: InflateGRO Method

**Note**: This historical method (Lemkul tutorial, CC-BY 4.0) teaches force field internals. For CHARMM36 production systems, use [CHARMM-GUI](http://www.charmm-gui.org) instead.

### Force Field Modification for Lipids
When lipid parameters don't exist in your chosen force field, you must merge them:

1. Copy the parent force field directory: `cp -r gromos53a6.ff/ gromos53a6_lipid.ff/`
2. Add lipid `[ atomtypes ]` (with atomic number column) → `ffnonbonded.itp`
3. Copy `[ nonbond_params ]` and `[ pairtypes ]` → `ffnonbonded.itp`
4. Copy `[ dihedraltypes ]` → `ffbonded.itp`
5. **Remove** deprecated interaction lines incompatible with your force field
6. Add `#include "dppc.itp"` to topology AFTER the protein position restraint section

**Critical**: The `#include` order matters — parent force field first, then lipid parameters, then molecule definitions.

### InflateGRO Packing Protocol
Embed a protein in a lipid bilayer via iterative scaling:

1. **Orient**: Align protein and membrane in same coordinate frame (`editconf -box`)
2. **Inflate**: Scale lipid positions ×4 to create void around protein
3. **Delete overlaps**: Remove lipid molecules within cutoff (Å) of protein atoms
4. **EM**: Energy minimize the inflated system (strong position restraints on protein)
5. **Shrink iteratively**: Scale ×0.95, EM again, repeat ~26× until area/lipid ≈ 71 Å² (target for DPPC ≈ 62 Å² after equilibration)
6. **Solvate**: Add water, then remove water from membrane core with `water_deletor.pl`

### Membrane-Specific Equilibration Rules
- `pcoupltype = semiisotropic` — xy and z scale independently (isotropic distorts bilayer)
- `tc-grps = Protein_Lipid Water_and_ions` — never couple lipids or ions separately
- `comm-grps = Protein_Lipid Water_and_ions` — prevent phase drift in interfacial systems
- Simulate **above lipid Tm**: DPPC ≥ 323 K (Tm = 315 K). Below Tm → gel phase.
- NPT should run 1+ ns — lipid equilibration is much slower than water

### Lipid Tm Reference
| Lipid | Area/lipid (Å²) | Tm (K) |
|---|---|---|
| DPPC | 62–64 | 315 |
| DMPC | 60.6 | 297 |
| POPC | 65.8 | 271 |
| POPE | 56 | 298 |
| POPG | 53 | 269 |

### Key Analysis Commands
```bash
# Deuterium order parameters (per acyl chain)
gmx make_ndx -f md.tpr -o sn1.ndx   # select acyl chain carbons only
gmx order -s md.tpr -f md.xtc -n sn1.ndx -d z -od deuter.xvg

# Membrane density profile (by component)
gmx density -s md.tpr -f md.xtc -n density_groups.ndx -o dens.xvg -d Z

# Lipid lateral diffusion
gmx make_ndx -f md.tpr -o p8.ndx     # select P8 atoms only
gmx msd -s md.tpr -f md.xtc -n p8.ndx -lateral z
```

## Biphasic System Construction

For liquid-liquid interfaces (e.g., water/cyclohexane):

1. Build each phase separately (homogeneous solvent boxes)
2. Combine with `editconf -translate` or manual coordinate concatenation
3. Use `gmx insert-molecules` (not `solvate`) for non-aqueous solvents
4. Gentle EM at the interface (steep, small `emstep`)
5. Semi-isotropic pressure coupling during equilibration

`gmx insert-molecules -f system.gro -ci solvent.gro -nmol 500 -try 50 -radius 0.12`
- `-try`: attempts per molecule (increase if placement fails in crowded box)
- `-radius`: vdW clash detection threshold
