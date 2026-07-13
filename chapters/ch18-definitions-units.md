# Chapter 18: Definitions & Units

## Core Idea
GROMACS uses a consistent unit system (nm, ps, kJ/mol, K, e, bar) that simplifies MD calculations. Understanding the notation conventions and precision considerations is essential for correct parameter specification.

## MD Units
| Quantity | Unit | Value |
|---|---|---|
| Length | nm | 1 nm = 10 Å |
| Time | ps | 1 ps = 10⁻¹² s |
| Mass | amu (u) | 1.66054×10⁻²⁷ kg |
| Energy | kJ/mol | 1 kJ/mol ≈ 0.239 kcal/mol |
| Temperature | K | Boltzmann constant implicit |
| Charge | e | 1.60218×10⁻¹⁹ C |
| Pressure | bar | 1 bar = 10⁵ Pa |
| Force | kJ/mol/nm | 1 kJ/mol/nm ≈ 16.02 pN |
| Velocity | nm/ps | 1 nm/ps = 1000 m/s |

## Key Constants
- **Electrostatic constant f**: 1/(4πε₀) = 138.935458 kJ mol⁻¹ nm e⁻²
- **Boltzmann constant k_B**: Implicitly included via temperature
- **Gas constant R**: 8.31451×10⁻³ kJ mol⁻¹ K⁻¹

## Notation
- **Bold**: Vector quantities (r, v, F)
- **Italic**: Scalar quantities (r, V, T)
- **Subscript i,j**: Particle indices
- **r_ij**: Vector from particle j to particle i
- **r_ij**: Distance between particles i and j

## Precision
- **Single precision** (default): 7 significant digits, sufficient for MD
- **Double precision**: 15 significant digits, required for normal mode analysis, energy minimization to very high accuracy
- **Mixed precision**: GROMACS internally uses mixed precision where beneficial

## Reduced Units
Used in some specialized contexts where quantities are expressed in dimensionless form using Lennard-Jones parameters σ and ε as base units.

## Key Takeaways
1. Always use consistent units: nm, ps, kJ/mol, K, e, bar
2. The electrostatic constant f = 138.935458 is used internally
3. Single precision is sufficient for 99% of MD simulations
4. Use double precision only for normal mode analysis

## Connects To
- **Ch 9**: MDP parameters (use these units)
- **Ch 23**: Non-bonded interaction formulas

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 矢量/粗斜体 | Vector (bold italic) | 如 r_i，位置向量 |
| 向量长度/斜体 | Scalar (italic) | 如 r_i，标量距离 |
| 位置矢量 | Position vector | r_i，粒子 i 的位置 |
| 粒子间矢量 | Inter-particle vector | r_ij = r_j − r_i |
| 粒子间距离 | Inter-particle distance | r_ij = |r_ij| |
| 力矢量 | Force vector | F_i，作用于粒子 i 的力 |
| 长度单位 | Length | nm (10⁻⁹ m)，基本MD单位 |
| 质量单位 | Mass | u (原子质量单位) = 1.66054×10⁻²⁷ kg |
| 时间单位 | Time | ps (10⁻¹² s)，基本MD单位 |
| 电荷单位 | Charge | e (基元电荷) = 1.60218×10⁻¹⁹ C |
| 温度单位 | Temperature | K，基本MD单位 |
| 能量单位 | Energy | kJ/mol，导出单位 |
| 力单位 | Force | kJ mol⁻¹ nm⁻¹，导出单位 |
| 压力单位 | Pressure | bar，导出单位 |
| 速度单位 | Velocity | nm/ps = 1000 m/s |
| 偶极矩单位 | Dipole moment | e·nm |
| 电势单位 | Electric potential | kJ mol⁻¹ e⁻¹ ≈ 0.010364 V |
| 电场单位 | Electric field | kJ mol⁻¹ nm⁻¹ e⁻¹ ≈ 1.037×10⁷ V/m |
| 静电转换因子 | Electrostatic constant f | f = 1/(4πε₀) = 138.935458 kJ mol⁻¹ nm e⁻² |
| Boltzmann/气体常数 | k_B and R (same value) | 0.0083144621 kJ mol⁻¹ K⁻¹ (MD单位中 k_B = R) |
| 普朗克常数 | Planck constant h | 0.399031271 kJ mol⁻¹ ps |
| 狄拉克常数 | Dirac constant ℏ | 0.0635077993 kJ mol⁻¹ ps |
| 光速 | Speed of light c | 299792.458 nm/ps |
| 阿伏加德罗常数 | Avogadro's number N_AV | 6.02214129×10²³ mol⁻¹ |
| 约化单位 | Reduced units | ε_ii = σ_ii = m_i = k_B = 1, 用于LJ系统 |
| 混合精度 | Mixed precision | 关键变量双精度，状态向量单精度，默认配置 |
| 双精度 | Double precision | 比混合精度慢20%-100%，需cmake -DGMX_DOUBLE=on |

**关键概念**:
1. **MD单位系统自洽**: 基本单位为 nm, ps, e, u, K。所有导出单位由此计算。不可随意修改单位，否则会导致不自洽。例如用Å代替nm则时间单位变为0.1 ps；用kcal/mol代替kJ/mol则时间单位变为0.488882 ps
2. **k_B = R**: 在MD单位中，Boltzmann常数和理想气体常数无区别，均为 0.0083144621 kJ mol⁻¹ K⁻¹。这是以摩尔为单位计算的结果
3. **约化温度特例**: 约化单位的温度是实际k_B T的 0.008314...倍。GROMACS温度 T=1 对应约化温度约0.008；约化温度=1时GROMACS温度应为120.27236
4. **混合精度足够**: 能量精确到最后一位，力的最后一位或两位不重要。维里精度比力差。MD混沌性导致混合精度轨迹比双精度发散更快
5. **双精度适用场景**: 简正模式分析、Hessian矩阵计算/对角化、长时间能量守恒(大系统)

**重要公式**:
- 静电转换: V = f·q²/r, F = f·q²/r², 其中 f = 1/(4πε₀) = 138.935458
- 电势/电场: Φ(r) = f·Σ_j q_j/|r−r_j|, E(r) = f·Σ_j q_j·(r−r_j)/|r−r_j|³
- LJ势(约化单位): V_LJ = 4ε[(σ/r)¹² − (σ/r)⁶]

Sources: GROMACS 2019.6 中文译版, §5.3
