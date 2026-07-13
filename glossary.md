# GROMACS Manual Glossary

## A
**AWH** — Accelerated Weight Histogram method; an adaptive biasing technique for free energy calculation along reaction coordinates (Ch 29)

## B
**BAR** — Bennett's Acceptance Ratio; method for calculating free energy differences between states (Ch 21)

**BD** — Brownian Dynamics; integrator where velocity = force/friction + thermal noise, used for implicit solvent (Ch 20)

**Berendsen thermostat** — Weak coupling thermostat that scales velocities toward target temperature; fast but does not produce correct canonical ensemble (Ch 20)

**Berendsen barostat** — Weak coupling pressure control; useful for equilibration, not recommended for production (Ch 20)

## C
**CHARMM** — Chemistry at Harvard Molecular Mechanics; force field family with extensive lipid and protein parameters (Ch 8)

**CG** — Conjugate Gradient energy minimization algorithm; more efficient than steepest descent near minima (Ch 21)

**Colvars** — Collective Variables module for free energy and enhanced sampling calculations (Ch 30)

**cpt file (.cpt)** — Binary checkpoint file containing full-precision coordinates, velocities, and simulation state for exact restarts (Ch 27)

**CUDA** — NVIDIA GPU programming platform; primary GPU acceleration path for GROMACS (Ch 11)

## D
**Domain decomposition** — Spatial parallelization scheme dividing the simulation box into domains assigned to MPI ranks (Ch 22)

## E
**edr file (.edr)** — Energy data file in binary XDR format; stores energy terms over time, read by `gmx energy` (Ch 27)

**Editconf** — `gmx editconf`; converts between structure formats, defines simulation boxes (Ch 16)

**Essential Dynamics** — PCA-based sampling method focusing on collective motions; also called PCA dynamics (Ch 21)

**Expanded Ensemble** — Method sampling multiple states (e.g. lambda values) in a single simulation (Ch 21)

## F
**FEP** — Free Energy Perturbation; computes free energy differences from ensemble averages (Ch 21)

**FFT** — Fast Fourier Transform library required for PME electrostatics; FFTW is recommended (Ch 2)

## G
**Genion** — `gmx genion`; replaces solvent molecules with ions to neutralize or set salt concentration (Ch 16)

**gmx** — GROMACS command-line wrapper; all tools accessed as `gmx <tool>` (Ch 13)

**Grompp** — `gmx grompp`; preprocessor combining .gro, .top, .mdp into .tpr run input file (Ch 13)

**gro file (.gro)** — GROMOS coordinate/velocity file; human-readable fixed-column format (Ch 27)

**GROMOS** — GROMOS force field family; historically tied to GROMACS, uses geometric combination rules (Ch 8)

## H
**HIP** — AMD GPU programming platform; GROMACS supports HIP for AMD GPU acceleration (Ch 11)

