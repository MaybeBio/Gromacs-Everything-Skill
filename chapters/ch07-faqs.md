# Chapter 7: Frequently Asked Questions

## Core Idea
Common GROMACS questions organized by topic: installation, system preparation, simulation methodology, force fields, and analysis. This is the first place to look when encountering a problem.

## Installation FAQs
- **Q: FFTW not found?** Install FFTW3 development package or build with `-DGMX_BUILD_OWN_FFTW=ON`
- **Q: GPU not detected?** Check CUDA toolkit/driver version; verify `-DGMX_GPU=CUDA` was set; run `nvidia-smi`
- **Q: MPI vs thread-MPI?** Thread-MPI is built-in and sufficient for single node; real MPI needed for multi-node
- **Q: How to check if build is correct?** Run `make check` with regression tests downloaded

## System Preparation FAQs
- **Q: Which force field?** AMBER99SB-ILDN for proteins, CHARMM36 for protein-lipid, GROMOS54A7 for united-atom simplicity
- **Q: Which water model?** Must match force field. TIP3P for AMBER/CHARMM, SPC for GROMOS
- **Q: How many ions?** Neutralize first, then add physiological concentration (~0.15 M)
- **Q: PDB has missing residues?** Model them with external tools (Modeller, SWISS-MODEL) before pdb2gmx

## Simulation Methodology FAQs
- **Q: dt=1 fs or 2 fs?** 2 fs with `constraints=h-bonds` is standard. 1 fs only if you need flexible water.
- **Q: EM before MD?** Always. Steepest descent until Fmax < 1000 kJ/mol/nm, then optionally CG/L-BFGS.
- **Q: How long should equilibration be?** 100 ps NVT + 100-500 ps NPT, until density and temperature stabilize.
- **Q: Nose-Hoover or V-rescale?** V-rescale for equilibration, either for production. V-rescale is more robust.

## Force Field FAQs
- **Q: Can I mix force fields?** Generally no. Use consistent force field for all components.
- **Q: Ligand parameters?** Use tools like ACPYPE (Antechamber), CGenFF, or GAFF for small molecules.
- **Q: Non-standard residues?** Use `gmx pdb2gmx -ter` for termini; create custom .rtp entries for modified residues.

## Analysis FAQs
- **Q: RMSD calculation reference?** Use the starting (energy-minimized) structure or crystal structure.
- **Q: How to center trajectory?** `gmx trjconv -center -pbc mol -ur compact`
- **Q: Hydrogen bonds?** Use `gmx hbond` with donor-acceptor cutoff 0.35 nm and angle cutoff 30 degrees.
- **Q: Cluster analysis?** `gmx cluster -method gromos -cutoff 0.2` (0.2 nm for similar conformations).

## Key Takeaways
1. Force field + water model are inseparable — match them
2. Always EM before MD
3. 2 fs timestep with constraints=h-bonds is the production standard
4. V-rescale is safer than Nose-Hoover for equilibration
5. Check the .log file after every mdrun for warnings

## Connects To
- **Ch 2**: Installation details
- **Ch 5**: System preparation
- **Ch 9**: MDP parameters
- **Ch 12**: Common errors

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 常见问题 | Frequently Asked Questions (FAQ) | 按主题分类 |
| 安装问题 | installation questions | 编译器/MPI/精度 |
| 系统准备 | system preparation | 溶剂/拓扑/文件格式 |
| 模拟方法 | simulation methodology | 耦合/约束/重启 |
| 参数化 | parameterization | 力场/过渡金属 |
| 分析与可视化 | analysis & visualization | 轨迹显示/PBC |
| 混合精度 | mixed precision | 默认推荐使用 |
| 单点能 | single-point energy | -rerun 选项计算 |
| 缺失原子 | missing atoms | 需外部程序添加 |
| 总电荷非整数 | non-integer total charge | 浮点运算误差 |
| 温度耦合组 | temperature coupling group | 避免过小组 |
| 力场不可混合 | force fields are not mixable | 一致性原则 |
| 分子成键可视化 | bond visualization | 与拓扑定义可能不同 |

**关键概念**: 中文手册中的 FAQ 按五大主题组织：安装问题、系统准备和预处理、模拟方法、参数化与力场、分析与可视化。关键经验包括：不需要使用 MPI 编译所有实用程序（只有 mdrun 支持 MPI）；默认混合精度即可满足大多数需求；溶剂盒子坐标文件位于 \$GMXDIR/share/gromacs/top 目录中；不要将少量离子耦合到它们自己的温度耦合组；共轭梯度最小化不能使用约束；力场参数不可混用；可视化时看到的孔洞只是周期性边界的折叠效应，使用 trjconv 可以修复。

Sources: GROMACS 2019.6 中文译版 (§3.4)
