# Chapter 29: Special Topics — AWH & Enhanced Sampling

## Core Idea
Accelerated Weight Histogram (AWH) is an adaptive biasing method that automatically computes the free energy along user-defined reaction coordinates. It requires no post-processing and converges faster than umbrella sampling for 1-3 CVs. Additional special methods include enforced rotation, electric fields, and computational electrophysiology.

## Adaptive Biasing with AWH

### How AWH Works
1. Define reaction coordinates (1-3 CVs)
2. AWH builds a bias potential that flattens the free energy landscape
3. Bias is updated adaptively using the weight histogram
4. Free energy = converged bias potential (inverted)
5. No post-processing needed — PMF comes directly from simulation

### AWH Setup (.mdp)
```ini
awh                     = yes
awh-nstout              = 100000
awh-nstsample           = 10
awh-nbias               = 1
awh1-ndim               = 1
awh1-dim1-coord-provider = pull
awh1-dim1-coord-index   = 1
awh1-dim1-start         = 0.0
awh1-dim1-end           = 3.0
awh1-dim1-force-constant = 0.0
awh1-dim1-cover-diameter = 0.2
awh1-dim1-diffusion     = 1e-5
awh1-share-group        = 0
awh1-target             = constant
awh1-error-init         = 10.0
awh1-growth             = exp-linear
awh1-equilibrate-histogram = no
```

### AWH Key Parameters
| Parameter | Purpose |
|---|---|
| `awh1-dim1-cover-diameter` | Gaussian width for sampling |
| `awh1-dim1-diffusion` | Estimated diffusion constant (controls update rate) |
| `awh1-error-init` | Initial bias uncertainty (kJ/mol) |
| `awh1-growth` | Bias growth strategy |
| `awh-share-group` | Share bias across multiple walkers |
| `awh-target` | Target distribution shape |

### AWH vs Umbrella Sampling
| Feature | AWH | Umbrella |
|---|---|---|
| Windows needed | 1 simulation | 20-40 |
| Post-processing | None (built-in) | WHAM/gmx wham |
| CVs | 1-3 | Typically 1-2 |
| Convergence | Adaptive, automatic | Manual inspection |
| Parallel efficiency | Multiple walkers | Independent windows |

## Enforced Rotation
- Apply torque to rotate groups at constant angular velocity
- Useful for: ATP synthase, motor proteins, viscosity from rotation
- `rotation = yes` in .mdp

## Electric Fields
- Apply external electric field: `E-x`, `E-y`, `E-z` (V/nm)
- Pulsed/oscillating: `E-xt = E0 * cos(omega * t)`
- Used for: ion transport, electroporation, dielectric response

## Computational Electrophysiology (CompEL)
- Double-membrane setup with ion concentration imbalance
- Creates transmembrane potential
- Counts ions traversing the channel
- Computes current from ion flux

## PMF Using Free Energy Code
- Alternative to pulling: use alchemical code for PMF
- Decouple molecule from environment along coordinate
- Can be more accurate for complex transformations

## Key Takeaways
1. AWH is the easiest PMF method — 1 simulation, no post-processing
2. AWH converges adaptively — monitor bias convergence, not simulation length
3. Diffusion constant estimate is the most important AWH parameter
4. CompEL enables direct electrophysiology simulations
5. AWH shares bias across multiple walkers for faster convergence

## Connects To
- **Ch 28**: Umbrella sampling (alternative PMF method)
- **Ch 30**: Colvars and PLUMED (alternative enhanced sampling)
- **Ch 39**: Advanced AWH MDP parameters
