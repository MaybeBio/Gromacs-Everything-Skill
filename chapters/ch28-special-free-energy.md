# Chapter 28: Special Topics — Free Energy & Pulling

## Core Idea
The pull code applies external forces to defined groups, enabling umbrella sampling, steered MD, and AFM pulling. Combined with the free energy code, it computes potential of mean force (PMF) along reaction coordinates.

## Free Energy Implementation
- `free-energy = yes` enables alchemical transformations
- Couple parameters: `couple-moltype`, `couple-lambda0`, `couple-lambda1`
- `couple-intramol = yes` for intramolecular interactions
- Lambda schedule: `init-lambda` + `delta-lambda` or explicit `init-lambda-state`
- `calc-lambda-neighbors = 1` for BAR analysis
- `nstdhdl = 10` for dH/dλ output frequency
- Separate VdW and Coulomb coupling: `couple-lambda0 = vdw-q`, `couple-lambda1 = none`

## Potential of Mean Force

### Pull Code Setup
```ini
pull                    = yes
pull-ncoords            = 1
pull-ngroups            = 1
pull-group1-name        = Chain_A
pull-group2-name        = Chain_B
pull-coord1-type        = umbrella      ; or constant-force, constant-rate
pull-coord1-geometry    = distance      ; direction, direction-periodic, cylinder
pull-coord1-dim         = N N Y
pull-coord1-rate        = 0.0
pull-coord1-k           = 1000          ; force constant (kJ/mol/nm²)
pull-coord1-start       = yes
```

### Pull Types
| Type | Purpose | Key parameters |
|---|---|---|
| `umbrella` | Harmonic restraint along coordinate | `pull-coord1-k` |
| `constraint` | Fixed distance (SHAKE-like) | No force constant needed |
| `constant-force` | Constant pulling force | `pull-coord1-k` as force magnitude |
| `flat-bottom` | Harmonic only outside range | `pull-coord1-tol` |
| `flat-bottom-high` | Harmonic only above range | Upper wall restraint |

### Pull Geometries
| Geometry | Description |
|---|---|
| `distance` | Center-of-mass distance |
| `direction` | Projection onto a vector |
| `direction-periodic` | Direction with periodic component |
| `cylinder` | Radial distance from axis |
| `angle` | Angle between groups |
| `dihedral` | Dihedral angle |
| `position` | Absolute position |

## Non-Equilibrium Pulling
- Constant velocity: `pull-coord1-rate > 0`
- Steered MD: time-dependent pulling
- Jarzynski equality for free energy from non-equilibrium work
- Multiple trajectories needed for convergence

## Workflow
```bash
# Generate umbrella windows
for i in $(seq 0 0.1 3.0); do
    # Set pull-coord1-init = $i in each window
    gmx grompp -f umbrella.mdp -c npt.gro -p topol.top -o window_${i}.tpr
done

# Run all windows
gmx mdrun -multidir window_*

# Analyze with WHAM
gmx wham -it tpr_files.dat -if pullf_files.dat -o profile.xvg -hist histo.xvg
```

## Key Takeaways
1. Umbrella sampling is the standard method for 1D PMF calculations
2. WHAM or umbrella integration for post-processing
3. Pull force constant ~1000 kJ/mol/nm² is typical
4. Use 20-40 windows for well-converged PMF
5. Non-equilibrium pulling needs many trajectories (Jarzynski)

## Connects To
- **Ch 21**: Free energy theory
- **Ch 25**: Soft-core potentials
- **Ch 29**: AWH (automatic alternative to umbrella sampling)
