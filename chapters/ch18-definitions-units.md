# Chapter 18: Definitions & Units

## Core Idea
GROMACS uses a consistent unit system (nm, ps, kJ/mol, K, e, bar) that simplifies MD calculations. Understanding the notation conventions and precision considerations is essential for correct parameter specification.

## MD Units
| Quantity | Unit | Value |
|---|---|---|
| Length | nm | 1 nm = 10 Å |
| Time | ps | 1 ps = 10⁻¹² s |
| Mass | amu (u) | 1.66054×10⁻²⁷ kg |
| Energy | kJ/mol | 1 kJ/mol ≈ 0.239 kcal/mol |
| Temperature | K | Boltzmann constant implicit |
| Charge | e | 1.60218×10⁻¹⁹ C |
| Pressure | bar | 1 bar = 10⁵ Pa |
| Force | kJ/mol/nm | 1 kJ/mol/nm ≈ 16.02 pN |
| Velocity | nm/ps | 1 nm/ps = 1000 m/s |

## Key Constants
- **Electrostatic constant f**: 1/(4πε₀) = 138.935458 kJ mol⁻¹ nm e⁻²
- **Boltzmann constant k_B**: Implicitly included via temperature
- **Gas constant R**: 8.31451×10⁻³ kJ mol⁻¹ K⁻¹

## Notation
- **Bold**: Vector quantities (r, v, F)
- **Italic**: Scalar quantities (r, V, T)
- **Subscript i,j**: Particle indices
- **r_ij**: Vector from particle j to particle i
- **r_ij**: Distance between particles i and j

## Precision
- **Single precision** (default): 7 significant digits, sufficient for MD
- **Double precision**: 15 significant digits, required for normal mode analysis, energy minimization to very high accuracy
- **Mixed precision**: GROMACS internally uses mixed precision where beneficial

## Reduced Units
Used in some specialized contexts where quantities are expressed in dimensionless form using Lennard-Jones parameters σ and ε as base units.

## Key Takeaways
1. Always use consistent units: nm, ps, kJ/mol, K, e, bar
2. The electrostatic constant f = 138.935458 is used internally
3. Single precision is sufficient for 99% of MD simulations
4. Use double precision only for normal mode analysis

## Connects To
- **Ch 9**: MDP parameters (use these units)
- **Ch 23**: Non-bonded interaction formulas
