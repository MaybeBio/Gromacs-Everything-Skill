# GROMACS Patterns & Workflows

## Standard MD Simulation Workflow
**When to use**: Any MD simulation from scratch
**How**:
1. `gmx pdb2gmx -f protein.pdb -o processed.gro -p topol.top -water <water_model> -ff <forcefield>` — Generate topology
2. `gmx editconf -f processed.gro -o box.gro -c -d 1.0 -bt <dodecahedron|cubic|octahedron>` — Define box
3. `gmx solvate -cp box.gro -cs spc216.gro -o solv.gro -p topol.top` — Add solvent
4. `gmx grompp -f ions.mdp -c solv.gro -p topol.top -o ions.tpr` — Prepare for ion addition
5. `gmx genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -neutral` — Add ions
6. `gmx grompp -f em.mdp -c solv_ions.gro -p topol.top -o em.tpr` — Energy minimization input
7. `gmx mdrun -v -deffnm em` — Run EM
8. `gmx grompp -f nvt.mdp -c em.gro -p topol.top -o nvt.tpr` — NVT equilibration
9. `gmx mdrun -v -deffnm nvt` — Run NVT
10. `gmx grompp -f npt.mdp -c nvt.gro -p topol.top -o npt.tpr` — NPT equilibration
11. `gmx mdrun -v -deffnm npt` — Run NPT
12. `gmx grompp -f md.mdp -c npt.gro -p topol.top -o md.tpr` — Production
13. `gmx mdrun -v -deffnm md` — Run production
**Trade-offs**: Full workflow ensures proper equilibration. Skipping NVT/NPT can cause crashes.

## Force Field Selection
**When to use**: Choosing interaction parameters for a new system
**How**:
| Force Field | Best for | Water model | Notes |
|---|---|---|---|
| AMBER99SB-ILDN | Proteins (modern) | TIP3P | Widely validated |
| CHARMM36 | Proteins, lipids, nucleic acids | TIP3P | Best for lipids |
| GROMOS54A7 | Proteins | SPC | Historically GROMACS-native |
| OPLS-AA/L | Small molecules, liquids | TIP4P | Good for organic solvents |

**Trade-offs**: AMBER and CHARMM have more validation data. GROMOS is simpler. OPLS good for non-biological systems.

## Box Type Selection
**When to use**: Defining simulation cell
**How**:
| Box type | Volume vs cube | Use when |
|---|---|---|
| Cubic | 100% | Crystalline systems, simple setups |
| Rhombic dodecahedron | 71% | Spherical solutes (most proteins) |
| Truncated octahedron | 77% | Spherical solutes, slightly more expensive |

**Trade-offs**: Rhombic dodecahedron saves ~29% CPU vs cube for solvated proteins. Use `-bt dodecahedron` with editconf.

## Temperature Coupling
**When to use**: Different simulation phases
**How**:
| Thermostat | Phase | Ensemble | tau-t |
|---|---|---|---|
| V-rescale | NVT/NPT equilibration | Correct NVT | 0.1 ps |
| Nose-Hoover | NPT equilibration | Correct NVT | 1.0 ps |
| Berendsen | Fast pre-equilibration | Approximate NVT | 0.1 ps |
| SD integrator | Production | Correct NVT | 2.0 ps |

**Why it works**: Berendsen converges fast but wrong ensemble. V-rescale is correct and stable. Use Berendsen only for initial relaxation.

## Pressure Coupling
**When to use**: NPT simulation phases
**How**:
| Barostat | When | tau-p | compressibility |
|---|---|---|---|
| Berendsen | Fast box equilibration | 2.0 ps | 4.5e-5 bar⁻¹ |
| Parrinello-Rahman | Production NPT | 5.0 ps | 4.5e-5 bar⁻¹ |

**Why it works**: PR barostat requires stable box first. Always couple semisotropically for membranes (xy separate from z).

## Constraint Algorithms
**When to use**: Constraining bond lengths for larger timestep
**How**:
| Algorithm | Best for | Speed | Notes |
|---|---|---|---|
| LINCS | All bonds (default) | Fastest | Order 4 default, parallelizes well |
| SHAKE | Special cases | Slower | Iterative, not well parallelized |
| SETTLE | 3-atom water only | Fastest (analytical) | Auto-applied for rigid water |

**Decision**: Use `constraints=h-bonds` with 2 fs dt. Use `constraints=all-bonds` for >2 fs dt or virtual sites.

