# Chapter 25: Advanced Interactions

## Core Idea
Advanced interaction features include soft-core potentials for free energy, virtual interaction sites for larger timesteps, LJ-PME for long-range dispersion, and force field organization. Soft-core prevents endpoint singularities in alchemical transformations; virtual sites enable 5 fs timesteps.

## Soft-Core Potentials
Prevent singularities when atoms appear/disappear in alchemical free energy calculations:

V_sc(r) = (1−λ)·V_A(r_A) + λ·V_B(r_B)

with modified distance: r_A = (α·σ_A⁶·λ^p + r⁶)^(1/6)

- **sc-alpha = 0.5**: Soft-core α parameter
- **sc-power = 1**: Soft-core power p
- **sc-sigma = 0.3 nm**: Soft-core σ (or read from atom types)
- **sc-coul = no**: Apply soft-core to Coulomb (usually not needed)

### Lambda Dynamics
- Lambda treated as dynamical variable
- Expanded ensemble or Wang-Landau for lambda sampling

## Virtual Interaction Sites
Massless sites constructed from real atom positions:

| Type | Construction | Use |
|---|---|---|
| 2 | Linear combination of 2 atoms | CH₂ groups |
| 3 | Linear combination of 3 atoms | CH₃ groups, water |
| 3fd | Fixed distance from 3 atoms | Aromatic H |
| 3fad | Fixed angle + distance | Lone pairs |
| 3out | Out-of-plane from 3 atoms | Out-of-plane H |
| 4fdn | Fixed distance from 4 atoms | TIP5P water |
| N | Linear from N atoms | General |

- Enable 5 fs timestep with `constraints=all-bonds`
- Eliminate fastest C-H vibrations
- Required for TIP4P and TIP5P water models

## Long-Range Electrostatics

### PME Detail
1. Real-space sum: short-range Coulomb + correction
2. Reciprocal sum: charge grid → FFT → potential → gradient → forces
3. Self-energy correction
4. Dipole correction for charged systems

### Ewald Sum
- Direct: erfc(βr)/r
- Reciprocal: exp(−k²/4β²)/k²
- Optimal β balances direct/reciprocal cost

### LJ-PME
- Mesh-based long-range dispersion
- Analogous to PME for LJ interactions
- `vdwtype = PME` enables LJ-PME
- Better than simple cutoff + dispersion correction

## Force Field Organization
GROMACS force fields in `share/gromacs/top/`:
```
amber99sb-ildn.ff/
├── forcefield.itp       # [defaults], [atomtypes], combination rules
├── forcefield.doc       # Documentation
├── atomtypes.atp        # Atom type definitions
├── ffnonbonded.itp      # Non-bonded parameters
├── ffbonded.itp         # Bonded parameters
├── aminoacids.rtp       # Standard amino acids
├── aminoacids.c.tdb     # C-terminal patches
├── aminoacids.n.tdb     # N-terminal patches
├── ions.itp             # Ion parameters
├── spc.itp / tip3p.itp  # Water model
└── *.hdb                # Hydrogen database
```

## Key Takeaways
1. Soft-core is essential for alchemical free energy — prevents λ=0,1 singularities
2. Virtual sites enable 5 fs timesteps by eliminating C-H vibrations
3. LJ-PME provides better long-range dispersion than cutoff + correction
4. Force fields are self-contained .ff directories
5. sc-alpha=0.5, sc-power=1, sc-sigma=0.3 are standard soft-core values

## Connects To
- **Ch 21**: Free energy calculations
- **Ch 23**: Non-bonded interactions
- **Ch 26**: Topology structure
- **Ch 30**: QM/MM, NNP
