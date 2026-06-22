# Chapter 13: Command-Line Reference

## Core Idea
The `gmx` wrapper provides access to ~100 analysis and preparation tools. This chapter covers the most commonly used commands with their purpose, key flags, and typical usage patterns.

## Essential Commands

### Simulation Pipeline

| Command | Purpose | Key Flags | Typical Usage |
|---|---|---|---|
| `gmx pdb2gmx` | PDB→GROMACS topology | `-f`, `-o`, `-p`, `-water`, `-ff`, `-ignh`, `-ter` | `gmx pdb2gmx -f prot.pdb -o prot.gro -p topol.top -water tip3p -ff amber99sb-ildn -ignh` |
| `gmx editconf` | Box definition, format conversion | `-f`, `-o`, `-c`, `-d`, `-bt` | `gmx editconf -f prot.gro -o box.gro -c -d 1.0 -bt dodecahedron` |
| `gmx solvate` | Add solvent | `-cp`, `-cs`, `-o`, `-p` | `gmx solvate -cp box.gro -cs spc216.gro -o solv.gro -p topol.top` |
| `gmx grompp` | Preprocess run input | `-f`, `-c`, `-p`, `-o`, `-n`, `-t`, `-maxwarn` | `gmx grompp -f em.mdp -c solv_ions.gro -p topol.top -o em.tpr` |
| `gmx mdrun` | Run simulation | `-s`, `-deffnm`, `-v`, `-cpi`, `-nb`, `-pme`, `-npme`, `-nt`, `-ntmpi`, `-maxh` | `gmx mdrun -deffnm md -v -cpi md.cpt` |
| `gmx genion` | Add/replace ions | `-s`, `-o`, `-p`, `-pname`, `-nname`, `-neutral`, `-conc` | `gmx genion -s ions.tpr -o neutral.gro -p topol.top -pname NA -nname CL -neutral` |
| `gmx genrestr` | Generate position restraints | `-f`, `-o`, `-fc` | `gmx genrestr -f prot.gro -o posre.itp -fc 1000 1000 1000` |

### Analysis

| Command | Purpose | Key Flags |
|---|---|---|
| `gmx energy` | Extract energy terms | `-f` (.edr), `-o`, `-xvg` |
| `gmx rms` | RMSD calculation | `-s` (.tpr), `-f` (.xtc), `-o` |
| `gmx rmsf` | RMS fluctuation | `-s`, `-f`, `-o`, `-res` |
| `gmx gyrate` | Radius of gyration | `-s`, `-f`, `-o` |
| `gmx rdf` | Radial distribution function | `-s`, `-f`, `-o`, `-ref`, `-sel` |
| `gmx msd` | Mean square displacement | `-s`, `-f`, `-o`, `-beginfit`, `-endfit` |
| `gmx hbond` | Hydrogen bond analysis | `-s`, `-f`, `-num`, `-dist`, `-ang` |
| `gmx cluster` | Conformational clustering | `-s`, `-f`, `-method`, `-cutoff`, `-cl` |
| `gmx sasa` | Solvent accessible surface area | `-s`, `-f`, `-o` |
| `gmx density` | Density distribution | `-s`, `-f`, `-o`, `-d`, `-sl` |
| `gmx mindist` | Minimum distance | `-s`, `-f`, `-od`, `-on` |
| `gmx distance` | Distance between groups | `-s`, `-f`, `-oav`, `-oall` |
| `gmx angle` | Angle/bond distributions | `-f`, `-n`, `-ov`, `-type` |
| `gmx rama` | Ramachandran plot | `-s`, `-f`, `-o` |
| `gmx chi` | Chi dihedral angles | `-s`, `-f`, `-o` |
| `gmx helix` | Helix analysis | `-s`, `-f`, `-o` |
| `gmx dssp` | Secondary structure assignment | `-f`, `-s`, `-o` |

### Conversion & Manipulation