## PME Parameter Tuning
**When to use**: Setting electrostatic parameters
**How**:
- `rcoulomb = 1.0 nm` (standard), `rcoulomb = 0.9 nm` (for 4 fs dt with HMR)
- `fourierspacing = 0.12 nm` (standard), `0.10 nm` (high accuracy)
- `pme-order = 4` (standard), `6` (high accuracy)
- Grid auto-determined by GROMACS; check .log for grid dimensions

**Performance tip**: MTS (`mts = yes`, `mts-level2-forces = longrange-nonbonded`, `mts-level2-factor = 2`) evaluates PME every other step.

## Performance Optimization
**When to use**: Slow simulation
**How**:
1. Use single precision unless you need double
2. Use newest compiler (GCC recommended)
3. Enable GPU: `-DGMX_GPU=CUDA` (NVIDIA), `HIP` (AMD), `SYCL` (Intel)
4. Use `constraints=h-bonds` (not all-bonds) for GPU-resident mode
5. Rhombic dodecahedron box for spherical solutes
6. Enable MTS for PME-heavy systems
7. Tune PME ranks: `gmx tune_pme -s topol.tpr`
8. For multi-GPU: assign separate PP and PME ranks

**Threshold**: ~500 atoms/core minimum for efficient parallelization.

## Free Energy Calculation
**When to use**: Computing binding free energies, solvation free energies
**How**:
1. Set `free-energy = yes` in .mdp
2. Define `couple-moltype`, `couple-lambda0/1` for which molecule couples
3. Choose `couple-intramol` for internal interactions
4. Set lambda schedule: `calc-lambda-neighbors = 1`, `nstdhdl = 10`
5. Use soft-core potentials: `sc-alpha = 0.5`, `sc-power = 1`, `sc-sigma = 0.3`
6. Run multiple lambda windows: `init-lambda = 0.0`, `delta-lambda = 0.05`
7. Analyze with `gmx bar` for MBAR free energy
**Trade-offs**: 20+ lambda windows needed for convergence. BAR/MBAR more efficient than TI.

## Enhanced Sampling Decision Tree
| Method | When to use | Key parameter |
|---|---|---|
| Umbrella sampling | Along 1-2 reaction coordinates | `pull = yes`, define pull groups |
| AWH | Automatic adaptive sampling | `awh = yes`, define coordinate |
| Replica exchange | Temperature/Hamiltonian | `replica-exchange = yes` |
| Expanded ensemble | Multiple lambda states | `expanded-ensemble = yes` |
| Metadynamics (PLUMED) | Complex CV space | PLUMED input file |
| Colvars | Standard CV library | Colvars config file |

## System Preparation Checklist
**When to use**: Starting any new simulation
**How**:
1. Check PDB: remove crystallographic waters, ligands, fix missing residues
2. Choose force field: match protonation states at target pH
3. Choose water model: must match force field (e.g., CHARMM36→TIP3P, GROMOS→SPC)
4. Box: protein-box distance ≥ 1.0 nm, use dodecahedron
5. Ions: neutralize system + physiological concentration (~0.15 M NaCl)
6. EM: `emtol = 1000 kJ/mol/nm` (loose) then `10.0 kJ/mol/nm` (tight)
7. NVT: 100 ps, `tcoupl = v-rescale`, `ref-t = 300K`
8. NPT: 100-500 ps, `pcoupl = Parrinello-Rahman`, check density stabilizes
9. Production: remove position restraints, use appropriate thermostat

## Common Error Diagnosis
| Symptom | Likely cause | Fix |
|---|---|---|
| LINCS warnings | Bad geometry, too large timestep | Reduce dt, check starting structure |
| PME load imbalance | Wrong PME rank count | `gmx tune_pme` |
| "System has non-zero total charge" | Missing ions | Add counterions with `gmx genion -neutral` |
| "Atom ... not found in ..." | Wrong topology | Check force field directory, `GMXLIB` env var |
| Temperature coupling groups | Wrong tc-grps | Use `tc-grps = Protein Water_and_ions` |

---

## Tutorial Workflow Patterns (Lemkul Tutorials, integrated into ch05/ch08/ch16/ch21/ch25/ch28)

