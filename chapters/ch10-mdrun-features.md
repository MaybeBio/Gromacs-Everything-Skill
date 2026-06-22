# Chapter 10: Useful mdrun Features

## Core Idea
mdrun supports re-running trajectories with different physics models, reproducible execution, graceful shutdown via signals, multi-simulation directories, and flexible simulation length control. These features enable debugging, high-throughput screening, and robust production workflows.

## Key Concepts

### Re-running a Simulation (`-rerun`)
- Takes a trajectory file (.trr) and recomputes energies and forces using the model in the .tpr
- Useful for: computing energies from a different force field, free energy analysis (dhdl.xvg generation), debugging
- No integration is performed — coordinates come from the trajectory
- Kinetic energy may be incorrect when rerunning, so free energy calculations from rerun need analytical KE correction

```bash
gmx mdrun -s topol.tpr -rerun traj.trr
```

### Reproducible Mode (`-reprod`)
- Forces deterministic execution for binary-identical output on the same hardware
- Reduces parallel efficiency significantly
- Use only for debugging or when exact reproducibility is required
- Does NOT guarantee reproducibility across different hardware or parallel decomposition

```bash
gmx mdrun -s topol.tpr -reprod
```

### Halting Running Simulations
- **SIGTERM**: Graceful shutdown at next neighbor list update (saves checkpoint)
- **SIGUSR1**: Graceful shutdown at next checkpoint interval (clean stop)
- **SIGINT** (Ctrl+C): Immediate stop with checkpoint save
- Checkpoint is always written when signal received, enabling safe restart

```bash
kill -SIGUSR1 <mdrun_pid>   # Clean stop at next checkpoint
kill -SIGTERM <mdrun_pid>   # Stop at next NS step
```

### Multi-Simulations (`-multidir`)
- Run multiple independent simulations from a single mdrun invocation
- Each simulation in its own subdirectory with its own .tpr
- Shares MPI communicator for efficient resource use
- Useful for: replica exchange, free energy lambda windows, ensemble runs

```bash
gmx mdrun -multidir sim{0..15} -v
# Each simN/ directory must contain topol.tpr
```

### Controlling Simulation Length
- `-nsteps`: Maximum number of steps (overrides .tpr value)
- `-maxh`: Maximum wall-clock time in hours (mdrun stops cleanly before time expires)
- `-nsteps -1`: Run until stopped by signal
- Use `-maxh` in queue systems to stop before wall-time limit

```bash
gmx mdrun -s topol.tpr -maxh 23.5  # Stop before 24h queue limit
gmx mdrun -s topol.tpr -nsteps 50000000  # Override .tpr nsteps
```

### Performance Counters
mdrun reports six performance metrics in the .log file:
1. **ns/day**: Nanoseconds simulated per 24 hours (most intuitive)
2. **hours/ns**: Hours per nanosecond
3. **Performance (ns/day)**: Aggregate throughput
4. **Msteps/s**: Million steps per second
5. **GFlops**: Effective floating-point performance
6. **Wall time**: Total elapsed time

```bash
# Check performance at end of log
tail -50 md.log | grep Performance
```

## Key Takeaways
1. Use `-rerun` for free energy analysis on existing trajectories
2. Use `-maxh` to safely stop before queue time limits expire
3. SIGUSR1 gives the cleanest stop — use it for checkpointed shutdown
4. `-multidir` is more efficient than separate mdrun processes for multi-replica runs
5. `-reprod` is for debugging only — don't use in production
6. Always check ns/day in the .log to verify expected performance

## Connects To
- **Ch 6**: Long simulations and checkpoint management
- **Ch 11**: Performance tuning
- **Ch 13**: mdrun command reference
