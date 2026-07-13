# Chapter 36: Developer Guide

## Core Idea
The GROMACS developer guide covers coding standards, contribution workflow, and testing for developers contributing to the GROMACS source code. GROMACS is written in C++17 with extensive use of SIMD intrinsics and GPU programming.

## Key Concepts

### Contribution Workflow
1. Discuss changes on the GROMACS developer mailing list (gmx-developers)
2. Fork the GROMACS GitLab repository
3. Create a feature branch
4. Follow GROMACS coding standards (see Doxygen docs)
5. Submit merge request on GitLab
6. Pass CI (continuous integration) pipeline
7. Code review by core developers

### Coding Standards
- C++17 standard
- clang-format for code formatting
- Extensive use of templates and SIMD abstractions
- GPU code: CUDA, OpenCL, SYCL, HIP backends
- Comprehensive Doxygen documentation

### Testing
- Regression tests: `make check` (requires regressiontests download)
- Unit tests: Google Test framework
- Performance tests: `gmx nonbonded-benchmark`
- CI pipeline: automated testing on multiple platforms

### Build System
- CMake-based with extensive option system
- Separate build for each configuration (CPU, CUDA, HIP, etc.)

## Key Takeaways
1. GROMACS is C++17 with extensive SIMD/GPU abstractions
2. Contributions go through GitLab merge requests
3. All changes require passing CI and code review
4. Regression tests validate correctness across platforms

## Connects To
- **Ch 2**: Build system details
- **Ch 37**: Doxygen API documentation

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 开发者指南 | Developer Guide | GROMACS代码开发模式说明 |
| 贡献 | Contribution | 帮助GROMACS发展 |
| 编码标准 | Coding Standards | C++17标准、clang-format |
| 合并请求 | Merge Request | 通过GitLab提交更改 |
| 代码审查 | Code Review | 由核心开发者审查更改 |
| CI/CD | Continuous Integration / Deployment | 自动测试多个平台 |
| 回归测试 | Regression Tests | make check 验证正确性 |
| 单元测试 | Unit Tests | Google Test框架 |
| 性能测试 | Performance Tests | gmx nonbonded-benchmark |
| 构建系统 | Build System | 基于CMake的选项系统 |
| SIMD内部函数 | SIMD Intrinsics | C内部函数编译为SIMD指令 |
| GPU加速 | GPU Acceleration | CUDA/OpenCL/SYCL/HIP后端 |
| 区域分解 | Domain Decomposition | 空间并行化，单元间通信 |
| 交叉编译 | Cross-Compiling | 为不同架构构建 |
| Gerrit | Gerrit | GROMACS代码审查系统 |
| Redmine | Redmine | 问题跟踪系统 |
| 贡献者 | Contributors | 代码贡献者名单（第5章前言） |
| LGPL | GNU Lesser General Public License | GROMACS使用的开放源码许可 |
| 文件读取自动解压 | Automatic Gzip Decompression | 读取压缩文件时自动解压 |
| 字节顺序 | Endianness | 运行输入文件和轨迹文件独立于硬件 |
| 有损压缩 | Lossy Compression | 轨迹坐标压缩输出 |
| x86/x86_64 | x86/x86_64 | SIMD支持的主流处理器架构 |
| AVX_512_KNL | AVX_512_KNL | Xeon Phi Knights Landing专用指令集 |
| 集群模式 | Cluster Mode | KNL处理器Quadrant/SNC模式 |
| MCDRAM | MCDRAM | Knights Landing高带宽堆栈内存 |
| gmxapi | gmxapi | 外部API，动态链接库和头文件 |
| 模块系统 | Module System | 管理不同架构GROMACS安装版本 |
| 线程MPI | Thread-MPI | 单节点工作站并行 |
| 跨节点并行 | Cross-Node Parallel | 使用MPI通信协议的多节点并行 |

Sources: GROMACS 2019.6 中文译版 (§5.1 前言, §5.1.2 自由软件, §2 安装指南, gmxapi 外部API)
