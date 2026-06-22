# Chapter 39: Advanced MDP Parameters

## Core Idea
Beyond basic simulation parameters, GROMACS supports advanced .mdp options for free energy calculations, enhanced sampling (AWH, expanded ensemble, pull code), and QM/MM. These parameters enable specialized simulation workflows.

## Free Energy Parameters
| Parameter | Default | Description |
|---|---|---|
| `free-energy` | no | Enable free energy calculation |
| `couple-moltype` | — | Molecule to couple/decouple |
| `couple-lambda0` | vdw-q | Interactions at lambda=0 |
| `couple-lambda1` | none | Interactions at lambda=1 |
| `couple-intramol` | no | Couple intramolecular |
| `init-lambda` | 0.0 | Starting lambda |
| `delta-lambda` | 0.0 | Lambda step per window |
| `init-lambda-state` | -1 | Explicit lambda state |
| `nstdhdl` | 10 | dH/dλ output frequency |
| `calc-lambda-neighbors` | -1 | Lambda neighbors for BAR |
| `sc-alpha` | 0.0 | Soft-core alpha |
| `sc-power` | 1 | Soft-core power |
| `sc-sigma` | 0.3 | Soft-core sigma (nm) |
| `sc-coul` | no | Soft-core Coulomb |

## Expanded Ensemble Parameters
| Parameter | Description |
|---|---|
| `expanded-ensemble` | Enable expanded ensemble |
| `nstexpanded` | Update frequency |
| `lmc-stats` | Lambda dynamics method |
| `lmc-weights-equil` | Weight equilibration steps |
| `weight-equil-wl-delta` | Wang-Landau delta |

## Pull Code Parameters (Advanced)
| Parameter | Description |
|---|---|
| `pull-print-com` | Print center of mass |
| `pull-print-ref-value` | Print reference value |
| `pull-print-components` | Print vector components |
| `pull-nstxout` | COM output frequency |
| `pull-nstfout` | Force output frequency |
| `pull-pbc-ref-prev-step-com` | PBC reference handling |

## AWH Parameters (Advanced)
| Parameter | Description |
|---|---|
| `awh-nstout` | Output frequency |
| `awh-nstsample` | Sampling frequency |
| `awh-share-group` | Bias sharing group |
| `awh-target` | Target distribution |
| `awh-error-init` | Initial error estimate |
| `awh-growth` | Error growth strategy |
| `awh-equilibrate-histogram` | Equilibration mode |

## QM/MM Parameters
| Parameter | Description |
|---|---|
| `QMMM` | Enable QM/MM |
| `QMMM-grps` | QM atom groups |
| `QMmethod` | QM method |
| `QMbasis` | Basis set |
| `QMcharge` | QM region charge |
| `QMmult` | QM multiplicity |
| `SH` | Surface hopping |

## Key Takeaways
1. Free energy needs soft-core: sc-alpha=0.5, sc-power=1, sc-sigma=0.3
2. Expanded ensemble is convenient for lambda sampling
3. AWH convergence depends critically on diffusion estimate
4. QM/MM requires separate QM software (CP2K or MiMiC)

## Connects To
- **Ch 9**: Basic MDP parameters
- **Ch 21**: Free energy theory
- **Ch 28-30**: Special topics
