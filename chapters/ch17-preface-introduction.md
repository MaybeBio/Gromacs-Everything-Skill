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

| 中文 | English |
|------|---------|
| 分子动力学模拟 | Molecular Dynamics (MD) simulation |
| 计算化学 | Computational chemistry |
| 分子建模 | Molecular modeling |
| 力场/势能函数 | Force field / Potential energy function |
| 能量最小化 | Energy minimization |
| 牛顿运动方程 | Newton's equations of motion |
| 各态历经 | Ergodicity |
| 系综 | Ensemble |
| 统计力学 | Statistical mechanics |

GROMACS中文手册由李继存组织翻译 (version 5.0.2), 理论基础来自该手册。手册指出: 势能函数 = 非键项 + 键合项 + 约束项, 这是所有力场的统一框架。在统计力学中, 足够长时间的MD模拟会采样平衡系综 (各态历经假设)。

Sources: GROMACS 5.0.2 中文手册 (李继存译), CC-BY compatible.
