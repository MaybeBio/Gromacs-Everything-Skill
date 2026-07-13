# Chapter 39: Advanced MDP Parameters

## Core Idea
Beyond basic simulation parameters, GROMACS supports advanced .mdp options for free energy calculations, enhanced sampling (AWH, expanded ensemble, pull code), and QM/MM. These parameters enable specialized simulation workflows.

## Free Energy Parameters
| Parameter | Default | Description |
|---|---|---|
| `free-energy` | no | Enable free energy calculation |
| `couple-moltype` | — | Molecule to couple/decouple |
| `couple-lambda0` | vdw-q | Interactions at lambda=0 |
| `couple-lambda1` | none | Interactions at lambda=1 |
| `couple-intramol` | no | Couple intramolecular |
| `init-lambda` | 0.0 | Starting lambda |
| `delta-lambda` | 0.0 | Lambda step per window |
| `init-lambda-state` | -1 | Explicit lambda state |
| `nstdhdl` | 10 | dH/dλ output frequency |
| `calc-lambda-neighbors` | -1 | Lambda neighbors for BAR |
| `sc-alpha` | 0.0 | Soft-core alpha |
| `sc-power` | 1 | Soft-core power |
| `sc-sigma` | 0.3 | Soft-core sigma (nm) |
| `sc-coul` | no | Soft-core Coulomb |

## Expanded Ensemble Parameters
| Parameter | Description |
|---|---|
| `expanded-ensemble` | Enable expanded ensemble |
| `nstexpanded` | Update frequency |
| `lmc-stats` | Lambda dynamics method |
| `lmc-weights-equil` | Weight equilibration steps |
| `weight-equil-wl-delta` | Wang-Landau delta |

## Pull Code Parameters (Advanced)
| Parameter | Description |
|---|---|
| `pull-print-com` | Print center of mass |
| `pull-print-ref-value` | Print reference value |
| `pull-print-components` | Print vector components |
| `pull-nstxout` | COM output frequency |
| `pull-nstfout` | Force output frequency |
| `pull-pbc-ref-prev-step-com` | PBC reference handling |

## AWH Parameters (Advanced)
| Parameter | Description |
|---|---|
| `awh-nstout` | Output frequency |
| `awh-nstsample` | Sampling frequency |
| `awh-share-group` | Bias sharing group |
| `awh-target` | Target distribution |
| `awh-error-init` | Initial error estimate |
| `awh-growth` | Error growth strategy |
| `awh-equilibrate-histogram` | Equilibration mode |

## QM/MM Parameters
| Parameter | Description |
|---|---|
| `QMMM` | Enable QM/MM |
| `QMMM-grps` | QM atom groups |
| `QMmethod` | QM method |
| `QMbasis` | Basis set |
| `QMcharge` | QM region charge |
| `QMmult` | QM multiplicity |
| `SH` | Surface hopping |

## Key Takeaways
1. Free energy needs soft-core: sc-alpha=0.5, sc-power=1, sc-sigma=0.3
2. Expanded ensemble is convenient for lambda sampling
3. AWH convergence depends critically on diffusion estimate
4. QM/MM requires separate QM software (CP2K or MiMiC)