### Protein-Ligand Parametrization (CGenFF)
**When to use**: Simulating a protein with a non-standard small molecule ligand
**How**:
1. Split PDB into protein-only and ligand-only files
2. Run `pdb2gmx` on protein alone
3. Add H to ligand in Avogadro → export as Sybyl .mol2
4. Fix .mol2: consistent residue names, ascending bond order (`sort_mol2_bonds.pl`)
5. Submit to CGenFF server → download .str → convert to GROMACS .itp
6. Edit .itp: remove standalone parts, embed bonded parameters, rename moleculetype
7. Merge ligand coordinates into protein .gro; update atom count
8. Add `#include "ligand.itp"` BEFORE `[ moleculetype ]` in topol.top
9. Add ligand to `[ molecules ]` section
10. Generate position restraints for ligand non-H atoms with `genrestr`
**Trade-offs**: CGenFF penalty scores tell you parameter quality. ATB (GROMOS), Antechamber (AMBER/GAFF), LigParGen (OPLS) are alternatives.

### Membrane Protein Construction (InflateGRO)
**When to use**: Embedding a protein in a lipid bilayer with non-standard force field
**How**:
1. Orient protein and membrane in same coordinate frame (`editconf -box`)
2. Concatenate protein + membrane .gro files
3. Inflate lipids (scale factor 4) to create space around protein
4. Delete overlapping lipids (cutoff radius in Å)
5. EM inflated system
6. Iteratively shrink (factor 0.95, ~26 rounds) until area per lipid converges
7. Solvate with water
8. Remove water from membrane core with `water_deletor.pl`
9. Add neutralizing ions
10. Equilibrate with semi-isotropic pressure coupling
**Trade-offs**: InflateGRO is deprecated for CHARMM36 systems — use CHARMM-GUI instead. Manual approach teaches force field internals.

### Free Energy: vdW Decoupling (Methane in Water)
**When to use**: Computing the vdW contribution to solvation free energy
**How**:
1. Prepare system with molecule to be decoupled
2. Set `free-energy=yes`, `couple-moltype`, `couple-lambda0=vdw-q`, `couple-lambda1=none`
3. Define 20+ lambda windows (vdw-lambdas = 0.00 0.05 ... 1.00)
4. Set soft-core: sc-alpha=0.5, sc-power=1, sc-sigma=0.3
5. For each lambda: EM → NVT → NPT → Production MD
6. Extract ∂H/∂λ with `nstdhdl=10`
7. Compute ΔG with `gmx bar`
**Trade-offs**: BAR is more efficient than TI but requires phase space overlap between windows. Pure vdW transformation (charges already zero) is the simplest case.

### Free Energy: Hydration (Two-Stage Decoupling)
**When to use**: Computing complete ΔG_hydr (Coulomb + vdW)
**How**:
1. Stage 1 — Coulomb discharge: `couple-lambda0=q`, keep vdW on, 20+ windows
2. Stage 2 — vdW decoupling: charges already zero, 20+ windows
3. ΔG_hydr = ΔG_coulomb + ΔG_vdW
4. Each window starts from previous window's equilibrated output
**Trade-offs**: Order is mandatory (charge first, vdW second). Reversing causes simulation blow-up. 42 total simulation sets for complete calculation.

### Umbrella Sampling (PMF via Pull Code)
**When to use**: Computing free energy profile along 1–2 reaction coordinates
**How**:
1. Pulling simulation: `pull=yes`, `pull-coord1-type=umbrella`, `pull-coord1-rate=0.01` (generate configurations)
2. Extract frames at desired COM spacings (e.g., 0.1 nm intervals)
3. Umbrella windows: `pull-coord1-rate=0.0`, `pull-coord1-k=1000` (fixed restraint per window)
4. Run 5–20 ns production per window
5. WHAM analysis: `gmx wham` reconstructs unbiased PMF
6. ΔG_bind = PMF_plateau − PMF_minimum
**Trade-offs**: Window spacing must allow histogram overlap. Force constant trades localization vs. sampling width.

### Virtual Site Construction (Manual Topology)
**When to use**: Enabling dt ≥ 4 fs by eliminating high-frequency bond vibrations
**How**:
1. Define real atoms (parent atoms) in topology
2. Construct virtual sites via function types (2=linear, 3=planar, 4=tetrahedral)
3. Transfer mass from vsites to parent atoms
4. Set `constraints=all-bonds` in .mdp
5. Use dt = 0.004 or 0.005 ps
**Trade-offs**: HMR (mass repartitioning) is simpler but only reaches 4 fs. Virtual sites reach 5 fs but require manual topology editing.
