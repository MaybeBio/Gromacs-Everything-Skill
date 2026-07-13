# Chapter 3: Known Issues

## Core Idea
GROMACS 2026.2 has several known platform-specific issues that users should check before compiling and running. These are being actively worked on for future releases.

## Known Issues

### "Cannot find a working standard library" with ROCm Clang
- **Symptom**: CMake fails to find standard library when using ROCm-bundled Clang
- **Workaround**: Use system Clang or GCC instead of ROCm-bundled Clang
- **Status**: Under investigation

### Expanded Ensemble Does Not Checkpoint Correctly
- **Symptom**: Checkpoint files from expanded ensemble simulations may not contain complete state information
- **Impact**: Restarts from checkpoint may not be exact
- **Workaround**: Avoid relying on checkpoint restarts for expanded ensemble simulations
- **Status**: Fix pending

### GCC 12-14 on POWER9 Architectures
- **Symptom**: Compilation issues or incorrect code generation with GCC 12-14 on POWER9
- **Workaround**: Use GCC 11 or test thoroughly with specific GCC version
- **Status**: Known compatibility issue

### NbnxmTest Crash with oneAPI 2024.1
- **Symptom**: The NbnxmTest (non-bonded kernel test) crashes when built with Intel oneAPI 2024.1
- **Impact**: Test suite failure, potential runtime issues with non-bonded calculations on SYCL backend
- **Workaround**: Use oneAPI 2024.0 or a newer version when available
- **Status**: Under investigation

### Severe Performance Regression with SVE and LLVM 20
- **Symptom**: Significant performance degradation on ARM SVE-enabled processors when compiled with LLVM 20
- **Impact**: Reduced simulation throughput on ARM-based HPC systems (e.g., NVIDIA Grace)
- **Workaround**: Use GCC instead of LLVM/Clang on SVE platforms
- **Status**: Performance investigation ongoing

### CMake OpenMP_CUDA Detection Issue on Windows
- **Symptom**: CMake fails to detect OpenMP_CUDA correctly on Windows
- **Impact**: May prevent GPU-accelerated builds on Windows
- **Workaround**: Manually specify OpenMP_CUDA paths or use WSL2 instead
- **Status**: Known issue with CMake's FindOpenMP on Windows

## Key Takeaways
1. Always check known issues before compiling — your platform may be affected
2. GCC is generally the safest compiler choice across all platforms
3. GPU builds (CUDA/HIP/SYCL) require extra attention to compiler and driver versions
4. Expanded ensemble users should be cautious about checkpoint restarts

## Connects To
- **Ch 2**: Installation guide — platform-specific compilation instructions
- **Ch 11**: Performance tuning — platform-specific optimization

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 已知问题 | known issues | 特定平台/编译器 |
| 兼容性 | compatibility | 编译器/硬件组合 |
| 致命错误 | fatal error | 编译/运行时的严重问题 |
| 警告信息 | warning message | 可接受或需关注 |
| 测试过的平台 | tested platforms | 官方验证的配置 |
| 基准测试 | benchmark | 性能参考数据 |
| 交叉编译 | cross-compilation | 不同架构间编译 |
| 平台特别说明 | platform-specific notes | 各架构的特殊要求 |
| 构建失败 | build failure | 编译过程中出错 |
| 性能回归 | performance regression | 性能显著下降 |
| 检查点 | checkpoint | 扩展系综的检查点问题 |
| 标准库 | standard library | libstdc++ 兼容性 |
| SIMD 不匹配 | SIMD mismatch | CPU 指令集不兼容 |

**关键概念**: GROMACS 在不同平台上可能存在特定的兼容性问题。常见的已知问题类型包括：编译器版本不兼容（如 GCC 12-14 在 POWER9 上）、GPU 驱动/工具包版本冲突、SIMD 指令集不匹配导致的性能退化。用户应在构建前检查已知问题列表，使用推荐的最新编译器版本（gcc 为最佳选择）。如遇到问题，首先检查 CMake 缓存中的检测结果，确认编译器、FFT 库和 GPU 后端的配置是否正确。对于 ARM/SVE 等新兴架构，编译器和数学库的版本选择尤为关键。

Sources: GROMACS 2019.6 中文译版 (§2.4-2.6)
