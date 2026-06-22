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