| Command | Purpose | Key Flags |
|---|---|---|
| `gmx trjconv` | Trajectory conversion | `-f`, `-s`, `-o`, `-pbc`, `-center`, `-ur`, `-fit`, `-dt`, `-b`, `-e` |
| `gmx convert-tpr` | Modify .tpr | `-s`, `-o`, `-extend`, `-until` |
| `gmx dump` | Print binary file contents | `-s` (.tpr), `-f` (.trr/.xtc/.edr/.cpt) |
| `gmx check` | Validate files | `-f`, `-s1`, `-s2` |
| `gmx eneconv` | Energy file conversion | `-f`, `-o` |
| `gmx make_ndx` | Create index groups | `-f`, `-o` |

### Free Energy & Enhanced Sampling

| Command | Purpose | Key Flags |
|---|---|---|
| `gmx bar` | Bennett acceptance ratio | `-f` (multiple .xvg), `-g` (.edr), `-b`, `-e`, `-temp` |
| `gmx awh` | AWH post-processing | `-f` (.edr), `-s` (.tpr), `-o` |
| `gmx lie` | LIE free energy | `-f` (.edr), `-o` |
| `gmx potential` | Electrostatic potential | `-s`, `-f`, `-o` |
| `gmx disre` | Distance restraint analysis | `-s`, `-f`, `-ds` |

### Other Tools

| Command | Purpose |
|---|---|
| `gmx covar` | Covariance matrix |
| `gmx anaeig` | Essential dynamics analysis |
| `gmx nmeig` | Normal mode eigenvectors |
| `gmx nmens` | Normal mode ensemble analysis |
| `gmx dos` | Density of states |
| `gmx dyecoupl` | Dye coupling analysis |
| `gmx dielectric` | Dielectric constant |
| `gmx dipoles` | Dipole moment analysis |
| `gmx current` | Current density |
| `gmx filter` | Frequency filtering |
| `gmx order` | Order parameters |
| `gmx principal` | Principal axes |
| `gmx polystat` | Polymer statistics |
| `gmx freevolume` | Free volume |
| `gmx gangle` | General angle calculation |
| `gmx h2order` | Water hydrogen order |
| `gmx hydorder` | Hydration order |
| `gmx pairdist` | Pair distance distribution |
| `gmx clustsize` | Cluster size distribution |
| `gmx densmap` | 2D density map |
| `gmx densorder` | Density ordering |
| `gmx rmsdist` | RMS distance matrix |
| `gmx confrms` | Conformation RMS comparison |
| `gmx extract-cluster` | Extract cluster structures |
| `gmx make_edi` | Essential dynamics input |
| `gmx mdmat` | Distance matrix |
| `gmx mk_angndx` | Angle index |
| `gmx nmr` | NMR relaxation |
| `gmx nmtraj` | Normal mode trajectory |
| `gmx nonbonded-benchmark` | Nonbonded kernel benchmark |
| `gmx pme_error` | PME error estimate |
| `gmx report-methods` | Report compilation methods |
| `gmx helixorient` | Helix orientation |
| `gmx insert-molecules` | Insert molecules |
| `gmx genconf` | Generate conformations |
| `gmx enemat` | Energy matrix |

## Common Patterns
```bash
# Trajectory processing (center + PBC + fit)
gmx trjconv -s topol.tpr -f traj.xtc -o centered.xtc -pbc mol -center -ur compact

# Extract specific time range
gmx trjconv -s topol.tpr -f traj.xtc -o subset.xtc -b 10000 -e 50000 -dt 10

# RMSD of backbone
gmx rms -s topol.tpr -f centered.xtc -o rmsd.xvg -tu ns

# RMSF by residue
gmx rmsf -s topol.tpr -f centered.xtc -o rmsf.xvg -res

# Hydrogen bonds over time
gmx hbond -s topol.tpr -f centered.xtc -num hbnum.xvg

# Cluster analysis
gmx cluster -s topol.tpr -f centered.xtc -method gromos -cutoff 0.2 -cl clusters.pdb
```

## Key Takeaways
1. Use `-deffnm` for consistent file naming across grompp/mdrun
2. `-h` on any gmx tool shows full option list
3. `gmx help commands` lists all available tools
4. Most analysis tools need `-s topol.tpr` and `-f traj.xtc`
5. `gmx trjconv` is the Swiss Army knife — center, PBC fix, format conversion, time slicing

## Connects To
- **Ch 4**: Getting started workflow
- **Ch 32**: Analysis methods
