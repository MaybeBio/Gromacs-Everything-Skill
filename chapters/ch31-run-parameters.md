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

## 中文术语对照 (Chinese Terminology)

**运行参数和程序** (来自中文手册 §7.1-7.3):

| 中文 | English | 说明 |
|------|---------|------|
| 在线手册 | Online manual | manual.gromacs.org |
| HTML手册 | HTML manual | $GMXLIB/html/online.html |
| UNIX手册页 | UNIX man pages | `man gmx-grompp` |
| 文件类型 | File types | 表7.1列出所有文件类型 |
| 运行参数 | Run parameters | .mdp文件中的所有参数 |
| 预处理 | Preprocessing | grompp阶段 |
| 运行控制 | Run control | 时间步数, 步长, 模拟时间 |
| 邻区搜索 | Neighbor searching | nstlist, rlist, ns-type |
| 静电 | Electrostatics | coulombtype, rcoulomb, PME参数 |
| VdW | Van der Waals | vdwtype, rvdw, DispCorr |
| 表格 | Tables | 用户自定义表格势能 |
| Ewald | Ewald | fourierspacing, pme-order, ewald-rtol |
| 温度耦合 | Temperature coupling | tcoupl, tc-grps, tau-t, ref-t |
| 压力耦合 | Pressure coupling | pcoupl, pcoupltype, tau-p, ref-p, compressibility |
| 模拟退火 | Simulated annealing | annealing, annealing-npoints |
| 速度产生 | Velocity generation | gen-vel, gen-temp, gen-seed |
| 键约束 | Bond constraints | constraints, constraint-algorithm |
| 能量组排除 | Energy group exclusions | energygrp-excl |
| 墙 | Walls | nwall, wall-type, wall-r-linpot |
| 质心牵引 | COM pulling | pull code全部参数 |
| NMR精修 | NMR refinement | NMR衍生约束 |
| 自由能计算(运行参数) | Free energy (run params) | free-energy, couple-*, lambda向量 |
| 扩展系综计算 | Expanded ensemble | expanded-ensemble, lmc-stats |
| 非平衡MD | Non-equilibrium MD | acc-grps, accelerate 等 |

Sources: GROMACS 5.0.2 中文手册 (李继存译) §7.1-7.3, CC-BY compatible.
