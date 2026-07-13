# Chapter 34: Averages & Fluctuations

## Core Idea
Thermodynamic properties from MD are ensemble averages with statistical errors. GROMACS implements block averaging and fluctuation formulas for computing properties like heat capacity, compressibility, and thermal expansion from simulation trajectories.

## Formulae for Averaging
- **Mean**: ⟨A⟩ = (1/N) Σ A_i
- **Variance**: σ² = ⟨A²⟩ − ⟨A⟩²
- **Standard error**: SEM = σ/√N_eff (corrected for correlation)
- **Block averaging**: Divide trajectory into blocks, compute mean of block means, standard deviation of block means = standard error

## Key Fluctuation Formulas

### Heat Capacity (NVT)
C_V = (⟨E²⟩ − ⟨E⟩²) / (k_B T²)

### Isothermal Compressibility (NPT)
κ_T = (⟨V²⟩ − ⟨V⟩²) / (k_B T ⟨V⟩)

### Thermal Expansion Coefficient (NPT)
α = (⟨V·H⟩ − ⟨V⟩·⟨H⟩) / (k_B T² ⟨V⟩)

## Implementation in GROMACS
- Energy averages accumulated in blocks
- `gmx energy -f ener.edr -fluct_props` computes fluctuation properties
- Block size affects error estimates — check convergence with block size

## Key Takeaways
1. Block averaging is the standard method for error estimation
2. Raw standard deviation underestimates error (correlated data)
3. Fluctuation properties converge slowly — need long trajectories
4. Heat capacity from NVT, compressibility from NPT

## Connects To
- **Ch 32**: Analysis tools
- **Ch 33**: Implementation details

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 系综平均值 | Ensemble Average | 宏观性质为统计系综的系综平均值 |
| 经典极限 | Classical Limit | 室温下k_B T/h ≈ 200 cm^{-1}，高于此频率的运动表现可能不正常 |
| 块平均 | Block Averaging | 将数据集划分为多块，计算块平均值之间的方差来估计误差 |
| 平均值 | Mean / Average | ⟨A⟩ = (1/N) Σ A_i |
| 方差 | Variance | σ² = ⟨A²⟩ - ⟨A⟩² |
| 标准误差 | Standard Error | SEM = σ/√N_eff，考虑时间相关性 |
| 自相关函数 | Autocorrelation Function | 评定数据点之间时间关联性 |
| 量子校正 | Quantum Correction | 对经典谐振子能量和比热的校正（公式5.3, 5.4） |
| 等容热容量 C_V | Heat Capacity at Constant Volume | (⟨E²⟩-⟨E⟩²)/(k_B T²)，NVT模拟 |
| 等压热容量 C_p | Heat Capacity at Constant Pressure | NPT模拟，需要Enthalpy和Temp |
| 热膨胀系数 | Thermal Expansion Coefficient (alpha) | (⟨V·H⟩-⟨V⟩·⟨H⟩)/(k_B T²⟨V⟩)，NPT模拟 |
| 等温压缩系数 | Isothermal Compressibility (kappa_T) | (⟨V²⟩-⟨V⟩²)/(k_B T⟨V⟩)，NPT模拟 |
| 绝热体积模量 | Adiabatic Bulk Modulus | 需要Vol和Temp |
| 涨落性质 | Fluctuation Properties | 能量波动计算的热力学性质 |
| 漂移 | Drift | 拟合最小二乘直线得到，第一点和最后一点的差值 |
| 量子谐振子 | Quantum Harmonic Oscillator | 处于零点能级½hν的基态，经典谐振子吸收过多能量 |
| 约束 | Constraints | 将键视为约束而非谐振子，允许更大的时间步长 |
| 构型空间采样 | Configurational Space Sampling | MD模拟产生具有代表性的平衡系综 |
| HMR | Hydrogen Mass Repartitioning | 增加氢原子质量，增大时间步长 |
| 维里定理 | Virial Theorem | 维里波动可能比平均值大两个数量级 |
| 最小二乘拟合 | Least-Squares Fit | 用于能量漂移分析 |
| 累积量 | Cumulant | 第三和第四累积量的相对偏差 |
| `gmx energy -fluct_props` | Fluctuation Properties Analysis | 计算基于能量波动的热力学性质 |
| 块平均解析曲线 | Analytical Block Averaging Curve | Hess (2002)推导，用于误差估计 |
| 正则系综 | Canonical Ensemble | NVT系综，采样产生正确分布 |
| 能量守恒 | Energy Conservation | 长时间能量守恒需双精度 |

Sources: GROMACS 2019.6 中文译版 (§5.12 平均值与波动, §5.2.2 分子动力学模拟)
