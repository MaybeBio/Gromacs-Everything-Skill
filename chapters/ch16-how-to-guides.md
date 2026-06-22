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
