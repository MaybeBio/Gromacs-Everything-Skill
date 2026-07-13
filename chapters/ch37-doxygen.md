# Chapter 37: Doxygen Documentation

## Core Idea
GROMACS provides Doxygen-generated API documentation for developers. The Doxygen docs cover the internal C++ class hierarchy, module structure, and API reference for all GROMACS components.

## Key Components Documented
- **MD modules**: Integrator, force calculation, constraints, coupling
- **NB (non-bonded)**: Neighbor search, pair interactions, PME
- **GPU backends**: CUDA, OpenCL, SYCL, HIP kernel interfaces
- **Analysis tools**: Trajectory analysis framework
- **File I/O**: Format readers/writers (.tpr, .xtc, .edr, etc.)
- **Utility libraries**: SIMD abstraction, random numbers, selection engine

## Accessing Doxygen Docs
- Online: https://manual.gromacs.org/doxygen/
- Build locally: `cmake -DGMX_BUILD_HELP=ON && make doxygen`
- Module overview: mainpage.h in source tree

## Key Takeaways
1. Doxygen docs are the primary API reference for GROMACS developers
2. Available online at manual.gromacs.org/doxygen
3. Covers all internal modules with class hierarchy and dependencies

## Connects To
- **Ch 36**: Developer guide and contribution workflow

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| Doxygen文档 | Doxygen Documentation | API参考，包含类层次和依赖关系 |
| API文档 | API Documentation | GROMACS内部C++类层次和模块结构 |
| 模块结构 | Module Structure | MD模块、非键模块、GPU后端等 |
| 类层次 | Class Hierarchy | C++类的继承和依赖关系 |
| MD模块 | MD Modules | 积分器、力计算、约束、耦合 |
| 非键相互作用 | Non-Bonded (NB) Interactions | 邻居搜索、配对相互作用、PME |
| GPU后端 | GPU Backend | CUDA/OpenCL/SYCL/HIP内核接口 |
| 分析框架 | Analysis Framework | 轨迹分析框架 |
| 文件I/O | File I/O | TPR/TRR/EDR等格式读写 |
| 工具库 | Utility Libraries | SIMD抽象、随机数、选择引擎 |
| 构建文档 | Building Documentation | cmake -DGMX_BUILD_HELP=ON |
| mainpage.h | mainpage.h | 源树中的模块概览文件 |
| Kmix精度 | Mixed Precision / Double Precision | 默认混合精度，cmake -DGMX_DOUBLE=on |
| 手动文档 | User Manual | GROMACS用户手册在线版和PDF |
| Sphinx | Sphinx | 文档构建工具 |
| gmxapi-cppdocs | gmxapi-cppdocs | 构建目标，在docs/api-user中生成 |
| 安装指南 | Installation Guide | GROMACS安装文档 |
| 参考手册 | Reference Manual | 手册第5章，实现细节 |
| 用户指南 | User Guide | 手册第3章包含快速设置和简短解释 |

Sources: GROMACS 2019.6 中文译版 (§5 参考手册, gmxapi 外部API, §2 安装指南)
