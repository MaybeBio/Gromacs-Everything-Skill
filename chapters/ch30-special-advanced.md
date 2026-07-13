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

| 中文 | English | Notes |
|------|---------|-------|
| 电场 | Electric field | 脉冲/振荡电场: E(t)=E0exp[-(t-t0)²/2σ²]cos[ω(t-t0)] |
| 计算电生理学 | Computational electrophysiology (CompEL) | 双膜系统，离子浓度不平衡建立跨膜电位 |
| 跨膜电位差 | Transmembrane potential ΔU | ΔU=Δq/C_membrane，由电荷不平衡维持 |
| 离子交换 | Ion swapping | 腔室间交换离子/水分子恢复参考离子数 |
| 交换频率 | swap-frequency | 离子交换尝试频率 |
| 划分组 | split-group | 定义腔室边界(通道中心) |
| 通道圆柱体 | Channel cylinder | 追踪通过每个通道的离子种类和数目 |
| 反转电位 | Reversal potential | I-U关系线性拟合零电流点 |
| 通道电导 | Channel conductance G | G=∑niqi/(Δt·ΔU) |
| 强制旋转 | Enforced rotation | 多种势能(各向同性/平行运动/径向运动/柔性轴) |
| 固定轴旋转 | Fixed axis rotation | 恒定ω绕v̂轴，支点u |
| 柔性轴旋转 | Flexible axis rotation | 板块划分+高斯加权，消除刚体行为 |
| 虚拟弹簧势 | Virtual spring potential | 原子附着于移动参考位置 |
| 平行运动势 | Parallel motion potential | 消除平行于旋转轴的力分量 |
| 径向运动势 | Radial motion potential | 消除/减少径向力分量 |
| 旋转组角度θ_av | Rotation group angle | 固定轴: 距离加权角度偏差均值 |
| 力矩τ | Torque | 旋转势能力矩，N·m |
| 三次样条插值 | Cubic spline interpolation | Vs(x)=A0+A1ε+A2ε²+A3ε³, h=x_{i+1}-x_i |
| 剪切粘度(平衡) | Shear viscosity (equilibrium) | 爱因斯坦关系: η=(V/2kBT)lim d/dt ⟨(∫Pxz)²⟩ |
| 剪切粘度(非平衡) | Shear viscosity (non-equilibrium) | 余弦加速剖面: ax(z)=Acos(2πz/lz) |
| 速度剖面 | Velocity profile | vx(z)=Vcos(2πz/lz), V=Aρ(lz/2π)²/η |
| 移除最快自由度 | Removing fastest DOF | 虚拟位点+H质量重分配，最大时间步长~4fs |
| 芳香基团面外振动 | Aromatic out-of-plane vibrations | 虚拟位点固定平面，消除反常二面角 |
| 氢原子虚拟位点 | Hydrogen virtual sites | 固定键距/固定键角/哑质点等多种构建方法 |
| 使用自由能代码求PMF | PMF via free energy code | 简谐键/约束+λ改变距离，或位置限制方法 |

**关键概念**:
- CompEL: 双膜体系，Δq不平衡建立ΔU，控制离子数目参考值，检测通道通量和电导
- 电场类型: 静态(σ=0,ω=0)、振荡(σ=0)、脉冲振荡(σ>0)
- 强制旋转势能类型丰富: 各向同性/平行运动/径向运动/二型径向运动，各有固定轴和柔性轴变体
- 柔性轴: 板块高斯加权求和(σ=0.7Δx, 总和≈1不变)，板块中分面可附着于质心(耐平动)
- 非平衡粘度: 余弦加速法，剪切速率sh_max=Aρlz/(2πη)，需确保系统不偏离平衡太远
- 虚拟位点构建: 羟基用约束固定键角；胺基用哑质点保留旋转自由度；芳环用3个约束原子
- 表格相互作用: 三次样条，查表+插值计算势能和力

Sources: GROMACS 2019.6 中文译版
