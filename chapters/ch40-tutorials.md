# Chapter 40: Tutorials & Resources

## Core Idea
GROMACS provides official tutorials and there are many third-party resources for learning. Background reading in molecular simulation theory is recommended before attempting production simulations.

## Official Tutorials
- **Lysozyme in water**: Standard protein solvation and MD tutorial (most common starting point)
- **Membrane protein**: CHARMM-GUI + GROMACS workflow
- **Free energy**: Alchemical transformation tutorial

## Third-Party Tutorials
- **Justin Lemkul's tutorials**: http://www.mdtutorials.com/gmx/ — comprehensive, widely used
- **GROMACS tutorial on proteins**: Various university course materials
- **CHARMM-GUI**: Web-based system builder that generates GROMACS input files

## Background Reading
- **MD fundamentals**: Allen & Tildesley, "Computer Simulation of Liquids"
- **Biomolecular simulation**: Leach, "Molecular Modelling: Principles and Applications"
- **Free energy methods**: Chipot & Pohorille, "Free Energy Calculations"
- **Force fields**: Ponder & Case, "Force Fields for Protein Simulations" (Adv. Protein Chem., 2003)

## External Tools That Work with GROMACS
| Tool | Purpose |
|---|---|
| **CHARMM-GUI** | Membrane system builder, solution builder |
| **ACPYPE** | AMBER/GAFF to GROMACS parameter conversion |
| **CGenFF** | CHARMM general force field parameters |
| **ATB** | Automated topology builder (GROMOS-compatible) |
| **VMD** | Visualization and analysis with GROMACS plugin |
| **PyMOL** | Molecular visualization |
| **PLUMED** | Enhanced sampling and free energy |
| **gmx_MMPBSA** | MM-PBSA binding free energy |
| **MDTraj/MDAnalysis** | Python trajectory analysis |
| **LOOS** | Lightweight object-oriented structure library |

## Where to Find Help
- **GROMACS website**: https://www.gromacs.org
- **User mailing list**: gmx-users@gromacs.org
- **Developer list**: gmx-developers@gromacs.org
- **GitLab**: https://gitlab.com/gromacs/gromacs
- **Online manual**: https://manual.gromacs.org

## Key Takeaways
1. Start with the lysozyme tutorial for your first simulation
2. Justin Lemkul's tutorials are the best third-party resource
3. CHARMM-GUI simplifies membrane system construction
4. Allen & Tildesley is the essential MD theory reference
5. Use the gmx-users mailing list for community support

## Connects To
- **Ch 4**: Getting started with GROMACS
- **Ch 16**: How-to guides for specific tasks

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 教程资料 | Tutorials | 涵盖使用GROMACS各个方面 |
| 背景阅读资料 | Background Reading | 模拟理论基础知识 |
| 系统准备 | System Preparation | 准备模拟系统的多种方法 |
| 蛋白质-配体 | Protein-Ligand | 原子级别自由能模拟 |
| 粗粒化 | Coarse-Grained (CG) | 简化模型，高密度系统 |
| 拓扑构建工具 | Topology Builder | 自动构建蛋白质/核酸拓扑 |
| 标准氨基酸 | Standard Amino Acids | 20种标准残基 |
| 修饰氨基酸 | Modified Amino Acids | 特殊修饰残基 |
| 核苷酸 | Nucleotides | 4种核苷酸和4种脱氧核苷酸 |
| 脂质 | Lipids | 膜模拟中的关键分子 |
| 碳纳米管 | Carbon Nanotubes (CNT) | Andrea Minoia教程 |
| 柔性水模型 | Flexible Water Model | 能量最小化使用 |
| 位置限制 | Position Restraints | 特定原子约束 |
| 距离限制 | Distance Restraints | NMR距离约束 |
| 溶剂化 | Solvation | 加水或其他溶剂分子 |
| CHARMM-GUI | CHARMM-GUI | 基于网页的系统构建器 |
| ATB | Automated Topology Builder | GROMOS兼容自动拓扑构建 |
| CGenFF | CHARMM General Force Field | 配体参数化服务器 |
| AmberTools / Antechamber | AMBER力场工具 | GAFF通用力场参数 |
| ACPYPE | AnteChamber PYthon Parser interfacE | AMBER到GROMACS参数转换 |
| 周期性边界 | Periodic Boundary Conditions | 模拟体相系统 |
| 可视化 | Visualization | VMD, PyMOL轨迹查看 |
| `editconf` | Edit Configuration | 盒子类型和尺寸定义 |
| `solvate` | Solvate | 填充溶剂分子 |
| `genion` | Add Ions | 中和系统或设盐浓度 |
| `grompp` | GROMACS Preprocessor | 预处理合并所有输入 |
| `mdrun` | MD Run | 主模拟引擎 |
| `trjconv` | Trajectory Conversion | 轨迹转换和PBC处理 |
| `gmx rms` | RMSD Analysis | 结构偏差分析 |
| `gmx rmsf` | RMSF Analysis | 原子涨落分析 |
| `gmx gyrate` | Radius of Gyration | 蛋白质紧凑度分析 |
| `gmx hbond` | Hydrogen Bond Analysis | 氢键数量和动力学 |
| `gmx energy` | Energy Analysis | 能量项提取和平均值 |
| `gmx wham` | WHAM Analysis | 伞形采样PMF重构 |
| `gmx bar` | BAR/MBAR Analysis | 自由能计算 |
| `gmx cluster` | Clustering | 结构聚类分析 |
| 膜蛋白模拟 | Membrane Protein Simulation | 嵌入脂质双层 |
| 伞形采样 | Umbrella Sampling | 沿反应坐标计算PMF |
| 自由能计算教程 | Free Energy Tutorial | 溶剂化自由能、结合自由能 |
| 自选力场 | Custom Force Field | 用户可修改力场参数 |

Sources: GROMACS 2019.6 中文译版 (§3.1.5 教程资料, §3.1.6 背景阅读资料, §3.2 系统准备, §1.1 简介)
