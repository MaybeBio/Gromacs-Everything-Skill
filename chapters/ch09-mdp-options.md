# Chapter 9: Molecular Dynamics Parameters (.mdp options)

## Core Idea
The .mdp file controls every aspect of a GROMACS simulation. Understanding the key parameters — integrator, timestep, temperature/pressure coupling, constraints, cutoffs, and output frequency — is essential for setting up correct and efficient simulations.

## Key Parameter Reference

### Preprocessing
| Parameter | Type | Default | Description |
|---|---|---|---|
| `include` | string | — | Directories to include in topology (-I format) |
| `define` | string | — | Preprocessor defines (-DPOSRES, -DFLEXIBLE) |

### Run Control
| Parameter | Type | Default | Description |
|---|---|---|---|
| `integrator` | enum | `md` | `md`, `md-vv`, `md-vv-avek`, `sd`, `bd`, `steep`, `cg`, `l-bfgs`, `nm`, `tpi`, `tpic`, `mimic` |
| `tinit` | real | 0 | Starting time (ps) |
| `dt` | real | 0.001 | Time step (ps). Standard: 0.002 |
| `nsteps` | int | 0 | Number of steps (-1 = no maximum) |
| `init-step` | int | 0 | Starting step number |
| `simulation-part` | int | 0 | Simulation part number |
| `mts` | enum | `no` | Multiple time stepping: `no`, `yes` |

### Integrator Details
| Integrator | Description | Best use |
|---|---|---|
| `md` | Leap-frog (default) | Production MD |
| `md-vv` | Velocity Verlet | Nose-Hoover/PR coupling |
| `md-vv-avek` | VV with averaged KE | Accurate Nose-Hoover |
| `sd` | Stochastic dynamics (Langevin) | Implicit solvent, thermostat |
| `bd` | Brownian dynamics | Coarse-grained, implicit solvent |
| `steep` | Steepest descent | Initial EM |
| `cg` | Conjugate gradient | Final EM refinement |
| `l-bfgs` | Quasi-Newton BFGS | Fast EM (not parallel) |
| `nm` | Normal mode analysis | Vibrational analysis |

### Output Control
| Parameter | Default | Description |
|---|---|---|
| `nstxout` | 0 | Steps between coordinate output (.trr) |
| `nstvout` | 0 | Steps between velocity output (.trr) |
| `nstfout` | 0 | Steps between force output (.trr) |
| `nstxout-compressed` | 0 | Steps between compressed output (.xtc) |
| `nstenergy` | 1000 | Steps between energy output (.edr) |
| `nstlog` | 1000 | Steps between log output (.log) |

### Neighbor Searching
| Parameter | Default | Description |
|---|---|---|
| `cutoff-scheme` | `Verlet` | `Verlet` (only option now) |
| `nstlist` | 10 | Steps between neighbor list update |
| `rlist` | 1.0 | Neighbor list cutoff (nm) |
| `verlet-buffer-tolerance` | 0.005 | Buffer tolerance (kJ/mol/ps) |

### Electrostatics
| Parameter | Default | Description |
|---|---|---|
| `coulombtype` | `PME` | `Cut-off`, `PME`, `P3M-AD`, `Reaction-Field`, `Ewald` |
| `rcoulomb` | 1.0 | Coulomb cutoff (nm) |
| `rcoulomb-modifier` | `Potential-shift` | Modifier at cutoff |
| `fourierspacing` | 0.12 | PME grid spacing (nm) |
| `pme-order` | 4 | PME interpolation order |
| `ewald-rtol` | 1e-5 | Ewald relative tolerance |

### VdW
| Parameter | Default | Description |
|---|---|---|
| `vdwtype` | `Cut-off` | `Cut-off`, `PME`, `Shift`, `Switch` |
| `rvdw` | 1.0 | VdW cutoff (nm) |
| `rvdw-modifier` | `Potential-shift` | Modifier at cutoff |
| `DispCorr` | `EnerPres` | Dispersion correction: `no`, `EnerPres`, `Ener` |

### Temperature Coupling
| Parameter | Default | Description |
|---|---|---|
| `tcoupl` | `no` | `no`, `berendsen`, `nose-hoover`, `v-rescale`, `andersen` |
| `tc-grps` | — | Temperature coupling groups |
| `tau-t` | — | Coupling time constant (ps) |
| `ref-t` | — | Reference temperature (K) |
| `nsttcouple` | -1 | Steps between temperature coupling |
| `nh-chain-length` | 10 | Nose-Hoover chain length |

### Pressure Coupling
| Parameter | Default | Description |
|---|---|---|
| `pcoupl` | `no` | `no`, `berendsen`, `Parrinello-Rahman`, `MTTK` |
| `pcoupltype` | `isotropic` | `isotropic`, `semiisotropic`, `anisotropic`, `surface-tension` |
| `tau-p` | — | Pressure coupling time constant (ps) |
| `ref-p` | — | Reference pressure (bar) |
| `compressibility` | — | Compressibility (bar⁻¹, typically 4.5e-5 for water) |
| `refcoord-scaling` | `no` | Coordinate scaling type: `no`, `all`, `com` |

