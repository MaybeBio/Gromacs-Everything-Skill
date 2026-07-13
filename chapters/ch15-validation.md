# Chapter 15: Validation

## Core Idea
GROMACS features are categorized by validation status: validated, experimental, or pending validation. Understanding these categories helps assess simulation reliability.

## Key Concepts
- **Validated features**: Tested extensively, suitable for production use
- **Experimental features**: Implemented but not fully validated — use with caution
- **Features with validation pending**: New features awaiting systematic testing

## Experimental Features (2026.2)
- Expanded ensemble checkpointing (known issue)
- MiMiC QM/MM coupling
- Neural Network Potentials (NNP)
- Fast Multipole Method (FMM)
- Multiple Time Stepping (MTS)

## Features with Validation Pending
- GPU FFT with vkFFT for some backends
- SYCL backend on some platforms
- New NNP architectures

## Key Takeaways
1. Check validation status before using features in production
2. Experimental features may have known issues — read release notes
3. Report validation results to help the community

## Connects To
- **Ch 3**: Known issues
- **Ch 38**: Release notes

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 验证 | validation | 功能可靠性评估 |
| 已验证功能 | validated features | 适合生产使用 |
| 实验性功能 | experimental features | 需谨慎使用 |
| 待验证功能 | validation pending | 新功能等待测试 |
| 兼容性 | compatibility | 不同方案的支持 |
| 双范围 | twin-range | 组方案不支持 |
| Buckingham 势 | Buckingham potential | Verlet 方案不支持 |
| 用户表格 | user tables | 自定义相互作用表 |
| 能量组排除 | energy group exclusions | Verlet 方案不支持 |
| 简正模式 | normal modes | 振动分析 |
| NNP 神经网络势 | Neural Network Potentials | 实验性功能 |
| FMM 快速多极子 | Fast Multipole Method | 实验性功能 |
| MiMiC QM/MM | MiMiC hybrid QM/MM | 实验性功能 |
| 杨-米尔斯耦合 | vkFFT GPU FFT | 部分后端待验证 |

**关键概念**: GROMACS 功能按照验证成熟度分级管理。已验证功能经过了广泛测试，适合生产级模拟使用。实验性功能已实现但尚未完全验证，使用时应谨慎并参考已知问题和发布说明。中文手册详细记录了组方案和 Verlet 方案在不同功能上的支持差异。随着 GROMACS 发展，一些旧功能（如组截断方案、Buckingham VdW 相互作用）正在被淘汰，而新功能（如神经网络势、FMM）处于实验阶段。用户应关注发布说明以了解最新功能的验证状态。

Sources: GROMACS 2019.6 中文译版 (§3.6, 发布说明)
