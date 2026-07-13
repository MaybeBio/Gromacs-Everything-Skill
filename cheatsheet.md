# GROMACS Quick Reference Cheatsheet

## Decision Rules

### Integrator Selection
| Goal | Integrator | Reason |
|---|---|---|
| Production MD (standard) | `md` (leap-frog) | Fast, stable, default |
| Production (Nose-Hoover) | `md-vv` | Better thermostat integration |
| Energy minimization | `steep` (initial) → `cg` or `l-bfgs` | Steep descent robust, CG converges faster |
| Normal mode analysis | `nm` | Requires double precision |
| Stochastic dynamics | `sd` | Langevin, good for implicit solvent |
| Free energy (TPI) | `tpi` | Test particle insertion |

### Thermostat Selection
| Phase | Thermostat | `tau-t` |
|---|---|---|
| Pre-equilibration (fast relax) | `Berendsen` | 0.1 ps |
| NVT equilibration | `v-rescale` | 0.1 ps |
| Production (canonical) | `v-rescale` or `Nose-Hoover` | 1.0–2.0 ps |

### Barostat Selection
| Phase | Barostat | `tau-p` |
|---|---|---|
| Initial box relaxation | `Berendsen` | 2.0 ps |
| Production NPT | `Parrinello-Rahman` | 5.0 ps |

### Constraint Algorithm
| When | Setting | dt max |
|---|---|---|
| Standard simulation | `constraints = h-bonds` | 2 fs |
| With HMR (mass repartitioning) | `constraints = h-bonds` | 4 fs |
| Virtual sites | `constraints = all-bonds` | 5 fs |

## Key Defaults & Thresholds

| Parameter | Default | When to change |
|---|---|---|
| `dt` | 0.001 ps (1 fs) | Set to 0.002 for standard MD |
| `nsteps` | 0 (no limit) | Always set explicitly |
| `rlist` | 1.0 nm | Usually = `rcoulomb` = `rvdw` |
| `rcoulomb` | 1.0 nm | 0.9 nm for 4 fs dt with HMR |
| `rvdw` | 1.0 nm | Same as rcoulomb |
| `fourierspacing` | 0.12 nm | 0.10 nm for high accuracy |
| `pme-order` | 4 | 6 for high accuracy |
| `nstlist` | 10 | 20-40 for GPU runs |
| `nstxout-compressed` | 5000 (5 ps) | Smaller for detailed analysis |
| `nstenergy` | 1000 | 100-500 for better energy resolution |
| `nstlog` | 1000 | Matches nstenergy |
| `emtol` | 10.0 kJ/mol/nm | 1000 for initial EM, 10 for final |
| `emstep` | 0.01 nm | 0.001 for delicate systems |
| `pcoupltype` | `isotropic` | `semiisotropic` for membranes |

## GPU vs CPU
| Situation | Recommendation |
|---|---|
| < 10k atoms | Single GPU sufficient |
| 10k–100k atoms | GPU + CPU (GPU-resident mode) |
| > 100k atoms | Multi-GPU with separate PME rank |
| Pure CPU cluster | MPI + OpenMP hybrid (1 thread/rank) |
| PME on GPU | Enable `-pme gpu` for best performance |
| GPU-resident mode | Requires `constraints=h-bonds`, verlet scheme |

## Free Energy Quick Reference
| Task | Method | Key mdp flags |
|---|---|---|
| Solvation free energy | TI + soft-core | `free-energy=yes`, `sc-alpha=0.5` |
| Binding free energy | FEP/MBAR | `free-energy=yes`, `calc-lambda-neighbors=1` |
| PMF along coordinate | Umbrella sampling | `pull=yes`, `pull-coord1-type=umbrella` |
| Automatic PMF | AWH | `awh=yes`, define awh coordinates |
| Alchemical transformation | Lambda windows | `couple-lambda0=vvdw-q`, `couple-lambda1=none` |

## Tells & Smells
| Observation | Diagnosis |
|---|---|
| Temperature drifting in NPT | PR barostat + Nose-Hoover need proper coupling |
| Density too low after NPT | Extend equilibration, check box type |
| LINCS warnings at step 0 | Bad starting geometry, atoms overlapping |
| "1-4 interaction between X atoms" | Check .top file pair definitions |
| Energy spikes in NVE | Too large timestep for constraints |
| PME grid dimensions change | Box size changed (check pressure coupling) |
| Slow performance, low GPU utilization | Too few atoms per GPU, add more system |
| Neighbor list overflow | Increase `rlist` buffer: `verlet-buffer-tolerance=-1` |
| Segfault at start | Wrong GMX_SIMD for CPU architecture |

