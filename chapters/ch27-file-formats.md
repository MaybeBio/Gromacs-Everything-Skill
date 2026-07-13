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

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 分子动力学参数文件 | MDP file (.mdp) | grompp输入参数，格式为 option = value |
| 坐标文件 | GROMACS structure (.gro) | 固定格式: %5d%-5s%5s%5d%8.3f%8.3f%8.3f%8.4f%8.4f%8.4f |
| GROMOS-96格式 | GROMOS-96 file (.g96) | 固定格式，浮点数15.9 |
| 蛋白质数据库格式 | PDB file (.pdb) | ATOM/HETATM记录，CRYST1存储盒子 |
| 运行输入文件(便携式二进制) | Run input file (.tpr) | 二进制，包含拓扑+坐标+参数，mdrun唯一输入 |
| 系统拓扑文件 | Topology file (.top) | ASCII文本，用#include包含.itp |
| 包含拓扑文件 | Include topology (.itp) | 可引用拓扑片段 |
| 残基拓扑文件 | Residue topology (.rtp) | pdb2gmx残基构建单元 |
| 索引文件 | Index file (.ndx) | [group_name]定义原子组 |
| 原子类型库 | Atom type library (.atp) | 原子序数+质量信息 |
| 原子重命名数据库 | Atom rename database (.arn) | 力场名→IUPAC/PDB名 |
| 氢原子数据库 | Hydrogen database (.hdb) | pdb2gmx构建缺失氢 |
| 虚拟位点数据库 | Virtual site database (.vsd) | 虚拟位点构建规则 |
| 末端数据库 | Terminal database (.tdb) | N端/C端封端信息 |
| 运行轨迹(全精度) | Trajectory (.trr) | x+v+f全精度便携二进制 |
| 压缩轨迹(便携式) | Compressed trajectory (.xtc) | 仅坐标，精度降低，pm单位 |
| TNG轨迹(下一代) | TNG trajectory (.tng) | 压缩、便携、任意精度 |
| 能量文件(便携式) | Energy file (.edr) | XDR协议存储能量 |
| 旧版能量文件 | Energy file (.ene) | 二进制，需用eneconv转换 |
| 检查点文件 | Checkpoint file (.cpt) | 完整状态(恒温/恒压变量+随机数+时间平均) |
| 日志文件 | Log file (.log) | 人类可读，含性能数据 |
| XVGR格式 | Xmgrace/XVG (.xvg) | 所有分析工具的输出格式 |
| XPM矩阵格式 | XPM matrix (.xpm) | 矩阵数据，与XPixMap兼容 |
| 通用数据文件 | Generic data (.dat) | 无固定格式 |
| 通用输出文件 | Generic output (.out) | 无固定格式 |
| XDR格式 | XDR format | 跨平台便携式二进制协议 |
| nm2t文件 | Atom name to type (.n2t) | 原子名→原子类型的初步转换 |

**关键概念**:
- .gro文件格式固定: 残基编号(5位)+残基名(5字符)+原子名(5字符)+原子编号(5位)+位置(3×8.3)+速度(3×8.4)
- .xtc坐标写入前乘以缩放因子(通常1000)，转为pm整数存储，利用原子空间邻近性压缩
- .cpt存储扩展系综恒温/恒压变量+随机数状态+NMR时间平均数据，确保精确重启
- .tpr使用XDR便携格式，可跨硬件平台读取，用`gmx dump -s topol.tpr`检查
- .mdp文件中选项顺序不重要但重复赋值取最后一次；选项名中的破折号与下划线等价
- .ndx文件原子编号从1开始；.xpm文件包含title/legend/x-label/y-label/type额外字段
- .xtc使用特定的XDR扩展例程存储3-D浮点坐标；magic number为1995

Sources: GROMACS 2019.6 中文译版
