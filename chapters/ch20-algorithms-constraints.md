# Chapter 20: Algorithms — Constraints & Coupling

## Core Idea
GROMACS provides multiple thermostat and barostat options for different simulation phases, plus three constraint algorithms (LINCS, SHAKE, SETTLE) for bond length constraints. The key principle: use V-rescale + Parrinello-Rahman for production, Berendsen for fast initial relaxation.

## Temperature Coupling

### Berendsen Thermostat
- **How**: Velocities scaled by λ = [1 + Δt/τ_T (T₀/T − 1)]^½
- **When**: Fast pre-equilibration
- **Why not production**: Does NOT produce correct canonical ensemble
- **tau_t**: 0.1 ps typical

### V-rescale (Velocity Rescaling)
- **How**: Berendsen-like scaling with stochastic correction term
- **When**: Equilibration and production (NVT or NPT)
- **Why**: Produces CORRECT canonical ensemble; more robust than Nose-Hoover
- **tau_t**: 0.1 ps (equilibration), 1.0-2.0 ps (production)

### Nose-Hoover
- **How**: Extended system with thermal reservoir, oscillatory relaxation
- **When**: Production with velocity Verlet integrator
- **Why**: Correct ensemble; can oscillate with poor settings
- **nh-chain-length**: 10 (default), longer chain = better but more expensive

### Andersen
- **How**: Stochastic velocity reassignment at random times
- **When**: Simple thermostat, rarely used in production

### Stochastic Dynamics (SD)
- **How**: Langevin equation: m·dv/dt = F − m·γ·v + R(t)
- **When**: Implicit solvent, coarse-grained simulations
- **tau_t**: 2 ps gives friction < water internal friction

## Pressure Coupling

### Berendsen Barostat
- **How**: Coordinates and box vectors scaled toward target pressure
- **When**: Fast initial box relaxation
- **Why not production**: Does NOT produce correct NPT ensemble
- **tau_p**: 2 ps typical
- **compressibility**: 4.5e-5 bar⁻¹ (water)

### Parrinello-Rahman
- **How**: Extended system with box vectors as dynamical variables
- **When**: Production NPT
- **Why**: Correct NPT ensemble; can oscillate — ensure stable box first
- **tau_p**: 5 ps typical
- **pcoupltype**: isotropic (solution), semiisotropic (membrane), anisotropic, surface-tension

### MTTK (Martyna-Tuckerman-Tobias-Klein)
- **How**: Fully correct integration of NPT equations
- **When**: When fully correct NPT ensemble is critical

## Constraint Algorithms

### LINCS (LINear Constraint Solver)
- **Default**, fastest, well-parallelized
- **lincs-order**: 4 (expansion order for coupling matrix)
- **lincs-iter**: 1 (correction iterations)
- **lincs-warnangle**: 30° (warning threshold for bond rotation)

### SHAKE
- Iterative Lagrange multiplier method
- Slower than LINCS, less parallel
- Uses relative tolerance: `shake-tol = 0.0001`

### SETTLE
- Analytical solution for rigid 3-atom water molecules
- Fastest possible, auto-applied for rigid water models
- No user parameters needed

## Simulated Annealing
- Time-dependent reference temperature via annealing groups
- Multiple annealing groups with different schedules
- Use `annealing = single` or `periodic`

## Key Takeaways
1. V-rescale for thermostat, Parrinello-Rahman for barostat in production
2. Berendsen methods are for equilibration only — wrong ensemble
3. LINCS is the default and recommended constraint algorithm
4. SETTLE is automatic for rigid water — no configuration needed
5. SD integrator with tau_t=2 ps is a good Langevin thermostat

## Connects To
- **Ch 9**: MDP coupling parameters
- **Ch 19**: MD integrator details
