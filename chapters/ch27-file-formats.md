# Chapter 27: File Formats

## Core Idea
GROMACS uses a combination of text and binary file formats. .gro, .top, .itp, .mdp, and .ndx are human-readable text; .tpr, .trr, .xtc, .edr, and .cpt are binary for efficiency. Understanding each format's role in the simulation pipeline is essential.

## File Format Reference

### Structure Files
| Format | Type | Content | Usage |
|---|---|---|---|
| **.gro** | Text | Coordinates, velocities, box vectors | Default GROMACS structure format |
| **.pdb** | Text | Coordinates (ATOM/HETATM records) | Standard input from PDB database |
| **.g96** | Text | GROMOS-96 format coordinates | Legacy GROMOS compatibility |
| **.xyz** | Text | Element + xyz coordinates | Simple, widely supported |

### Topology Files
| Format | Type | Content | Usage |
|---|---|---|---|
| **.top** | Text | Complete system topology | Main topology file, #includes .itp files |
| **.itp** | Text | Molecular fragment topology | Included by .top files; reusable building blocks |
| **.rtp** | Text | Residue topology building blocks | Force field residue definitions |
| **.hdb** | Text | Hydrogen database | Rules for adding H atoms |
| **.tdb** | Text | Terminal database | N/C-terminal patches |

### Parameter & Input Files
| Format | Type | Content | Usage |
|---|---|---|---|
| **.mdp** | Text | All simulation parameters | Input to grompp |
| **.tpr** | Binary | Complete run input | Sole required mdrun input; bundle of .gro+.top+.mdp+.ndx |
| **.ndx** | Text | Atom index groups | Optional; defines groups for coupling/analysis |

### Trajectory Files
| Format | Type | Content | Size |
|---|---|---|---|
| **.trr** | Binary | Coordinates, velocities, forces (full precision) | Very large |
| **.xtc** | Binary | Coordinates only (compressed, reduced precision) | Smaller, sufficient for most analysis |
| **.tng** | Binary | Compressed trajectory (next-gen) | Portable, efficient |

### Energy & Checkpoint
| Format | Type | Content | Usage |
|---|---|---|---|
| **.edr** | Binary XDR | Energy terms over time | Input to `gmx energy` |
| **.cpt** | Binary | Full-precision simulation state | Exact restart checkpoint |
| **.log** | Text | Simulation log, performance data | Human-readable output from mdrun |
| **.xvg** | Text | Xmgrace plot data | Output from all analysis tools |

## File Format Details

### .gro File Format
```
MD of 2 waters, t= 0.0
    6
    1WATER  OW1    1   0.126   1.624   1.679  0.4027  0.0652  0.0826
    1WATER  HW2    2   0.190   1.661   1.747  0.3968  0.0848  0.1348
    ...
   3.00000   3.00000   3.00000
```
- Line 1: Title
- Line 2: Number of atoms
- Lines 3-N+2: residue number, residue name, atom name, atom number, x, y, z (nm), vx, vy, vz (nm/ps) [optional]
- Last line: Box vectors (nm)

### .top File Format
- `[ defaults ]` — combination rule, function types
- `[ atomtypes ]` — non-bonded parameter definitions
- `[ moleculetype ]` — molecule name, nrexcl
- `[ atoms ]` — nr, type, resnr, residue, atom, cgnr, charge, mass
- `[ bonds ]`, `[ pairs ]`, `[ angles ]`, `[ dihedrals ]`, etc.
- `[ system ]` — system name
- `[ molecules ]` — molecule names and counts

### .tpr File
- Binary, portable (endian-independent)
- Contains everything mdrun needs: topology, coordinates, parameters
- Can be inspected with `gmx dump -s topol.tpr`
- Can be modified with `gmx convert-tpr`

## Key Takeaways
1. .tpr is the single required input for mdrun — everything bundled in one binary
2. .xtc is 10-30x smaller than .trr — use .xtc for routine analysis
3. .cpt files are critical for crash recovery — always save them
4. .gro files have fixed-width columns — format is fragile to editing
5. .xvg is the universal analysis output format — compatible with Grace, gnuplot, matplotlib

## Connects To
- **Ch 4**: File types in the workflow
- **Ch 26**: Topology file structure details
- **Ch 13**: Tools that read/write each format
