# Chapter 38: Release Notes

## Core Idea
GROMACS 2026.2 is a patch release focusing on bug fixes, platform compatibility improvements, and performance enhancements. Release notes are available online at http://manual.gromacs.org/current/release-notes/index.html.

## Key Changes in 2026.2
- Bug fixes for expanded ensemble checkpointing
- Platform support: GCC 12-14 on POWER9, oneAPI 2024.x compatibility
- Performance improvements for GPU-resident mode
- SVE optimization refinements for ARM platforms
- SYCL backend updates for Intel oneAPI
- CMake improvements for Windows and exotic configurations

## Release History
- **2026.2**: Bug fixes, platform compatibility (May 2026)
- **2026.1**: Previous patch release
- **2026**: Major release with new features

## Known Limitations
- See Ch 3 for known issues affecting 2026.2
- Expanded ensemble checkpoint issue
- SVE performance regression with LLVM 20
- NbnxmTest crash with oneAPI 2024.1

## Key Takeaways
1. 2026.2 is a bug-fix release — no major new features
2. Check known issues before upgrading
3. GPU-resident mode has improved performance
4. Online release notes have full changelog

## Connects To
- **Ch 3**: Known issues
- **Ch 15**: Validation status

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 发布说明 | Release Notes | GROMACS各版本变更说明 |
| 版本 | Version | GROMACS 4.6, 5.0, 2019.6, 2026, 2026.2 等 |
| gmx封装二进制文件 | gmx Wrapper Binary | 5.0版引入，格式: gmx <tool> |
| 符号链接 | Symbolic Links | 5.0版向后兼容旧工具名称 |
| 向后兼容 | Backward Compatibility | 默认创建旧工具名称符号链接 |
| Verlet截断方案 | Verlet Cutoff Scheme | 4.6版支持两种截断方案 |
| 组方案 | Group Scheme | 基于粒子组的截断方案，已不推荐 |
| 截断方案 | Cutoff Scheme | Verlet方案 vs 组方案 |
| 电荷组 | Charge Groups | 普通截断静电算法需要 |
| 用户表格势能 | User Table Potential | 组方案中支持，Verlet方案待实现 |
| 内置GPU支持 | Native GPU Support | 4.6版起CUDA GPU加速 |
| 自由体积 | Free Volume | 使用蒙特卡洛采样计算（5.0版） |
| gmx sasa | SASA Surface Area | 5.0版重写并改名 |
| API | Application Programming Interface | FFTW兼容API |
| cmake | CMake | 构建系统 |
| 线程MPI | Thread-MPI | 单节点并行 |
| MPI | Message Passing Interface | 标准并行通信协议 |
| 回归测试 | Regression Tests | 测试自动下载运行 |
| GMX_SIMD | GMX_SIMD | CPU架构SIMD指令集选择 |
| 已知问题 | Known Issues | 影响特定版本的已知问题 |
| 补丁版本 | Patch Release | 修正bug的版本（如2019.6） |
| 重大版本 | Major Release | 包含新功能特性 |
| 自由能计算/FEP | Free Energy Calculation | 自由能微扰/热力学积分 |
| 模块系统 | Module System | 管理不同GROMACS安装 |
| gmx tune_pme | PME Tuning | 优化PME参数 |
| 测试套件 | Test Suite | 包含回归测试和单元测试 |
| 能量漂移 | Energy Drift | 温度控制的重要性 |
| 内存泄漏 | Memory Leak | 工具修复的常见问题 |

Sources: GROMACS 2019.6 中文译版 (§1.1 简介, §1.2 发布说明, §5.0版, gmxapi)