### Velocity Generation
| Parameter | Default | Description |
|---|---|---|
| `gen-vel` | `no` | Generate velocities: `no`, `yes` |
| `gen-temp` | — | Temperature for velocity generation (K) |
| `gen-seed` | -1 | Random seed (-1 = from process ID) |

### Constraints
| Parameter | Default | Description |
|---|---|---|
| `constraints` | `none` | `none`, `h-bonds`, `all-bonds`, `h-angles`, `all-angles` |
| `constraint-algorithm` | `LINCS` | `LINCS`, `SHAKE` |
| `continuation` | `no` | Continuing from checkpoint: `no`, `yes` |
| `lincs-order` | 4 | LINCS expansion order |
| `lincs-iter` | 1 | LINCS iterations |
| `lincs-warnangle` | 30 | LINCS warning angle (degrees) |

### Energy Minimization
| Parameter | Default | Description |
|---|---|---|
| `emtol` | 10.0 | EM tolerance (kJ/mol/nm) |
| `emstep` | 0.01 | Initial EM step size (nm) |
| `nstcgsteep` | 1000 | Steps between steep for CG |
| `nbfgscorr` | 10 | BFGS correction steps |

## Worked Example: Standard Production MD .mdp
```ini
integrator          = md
dt                  = 0.002
nsteps              = 50000000    ; 100 ns
nstxout-compressed  = 5000        ; every 10 ps
nstenergy           = 1000        ; every 2 ps
nstlog              = 1000

cutoff-scheme       = Verlet
nstlist             = 20
rlist               = 1.0
coulombtype         = PME
rcoulomb            = 1.0
vdwtype             = Cut-off
rvdw                = 1.0
fourierspacing      = 0.12
pme-order           = 4

tcoupl              = v-rescale
tc-grps             = Protein Non-Protein
tau-t               = 0.1   0.1
ref-t               = 300   300

pcoupl              = Parrinello-Rahman
pcoupltype          = isotropic
tau-p               = 5.0
ref-p               = 1.0
compressibility     = 4.5e-5

constraints         = h-bonds
constraint-algorithm = LINCS
continuation        = yes
```

## Key Takeaways
1. `dt=0.002` with `constraints=h-bonds` is the production standard
2. Set `rlist = rcoulomb = rvdw` (all 1.0 nm) for simplicity
3. Use `tc-grps = Protein Non-Protein` to couple protein and solvent separately
4. V-rescale for thermostat, Parrinello-Rahman for barostat in production
5. `continuation=yes` is critical for restarts from checkpoint

## Connects To
- **Ch 4**: Getting started workflow
- **Ch 20**: Thermostat/barostat algorithms
- **Ch 25**: Electrostatic and VdW interaction details

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 预处理 | preprocessing | include / define |
| 运行控制 | run control | integrator / dt / nsteps |
| 积分器 | integrator | md, md-vv, sd, steep, cg 等 |
| 时间步长 | time step (dt) | 标准 0.002 ps (2 fs) |
| 步数 | number of steps (nsteps) | -1 表示无上限 |
| 输出控制 | output control | nstxout / nstvout / nstenergy |
| 邻区搜索/邻居列表 | neighbor searching / neighbor list | nstlist / rlist / verlet-buffer-tolerance |
| 截断方案 | cutoff scheme | 默认 Verlet，组方案已废弃 |
| 库仑相互作用 | Coulomb interaction | coulombtype / rcoulomb |
| PME (粒子网格Ewald) | Particle-Mesh Ewald | fourierspacing / pme-order |
| 范德华相互作用 | van der Waals | vdwtype / rvdw / DispCorr |
| 温度耦合/控温器 | temperature coupling / thermostat | tcoupl / tc-grps / tau-t / ref-t |
| 压力耦合/控压器 | pressure coupling / barostat | pcoupl / pcoupltype / tau-p / ref-p |
| 键约束 | bond constraints | constraints / constraint-algorithm |
| LINCS 算法 | LINCS algorithm | 线性约束求解器，默认 |
| 速度产生 | velocity generation | gen-vel / gen-temp / gen-seed |
| 能量最小化容差 | EM tolerance | emtol / emstep |
| 质心牵引 | center-of-mass pulling | pull / pull-ncoords |
| 自由能计算 | free energy calculation | free-energy / couple-lambda 等 |

**关键概念**: .mdp 文件包含模拟的所有设置参数。最重要的参数包括：积分器（md 为默认蛙跳积分器）、时间步长（标准 2 fs 配合 h-bonds 约束）、邻区搜索（Verlet 方案为默认且唯一推荐）、库仑类型（PME 为长程静电标准方法）、温度耦合（v-rescale 为最稳健选择）、压力耦合（Parrinello-Rahman 为生产模拟标准）、以及约束（constraints=h-bonds 配合 LINCS 算法）。中文手册的 mdp 选项按照预处理、运行控制、输出控制、邻区搜索、静电、VdW、温度耦合、压力耦合、约束、速度产生、牵引和自由能计算等主题组织。

Sources: GROMACS 2019.6 中文译版 (§3.8)