## Connects To
- **Ch 9**: Basic MDP parameters
- **Ch 21**: Free energy theory
- **Ch 28-30**: Special topics

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 自由能计算(启用) | free-energy = yes | 开启alchemical自由能微扰/积分 |
| 耦合分子类型 | couple-moltype | 隐式定义B态的分子类型 |
| 耦合初始/终态 | couple-lambda0 / couple-lambda1 | q/vdw/vdw-q/none，λ=0和λ=1时的参数 |
| 分子内耦合 | couple-intramol | 是否耦合所选分子类型的分子内相互作用 |
| fep-lambdas | fep-lambdas | λ默认数组[0-1]，其他lambda数组未指定时沿用 |
| coul-lambdas | coul-lambdas | 库仑分量λ向量，可与vdw独立调度 |
| vdw-lambdas | vdw-lambdas | VDW分量λ向量 |
| bonded-lambdas | bonded-lambdas | 成键分量λ向量，控制未标记为限制的成键项 |
| restraint-lambdas | restraint-lambdas | 限制项λ向量 |
| mass-lambdas | mass-lambdas | 质量分量λ向量 |
| 初始λ状态 | init-lambda-state | 显式指定λ状态索引(-1为不指定) |
| dH/dλ输出频率 | nstdhdl | dhdl.xvg输出间隔 |
| λ邻居数 | calc-lambda-neighbors | BAR分析需要的邻居数(通常1) |
| 软核α | sc-alpha | 软核势参数，典型值0.5 |
| 软核幂 | sc-power | 软核势指数，典型值1 |
| 软核σ | sc-sigma | 软核半径(nm)，典型值0.3 |
| 软核库仑 | sc-coul | 是否对库仑使用软核 |
| sc-sigma6 | sc-sigma6 | 分离的C6参数软核σ |
| 扩展系综 | expanded-ensemble | 在λ态之间做Monte Carlo跳转 |
| 扩展系综更新频率 | nstexpanded | MC移动尝试间隔 |
| lmc-stats | lmc-stats | Lambda动力学统计方法(metropolis-gibbs/wang-landau等) |
| lmc-weights-equil | lmc-weights-equil | 权重平衡步数 |
| Wang-Landau增量 | weight-equil-wl-delta | WL算法增量 |
| AWH偏置 | awh = yes | 启用加速权重直方图 |
| AWH输出频率 | awh-nstout | 能量文件输出间隔 |
| AWH采样频率 | awh-nstsample | 采样间隔 |
| 偏置共享组 | awh-share-group | 共享偏置的walker组编号 |
| 目标分布类型 | awh-target | constant/cutoff/boltzmann/local-boltzmann |
| 初始误差 | awh-error-init | 初始偏置不确定性(kJ/mol) |
| 增长策略 | awh-growth | 误差增长方式(exp-linear) |
| 直方图平衡 | awh-equilibrate-histogram | 是否在增长前平衡直方图 |
| 覆盖直径 | cover-diameter | 共享偏置时覆盖判据的空间范围 |
| 扩散常数估计 | diffusion | 反应坐标的扩散系数估计(m²/s) |
| 力常数 | force-constant | AWH简谐耦合力常数 |
| 量子力学/分子力学 | QMMM | 启用QM/MM |
| QM原子组 | QMMM-grps | QM区域原子索引组 |
| QM方法 | QMmethod | RHF/UHF/B3LYP等 |
| QM基组 | QMbasis | STO-3G/6-31G*等 |
| QM电荷/多重度 | QMcharge / QMmult | QM区域的总电荷和自旋多重度 |
| 表面跳跃 | SH | Surface hopping非绝热动力学 |
| 强制旋转 | rotation = yes | 启用强制旋转模块 |
| 旋转组 | rot-group* | 索引组名称 |
| 旋转类型 | rot-type | iso/iso-pf/pm/pm-pf/rm/rm-pf/rm2/rm2-pf/flex/flex-t/flex2/flex2-t |
| 力常数/速率/向量/支点 | k/rate/vec/pivot | 旋转势能参数 |
| 板块间距 | slab-dist | 柔性轴板块间距 |
| 最小高斯截断 | min-gauss | 柔性轴板块贡献截断(0.001) |
| 电场-X/Y/Z | electric-field-x/y/z | 施加电场E0 ω t0 σ |
| 牵引打印COM | pull-print-com | 是否输出COM坐标 |
| 牵引力输出频率 | pull-nstfout | pullf.xvg输出频率 |
| PBC参考处理 | pull-pbc-ref-prev-step-com | 上一步COM作为参考位置 |

**关键概念**:
- Lambda向量独立控制: 典型策略coul-lambdas先于vdw-lambdas，bonded-lambdas与vdw-lambdas同步
- 软核势: 避免λ=0时粒子消失导致的奇异性，sc-alpha=0.5, sc-power=1为常见设置
- 扩展系综: 在λ态间MC跳转，无需模拟所有中间态，Wang-Landau方法自动调整权重
- AWH关键参数: diffusion估计最重要，force-constant应取时间步长支持的最大值
- 强制旋转: 固定轴(各向同性/平行/径向)+柔性轴(板块划分高斯加权)，12种势能变体
- 电场: 4参数格式 E0 ω t0 σ，σ=0为非脉冲，ω=0为静态
- QM/MM: 需外部QM程序(CP2K/MiMiC)，链接原子封端QM/MM边界

Sources: GROMACS 2019.6 中文译版
