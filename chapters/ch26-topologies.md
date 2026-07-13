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

| 中文 | English | Notes |
|------|---------|-------|
| 拓扑文件 | Topology file (.top) | 系统完整拓扑描述，三层级: 参数→分子→系统 |
| 固有属性/固定属性 | Fixed/static properties | 原子类型、电荷、键连参数 |
| 动态属性 | Dynamic properties | 位置、速度、力（存储在.gro/.trr中） |
| 粒子类型 | Particle type | A(原子)、S(壳层)、V/D(虚拟位点) |
| 原子类型 | Atom type | 力场参数化基本单位，远多于元素种类 |
| 虚拟相互作用位点 | Virtual interaction sites | 由其他粒子位置构建的相互作用位点 |
| 组合规则 | Combination rule | 非键参数混合规则，cr=1几何平均，cr=2 Lorentz-Berthelot |
| gen-pairs | gen-pairs | yes时根据fudgeLJ自动生成1-4相互作用参数 |
| fudgeLJ / fudgeQQ | fudgeLJ / fudgeQQ | LJ和库仑1-4相互作用修正因子 |
| 分子类型 | Moleculetype | 分子名称和排除数nrexcl |
| 排除 | Exclusions | 排除不多于nrexcl条键的非键相互作用 |
| 电荷组 | Charge group | 电荷组总电荷应接近零 |
| 对相互作用 | Pair interactions | 1-4相互作用，函数类型1(普通)、2(自由能) |
| 包含拓扑文件 | Include topology (.itp) | 模块化分子片段，可复用 |
| 残基数据库 | Residue database (.rtp) | pdb2gmx构建单元，含原子、键、电荷 |
| 氢原子数据库 | Hydrogen database (.hdb) | 8种氢原子添加方法(平面、四面体、水等) |
| 末端数据库 | Terminal database (.tdb) | N端(.n.tdb)和C端(.c.tdb)修补 |
| 残基-构建单元转换 | r2b file | 将标准残基名转换为力场构建单元名 |
| 原子重命名数据库 | Atomic renaming (.arn) | 坐标文件原子名→力场原子名的转换 |
| 虚拟位点数据库 | Virtual site database (.vsd) | CH3/NH3质心类型、芳族侧链参数 |
| 特殊键数据库 | Special bonds (specbond.dat) | 二硫键、血红素等非骨架化学键 |
| SETTLE算法 | SETTLE algorithm | 水分子约束的解析解 |
| 约束 | Constraints | LINCS或SHAKE算法，函数类型1产生排除 |
| 自由能拓扑 | Free energy topology | B态参数添加在A态参数之后 |
| 力场目录 | Force field directory (.ff) | forcefield.itp + ffbonded.itp + ffnonbonded.itp |
| #ifdef预处理 | #ifdef preprocessor | 条件编译，配合define = -DXXX使用 |

**关键概念**:
- 拓扑文件的三层结构: 参数级别(力场设置)→分子级别(moleculetype定义)→系统级别([system]和[molecules])
- [defaults]中的nbfunc(1=LJ, 2=Buckingham)和comb-rule决定非键参数的计算方式
- pdb2gmx通过.rtp(残基)、.hdb(加氢)、.tdb(末端)、.arn(重命名)、r2b(转换)构建拓扑
- 自由能计算需在[atoms]行添加typeB、chargeB、massB，成键相互作用可指定B态参数
- 虚拟位点类型: 2体(type 1)、3体(type 1/2/3/4)、4体(type 2)、N体(COG/COM/COW)
- bondtypes/angletypes/dihedraltypes中使用通配符X匹配原子类型，精确匹配优先

Sources: GROMACS 2019.6 中文译版
