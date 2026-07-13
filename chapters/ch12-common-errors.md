# Chapter 12: Common Errors

## Core Idea
Most GROMACS errors occur during three stages: pdb2gpx (force field/topology issues), grompp (parameter/topology validation), and mdrun (runtime crashes). Each error has a specific diagnostic message and a systematic fix.

## pdb2gmx Errors

### Force Field Not Found
```
No force fields found (files with name 'forcefield.itp' in subdirectories ending on '.ff')
```
- **Cause**: GMXLIB not set or force field directory missing
- **Fix**: `export GMXLIB=/path/to/gromacs/share/top` or use absolute path

### Atom Naming Mismatch
```
Atom ... not found in residue ...
```
- **Cause**: PDB atom names don't match force field .rtp entries
- **Fix**: Edit PDB to use standard atom naming, or create custom .rtp entry

### Missing Residue
```
Residue 'XXX' not found in residue topology database
```
- **Cause**: Non-standard residue (ligand, modified amino acid)
- **Fix**: Create custom .rtp entry or use external parameterization (ACPYPE, CGenFF)

### Terminus Handling
- Use `-ter` flag for interactive terminus selection
- Common issue: N-terminal NH3+ vs NH2, C-terminal COO- vs COOH

## grompp Errors

### Topology Directive Order
```
The directives in the .top and .itp files have rules about the order in which they can appear
```
- **Cause**: Wrong order of `[moleculetype]`, `[atoms]`, `[bonds]` sections
- **Fix**: Follow correct topology directive order

### Missing Include File
```
#include file 'xxx.itp' not found
```
- **Cause**: Include file not in search path
- **Fix**: Check GMXLIB, add `-I` to include parameter

### Non-Zero Total Charge
```
System has non-zero total charge
```
- **Cause**: Unbalanced ions, missing counterions
- **Fix**: Use `gmx genion -neutral` or manually adjust ion count

### Inconsistent Atom Count
```
Number of atoms in topology is not equal to number of atoms in coordinate file
```
- **Cause**: Topology and structure files don't match
- **Fix**: Regenerate topology with current structure, or check for missing/wrong molecules

## mdrun Errors

### LINCS Warnings
```
Step N: Water molecule starting at atom M can not be settled
```
- **Cause**: Bad geometry, overlapping atoms, too large timestep
- **Fix**: Run EM first, reduce dt, check starting structure

### Neighbor List Overflow
```
Neighbor list overflow
```
- **Cause**: Atoms moved too far between neighbor list updates
- **Fix**: Reduce nstlist, increase rlist buffer, set `verlet-buffer-tolerance=-1`

### PME Load Imbalance
- **Cause**: Wrong number of PME ranks
- **Fix**: `gmx tune_pme -s topol.tpr`

### Segfault at Start
- **Cause**: Wrong GMX_SIMD for CPU, incompatible GPU driver, bad build
- **Fix**: Check `gmx mdrun -version`, recompile with correct SIMD

### Domain Decomposition Error
```
There is no domain decomposition for N nodes that is compatible with the given box
```
- **Cause**: Too many MPI ranks for system size
- **Fix**: Reduce MPI ranks, ensure ~500 atoms/core minimum

## Key Takeaways
1. pdb2gmx errors are usually naming/format issues in the input PDB
2. grompp errors are parameter validation — read the error message carefully
3. LINCS warnings at step 0 = bad starting geometry (run EM first)
4. Neighbor list overflow = atoms moving too fast (reduce dt or nstlist)
5. Always check .log file after mdrun for warnings
6. `GMXLIB` is the most common environment variable issue

## Tutorial-Specific Troubleshooting (Lemkul)

### Membrane System Crashes
- **Intra-headgroup H-bonds collapsing**: Use position restraints or freeze groups on lipid headgroups during initial equilibration; reduce charges on headgroup H atoms temporarily
- **Acyl chain overlap from InflateGRO**: Don't over-pack — if area/lipid is already target, stop shrinking
- **Water/ion-headgroup overlap**: genion may place CL⁻ next to phosphate → explosion. Visualize after genion, manually remove problematic ions if needed
- **Incorrect cutoff values**: Berger lipids parametrized with 1.0 nm cutoff; GROMOS96 with 0.9 nm electrostatics, 1.4 nm vdW. Mismatched cutoffs = wrong physics

### Ligand Simulation Failures
- **Coupling ligand to its own thermostat group** (`tc-grps = Protein JZ4 SOL CL`): Ligand (22 atoms) and ions (few atoms) lack sufficient DOF → thermostat instability → crash. Always group: `Protein_Ligand Water_and_ions`
- **Wrong `#include` order for ligand .itp**: Must be AFTER parent force field but BEFORE `[ moleculetype ]`. Wrong order = undefined atom types → fatal error
- **CGenFF topology not edited**: Server output is a standalone system topology. Must remove standalone parts (forcefield include, water, ions, system section) to convert to .itp

### Free Energy Calculation Failures
- **Simulation blows up at intermediate λ**: Missing or wrong soft-core parameters. Verify `sc-alpha=0.5`, `sc-power=1`, `sc-sigma=0.3`
- **Decoupling vdW before charge**: Charged atoms approach at zero distance → infinite Coulomb force. Always discharge Coulomb first
- **BAR fails with NaN**: Check for overlapping phase space between neighboring lambda windows — insufficient overlap = BAR can't converge. Increase number of windows

### Umbrella Sampling Issues
- **WHAM fails**: No histogram overlap between adjacent windows. Decrease window spacing or increase `pull-coord1-k`
- **PMF not smooth**: Insufficient sampling per window. Run > 10 ns per window
- **Pull reference group drifts**: Reference must be immobile — apply position restraints to reference group

## Connects To
- **Ch 4**: Getting started workflow
- **Ch 9**: MDP parameters
- **Ch 13**: Command reference

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 常见错误 | common errors | 按阶段分类 |
| 内存不足 | out of memory | 分配时错误 |
| 残基未找到 | residue not found | 拓扑数据库中缺少 |
| 过长的键 | long bonds | 缺失原子导致 |
| 链标识符 | chain identifier | 非相邻使用同一标识 |
| 力场文件未找到 | force field not found | GMXLIB 路径问题 |
| 指令顺序错误 | invalid directive order | topology 节段顺序 |
| 总电荷非零 | non-zero total charge | 离子不平衡 |
| 致命错误 | fatal error | 无此分子类型 |
| 温度耦合组 | T-coupling group | 原子数过少 |
| 截断长度 | cutoff length | 超过盒子一半 |
| LINCS 警告 | LINCS warnings | 水分子无法收敛 |
| 步长太小 | stepsize too small | 能量无变化 |
| 力无穷大 | force not finite | 原子受力非有限 |
| 区域分解错误 | domain decomposition error | 进程数与盒子不匹配 |
| 爆破 | system blowing up | 系统不稳定 |

**关键概念**: GROMACS 错误按模拟阶段分为三类：pdb2gmx 错误（与输入 PDB 的命名/格式有关），grompp 错误（参数验证阶段），以及 mdrun 错误（运行时）。pdb2gmx 常见问题包括残基命名不匹配、原子缺失、力场文件找不到（检查 GMXLIB）。grompp 常见问题包括拓扑指令顺序错误、系统总电荷非零、拓扑与坐标原子数不匹配。mdrun 常见问题包括 LINCS/SETTLE 警告（通常是初始几何不好，先运行 EM）、模拟爆破（步长过大、力场参数错误）、和区域分解不兼容（进程数过多）。中文手册对每种错误都给出了中文错误信息和说明。

Sources: GROMACS 2019.6 中文译版 (§3.11)
