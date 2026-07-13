# Chapter 1: Downloads

## Core Idea
GROMACS 2026.2 source code and regression tests are distributed via FTP/HTTPS from the official servers. Source compilation is the primary installation method.

## Key Concepts
- **Source tarball**: `gromacs-2026.2.tar.gz` from ftp.gromacs.org (md5: 74c859cb7f53919f9c7760c287b43323)
- **Regression tests**: `regressiontests-2026.2.tar.gz` for verifying build correctness (md5: 074be6e754e7db6e89a1c78347a1d0ac)
- **Other versions**: Available at the GROMACS website

## Key Takeaways
1. Always verify md5sum after download to ensure file integrity
2. Regression tests should be downloaded separately and used to validate your build
3. Source compilation is the standard way to get GROMACS — binary packages are not officially distributed

## Connects To
- **Ch 2**: Installation guide — what to do after downloading

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 分子动力学模拟 | molecular dynamics simulation | MD |
| 源代码 | source code | 压缩包格式 .tar.gz |
| 回归测试 | regression tests | 用于验证构建正确性 |
| 校验码/校验和 | md5sum / checksum | 验证文件完整性 |
| 牛顿运动方程 | Newton's equations of motion | 理论基础 |
| 自由软件 | free software | LGPL 许可证 |
| 非键相互作用 | non-bonded interactions | 模拟中最耗时的计算 |
| SIMD | Single Instruction Multiple Data | 指令级并行 |
| GPU加速 | GPU acceleration | 基于 CUDA |
| C预处理器 | C preprocessor (cpp) | 拓扑文件的 #include 机制 |
| 字节顺序 | endianness | 文件与硬件无关 |
| 有损压缩 | lossy compression | 轨迹坐标压缩存储 |
| 轨迹查看器 | trajectory viewer | 内置 gmx view |
| 线程MPI | thread-MPI | 单节点并行 |

**关键概念**: GROMACS 是一个通用的分子动力学模拟软件包，可用于计算具有数百到数百万个粒子的体系的牛顿运动方程。它主要用于生化分子（蛋白质、脂质、核酸），但由于其非键计算速度极快，也被广泛用于聚合物等非生物体系。GROMACS 代码中的算法经过高度优化，从 4.6 版本开始内置 GPU 加速功能。源代码和回归测试从官方网站下载后，应验证 md5sum 确保文件完整性。

Sources: GROMACS 2019.6 中文译版 (§1.1-1.3)