## I
**itp file (.itp)** — Include topology file; contains molecular topology fragments (#include'd into .top files) (Ch 26)

## L
**L-BFGS** — Low-memory Broyden-Fletcher-Goldfarb-Shanno quasi-Newton minimizer; often faster than CG but not parallelized (Ch 21)

**Leap-frog** — Default GROMACS MD integrator; staggered update of velocities and positions, numerically stable (Ch 19)

**LINCS** — LINear Constraint Solver; default constraint algorithm for bonds, faster than SHAKE (Ch 20)

**LJ-PME** — Lennard-Jones PME; long-range dispersion correction using PME mesh (Ch 25)

**log file (.log)** — Text log from mdrun containing performance data, cycle counters, and warnings (Ch 27)

## M
**MDP file (.mdp)** — Molecular Dynamics Parameters file; contains all simulation parameters (integrator, dt, nsteps, temperature, pressure, cutoffs, etc.) (Ch 9)

**Mdrun** — `gmx mdrun`; the main MD simulation engine; performs integration and writes trajectories (Ch 13)

**MTS** — Multiple Time Stepping; evaluates PME less frequently than direct-space forces for performance (Ch 9)

**MPI** — Message Passing Interface; used for multi-node parallelization via domain decomposition (Ch 22)

## N
**ndx file (.ndx)** — Index file; defines atom groups for temperature coupling, analysis, restraints, etc. (Ch 27)

**Nose-Hoover** — Canonical thermostat producing correct NVT ensemble via extended system Hamiltonian (Ch 20)

## O
**OpenCL** — Open standard for heterogeneous computing; GROMACS supports OpenCL GPU backend (Ch 11)

**OpenMP** — Shared-memory threading; used for intra-node parallelization in GROMACS (Ch 22)

**OPLS** — Optimized Potentials for Liquid Simulations; force field family using geometric combination rules (Ch 8)

## P
**Parrinello-Rahman** — Isothermal-isobaric barostat producing correct NPT ensemble (Ch 20)

**pdb file (.pdb)** — Protein Data Bank coordinate format; standard input/output for many tools (Ch 27)

**pdb2gmx** — `gmx pdb2gmx`; converts PDB file to GROMACS topology (.top) and structure (.gro) (Ch 13)

**PME** — Particle-Mesh Ewald; efficient method for long-range electrostatics using FFT grid (Ch 25)

**PPPM** — Particle-Particle Particle-Mesh; alternative to PME, not recommended for new simulations (Ch 25)

**Pull code** — Applies external forces to defined groups; used for umbrella sampling, SMD, AFM pulling (Ch 28)

## Q
**QM/MM** — Quantum Mechanics/Molecular Mechanics hybrid simulation; GROMACS interfaces with CP2K or MiMiC (Ch 30)

## R
**Replica Exchange** — Parallel simulation method where replicas at different temperatures/hamiltonians exchange configurations (Ch 21)

**RMSD** — Root Mean Square Deviation; structural comparison metric (Ch 32)

**RMSF** — Root Mean Square Fluctuation; per-atom/ residue flexibility measure (Ch 32)

**RDF** — Radial Distribution Function; pair correlation g(r), used for structural analysis (Ch 32)

## S
**SETTLE** — Analytical constraint algorithm specific to rigid 3-atom water molecules; faster than iterative methods (Ch 20)

**SHAKE** — Iterative constraint algorithm; alternative to LINCS, resets bond lengths via Lagrange multipliers (Ch 20)

**Solvate** — `gmx solvate`; fills simulation box with solvent molecules (Ch 16)

**Steep** — Steepest Descent energy minimization; simple, robust, good for initial relaxation (Ch 21)

**SYCL** — Khronos standard for heterogeneous computing; supports Intel GPUs (Ch 11)

## T
**TI** — Thermodynamic Integration; free energy method integrating ∂H/∂λ along a lambda path (Ch 21)

**tng file (.tng)** — Trajectory Next Generation; compressed, portable trajectory format (Ch 27)

**top file (.top)** — System topology file; lists molecules, force field parameters, includes .itp files (Ch 26)

**tpr file (.tpr)** — Portable run input; binary file containing all simulation parameters and coordinates; sole input needed by mdrun (Ch 27)

**trjconv** — `gmx trjconv`; trajectory conversion and manipulation tool (Ch 13)

**trr file (.trr)** — Full-precision trajectory file with coordinates, velocities, and forces (Ch 27)

## V
**V-rescale** — Velocity rescaling thermostat with stochastic term; produces correct canonical ensemble (Ch 20) 

**Velocity Verlet** — Integrator alternative to leap-frog; produces full-step velocities, better for Nose-Hoover/PR coupling (Ch 19)

## X
**xtc file (.xtc)** — Compressed trajectory format using reduced precision; much smaller than .trr, suitable for analysis (Ch 27)

**xvg file (.xvg)** — Xmgrace plot data format; used for all GROMACS analysis tool output (Ch 27)

---

## Tutorial & Theory Additions

### A-B
**Area per lipid** — Key membrane simulation validation metric; DPPC experimental value ~62–64 Å² (ch16)

**BAR (Bennett Acceptance Ratio)** — Free energy analysis method using energy differences between neighboring lambda states; more efficient than TI, implemented via `gmx bar` (ch21)

**Berger lipids** — United-atom lipid parameters combining GROMOS atom types with OPLS partial charges; Ryckaert-Bellemans dihedral potentials with 1-4 LJ scaling factor 0.125 (ch16)

**CGenFF** — CHARMM General Force Field server; generates ligand topologies with penalty scores (<10: reliable; 10–50: validate; >50: reparametrize) (ch08)

**Decoupling (alchemical)** — Systematic removal of solute-solvent interactions via lambda scaling; discharge first (Coulomb → 0) then decouple vdW (LJ → 0) (ch21)

**Hydrogen bonds (GROMACS criteria)** — Donor-H distance ≤ 0.35 nm and donor-acceptor-H angle ≤ 30° (not donor-H-acceptor) (ch05)

### I-L
**InflateGRO** — Perl script for embedding proteins into lipid bilayers; inflates lipids (scale factor >1), deletes overlapping lipids, then iteratively shrinks (scale factor <1) until area per lipid converges (ch16)

**Insert-molecules (`gmx insert-molecules`)** — Adds molecules to simulation box with steric clash detection; used for non-water solvents where `solvate` is inappropriate (ch16)

**Lambda window** — A discrete value of the coupling parameter λ; 20+ windows (Δλ ≈ 0.05) needed for BAR convergence in free energy calculations (ch21)

**Ligand parametrization** — Process of generating topology/parameters for non-standard residues; requires external tools/servers specific to force field family (ch08)

### M-P
**Membrane packing** — InflateGRO methodology: orient → inflate → delete overlaps → EM → iteratively shrink to target area per lipid (ch16)

**Penalty score (CGenFF)** — Per-parameter quality metric; <10 = reliable for immediate use, 10–50 = validate before use, >50 = manual reparametrization required (ch08)

**Position restraints (ligand)** — Separate `posre_ligand.itp` file for restraining ligand during equilibration; controllable via independent `#ifdef POSRES_LIG` block (ch08)

**Pulling simulation** — SMD-like simulation where a harmonic restraint moves along a reaction coordinate at a defined rate; precursor to umbrella sampling windows (ch28)

### R-Z
**Reaction coordinate** — The degree of freedom along which PMF is calculated (e.g., COM distance between two groups); must be physically meaningful (ch28)

**Soft-core potentials** — Modified LJ/Coulomb functional form preventing singularities at intermediate λ values; parameters: sc-alpha=0.5, sc-power=1, sc-sigma=0.3 (ch25, ch21)

**Umbrella sampling** — Biased MD method: run multiple independent simulations with harmonic restraints at fixed positions along a reaction coordinate; WHAM reconstructs the unbiased PMF (ch28)

**Virtual sites** — Massless interaction centers constructed from real atoms; eliminate high-frequency bond vibrations → enable dt = 4–5 fs with constraints=all-bonds (ch25)

**Water deletor** — Custom Perl script for removing water molecules from the hydrophobic core of lipid bilayers after `gmx solvate` (ch16)

**WHAM (Weighted Histogram Analysis Method)** — Algorithm for reconstructing unbiased PMF from umbrella sampling histograms by deconvolving the known bias potential (ch28)

**蛙跳式算法 (Leap-frog)** — GROMACS默认MD积分器，对位置三阶精度，时间可逆 (ch17/ch19/ch20)

**速度重缩放 (V-rescale)** — 带有随机项的速度重缩放恒温器，产生正确的正则系综 (ch17/ch19/ch20)

**系综 (Ensemble)** — 统计力学术语，NVT(正则)/NPT(等温等压)/NVE(微正则)等 (ch17/ch19/ch20)

## 中文术语扩展 (Extended Chinese Terminology)

_来源：GROMACS 2019.6 中文译版_

**牛顿运动方程 / Newton's Equations of Motion** — MD 模拟求解的核心方程：m_i * d²r_i/dt² = F_i (ch17/ch19)

**周期性边界条件 / Periodic Boundary Conditions (PBC)** — 最小映像约定，截断半径不能超过盒子大小的一半 (ch19)

**邻居列表 / Neighbor List** — 将O(N^2)配对搜索分摊到nstlist步骤上，减少计算成本 (ch33)

**区域分解 / Domain Decomposition** — 将模拟盒子在空间上划分为多个域，分配到不同MPI进程 (ch22/ch33)

**维里 / Virial** — 压力计算的物理量，为所有力加和，混合精度情况下其波动可能比平均值大两个数量级 (ch33)

**最陡下降法 / Steepest Descent** — 沿负梯度方向前进的能量最小化方法，简单稳健但收敛可能慢 (ch21)

**共轭梯度法 / Conjugate Gradient** — 使用前步梯度信息的能量最小化方法，在极小点附近收敛快 (ch21)

**Born-Oppenheimer近似 / Born-Oppenheimer Approximation** — 电子基态近似假设，原子位置变化时电子瞬时调整 (ch17)

**模拟退火 / Simulated Annealing** — 高温模拟后缓慢降温来搜索全局极小值的方法 (ch33)

**保守力场 / Conservative Force Field** — 仅依赖于原子位置的势能函数，不包含电子运动 (ch08)

**截断半径 / Cutoff Radius** — Lennard-Jones和库仑相互作用的距离限制 (ch09/ch25)

**量子校正 / Quantum Correction** — 对经典谐振子的能量和比热进行量子力学修正 (ch34)

**块平均 / Block Averaging** — 将轨迹分块计算平均值，用块平均值之间的方差估计标准误差 (ch34)

**自相关函数 / Autocorrelation Function (ACF)** — 衡量数据点之间时间关联性的函数 (ch34)

**涨落性质 / Fluctuation Properties** — 从能量涨落计算热容、压缩系数、热膨胀系数等 (ch34)

**软核势 / Soft-core Potentials** — 修改的LJ/库仑函数形式，防止lambda中值处的奇点，参数sc-alpha=0.5, sc-power=1, sc-sigma=0.3 (ch21/ch25)

**解耦 (炼金术变换) / Decoupling (Alchemical Transformation)** — 通过lambda标度系统地移除溶质-溶剂相互作用：先放电(Coulomb→0)再解耦vdW(LJ→0) (ch21)

**伞形采样 / Umbrella Sampling** — 沿反应坐标设置多个独立模拟窗口，每个窗口施加简谐限制，WHAM重构无偏PMF (ch28)

**平均力势 / Potential of Mean Force (PMF)** — 沿反应坐标的自由能函数，通过伞形采样或AWH等方法计算 (ch28)

**加权直方图分析方法 / WHAM (Weighted Histogram Analysis Method)** — 通过解卷积已知偏置势函数，从伞形采样直方图重构无偏PMF (ch28)

**反应坐标 / Reaction Coordinate** — 计算PMF所沿的自由度（如两组质心距离），必须具有物理意义 (ch28)

**Lambda窗口 / Lambda Window** — 耦合参数λ的离散值；对于BAR收敛，需要20+个窗口(Δλ ≈ 0.05) (ch21)

**拉伸模拟 / Pulling Simulation** — SMD类模拟，简谐限制沿反应坐标以指定速率移动，为伞形采样窗口提供起始构型 (ch28)

**膜蛋白嵌入 / Membrane Protein Embedding (membed)** — ProtSqueeze技术：xy平面收缩蛋白质→删除重叠脂质→逐步恢复蛋白质原子 (ch16)

**半各向同性压力耦合 / Semi-isotropic Pressure Coupling** — xy平面和z方向分别独立耦合压力，膜模拟必需 (ch16/ch20)

**面积/脂质 (Area per Lipid)** — 膜模拟关键验证量，DPPC实验值约62-64 Å² (ch16)

**Berger脂质 / Berger Lipids** — 联合原子脂质参数，组合GROMOS原子类型和OPLS部分电荷 (ch16)

**CGenFF惩罚分数 / CGenFF Penalty Score** — 每个参数的质量指标：<10可靠使用，10-50验证后使用，>50需要手动重新参数化 (ch08)

**配体参数化 / Ligand Parametrization** — 为非标准残基生成拓扑/参数的过程，需要特定力场族的外部工具/服务器 (ch08)

**虚拟位点 / Virtual Sites** — 由真实原子构建的无质量相互作用中心，消除高频键振动，允许dt=4-5 fs (ch25)

**氢键标准 (GROMACS)** — 供体-H距离 ≤ 0.35 nm 且 供体-受体-H角度 ≤ 30° (ch05)

**InflateGRO** — Perl脚本：膨胀脂质(scale>1)→删除重叠脂质→逐步收缩(scale<1)至目标面积/脂质 (ch16)

**水删除器 / Water Deletor** — 删除脂质双层疏水核心中水分子的自定义Perl脚本 (ch16)

**插入分子 / gmx insert-molecules** — 通过位阻冲突检测向模拟盒子添加分子，用于非水溶剂的添加 (ch16)

**pdb2gmx** — 将PDB文件转换为GROMACS拓扑(.top)和结构(.gro)，自动选择力场和水模型 (ch13)

**电荷中性化 / Charge Neutralization** — gmx genion替换溶剂分子为离子，以中和总电荷或设置盐浓度 (ch05/ch16)

**约束算法 / Constraint Algorithm** — LINCS(默认)/SHAKE/SETTLE保持键长不变，允许更大时间步长 (ch20)

**积分器 / Integrator** — 蛙跳式(默认)/Velocity Verlet/随机动力学(sd)，求解运动方程 (ch19)

**恒温器 / Thermostat** — 温度耦合方法：Berendsen(快速但近似)/V-rescale(正确NVT)/Nose-Hoover(扩展系综) (ch20)

**恒压器 / Barostat** — 压力耦合方法：Berendsen(快速初始化)/Parrinello-Rahman(正确NPT) (ch20)

**粒子网格Ewald / Particle-Mesh Ewald (PME)** — 使用FFT网格高效计算长程静电相互作用 (ch25)

**能量最小化 / Energy Minimization (EM)** — 移除动能和结构异常，起始结构预处理必需步骤 (ch21)

**力场 / Force Field** — 提供原子间相互作用参数：AMBER, CHARMM, GROMOS, OPLS等 (ch08)

**混合精度 / Mixed Precision** — 状态向量(坐标/速度/力)单精度，关键变量双精度，默认选项 (ch33)

**双精度 / Double Precision** — 所有变量双精度，比混合精度慢20%-100%，用于简正分析等 (ch33)

**自由能微扰 / Free Energy Perturbation (FEP)** — 从系综平均计算自由能差的方法 (ch21)

**热力学积分 / Thermodynamic Integration (TI)** — 沿lambda路径积分∂H/∂λ的自由能方法 (ch21)

**Bennett接受率法 / Bennett Acceptance Ratio (BAR)** — 比TI更高效的自由能计算方法 (ch21)

**扩展系综 / Expanded Ensemble** — 在单次模拟中采样多个状态（如lambda值）的方法 (ch21)

**副本交换 / Replica Exchange** — 不同温度/哈密顿量的副本之间交换构型的并行模拟方法 (ch21)

**加速权重直方图 / Accelerated Weight Histogram (AWH)** — 沿反应坐标的自适应偏置自由能计算方法 (ch29)

**简正模式分析 / Normal Mode Analysis** — 计算简正模式频率，分析振动谱和热化学性质 (ch34)

**主成分分析 / Principal Component Analysis (PCA)** — 又称为本质动力学，关注集体运动 (ch21/ch32)

**均方根偏差 / RMSD** — 结构比较指标，蛋白质稳定性分析 (ch32)

**均方根涨落 / RMSF** — 每个原子/残基的柔性度量，蛋白质局部运动分析 (ch32)

**径向分布函数 / Radial Distribution Function (RDF)** — 对相关函数g(r)，结构分析使用 (ch32)

**偶极矩 / Dipole Moment** — 总偶极及其涨落，介电常数计算 (ch32)

**Lennard-Jones势 / Lennard-Jones Potential** — V_LJ = 4ε[(σ/r)^12-(σ/r)^6]，非键排斥和色散 (ch23)

**色散校正 / Dispersion Correction** — 长程LJ相互作用平均校正，脂质单层使用 (ch25)

**自由能计算 / Free Energy Calculation** — 结合自由能/溶剂化自由能计算 (ch21/ch28)

**可转移分子间势能 / Transferable Intermolecular Potential** — TIP3P/TIP4P/SPC等水模型 (ch05)

**拓扑文件 / Topology File (.top/.itp)** — 系统组成、力场参数、化学结构 (ch26)

**运行输入文件 / Run Input File (.tpr)** — 二进制文件，包含所有模拟参数和坐标 (ch27)

**轨迹文件 / Trajectory File (.xtc/.trr)** — 坐标随时间变化，.xtc压缩格式用于分析 (ch27)
