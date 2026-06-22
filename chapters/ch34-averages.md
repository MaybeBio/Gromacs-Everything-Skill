# Chapter 34: Averages & Fluctuations

## Core Idea
Thermodynamic properties from MD are ensemble averages with statistical errors. GROMACS implements block averaging and fluctuation formulas for computing properties like heat capacity, compressibility, and thermal expansion from simulation trajectories.

## Formulae for Averaging
- **Mean**: ⟨A⟩ = (1/N) Σ A_i
- **Variance**: σ² = ⟨A²⟩ − ⟨A⟩²
- **Standard error**: SEM = σ/√N_eff (corrected for correlation)
- **Block averaging**: Divide trajectory into blocks, compute mean of block means, standard deviation of block means = standard error

## Key Fluctuation Formulas

### Heat Capacity (NVT)
C_V = (⟨E²⟩ − ⟨E⟩²) / (k_B T²)

### Isothermal Compressibility (NPT)
κ_T = (⟨V²⟩ − ⟨V⟩²) / (k_B T ⟨V⟩)

### Thermal Expansion Coefficient (NPT)
α = (⟨V·H⟩ − ⟨V⟩·⟨H⟩) / (k_B T² ⟨V⟩)

## Implementation in GROMACS
- Energy averages accumulated in blocks
- `gmx energy -f ener.edr -fluct_props` computes fluctuation properties
- Block size affects error estimates — check convergence with block size

## Key Takeaways
1. Block averaging is the standard method for error estimation
2. Raw standard deviation underestimates error (correlated data)
3. Fluctuation properties converge slowly — need long trajectories
4. Heat capacity from NVT, compressibility from NPT

## Connects To
- **Ch 32**: Analysis tools
- **Ch 33**: Implementation details
