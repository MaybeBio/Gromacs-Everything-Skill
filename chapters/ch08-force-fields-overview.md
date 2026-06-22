# Chapter 8: Force Fields in GROMACS

## Core Idea
GROMACS supports AMBER, CHARMM, GROMOS, and OPLS force field families. Each comes with specific water model requirements, combination rules, and parametrization philosophy. Choosing the right force field for your system is critical for physically meaningful results.

## Frameworks Introduced
- **Force field selection heuristic**: Protein-only → AMBER. Protein+Lipid+NA → CHARMM. Simple/small → GROMOS. Organic liquids → OPLS.
- **Consistency rule**: Force field, water model, and ion parameters must all come from the same family.

## Force Field Families

### AMBER
- **Variants**: ff99SB, ff99SB-ILDN (improved side-chain torsions), ff14SB, ff19SB
- **Water**: TIP3P (standard), TIP4P-Ew (alternative)
- **Combination rules**: Lorentz-Berthelot (arithmetic sigma, geometric epsilon)
- **Best for**: Proteins, nucleic acids
- **Ion parameters**: Joung-Cheatham recommended

### CHARMM
- **Variants**: CHARMM36, CHARMM36m (improved for intrinsically disordered proteins)
- **Water**: TIP3P (CHARMM-modified)
- **Combination rules**: Lorentz-Berthelot
- **Best for**: Protein-membrane systems, protein-lipid-nucleic acid
- **Features**: CMAP cross-term corrections, extensive lipid parameters
- **Ion parameters**: NBFIX corrections for Na+/K+

### GROMOS
- **Variants**: 53a6, 54a7
- **Water**: SPC (standard), SPC/E
- **Combination rules**: Geometric (C6 and C12)
- **Best for**: United-atom protein simulations (computationally lighter)
- **Features**: Aliphatic CHn groups as single particles, fewer interaction sites

### OPLS
- **Variants**: OPLS-AA, OPLS-AA/L (improved liquids)
- **Water**: TIP4P
- **Combination rules**: Geometric for both sigma and epsilon (type 3)
- **Best for**: Small organic molecules, liquid simulations
- **Features**: Extensive small molecule parametrization

## Key Concepts
- **Combination rules**: How non-bonded parameters for atom pairs (i,j) are derived from pure atom types (i,i and j,j)
  - Type 1: Geometric for both C6 and C12 (GROMOS)
  - Type 2: Lorentz-Berthelot — arithmetic sigma, geometric epsilon (AMBER, CHARMM)
  - Type 3: Geometric for both sigma and epsilon (OPLS)
- **Atom types**: Force-field-specific numeric codes mapping chemical environments to parameters
- **.rtp files**: Residue topology files defining building blocks (amino acids, nucleotides)
- **.ff directories**: Complete force field packages in `share/gromacs/top/`

## Code Examples
```bash
# List available force fields
ls $GMXLIB

# Generate topology with specific force field
gmx pdb2gmx -f protein.pdb -o processed.gro -water tip3p -ff amber99sb-ildn -ignh
gmx pdb2gmx -f protein.pdb -o processed.gro -water tip3p -ff charmm36-mar2019 -ignh
gmx pdb2gmx -f protein.pdb -o processed.gro -water spc -ff gromos54a7 -ignh
```

## Key Takeaways
1. AMBER99SB-ILDN is the safest default for most protein simulations
2. CHARMM36 is the best choice for membrane protein simulations
3. Water model is force-field-specific — never mix SPC with AMBER or TIP3P with GROMOS
4. Combination rules are force-field-specific — they affect non-bonded pair interactions
5. Ion parameters matter — use the recommended set for each force field

## Connects To
- **Ch 5**: System preparation with force fields
- **Ch 16**: Adding residues, parameterizing novel molecules
- **Ch 25**: Force field interaction function details
- **Ch 26**: Topology file structure
