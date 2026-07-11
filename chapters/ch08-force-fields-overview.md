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

## Ligand Parametrization: CGenFF Workflow (Lemkul Tutorial)

When simulating a non-standard small molecule with CHARMM, use the CGenFF server pipeline. CC-BY 4.0.

### The Pipeline
1. **Split**: Separate protein and ligand from PDB
2. **Protein**: `pdb2gmx` on protein alone (ligand is not in `.rtp`)
3. **Ligand**: Add H in Avogadro → export as Sybyl `.mol2`
4. **Fix .mol2**: Consistent residue names, ascending bond order (`sort_mol2_bonds.pl`)
5. **CGenFF**: Upload to server → download `.str` → convert to GROMACS `.itp`
6. **Edit .itp**: Remove standalone system parts, embed bonded parameters inline
7. **Merge**: Concatenate ligand atoms into protein `.gro`, update atom count
8. **Topology**: `#include "ligand.itp"` AFTER parent FF `#include`, BEFORE `[ moleculetype ]`

### Penalty Score Interpretation
CGenFF provides per-parameter quality scores — this is what makes it trustworthy:
- **< 10**: Reliable, ready for production use
- **10–50**: Validation warranted (check charges, run test simulations)
- **> 50**: Manual reparametrization required — parameter is unreliable

### Server Choice by Force Field
| Force Field | Server | Notes |
|---|---|---|
| CHARMM | **CGenFF** | Penalty scores; most transparent |
| AMBER/GAFF | Antechamber + **acpype** | Python wrapper writes GROMACS topology |
| GROMOS | **ATB** | Automated Topology Builder, 54A7 compatible |
| OPLS-AA | **LigParGen** | Jorgensen group, OPLS-specific |

### Force Field Modification Pattern
For systems needing new molecule types within an existing FF:
1. Copy parent `.ff` directory: `cp -r gromos53a6.ff/ gromos53a6_lipid.ff/`
2. Add new atom types → `ffnonbonded.itp` (`[ atomtypes ]` section)
3. Add nonbonded parameters → `ffnonbonded.itp` (`[ nonbond_params ]`, `[ pairtypes ]`)
4. Add bonded parameters → `ffbonded.itp` (`[ dihedraltypes ]` etc.)
5. Remove incompatible legacy entries
6. Point topology to modified FF: `#include "gromos53a6_lipid.ff/forcefield.itp"`

Never modify the original force field directory — always work on a copy.

## Connects To
- **Ch 5**: System preparation with force fields
- **Ch 16**: Advanced preparation — membrane protein FF modification
- **Ch 25**: Force field interaction function details
- **Ch 26**: Topology file structure
- **Ch 43** (Legacy Reference): Full ligand tutorial walkthrough (archived in book-to-skill source)
