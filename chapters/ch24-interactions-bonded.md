# Chapter 24: Bonded Interactions

## Core Idea
Bonded interactions are defined by the molecular topology and include bond stretching, angle bending, proper/improper dihedrals, and restraints. GROMACS supports multiple functional forms for each, matching different force field conventions.

## Bond Stretching
| Form | Potential | Used by |
|---|---|---|
| Harmonic | ½k_b(b − b₀)² | Most force fields |
| Fourth power | ¼k_b(b² − b₀²)² | GROMOS-96 |
| Morse | D_e[1 − exp(−β(b − b₀))]² | Special cases |
| Cubic | ½k_b(b − b₀)² + ½k_b·k_cub(b − b₀)³ | OPLS |
| FENE | −½k_b·b₀²·ln[1−(b/b₀)²] | Polymers |
| Restrained | Harmonic with flat bottom | ReaxFF |

## Angle Bending
| Form | Potential | Used by |
|---|---|---|
| Harmonic | ½k_θ(θ − θ₀)² | AMBER, CHARMM, OPLS |
| GROMOS-96 | ½k_θ(cos θ − cos θ₀)² | GROMOS |
| Cosine-based | ½k_θ(cos θ − cos θ₀)² | GROMOS variant |
| Urey-Bradley | Harmonic + 1-3 distance term | CHARMM |
| Linear | k_θ(1 − cos θ) | Special |
| Restrained | Harmonic with flat bottom | ReaxFF |

## Dihedrals

### Proper Dihedrals (4-atom torsion)
| Type | Form | Used by |
|---|---|---|
| Periodic | k_φ[1 + cos(nφ − φ_s)] | AMBER, CHARMM, OPLS |
| Ryckaert-Bellemans | Σ_n C_n·cosⁿ(ψ) with ψ = φ − 180° | GROMOS, OPLS |
| Fourier | ½[F₁(1+cos φ) + F₂(1−cos 2φ) + F₃(1+cos 3φ) + F₄(1−cos 4φ)] | OPLS |
| CMAP | Grid-based cross-term for φ/ψ backbone | CHARMM |
| Multiple periodic | Sum of periodic terms | Multiplicity |

### Improper Dihedrals (out-of-plane)
| Type | Purpose |
|---|---|
| Harmonic | Maintain planar groups (rings) |
| Periodic | Maintain chirality |

## 1-4 Interactions
- Non-bonded interactions between atoms separated by 3 bonds
- Scaled: `fudgeLJ` (default 1.0), `fudgeQQ` (default 1.0)
- Listed in `[pairs]` section of topology

## Restraints

### Position Restraints
- Harmonic potential: ½k_pr(r − R_ref)²
- Used during equilibration to keep solute near starting position
- `-DPOSRES` define activates them

### Distance Restraints
- Time-averaged or instantaneous
- Upper/lower bounds: flat-bottom harmonic well
- NMR-derived restraints

### Angle Restraints
- Similar to distance restraints for angles
- Used with NMR data

### Orientation Restraints
- Restrain orientation of vectors relative to reference
- NMR order parameter restraints (RDC)

### Dihedral Restraints
- Restrain dihedral angles to target values
- Used with NMR J-coupling data

## Key Takeaways
1. Harmonic bonds/angles are the default for most force fields
2. Ryckaert-Bellemans dihedrals are used by GROMOS and OPLS
3. CMAP correction is CHARMM-specific for backbone φ/ψ
4. Position restraints are essential during equilibration
5. 1-4 interactions are scaled separately from other non-bonded

## Connects To
- **Ch 23**: Non-bonded interactions
- **Ch 25**: Virtual sites and restraints implementation
- **Ch 26**: Topology file format
