# Gromacs-Everything-Skill

[![GROMACS](https://img.shields.io/badge/GROMACS-2026.2-00A86B?logo=gnometerminal)](https://www.gromacs.org)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-6C47FF)](https://claude.ai/code)
[![License](https://img.shields.io/badge/License-LGPL%202.1%2B-blue)](https://www.gnu.org/licenses/lgpl-2.1.html)

将 **GROMACS Molecular Dynamics Documentation Release 2026.2**（953页）蒸馏为 Claude Code 可用的结构化知识库，内置 10 个核心决策框架、14 个工作流模式、~100 个术语定义。

> **Summary**: A Claude Code skill distilled from 953-page GROMACS Molecular Dynamics Documentation Release 2026.2. Features 10 core decision frameworks, 14 workflow patterns, ~100 term definitions, and bilingual support (Chinese+English).
>
> [Manual](./manual-2026.2.pdf) | [Online Version](https://manual.gromacs.org/2026.2/manual-2026.2.pdf) 
> 
> [Skill](./SKILL.md) | [Cheatsheet](./cheatsheet.md) | [Glossary](./glossary.md) | [Patterns](./patterns.md)

---

## 功能亮点

- **10 个核心决策框架**：从模拟流水线到错误诊断，覆盖 MD 全流程的关键决策点
- **14 个工作流模式**：完整模拟流程、力场选择、热浴/压浴决策、性能优化、自由能计算等
- **~100 个术语定义**：A-Z 全覆盖 GROMACS 关键术语，快速查阅
- **中英双文支持**：README、心智模型、主题索引均为中英对照
- **自动章节检索**：按话题自动定位相关章节，无需手动翻阅
- **错误诊断启发式**：9 种常见错误（LINCS warnings、PME load imbalance 等）的快速诊断与修复

---

## 快速开始

### 安装

```bash
# 方式 1: 使用 npx（推荐）
npx skills add MaybeBio/Gromacs-Everything-Skill -a claude-code

# 方式 2: 手动克隆
git clone https://github.com/MaybeBio/Gromacs-Everything-Skill ~/.claude/skills/Gromacs-Everything-Skill
```

### 使用

在 Claude Code 中，直接用自然语言触发：

```
我的蛋白质体系怎么选力场？
How do I tune PME for GPU-resident mode?
LINCS warnings 怎么修？
帮我设置一个膜蛋白 NPT 模拟的参数
什么是 AWH，什么时候该用它？
```

---

## 核心框架

本技能内置以下 10 个核心决策框架，覆盖 GROMACS 模拟全流程的关键决策点：

| 框架 | 英文 | 核心内容 |
|------|------|----------|
| 1 | 6-phase Simulation Pipeline | 每次模拟的完整路径：体系准备 → EM → NVT → NPT → MD → 分析 |
| 2 | Force Field Selection Heuristic | 蛋白质? 膜? 小分子? 不同场景不同选择（AMBER/CHARMM/GROMOS/OPLS） |
| 3 | Thermostat/Barostat Decision Matrix | 不同阶段用不同的耦合：Berendsen（快速松弛）、V-rescale/Nose-Hoover（生产）、PR barostat |
| 4 | Timestep Selection | 1fs/2fs/4fs/5fs 分别适用什么场景，配合约束算法选择 |
| 5 | PME Tuning Rule | rcoulomb = rvdw = rlist, fourierspacing = 0.12，GPU-resident 模式优化 |
| 6 | Box Selection Principle | 球形溶质用菱形十二面体，节省 29% 溶剂原子 |
| 7 | Performance Optimization Priority | 8 个优化点按优先级排列：GPU 加速、MTS、PME rank 调优等 |
| 8 | Free Energy Calculation Strategy | 5 种方法（TI/FEP/BAR/AWH/Umbrella）的适用场景对比 |
| 9 | Constraint Algorithm Selection | LINCS/SHAKE/SETTLE 何时用哪个，配合时间步长选择 |
| 10 | Error Diagnosis Heuristic | 9 种常见错误（LINCS warnings、非零电荷、PME 负载等）的快速诊断 |

---

## 工作流模式

| 模式 | 适用场景 |
|------|----------|
| **Standard MD Workflow** | 任何从头开始的 MD 模拟（13 步完整流程） |
| **Force Field Selection** | 为新体系选择交互参数 |
| **Box Type Selection** | 定义模拟盒子类型（立方体/菱形十二面体/截角八面体） |
| **Temperature Coupling** | 不同模拟阶段的热浴选择 |
| **Pressure Coupling** | NPT 模拟阶段的压浴选择 |
| **Constraint Algorithms** | 为更大时间步长约束键长 |
| **PME Parameter Tuning** | 设置静电参数 |
| **Performance Optimization** | 模拟速度慢时的优化策略 |
| **Free Energy Calculation** | 计算结合自由能、溶剂化自由能 |
| **Enhanced Sampling Decision Tree** | 选择增强采样方法（Umbrella/AWH/Replica exchange 等） |
| **System Preparation Checklist** | 开始任何新模拟前的检查清单 |
| **Common Error Diagnosis** | 错误症状快速诊断与修复 |

---

## 核心能力

| 领域 | 英文 | 涵盖内容 |
|------|------|----------|
| **模拟流程** | MD Pipeline | pdb2gmx → editconf → solvate → genion → grompp → mdrun (EM → NVT → NPT → MD) |
| **力场** | Force Fields | AMBER99SB-ILDN, CHARMM36, GROMOS54A7, OPLS-AA/L |
| **热浴/压浴** | Thermostats & Barostats | Berendsen, V-rescale, Nose-Hoover, Parrinello-Rahman |
| **约束算法** | Constraints | LINCS, SHAKE, SETTLE |
| **长程静电** | Long-range Electrostatics | PME, Ewald, PPPM, Reaction Field, FMM |
| **自由能** | Free Energy | TI, FEP, BAR, MBAR, soft-core potentials |
| **增强采样** | Enhanced Sampling | Replica exchange, expanded ensemble, AWH, umbrella sampling, metadynamics |
| **GPU 加速** | GPU Acceleration | CUDA, OpenCL, SYCL, HIP, GPU-resident mode |
| **并行化** | Parallelization | Domain decomposition, MPI, OpenMP |
| **文件格式** | File Formats | .gro, .top, .itp, .mdp, .tpr, .xtc, .trr, .edr, .cpt, .ndx, .xvg |
| **高级方法** | Advanced Methods | QM/MM (CP2K, MiMiC), Neural Network Potentials, MTS, HMR |

---

## 相关资源

### 官方 / Official

| 资源 | 英文 | 链接 |
|------|------|------|
| GROMACS 官网 | Homepage | [gromacs.org](https://www.gromacs.org) |
| 在线手册 | Online Manual | [manual.gromacs.org](https://manual.gromacs.org/current/index.html) |
| 用户指南 | User Guide | [manual.gromacs.org/user-guide](https://manual.gromacs.org/current/user-guide/index.html) |
| 官方教程 | Official Tutorials | [tutorials.gromacs.org](https://tutorials.gromacs.org) |
| 源码仓库 | Source Code | [gitlab.com/gromacs/gromacs](https://gitlab.com/gromacs/gromacs) · [github.com/gromacs/gromacs](https://github.com/gromacs/gromacs) |
| 官方论坛 | Forum | [gromacs.bioexcel.eu](https://gromacs.bioexcel.eu) |
| Python API (gmxapi) | Python API | [manual.gromacs.org/gmxapi](https://manual.gromacs.org/current/gmxapi/index.html) |

### 第三方 / Third-Party

| 资源 | 英文 | 链接 |
|------|------|------|
| Justin Lemkul 教程 | MD Tutorials | [mdtutorials.com/gmx](http://www.mdtutorials.com/gmx) |

---

## 学习路线

```
GROMACS 用户指南 -> 第三方教程（Justin Lemkul）模拟 -> 实操: (本skill定位RAG -> 论坛问答 + 官方文档查证) -> Loop
```

> **核心原则 / Core Principle**: 做中学，一切第一性原理的引用论证，都以官方文档为准，本 skill 只是在传统 RAG 学习模式上加速收敛。

---

## 源文档

- **GROMACS Documentation Release 2026.2** by GROMACS development team: [manual.gromacs.org/2026.2/manual-2026.2.pdf](https://manual.gromacs.org/2026.2/manual-2026.2.pdf)
- virgiliojr94/book-to-skill
- 本项目不替代官方文档，而是将其组织为 AI 可直接使用的结构化知识


---

## 文件结构

```
Gromacs-Everything-Skill/
├── SKILL.md              # 技能主配置（Claude Code 入口）
├── README.md             # 本文件
├── patterns.md           # 14 个工作流/决策模式
├── cheatsheet.md         # 速查表（积分器、热浴、命令模式）
├── glossary.md           # ~100 个 GROMACS 术语
├── manual-2026.2.pdf     # 源文档
└── chapters/             # 40 个章节摘要
    ├── ch01-ch07         实用指南（下载、安装、快速入门、体系准备、FAQ）
    ├── ch08-ch16         参考手册（力场、MDP、mdrun、性能、错误）
    ├── ch17-ch27         理论背景（算法、PBC、约束、自由能、相互作用、拓扑）
    ├── ch28-ch34         特殊专题（自由能、AWH、QM/MM、分析）
    └── ch35-ch40         附录（参考文献、开发者、教程）
```

---

## License

本 skill 内容源自 GROMACS 官方文档，遵循 [LGPL 2.1+](https://www.gnu.org/licenses/lgpl-2.1.html) 许可证。

This skill content is derived from GROMACS official documentation, licensed under [LGPL 2.1+](https://www.gnu.org/licenses/lgpl-2.1.html).

--- 

## Contributing

欢迎提交 Issue 和 Pull Request。

如果你在使用中发现章节索引错误、术语定义偏差或决策框架不合理，欢迎在 Issue 中附上复现步骤（官方文档链接 + 具体问题 + 建议修正）。

## To-Do
