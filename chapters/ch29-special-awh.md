# Chapter 29: Special Topics — AWH & Enhanced Sampling

## Core Idea
Accelerated Weight Histogram (AWH) is an adaptive biasing method that automatically computes the free energy along user-defined reaction coordinates. It requires no post-processing and converges faster than umbrella sampling for 1-3 CVs. Additional special methods include enforced rotation, electric fields, and computational electrophysiology.

## Adaptive Biasing with AWH

### How AWH Works
1. Define reaction coordinates (1-3 CVs)
2. AWH builds a bias potential that flattens the free energy landscape
3. Bias is updated adaptively using the weight histogram
4. Free energy = converged bias potential (inverted)
5. No post-processing needed — PMF comes directly from simulation

### AWH Setup (.mdp)
```ini
awh                     = yes
awh-nstout              = 100000
awh-nstsample           = 10
awh-nbias               = 1
awh1-ndim               = 1
awh1-dim1-coord-provider = pull
awh1-dim1-coord-index   = 1
awh1-dim1-start         = 0.0
awh1-dim1-end           = 3.0
awh1-dim1-force-constant = 0.0
awh1-dim1-cover-diameter = 0.2
awh1-dim1-diffusion     = 1e-5
awh1-share-group        = 0
awh1-target             = constant
awh1-error-init         = 10.0
awh1-growth             = exp-linear
awh1-equilibrate-histogram = no
```

### AWH Key Parameters
| Parameter | Purpose |
|---|---|
| `awh1-dim1-cover-diameter` | Gaussian width for sampling |
| `awh1-dim1-diffusion` | Estimated diffusion constant (controls update rate) |
| `awh1-error-init` | Initial bias uncertainty (kJ/mol) |
| `awh1-growth` | Bias growth strategy |
| `awh-share-group` | Share bias across multiple walkers |
| `awh-target` | Target distribution shape |

### AWH vs Umbrella Sampling
| Feature | AWH | Umbrella |
|---|---|---|
| Windows needed | 1 simulation | 20-40 |
| Post-processing | None (built-in) | WHAM/gmx wham |
| CVs | 1-3 | Typically 1-2 |
| Convergence | Adaptive, automatic | Manual inspection |
| Parallel efficiency | Multiple walkers | Independent windows |

## Enforced Rotation
- Apply torque to rotate groups at constant angular velocity
- Useful for: ATP synthase, motor proteins, viscosity from rotation
- `rotation = yes` in .mdp

## Electric Fields
- Apply external electric field: `E-x`, `E-y`, `E-z` (V/nm)
- Pulsed/oscillating: `E-xt = E0 * cos(omega * t)`
- Used for: ion transport, electroporation, dielectric response

## Computational Electrophysiology (CompEL)
- Double-membrane setup with ion concentration imbalance
- Creates transmembrane potential
- Counts ions traversing the channel
- Computes current from ion flux

## PMF Using Free Energy Code
- Alternative to pulling: use alchemical code for PMF
- Decouple molecule from environment along coordinate
- Can be more accurate for complex transformations

## Key Takeaways
1. AWH is the easiest PMF method — 1 simulation, no post-processing
2. AWH converges adaptively — monitor bias convergence, not simulation length
3. Diffusion constant estimate is the most important AWH parameter
4. CompEL enables direct electrophysiology simulations
5. AWH shares bias across multiple walkers for faster convergence

## Connects To
- **Ch 28**: Umbrella sampling (alternative PMF method)
- **Ch 30**: Colvars and PLUMED (alternative enhanced sampling)
- **Ch 39**: Advanced AWH MDP parameters

## 中文术语对照 (Chinese Terminology)

**专题方法** (来自中文手册 §6.1-6.7):

| 中文 | English | 说明 |
|------|---------|------|
| 平均力势 (PMF) | Potential of Mean Force | 沿反应坐标的自由能 |
| 非均衡牵引 | Non-equilibrium pulling | Jarzynski等式求自由能 |
| 牵引代码 | Pull code | 施加外力/约束于定义组 |
| 强制旋转 | Enforced rotation | 固定轴旋转 + 柔性轴旋转 |
| 计算电生理学 | Computational electrophysiology (CompEL) | 双膜系统模拟离子通道 |
| 自适应偏置 | Adaptive biasing (AWH) | 自动计算PMF |
| 伞形采样 | Umbrella sampling | 谐波约束沿反应坐标 |
| 自由能代码求PMF | PMF via free energy code | 用alchemical代码替代牵引 |
| 移除最快自由度 | Removing fastest DOF | 虚拟位点+HMR, 增大时间步长 |
| 氢原子键-键角振动 | H-atom bond-angle vibrations | HMR的物理基础 |
| 芳香基团面外振动 | Aromatic out-of-plane vibrations | 虚拟位点消除的额外自由度 |
| 粘度计算 | Viscosity calculation | Einstein关系, 压力张量自相关 |
| 表格相互作用函数 | Tabulated interaction functions | 用户自定义势能表 |
| 混合量子经典 (QM/MM) | Hybrid QM/MM | GROMACS+CP2K/MiMiC |
| 自适应分辨率 | Adaptive resolution (AdResS) | 不同区域不同分辨率 |
| 交互式MD | Interactive MD (iMD) | VMD实时操控模拟 |

Sources: GROMACS 5.0.2 中文手册 (李继存译) §6.1-6.14, CC-BY compatible.
