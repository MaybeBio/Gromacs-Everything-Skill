# Chapter 21: Algorithms — Free Energy & Enhanced Sampling

## Core Idea
GROMACS supports multiple free energy methods: thermodynamic integration (TI), free energy perturbation (FEP), Bennett's acceptance ratio (BAR), and replica exchange. Energy minimization uses steepest descent (initial) and conjugate gradient or L-BFGS (refinement).

## Energy Minimization

| Method | Algorithm | Parallel | Use |
|---|---|---|---|
| Steepest descent | Gradient-based, robust | Yes | Initial relaxation |
| Conjugate gradient | Polak-Ribiere CG | Yes | Refinement near minimum |
| L-BFGS | Quasi-Newton | No (yet) | Fastest convergence |

- `emtol = 10.0 kJ/mol/nm` — stop when max force < tolerance
- `emstep = 0.01 nm` — initial step size
- CG + steep hybrid: `nstcgsteep = 1000` (do steep every 1000 CG steps)

## Free Energy Calculations

### Thermodynamic Integration (TI)
- ΔG = ∫₀¹ ⟨∂H/∂λ⟩_λ dλ
- Lambda: coupling parameter (0=initial, 1=final state)
- Soft-core potentials prevent endpoint singularities
- `sc-alpha = 0.5`, `sc-power = 1`, `sc-sigma = 0.3`

### Free Energy Perturbation (FEP)
- ΔG = −k_B T ln⟨exp(−ΔH/k_B T)⟩
- Forward and reverse estimates bracket the true value
- BAR combines both for optimal estimate

### Bennett's Acceptance Ratio (BAR)
- Optimal combination of forward and reverse FEP
- `gmx bar -f dhdl1.xvg dhdl2.xvg ... -g ener1.edr ener2.edr ... -temp 300`

### Alchemical Free Energy Workflow
1. Set `free-energy = yes` in .mdp
2. Define `couple-moltype` (which molecule decouples)
3. Set `couple-lambda0 = vdw-q`, `couple-lambda1 = none`
4. Define lambda schedule: `init-lambda`, `delta-lambda`
5. Run 20+ lambda windows in parallel with `-multidir`
6. Analyze with `gmx bar`

## Enhanced Sampling

### Replica Exchange
- Multiple replicas at different temperatures/hamiltonians
- Exchanges attempted every `nst-replex` steps
- Temperature: enhances sampling of high-barrier regions
- Hamiltonian: exchanges lambda values for alchemical transitions

### Essential Dynamics
- PCA-based sampling along collective eigenvectors
- Amplifies motions along essential subspace

### Expanded Ensemble
- Single simulation samples multiple states (lambda, temperature)
- Weight hopping or Wang-Landau for state weights
- Checkpoint issue: known problem in 2026.2

## Normal Mode Analysis
- Requires double precision compilation
- `integrator = nm` in .mdp
- Computes eigenvalues/eigenvectors of Hessian matrix
- Use for: vibrational analysis, entropy estimation, flexibility assessment

## Key Takeaways
1. Steepest descent → CG/L-BFGS: two-stage EM strategy
2. Soft-core potentials are essential for alchemical free energy
3. BAR/MBAR gives optimal free energy estimates
4. 20+ lambda windows typically needed for converged ΔG
5. Replica exchange helps overcome high energy barriers

## Connects To
- **Ch 25**: Soft-core potential details
- **Ch 28**: Pull code free energy methods
- **Ch 29**: AWH adaptive biasing
- **Ch 39**: Advanced free energy MDP parameters
