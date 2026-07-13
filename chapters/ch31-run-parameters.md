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

| 中文 | English | Notes |
|------|---------|-------|
| 在线手册 | Online manual | manual.gromacs.org |
| 文件类型 | File types | 参数/结构/拓扑/运行输入/轨迹/能量/其他七大类 |
| 预处理 | Preprocessing | grompp将.gro+.top+.mdp+.ndx打包为.tpr |
| 运行控制 | Run control | integrator/dt/nsteps/tinit/simulation-part |
| 邻区搜索 | Neighbor searching | cutoff-scheme(Verlet)/nstlist/rlist/ns-type |
| 静电 | Electrostatics | coulombtype(Cut-off/PME/Ewald/Reaction-Field)/rcoulomb |
| Ewald求和参数 | Ewald parameters | fourierspacing/pme-order/ewald-rtol |
| VdW | Van der Waals | vdwtype/rvdw/DispCorr(EnerPres) |
| 温度耦合 | Temperature coupling | tcoupl(berendsen/v-rescale/nose-hoover)/tc-grps/tau-t/ref-t |
| 压力耦合 | Pressure coupling | pcoupl/pcoupltype(berendsen/Parrinello-Rahman)/tau-p/ref-p/compressibility |
| 模拟退火 | Simulated annealing | annealing/annealing-npoints/annealing-time |
| 速度产生 | Velocity generation | gen-vel(从Maxwell分布)/gen-temp/gen-seed |
| 键约束 | Bond constraints | constraints(all-bonds/h-bonds)/constraint-algorithm(LINCS/SHAKE) |
| 能量监测组 | Energy monitoring groups | energygrps定义组间非键能量输出 |
| 墙 | Walls | nwall/wall-type(9-3/10-4/12-6)/wall-r-linpot |
| 质心牵引 | COM pulling | pull=yes启用，pull-coord*-*定义坐标参数 |
| 自由能(运行参数) | Free energy parameters | free-energy/couple-moltype/couple-lambda0/1/couple-intramol |
| Lambda向量 | Lambda vectors | fep-lambdas/coul-lambdas/vdw-lambdas/bonded-lambdas/restraint-lambdas/mass-lambdas |
| 软核势 | Soft-core potential | sc-alpha/sc-power/sc-sigma/sc-sigma6 |
| 扩展系综 | Expanded ensemble | expanded-ensemble/nstexpanded/lmc-stats/lmc-weights-equil |
| 非平衡MD | Non-equilibrium MD | acc-grps(加速度组)/accelerate(加速度矢量) |
| AWH参数 | AWH parameters | awh=yes/awh-nstout/awh-nstsample |
| 强制旋转参数 | Enforced rotation params | rotation/rot-group*/rot-type/k/rate/vec/pivot |
| 电场参数 | Electric field params | electric-field-x/E-xt/E0/omega/t0/sigma |
| QM/MM参数 | QM/MM params | QMMM/QMMM-grps/QMmethod/QMbasis/QMcharge/QMmult |
| NMR精修 | NMR refinement | NMR衍生距离/取向约束 |

**关键概念**:
- .mdp文件格式为 option = value，等号左侧的破折号与下划线等效
- 不指定的选项会使用默认值，grompp生成的mdout.mdp列出所有选项及默认值
- 温度耦合组: 每组可独立设定tau-t和ref-t，通常Protein和SOL+NPT分别耦合
- 压力耦合: 各向同性(isotropic)/半各向同性(semiisotropic)/各向异性(anisotropic)/表面张力(surface-tension)
- gen-vel在无.cpt重启文件时产生初始速度，gen-seed用于可重复性
- 自由能lambda向量: 分量独立，可在同一模拟中线性关闭库仑再用软核关闭VDW
- 非平衡MD: acc-grps和accelerate在选定组上施加恒定加速度

Sources: GROMACS 2019.6 中文译版