## Common Command Patterns
```bash
# Quick single-node production
gmx mdrun -deffnm md -v -cpi md.cpt -noappend

# Multi-simulation (replicas)
gmx mdrun -multidir sim{0..7} -v

# GPU with PME offload
gmx mdrun -deffnm md -nb gpu -pme gpu -bonded gpu -update gpu

# MPI with separate PME ranks
mpirun -np 16 gmx_mpi mdrun -s topol.tpr -npme 4

# Analysis
gmx rms -s topol.tpr -f traj.xtc
gmx rmsf -s topol.tpr -f traj.xtc -o rmsf.xvg -res
gmx energy -f ener.edr -o potential.xvg
gmx gyrate -s topol.tpr -f traj.xtc
gmx hbond -s topol.tpr -f traj.xtc -num hbnum.xvg
gmx cluster -s topol.tpr -f traj.xtc -method gromos -cutoff 0.2
```

---

## Tutorial Decision Rules (Lemkul Tutorials, integrated into chapters)

### Ligand Parametrization Server Selection
| Force Field | Server/Tool | Notes |
|---|---|---|
| CHARMM | CGenFF | Penalty scores indicate quality; <10 reliable |
| AMBER/GAFF | Antechamber + acpype | Python wrapper writes GROMACS topologies |
| GROMOS96 54A7 | ATB (Automated Topology Builder) | Newer server, GROMOS-compatible |
| OPLS-AA | LigParGen | Jorgensen group, OPLS topologies |

### Membrane Simulation Rules
| Parameter | Value | Reason |
|---|---|---|
| `pcoupltype` | `semiisotropic` | xy and z deform independently |
| `ref_p` | 1.0 1.0 (xy z) | Separate reference pressure for lateral/normal |
| `tc-grps` | `Protein_DPPC Water_and_ions` | Different diffusion rates |
| `comm-grps` | `Protein_DPPC Water_and_ions` | Prevent phase drift in interfacial systems |
| T for DPPC | ≥ 323 K | Above phase transition (Tm = 315 K) |
| Area/lipid target | ~62–64 Å² (DPPC) | Experimental value; verify after equilibration |

### Free Energy Quick Reference (Tutorial Context)
| Task | Lambda windows | Key MDP | Analysis |
|---|---|---|---|
| vdW decoupling | 21 (Δλ=0.05) | sc-alpha=0.5, couple-lambda0=vdw-q | `gmx bar` |
| Coulomb discharge | 21 (Δλ=0.05) | sc-coul=yes, couple-lambda0=q | `gmx bar` |
| Hydration FE | 42 total (2 stages) | Sequential: q→0 then vdW→0 | ΔG_coul + ΔG_vdW |

### Umbrella Sampling Quick Reference
| Parameter | Typical Value | When to Adjust |
|---|---|---|
| `pull-coord1-k` | 1000 kJ/mol/nm² | Tighten if no overlap, loosen if too narrow |
| `pull-coord1-rate` | 0.01 nm/ps (pulling) | Lower = closer to equilibrium |
| `pull-coord1-rate` | 0.0 (umbrella) | Fixed position for sampling |
| Window spacing | 0.1–0.2 nm | Must produce overlapping histograms |
| Production/window | 5–20 ns | Longer for converged PMF |

### Timestep Selection (with Tutorial Methods)
| dt | Constraints | Method | Tutorial |
|---|---|---|---|
| 1 fs | none | Standard (no constraints) | — |
| **2 fs** | `h-bonds` | **Standard for all tutorials** | ch05 |
| 4 fs | `h-bonds` + HMR | Mass repartitioning | ch25 |
| 5 fs | `all-bonds` + vsites | Virtual sites | ch25 |

