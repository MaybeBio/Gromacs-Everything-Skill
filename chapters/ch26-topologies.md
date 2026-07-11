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

## 中文术语对照 (Chinese Terminology)

**拓扑文件概述** (来自中文手册 §5.1): 拓扑文件 `*.top` 列出每个原子的**固定属性**(原子类型、电荷、键连), 区别于坐标文件 `*.gro` 中的**动态属性**(位置、速度、力)。

| 中文 | English | 说明 |
|------|---------|------|
| 拓扑文件 | Topology file (.top) | 系统完整描述 |
| 固定属性 | Fixed/static properties | 原子类型, 电荷, 键连 |
| 动态属性 | Dynamic properties | 位置, 速度, 力 (在.gro/.trr中) |
| 分子类型 | Moleculetype | `[ moleculetype ]` 定义分子名称和排除数 |
| 原子类型 | Atom type | 比元素种类更多, 力场参数化的基本单位 |
| 粒子类型 | Particle type | A(原子), S(壳层), V/D(虚拟位点) |
| 包含文件 | Include file (.itp) | 模块化的拓扑片段 |
| 力场目录 | Force field directory (.ff) | 完整的力场参数集 |
| 残基数据库 | Residue database (.rtp) | pdb2gmx的构建块 |
| 氢数据库 | Hydrogen database (.hdb) | 添加H原子的规则 |
| 末端数据库 | Terminal database (.tdb) | N端/C端修补 |
| 原子重命名 | Atom renaming (.r2b) | PDB→力场命名的转换 |
| 虚拟位点数据库 | Virtual site database (.vsd) | 虚拟位点构建规则 |
| 排除 | Exclusions | 排除已通过键合处理的非键对 |
| 预处理指令 | Preprocessor directives | `#include`, `#ifdef`, `#define` |
| 指令顺序 | Directive order | `.top`文件中各段的严格顺序 |
| 对相互作用 | Pair interactions | 1-4相互作用, 列于 `[ pairs ]` |
| 隐式溶剂化模型 | Implicit solvation | 连续介质模型, GBSA |
| 特殊键 | Special bonds | pdb2gmx添加的额外连接 |
| GMXLIB | GMXLIB | 力场文件搜索路径环境变量 |

Sources: GROMACS 5.0.2 中文手册 (李继存译) §5.1-5.7, CC-BY compatible.
