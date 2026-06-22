# Chapter 6: Managing Long Simulations

## Core Idea
GROMACS supports robust checkpoint-based restart for long simulations. The .cpt file contains full-precision coordinates, velocities, and simulation state, enabling exact continuation. Understanding reproducibility and the difference between binary and analytical equivalence is important for production simulations.

## Frameworks Introduced
- **Checkpoint-restart pattern**: Use `-cpi state.cpt` to continue; `-noappend` to avoid overwriting output; `-cpo state_new.cpt` to name output checkpoint
- **Extension pattern**: Use `gmx convert-tpr -extend <time>` to extend completed simulation; then `gmx mdrun -s next.tpr -cpi state.cpt`

## Key Concepts

### Appending vs New Output
- By default, mdrun appends to existing output files (trajectory, energy, log)
- Use `-noappend` to start fresh output files (deletes previous output)
- Appending preserves the full simulation history in a single set of files

### Checkpoint Files (.cpt)
- Binary files containing full-precision coordinates, velocities, forces, and simulation state
- Named `state.cpt` by default; rename with `-cpo`
- Can be queried with `gmx check -f state.cpt`
- Required for exact continuation

### Extending .tpr Files
- When a simulation completes, use `gmx convert-tpr -s old.tpr -extend <time> -o next.tpr`
- Then continue: `gmx mdrun -s next.tpr -cpi state.cpt`
- This preserves the original simulation parameters while extending the runtime

### Changing MDP Options for Restart
- Create new .tpr with modified .mdp: `gmx grompp -f new.mdp -c old.gro -p topol.top -t state.cpt -o new.tpr`
- The `-t state.cpt` flag copies full-precision coordinates and velocities from checkpoint
- This allows changing parameters (e.g., switching from NVT to NPT, changing thermostat) mid-simulation

### Restarts Without Checkpoint Files
- If .cpt is lost, use last .gro frame: `gmx grompp -f new.mdp -c lastframe.gro -p topol.top -o new.tpr`
- Velocities are generated from Maxwell-Boltzmann distribution at `gen-temp`
- This is NOT an exact continuation — velocities are random

### Are Continuations Exact?
- **Exact binary reproducibility**: Requires same .tpr, same .cpt, same executable, same hardware, same number of MPI ranks, same OpenMP threads, same `-reprod` flag
- **Analytical equivalence**: Leap-frog and velocity Verlet trajectories are analytically (not binary) identical when started from corresponding points
- **Long-term divergence**: Even with exact restarts, trajectories diverge due to floating-point non-associativity in parallel runs

### Reproducibility
- Use `-reprod` flag to force deterministic execution (binary identical output on same hardware)
- Performance impact: significant reduction in parallel efficiency
- Generally NOT needed — statistical equivalence is sufficient for analysis

## Code Examples
```bash
# Standard restart from checkpoint
gmx mdrun -s topol.tpr -cpi state.cpt -noappend

# Extend completed simulation by 100 ns
gmx convert-tpr -s topol.tpr -extend 100000 -o next.tpr
gmx mdrun -s next.tpr -cpi state.cpt

# Change parameters and continue
gmx grompp -f production.mdp -c npt.gro -t state.cpt -p topol.top -o prod.tpr
gmx mdrun -s prod.tpr -cpi state.cpt

# Query checkpoint contents
gmx check -f state.cpt

# Reproducible run (same binary output on same hardware)
gmx mdrun -s topol.tpr -reprod
```

## Key Takeaways
1. Always save checkpoint files — they're your insurance against crashes
2. Use `-noappend` only when you want to start fresh output files
3. `gmx convert-tpr -extend` is the proper way to extend completed simulations
4. For exact restarts after changing parameters, use `grompp -t state.cpt`
5. Binary reproducibility requires identical hardware and parallel decomposition — use `-reprod` for debugging only
6. Statistical equivalence is sufficient for production analysis

## Connects To
- **Ch 10**: mdrun features (rerun, multi-simulation)
- **Ch 15**: Validation and reproducibility
- **Ch 27**: .cpt file format details
