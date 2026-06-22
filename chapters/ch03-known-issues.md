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
