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

| 中文 | English | Notes |
|------|---------|-------|
| 加速权重直方图 | Accelerated Weight Histogram (AWH) | 自适应偏置方法，自动计算PMF |
| 参考坐标λ | Reference coordinate λ | AWH偏置作用的坐标，通过简谐势耦合到反应坐标 |
| 反应坐标ξ(x) | Reaction coordinate ξ(x) | 实际的物理坐标(如距离) |
| 偏置函数g(λ) | Bias function g(λ) | 调整沿λ分布的自由变量 |
| 目标分布ρ(λ) | Target distribution ρ(λ) | 预定义的采样分布(通常均匀) |
| 概率权重ω(λ|x) | Probability weight ω(λ|x) | AWH方法的核心样本权重 |
| 参考权重直方图W(λ) | Reference weight histogram | 代表先前采样的"理想"历史 |
| 自由能估计Fn(λ) | Free energy estimate | 每次更新后的自由能近似值 |
| 简谐耦合势Q(ξ,λ) | Harmonic coupling Q(ξ,λ) | ½βk(ξ-λ)²，k大为紧耦合 |
| 力常数k | Force constant k | 应大于PMF形貌曲率，取时间步长支持的最大值 |
| 卷积偏置势Un(ξ) | Convolution bias potential | 默认施加方式，对ξ直接施加卷积偏置 |
| 吉布斯采样 | Gibbs sampling | 替代施加方式，蒙特卡洛采样λ |
| 初始阶段 | Initial stage | Wang-Landau启发式稳健增长，更新大小近指数衰减 |
| 最终阶段 | Final stage | 权重直方图以采样率线性增长，自由能缓慢收敛 |
| 覆盖判据 | Covering criterion | 所有维度所有点被"访问"过即覆盖 |
| 覆盖直径D_cover | Cover diameter | 共享偏置时的覆盖范围参数 |
| 初始更新大小N0 | Initial update size N0 | 1/N0 ~ D·ε0²，慢系统需更大N0 |
| 增长因子γ | Growth factor γ | 覆盖后N按γ=3倍缩放 |
| 局部Boltzmann目标 | Local Boltzmann target | ρ(λ)∝W(λ)，类似良好回火元动力学 |
| 自由能截断目标 | Cut-off target ρ_cut(λ) | 指数抑制F(λ)>F_cut高能区域 |
| Boltzmann缩放目标 | Boltzmann scaling target | ρ∝exp(-s_β·F)，s_β控制平坦化程度 |
| 摩擦张量ημν(λ) | Friction tensor ημν(λ) | 与局部扩散系数成反比，峰值为慢动力学区域 |
| 多walker共享 | Multiple walkers sharing | 多个模拟共享单个偏置势加速收敛 |

**关键概念**:
- AWH核心方程: P(λ) = (1/Z)·exp(g(λ)-F(λ))，当g(λ)=lnρ(λ)+F(λ)时P(λ)=ρ(λ)
- 自由能更新: ΔFn = -ln[ (Wn+∑ω) / (Wn+∑ρ) ]，更新大小∝1/Nn随采样衰减
- 初始阶段：保持Nn不变直到覆盖，然后乘以γ=3；退出条件为新样本权重≥旧样本权重
- 覆盖判据: 多维时每个维度每个λ点都需满足ΔW(λ*)≥w_peak = Π(Δλμ/√(2π)·σk)
- 目标分布可选: 均匀ρ_const / 截断ρ_cut / Boltzmann缩放ρ_Boltz / 局部Boltzmann ρ_Boltz,loc
- PMF Φ(ξ)通过实时重加权样本计算，与F(λ)收敛速度相同
- 摩擦张量η(λ)高的区域需要更长采样时间

Sources: GROMACS 2019.6 中文译版
