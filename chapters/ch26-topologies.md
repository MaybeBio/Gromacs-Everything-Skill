# Chapter 26: Topologies

## Core Idea
The GROMACS topology (.top) file defines the complete system: atom types, molecular structure, bonded/non-bonded parameters, and system composition. Include files (.itp) modularize force field parameters. The C-preprocessor-style `#include` mechanism enables conditional topology building.

## Topology File Structure
```
; System topology
#include "amber99sb-ildn.ff/forcefield.itp"
#include "amber99sb-ildn.ff/tip3p.itp"
#include "amber99sb-ildn.ff/ions.itp"

[ system ]
Protein in water

[ molecules ]
Protein_A     1
SOL         12000
NA            45
CL            50
```

## Directive Order (strict!)
1. `[ defaults ]` — combination rule, non-bonded function types
2. `[ atomtypes ]` — atom type definitions
3. `[ moleculetype ]` — molecule name and exclusions
4. `[ atoms ]` — atom listing: nr, type, resnr, residue, atom, cgnr, charge, mass
5. `[ bonds ]` — bond constraints
6. `[ pairs ]` — 1-4 interaction parameters
7. `[ angles ]` — angle parameters
8. `[ dihedrals ]` — proper dihedral parameters
9. `[ impropers ]` — improper dihedral parameters
10. `[ exclusions ]` — explicit atom exclusions
11. `[ constraints ]` — bond constraints (alternative to [bonds])
12. `[ virtual_sitesN ]` — virtual site constructions
13. `[ position_restraints ]` — position restraint parameters

## Include File (.itp)
```itp
[ moleculetype ]
SOL   3

[ atoms ]
1  OW  1  SOL  OW  1  -0.820
2  HW  1  SOL  HW2 1   0.410
3  HW  1  SOL  HW3 1   0.410

[ constraints ]
1  2  1  0.1000
1  3  1  0.1000

[ exclusions ]
2 3
```

## Preprocessor Features
- `#include "file.itp"` — inline include (searches current dir, GMXLIB, -I paths)
- `#ifdef POSRES_WATER` / `#endif` — conditional block
- `#define POSRES_WATER` — activate conditionals
- mdp `define = -DPOSRES` — set defines from .mdp

## Force Field .ff Directory
```
forcefield.itp    → [ defaults ] + [ atomtypes ] + non-bonded parameters
ffbonded.itp      → Bonded parameters
ffnonbonded.itp   → Non-bonded parameters per atom type
aminoacids.rtp    → Residue building blocks
*.tdb             → Terminal patches (N-term, C-term)
*.hdb             → Hydrogen database
ions.itp          → Ion parameters
watermodel.itp    → Water model parameters
```

## System Assembly
```bash
# pdb2gmx generates topology + structure
gmx pdb2gmx -f protein.pdb -o processed.gro -p topol.top

# Then manually add to topol.top:
[ molecules ]
Protein_chain_A     1
SOL              12345
NA                   45
```

## Key Takeaways
1. Directive order is strict — wrong order causes grompp errors
2. .itp files are modular building blocks
3. Preprocessor defines enable conditional topology building
4. #include mechanism prevents duplication and typos
5. GMXLIB env var controls force field search path

## Connects To
- **Ch 8**: Force field overview
- **Ch 12**: Common topology errors
- **Ch 27**: File format specifications