### Tutorial Analysis Commands Reference
```bash
# Standard analysis (ch05 / ch32)
gmx trjconv -s md.tpr -f md.xtc -o md_noPBC.xtc -pbc mol -center
gmx rms -s md.tpr -f md_noPBC.xtc -o rmsd.xvg -tu ns
gmx rmsf -s md.tpr -f md_noPBC.xtc -o rmsf.xvg -res
gmx gyrate -s md.tpr -f md_noPBC.xtc -o gyrate.xvg -sel Protein -tu ns
gmx dssp -s md.tpr -f md_noPBC.xtc -tu ns -o dssp.dat -num dssp_num.xvg
gmx hbond -s md.tpr -f md_noPBC.xtc -tu ns -num hbnum.xvg

# Membrane analysis (ch16)
gmx order -s md.tpr -f md.xtc -n sn1.ndx -d z -od deuter.xvg
gmx density -s md.tpr -f md.xtc -n density_groups.ndx -o dens.xvg -d Z
gmx msd -s md.tpr -f md.xtc -n p8.ndx -lateral z

# Free energy analysis (ch21)
gmx bar -f md*_dhdl.xvg -o bar.xvg

# Umbrella sampling analysis (ch28)
gmx wham -it tpr-files.dat -if pullf-files.dat -o profile.xvg -hist histo.xvg
```

---

## 中文参数速查 (Chinese Parameter Quick Reference)

_Sources: GROMACS 2019.6 中文译版_

### 积分器选择 (Integrator Selection) [中文版]
| 目标 | 积分器 | 原因 |
|---|---|---|
| 成品MD（标准） | `md` (蛙跳式) | 快速、稳定、默认 |
| 成品MD（Nose-Hoover） | `md-vv` (Velocity Verlet) | 更好的恒温器耦合 |
| 能量最小化 | `steep` (最陡下降) → `cg` (共轭梯度) | 稳健的初步松弛 |
| 简正模式分析 | `nm` | 需双精度编译 |
| 随机动力学 | `sd` (Langevin) | 隐性溶剂适用 |

### 温度耦合决策规则 (Thermostat Selection) [中文版]
| 阶段 | 恒温器 | `tau-t` | 注 |
|---|---|---|---|
| 预平衡（快速松弛） | `Berendsen` | 0.1 ps | 不产生正确正则系综 |
| NVT平衡 | `V-rescale` (速度重缩放) | 0.1 ps | 正确正则系综 |
| 成品模拟 | `V-rescale` 或 `Nose-Hoover` | 1.0-2.0 ps | Nosé-Hoover扩展系综 |

### 压力耦合决策规则 (Barostat Selection) [中文版]
| 阶段 | 恒压器 | `tau-p` | 注 |
|---|---|---|---|
| 初始盒子松弛 | `Berendsen` | 2.0 ps | 快速接近目标压力 |
| 成品NPT | `Parrinello-Rahman` | 5.0 ps | 正确NPT系综 |
| 膜模拟 | 半各向同性 (semiisotropic) | xy和z独立 | 必须 `pcoupltype=semiisotropic` |

### 约束算法 (Constraint Algorithm) [中文版]
| 条件 | 设置 | dt上限 | 注 |
|---|---|---|---|
| 标准模拟 | `constraints = h-bonds` | 2 fs | 只约束含氢键 |
| HMR (质量重分配) | `constraints = h-bonds` | 4 fs | 增加氢原子质量 |
| 虚拟位点 (Virtual Sites) | `constraints = all-bonds` | 5 fs | 需要手动拓扑编辑 |

### 关键参数默认值与阈值 (Key Defaults & Thresholds) [中文版]
| 参数 | 默认值 | 何时更改 |
|---|---|---|
| `dt` | 0.001 ps (1 fs) | 设为0.002用于标准MD |
| `nsteps` | 0 (无限制) | 始终明确设置 |
| `rlist` | (无硬默认) | = `rcoulomb` = `rvdw`，通常1.0 nm |
| `rcoulomb` | 1.0 nm | 0.9 nm用于HMR 4 fs dt |
| `rvdw` | 1.0 nm | = rcoulomb |
| `fourierspacing` | 0.12 nm | 0.10 nm用于高精度 |
| `pme-order` | 4 | 6用于高精度 |
| `nstlist` | 10 | 20-40用于GPU运行 |
| `nstxout-compressed` | 5000 (5 ps) | 更小值用于详细分析 |
| `emtol` | 10.0 kJ/mol/nm | 1000用于初步EM，10用于最终 |
| `emstep` | 0.01 nm | 0.001用于精细系统 |
| `pcoupltype` | `isotropic` (各向同性) | `semiisotropic` (半各向同性) 用于膜 |

