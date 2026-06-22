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
