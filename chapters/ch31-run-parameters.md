# Chapter 31: Run Parameters & Programs

## Core Idea
GROMACS programs work in a pipeline: topology generation → preprocessing → simulation → analysis. Understanding the data flow between programs and the role of each file type is fundamental to the GROMACS workflow.

## Program Workflow

```
PDB file (.pdb)
    │
    ▼
gmx pdb2gmx ──→ .gro + .top
    │
    ▼
gmx editconf ──→ box.gro
    │
    ▼
gmx solvate ──→ solv.gro + updated .top
    │
    ▼
gmx grompp + gmx genion ──→ ions.tpr → neutral.gro
    │
    ▼
gmx grompp (EM) ──→ em.tpr → gmx mdrun (EM) ──→ em.gro
    │
    ▼
gmx grompp (NVT) ──→ nvt.tpr → gmx mdrun (NVT) ──→ nvt.gro + nvt.cpt
    │
    ▼
gmx grompp (NPT) ──→ npt.tpr → gmx mdrun (NPT) ──→ npt.gro + npt.cpt
    │
    ▼
gmx grompp (MD) ──→ md.tpr → gmx mdrun (MD) ──→ traj.xtc + ener.edr + md.cpt + md.log
    │
    ▼
Analysis tools ──→ .xvg output files
```

## File Types by Role

### Input Files
| File | Created by | Used by |
|---|---|---|
| .pdb | External (PDB, modeling) | pdb2gpx |
| .gro | pdb2gmx, editconf, solvate, genion, mdrun | grompp, analysis tools |
| .top | pdb2gmx (manual editing) | grompp |
| .itp | Force field packages, custom | #include in .top |
| .mdp | User-written | grompp |
| .ndx | make_ndx, auto-generated | grompp, analysis tools |
| .tpr | grompp | mdrun, analysis tools |

### Output Files
| File | Created by | Content |
|---|---|---|
| .trr | mdrun | Full trajectory (positions, velocities, forces) |
| .xtc | mdrun | Compressed trajectory (positions only) |
| .edr | mdrun | Energy terms over time |
| .cpt | mdrun | Checkpoint for exact restart |
| .log | mdrun | Log file with performance data |
| .xvg | Analysis tools | Plot data (energy, RMSD, RDF, etc.) |

## Online Documentation
- Full documentation at manual.gromacs.org
- `gmx help <command>` for command-specific help
- `gmx help commands` for full command list
- `gmx <command> -h` for detailed options

## Key Takeaways
1. The .tpr file is the single input mdrun needs
2. grompp validates and bundles all inputs into .tpr
3. Every simulation phase (EM, NVT, NPT, MD) uses its own .mdp
4. Analysis tools read .xtc + .tpr (or .gro) to produce .xvg output

## Connects To
- **Ch 4**: Getting started workflow
- **Ch 13**: Command reference
- **Ch 27**: File format specifications