### 自由能计算速查 (Free Energy Quick Reference) [中文版]
| 任务 | 方法 | 关键MDP标志 |
|---|---|---|
| 溶剂化自由能 | TI + 软核势 | `free-energy=yes`, `sc-alpha=0.5` |
| 结合自由能 | FEP/MBAR | `free-energy=yes`, `calc-lambda-neighbors=1` |
| 沿坐标的PMF | 伞形采样 | `pull=yes`, `pull-coord1-type=umbrella` |
| 自动PMF | AWH | `awh=yes`, 定义awh坐标 |
| 炼金术变换 | Lambda窗口 | `couple-lambda0=vdw-q`, `couple-lambda1=none` |

### 常见问题诊断 (Tells & Smells) [中文版]
| 观察 | 诊断 |
|---|---|
| NPT中温度漂移 | PR恒压器+Nose-Hoover需要适当的耦合 |
| NPT后密度太低 | 延长平衡时间，检查盒子类型 |
| 第0步LINCS警告 | 起始结构几何差，原子重叠 |
| "1-4相互作用在X原子间" | 检查.top文件中的对定义 |
| NVE中能量尖峰 | 时间步长太大，约束条件不满足 |
| PME网格维数改变 | 盒子尺寸变化（检查压力耦合） |
| 性能慢、GPU利用率低 | 每个GPU原子太少 |
| 邻居列表溢出 | 增加 `rlist` 缓冲区: `verlet-buffer-tolerance=-1` |
| 启动时Segfault | CPU架构GMX_SIMD设置错误 |
| 盒向量信息缺失 | 使用 `gmx trjconv -pbc mol -center` |
| 无限/NaN坐标 | 力场问题，检查起始结构和参数 |
| 动能异常高/低 | 检查初始温度、约束和力场 |

### 膜模拟决策规则 (Membrane Simulation Rules) [中文版]
| 参数 | 值 | 原因 |
|---|---|---|
| `pcoupltype` | `semiisotropic` | xy和z独立变形 |
| `ref_p` | 1.0 1.0 (xy z) | 横向和法向分别参考压力 |
| `tc-grps` | `Protein_DPPC Water_and_ions` | 不同组扩散率不同 |
| `comm-grps` | `Protein_DPPC Water_and_ions` | 防止界面系统中的质心漂移 |
| DPPC温度 | ≥ 323 K | 高于相变温度(Tm = 315 K) |
| 面积/脂质目标 | ~62-64 Å² (DPPC) | 实验值，平衡后验证 |

### 配体参数化服务器选择 (Ligand Parametrization Server) [中文版]
| 力场 | 服务器/工具 | 注 |
|---|---|---|
| CHARMM | CGenFF | 惩罚分数指示质量 |
| AMBER/GAFF | Antechamber + acpype | Python包装器写GROMACS拓扑 |
| GROMOS96 54A7 | ATB (Automated Topology Builder) | GROMOS兼容 |
| OPLS-AA | LigParGen | Jorgensen课题组 |

### 伞形采样速查 (Umbrella Sampling Quick Reference) [中文版]
| 参数 | 典型值 | 何时调整 |
|---|---|---|
| `pull-coord1-k` | 1000 kJ/mol/nm² | 无重叠时收紧，太窄时放松 |
| `pull-coord1-rate` | 0.01 nm/ps (拉伸) | 更小值接近平衡态 |
| `pull-coord1-rate` | 0.0 (伞形) | 固定位置采样 |
| 窗口间距 | 0.1-0.2 nm | 必须产生重叠的直方图 |
| 成品/窗口 | 5-20 ns | 更长时间用于收敛的PMF |

### 时间步长选择 (Timestep Selection) [中文版]
| dt | 约束 | 方法 | 注 |
|---|---|---|---|
| 1 fs | none | 无约束标准 | 保守选项 |
| **2 fs** | `h-bonds` | **所有教程标准** | 最常用 |
| 4 fs | `h-bonds` + HMR | 质量重分配 | 增加H质量 |
| 5 fs | `all-bonds` + vsites | 虚拟位点 | 手动拓扑编辑 |
