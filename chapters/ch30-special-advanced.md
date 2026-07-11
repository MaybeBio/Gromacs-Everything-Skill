# Chapter 30: Special Topics — Advanced Methods

## Core Idea
GROMACS supports advanced simulation methods beyond classical MD: QM/MM (CP2K, MiMiC), collective variable modules (PLUMED, Colvars), neural network potentials (NNP), fast multipole method (FMM), and specialized techniques (viscosity, shear, interactive MD, membrane embedding).

## QM/MM Hybrid Simulations

### CP2K Interface
- GROMACS handles MM, CP2K handles QM
- QM atoms selected via `QMMM-grps` in index file
- Link atoms cap broken QM/MM bonds
- Communication via socket-based interface
- Requires CP2K compiled with GROMACS interface

### MiMiC QM/MM
- Alternative QM/MM framework
- CPMD (or other QM code) does integration
- `integrator = mimic` in .mdp
- More flexible QM code selection
- Experimental feature in 2026.2

## Collective Variable Modules

### PLUMED
- External library patching via PLUMED plugin
- Extensive CV library + enhanced sampling (metadynamics, OPES, VES)
- PLUMED input file: `plumed.dat`
- Use: `gmx mdrun -plumed plumed.dat`

### Colvars Module
- Built-in Colvars collective variables module
- Standard CV types: distance, angle, RMSD, coordination, alpha, etc.
- Colvars config file: `colvars.in`
- Wall, harmonic, and ABF (adaptive biasing force) restraints

## Neural Network Potentials (NNP)
- Machine-learned force fields replacing classical potentials
- QM-quality accuracy at near-classical cost
- GROMACS interface to external NNP libraries
- Experimental feature — validate carefully

## Fast Multipole Method (FMM)
- Alternative to PME for long-range electrostatics
- O(N) scaling (vs O(N log N) for PME)
- Better for very large systems (>1M atoms)
- Experimental feature in 2026.2

## Additional Methods

### Removing Fastest Degrees of Freedom
- Virtual sites for hydrogen atoms
- Eliminates C-H, O-H bond vibrations
- Enables 4-5 fs timestep

### Viscosity Calculation
- Einstein relation from pressure tensor autocorrelation
- Non-equilibrium: cosine acceleration method
- `gmx energy` extracts pressure tensor components

### Shear Simulations
- Lees-Edwards boundary conditions
- Cos(x) acceleration for shear flow
- Used for rheology, non-Newtonian fluids

### Tabulated Interaction Functions
- User-defined potential tables
- `table.xvg` (non-bonded), `tablep.xvg` (pair)
- 7 columns: x, f(x), -f'(x), g(x), -g'(x), h(x), -h'(x)

### Interactive Molecular Dynamics (iMD)
- Real-time steering via VMD plugin
- Apply forces interactively during simulation
- Educational and exploratory tool

### Membrane Embedding
- Automated insertion of proteins into membranes
- `gmx membed` tool
- Minimizes unfavorable contacts

### 3D Density Forces
- Apply forces from 3D density maps (e.g., cryo-EM)
- Guide simulation toward experimental density

## Key Takeaways
1. QM/MM with CP2K is the standard for reactive simulations
2. PLUMED provides the most extensive enhanced sampling CV library
3. Neural network potentials are experimental — validate thoroughly
4. FMM may replace PME for million-atom systems
5. Interactive MD is useful for exploration and education

## Connects To
- **Ch 21**: Free energy methods
- **Ch 29**: AWH adaptive biasing
- **Ch 25**: Virtual sites for removing fast DOF

## 中文术语对照 (Chinese Terminology)

**高级方法** (来自中文手册 §6.8-6.14):

| 中文 | English | 说明 |
|------|---------|------|
| 混合量子经典 | QM/MM (Hybrid QM/Classical) | GROMACS(MM) + CP2K/MiMiC(QM) |
| 链接原子 | Link atoms | 封端QM/MM边界 |
| 自适应分辨率方案 | Adaptive resolution scheme (AdResS) | 不同区域不同精度 |
| 粗粒化力场 | Coarse-grained force fields | MARTINI, PLUM |
| 集体变量 | Collective variables (Colvars) | 描述集合运动, PLUMED接口 |
| 神经网络势 | Neural network potentials (NNP) | 机器学习力场 |
| 快速多极子方法 | Fast multipole method (FMM) | O(N)长程静电替代PME |
| 交互式分子动力学 | Interactive MD (iMD) | VMD插件实时操控 |
| 剪切模拟 | Shear simulations | Lees-Edwards边界 |
| 表格势能函数 | Tabulated potentials | 三次样条插值, 用户自定义 |
| 粘度 | Viscosity | 压力张量自相关或余弦加速 |
| 膜嵌入 | Membrane embedding | `gmx membed` 自动嵌入 |

**力场** (来自中文手册 §4.10):

| 中文 | English | 说明 |
|------|---------|------|
| GROMOS-96 | GROMOS-96 | 联合原子, 几何组合规则 |
| OPLS/AA | OPLS-AA | 全原子, 有机分子 |
| AMBER | AMBER | 全原子, 蛋白质/核酸 |
| CHARMM | CHARMM | 全原子, CMAP修正 |
| MARTINI | MARTINI | 粗粒化力场, 4:1映射 |
| PLUM | PLUM | 蛋白质-配体粗粒化力场 |

Sources: GROMACS 5.0.2 中文手册 (李继存译) §4.10, §6.8-6.14, CC-BY compatible.
