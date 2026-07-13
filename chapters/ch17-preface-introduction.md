# Chapter 17: Preface & Introduction

## Core Idea
GROMACS is free software (LGPL) for molecular dynamics simulation of biomolecular systems. It is one of the fastest MD engines available, with extensive GPU acceleration and a comprehensive analysis toolkit.

## Citation
When publishing results obtained with GROMACS, cite the relevant reference papers listed in the manual. The primary citation depends on the features used (e.g., GPU acceleration, free energy methods).

## Free Software
GROMACS is distributed under the GNU Lesser General Public License (LGPL). This means:
- Free to use, modify, and distribute
- Can be linked with proprietary code
- Modifications to GROMACS itself must be shared

## Computational Chemistry Background
- **Molecular Modeling**: Representing molecules with force fields (potential energy functions)
- **MD Simulation**: Numerical integration of Newton's equations of motion
- **Energy Minimization**: Finding local minima on the potential energy surface
- **Ergodicity**: Long-enough MD samples the equilibrium ensemble

## Key Takeaways
1. Always cite GROMACS in publications — specific papers for specific features
2. GROMACS is LGPL — free for academic and commercial use
3. MD simulations sample the equilibrium ensemble if run long enough

## Connects To
- **Ch 18**: Definitions and units
- **Ch 19**: MD algorithm details

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 分子动力学模拟 | Molecular Dynamics (MD) simulation | 求解牛顿运动方程，生成相空间轨迹 |
| 计算化学 | Computational chemistry | 从量子力学到复杂大分子聚集体的动力学 |
| 分子建模 | Molecular modeling | 用近真实原子模型描述复杂化学系统 |
| 力场/势能函数 | Force field / Potential energy function | 成对累加，不含极化，V = 非键项 + 键合项 + 约束项 |
| 能量最小化 | Energy minimization | 移除动能，降低热噪声，找最近局部极小点 |
| 蒙特卡洛模拟 | Monte Carlo (MC) simulation | 另一种生成平衡系综的方法，不需计算力 |
| 牛顿运动方程 | Newton's equations of motion | m_i·d²r_i/dt² = F_i = −∂V/∂r_i |
| 各态历经 | Ergodicity | 长时间MD采样平衡系综的条件 |
| 系综 | Ensemble | NVE(微正则)、NVT(正则)、NPT(等温等压) |
| 统计力学 | Statistical mechanics | 为简化提供理论框架，层次从原子到介观 |
| 静态平衡性质 | Static equilibrium properties | 结合常数、平均势能、径向分布函数 |
| 动态/非平衡性质 | Dynamic/non-equilibrium properties | 粘度、扩散过程、相变动力学 |
| 量子力学/经典力学 | Quantum/Classical mechanics | MD使用经典力学，氢原子/低温量子效应显著 |
| 隧穿效应 | Tunneling effect | 质子在氢键中量子隧穿，经典MD无法处理 |
| 玻恩-奥本海默近似 | Born-Oppenheimer approximation | 电子始终处于基态，瞬间调整 |
| 超曲面 | Hyper-surface | 势能函数在多维空间的复杂形貌 |
| 全局极小值/局部极小值 | Global/local minimum | 无法保证找到全局极小点 |
| 鞍点 | Saddle point | Hessian矩阵仅有一个负特征值 |
| 模拟退火 | Simulated annealing | 高温→慢冷，加速构型搜索 |
| 软核势 | Soft-core potential | 替换排斥势以降低势垒 |
| 约束 | Constraint | 将键视为约束比经典谐振子更接近量子基态 |
| GNU Lesser GPL | LGPL | 自由使用/修改/分发，可与专有代码链接 |

**关键概念**:
1. **势能函数统一框架**: V(r₁,...,r_N) = 非键项(LJ+Coulomb) + 键合项(键伸缩+键角+二面角) + 约束项(位置/角度/距离/方向/二面角约束)
2. **经典MD的五个近似**: (a) 使用经典力学(高频振动需校正); (b) 电子始终处于基态(BO近似); (c) 力场是近似的; (d) 力场是成对累加的(忽略极化); (e) 长程相互作用被截断
3. **量子校正公式**: U^QM = U^cl + kT(x/2 − 1 + x/(e^x−1)), 其中 x = hν/kT。室温下高于 100 cm⁻¹ 的频率表现可能不正常
4. **周期性边界条件**消除表面效应，但引入了周期性假象；对非周期性体系，周期边界假象远小于真空边界假象
5. **能量最小化方法三类**: 只需函数值(单纯形)、需导数(最陡下降/CG)、需二阶导数(Hessian矩阵) — GROMACS使用第二类

**重要公式**:
- 牛顿方程: m_i·d²r_i/dt² = −∂V/∂r_i = F_i
- 量子能量校正: U^QM = U^cl + kT(½x − 1 + x/(e^x−1))
- 退火策略: 高温模拟→缓慢冷却→重复; 可增大氢质量(至10倍)以增大时间步长

Sources: GROMACS 2019.6 中文译版, §5.1-5.2
