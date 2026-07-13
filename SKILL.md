---
name: gromacs
description: "Knowledge base from \"GROMACS Documentation Release 2026.2\" by the GROMACS development team, enriched with Justin A. Lemkul's practical tutorial workflows (CC-BY 4.0) and Chinese manual theory notes. Use when applying GROMACS for molecular dynamics simulations: system preparation, force field selection, ligand parametrization, .mdp parameter tuning, mdrun performance optimization, free energy calculations (alchemical + umbrella sampling), membrane protein simulation, virtual sites, trajectory analysis, and troubleshooting."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# GROMACS Documentation Release 2026.2
**Author**: GROMACS development team | **Enriched with**: Justin A. Lemkul tutorials (CC-BY 4.0), Chinese manual (GROMACS 2019.6 中文译版) | **Chapters**: 40 | **Generated**: 2026-07-13

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `PME`, `free energy`, `CGenFF`, `umbrella sampling`, `virtual sites`, `membrane simulation`, or any indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch05`, `ch16`, etc.; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index
- **Troubleshoot** — describe your error or problem; I match it to known issues and solutions
- **Practical workflow** — ask for topic + "workflow" (e.g., "free energy workflow") for step-by-step tutorial examples

When you ask about a topic not covered in Core Frameworks below, I will read the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### The GROMACS Simulation Pipeline
Use this **6-phase mental model** for every simulation project:

1. **System Preparation** — `pdb2gmx` (topology) → `editconf` (box) → `solvate` (solvent) → `genion` (ions)
2. **Energy Minimization** — Remove steric clashes with `steep` integrator before any dynamics
3. **NVT Equilibration** — Stabilize temperature with `v-rescale` thermostat, position restraints on solute
4. **NPT Equilibration** — Stabilize density with `Parrinello-Rahman` barostat, release restraints gradually
5. **Production MD** — Remove restraints, run with appropriate thermostat/barostat
6. **Analysis** — `trjconv -pbc mol -center` first, then `rmsd`, `rmsf`, `gyrate`, `hbond`, `energy`, `cluster`, `rdf`, `msd`

### Force Field Selection Heuristic
- **Proteins (general)**: AMBER99SB-ILDN with TIP3P water — most validated
- **Proteins + Lipids**: CHARMM36 with TIP3P water — best lipid parameters
- **Simple/united-atom**: GROMOS54A7 with SPC water — computationally lighter
- **Small organic molecules**: OPLS-AA/L — good liquid properties
- **Rule**: Water model must match force field. Never mix.

### Thermostat/Barostat Decision Matrix
| Phase | Thermostat | Barostat | Why |
|---|---|---|---|
| EM | none | none | Minimization, no dynamics |
| NVT (initial) | Berendsen, tau-t=0.1 | none | Fast convergence, wrong ensemble OK |
| NVT (equilibration) | V-rescale, tau-t=0.1 | none | Correct NVT ensemble |
| NPT (equilibration) | V-rescale, tau-t=0.1 | Berendsen, tau-p=2.0 | Fast box relaxation |
| NPT (production) | V-rescale or Nose-Hoover | Parrinello-Rahman, tau-p=5.0 | Correct NPT ensemble |
| SD production | SD integrator, tau-t=2.0 | Parrinello-Rahman | Langevin, correct NVT |

### Timestep Selection
- **1 fs**: No constraints, flexible water — rarely needed
- **2 fs**: `constraints=h-bonds` — **standard, use this**
- **4 fs**: HMR (`mass-repartition-factor`), `constraints=h-bonds` — with caution
- **5 fs**: Virtual sites, `constraints=all-bonds` — specialized

### PME Tuning Rule
**Set rcoulomb = rvdw = rlist, fourierspacing = 0.12 nm, pme-order = 4.**
- This is the standard that works for 99% of simulations.
- For GPU-resident mode: use `constraints=h-bonds` (not all-bonds).
- For PME-heavy systems: enable MTS (`mts=yes`, `mts-level2-forces=longrange-nonbonded`, `mts-level2-factor=2`).

### Box Selection Principle
**Use a rhombic dodecahedron (`-bt dodecahedron`) for all spherical solutes.**
- Saves 29% solvent vs cubic box at same image distance.
- Use cubic only for crystalline systems or simple setups.
- Use `-c -d 1.0` (center molecule, 1.0 nm minimum distance to box edge).

### Performance Optimization Priority
1. **Single precision** (default) — double only for normal mode analysis
2. **New GCC compiler** — use latest version
3. **GPU acceleration** — `-DGMX_GPU=CUDA` (NVIDIA) or `HIP` (AMD)
4. **GPU-resident mode** — requires `constraints=h-bonds`, verlet scheme
5. **Rhombic dodecahedron box** — 29% fewer solvent atoms
6. **MTS** — evaluate PME every 2nd step
7. **PME rank tuning** — `gmx tune_pme -s topol.tpr`
8. **Domain decomposition** — ~500 atoms/core minimum

### Free Energy Calculation Strategy
- **Alchemical (TI/FEP)**: For binding/solvation free energies. Always **discharge Coulomb first**, then decouple vdW. Use soft-core (`sc-alpha=0.5`, `sc-power=1`, `sc-sigma=0.3`), 20+ lambda windows (Δλ≈0.05), analyze with BAR/MBAR. Two-stage hydration FE: ΔG_hydr = ΔG_coul + ΔG_vdW.
- **Umbrella sampling (pull code)**: For PMF along 1-2 coordinates. Pulling→extract frames→umbrella windows→WHAM. Verify histogram overlap. `pull-coord1-k ≈ 1000`, window spacing ~0.1 nm.
- **AWH**: Adaptive, automatic. Best for 1-3 CVs. No post-processing needed.
- **Expanded ensemble**: For sampling multiple lambda states in one simulation.
- **Replica exchange**: For overcoming energy barriers. Temperature or Hamiltonian exchange.

### Ligand Parametrization Decision Tree
- **CHARMM force field**: CGenFF server → penalty scores <10 = reliable, 10–50 = validate, >50 = reparametrize
- **AMBER/GAFF**: Antechamber + acpype
- **GROMOS**: ATB (Automated Topology Builder)
- **OPLS**: LigParGen (Jorgensen group)
- **Rule**: `#include "ligand.itp"` MUST appear after parent force field include and before first `[ moleculetype ]`

### Membrane Simulation Rules
- **Semi-isotropic pressure coupling** mandatory for bilayers: `pcoupltype = semiisotropic`
- **Separate COM motion removal**: group membrane+protein separately from water+ions
- **Temperature coupling**: `tc-grps = Protein_Lipid Water_and_ions` — never couple lipids or ions separately
- **Simulate above lipid Tm**: DPPC ≥ 323 K (Tm = 315 K), DMPC ≥ 297 K, POPC ≥ 271 K
- **Area per lipid validation**: DPPC experimental ~62–64 Å²

### Virtual Sites for Large Timesteps
- Transfer mass from eliminated atoms to parent atoms
- Function types: 2=linear, 3=planar, 4=tetrahedral
- With vsites: `constraints=all-bonds`, dt=5 fs
- HMR (easier alternative): dt=4 fs with `constraints=h-bonds`

### Constraint Algorithm Selection
- **LINCS** (default): Fast, parallelizable, order 4. Use for all bonds.
- **SHAKE**: Iterative, slower. Use only when LINCS fails.
- **SETTLE**: Analytical for 3-atom water. Auto-applied for rigid water models.

### Error Diagnosis Heuristic
- **"LINCS warnings"** → Bad geometry or too large dt. Check starting structure, reduce dt.
- **"non-zero total charge"** → Missing counterions. Use `gmx genion -neutral`.
- **"Atom ... not found in ..."** → Wrong topology/force field. Check `GMXLIB`.
- **Temperature drifting** → Check thermostat (PR barostat + Nose-Hoover need proper coupling).
- **PME load imbalance** → Wrong PME rank count. Use `gmx tune_pme`.
- **Neighbor list overflow** → Increase rlist buffer or set `verlet-buffer-tolerance=-1`.
- **Segfault at start** → Wrong `GMX_SIMD` for CPU architecture.
- **WHAM fails to converge** → No histogram overlap between umbrella windows. Decrease spacing or increase k.

---

## Chapter Index

**All 40 chapters include comprehensive Chinese terminology, concepts, and formulas from GROMACS 2019.6 中文译版.**

| # | Title | Key Content |
|---|-------|-------------|
| [ch01](chapters/ch01-downloads.md) | Downloads | Source code, regression tests |
| [ch02](chapters/ch02-installation.md) | Installation Guide | CMake, compilers, FFTW, GPU backends |
| [ch03](chapters/ch03-known-issues.md) | Known Issues | ROCm, GCC POWER9, oneAPI, SVE |
| [ch04](chapters/ch04-getting-started.md) | Getting Started | Flow chart, file types, environment |
| [ch05](chapters/ch05-system-preparation.md) | System Preparation [+tutorial] | Full pipeline, **worked example: Lysozyme 12-step workflow** (Lemkul) |
| [ch06](chapters/ch06-long-simulations.md) | Managing Long Simulations | Checkpoints, reproducibility, restarts |
| [ch07](chapters/ch07-faqs.md) | FAQs | Installation, prep, methodology, analysis |
| [ch08](chapters/ch08-force-fields-overview.md) | Force Fields [+tutorial] | AMBER/CHARMM/GROMOS/OPLS, **ligand parametrization (CGenFF)**, force field modification |
| [ch09](chapters/ch09-mdp-options.md) | MDP Parameters | Integrators, dt, cutoffs, coupling, constraints |
| [ch10](chapters/ch10-mdrun-features.md) | Useful mdrun Features | Re-run, multi-sim, reproducibility |
| [ch11](chapters/ch11-performance-tuning.md) | Performance Tuning | GPU, MPI, domain decomposition, MTS |
| [ch12](chapters/ch12-common-errors.md) | Common Errors [+tutorial] | pdb2gmx/grompp/mdrun diagnostics, **membrane/ligand/FE/umbrella troubleshooting** |
| [ch13](chapters/ch13-command-reference.md) | Command-Line Reference | All gmx tools with flags and usage |
| [ch14](chapters/ch14-terminology.md) | Terminology | MDP option glossary, GROMACS terms |
| [ch15](chapters/ch15-validation.md) | Validation | Experimental features, pending validation |
| [ch16](chapters/ch16-how-to-guides.md) | How-To Guides [+tutorial] | Solvation, **membrane protein (InflateGRO)**, parameterization, **biphasic systems** |
| [ch17](chapters/ch17-preface-introduction.md) | Preface & Introduction | Citation, free software, MD intro, Chinese terminology |
| [ch18](chapters/ch18-definitions-units.md) | Definitions & Units | Notation, MD units, precision |
| [ch19](chapters/ch19-algorithms-pbc.md) | Algorithms: PBC & MD | Box types, neighbor searching, integrators |
| [ch20](chapters/ch20-algorithms-constraints.md) | Algorithms: Constraints & Coupling | LINCS/SHAKE/SETTLE, thermostats, barostats |
| [ch21](chapters/ch21-algorithms-free-energy.md) | Algorithms: Free Energy [+tutorial] | TI/FEP/BAR/REMD/EM, **practical FE workflows** |
| [ch22](chapters/ch22-algorithms-parallel.md) | Algorithms: Parallelization | Domain decomposition, GPU, load balancing |
| [ch23](chapters/ch23-interactions-nonbonded.md) | Non-bonded Interactions | LJ/Buckingham/Coulomb/PME/cutoffs |
| [ch24](chapters/ch24-interactions-bonded.md) | Bonded Interactions | Bonds, angles, dihedrals, restraints |
| [ch25](chapters/ch25-interactions-advanced.md) | Advanced Interactions [+tutorial] | Soft-core, LJ-PME, **virtual sites construction (CO₂)** |
| [ch26](chapters/ch26-topologies.md) | Topologies | .top/.itp structure, force field files |
| [ch27](chapters/ch27-file-formats.md) | File Formats | .gro, .pdb, .tpr, .xtc, .edr, .cpt, .ndx |
| [ch28](chapters/ch28-special-free-energy.md) | Special: Free Energy & Pulling [+tutorial] | PMF, **umbrella sampling workflow (4 steps: pull→extract→windows→WHAM)** |
| [ch29](chapters/ch29-special-awh.md) | Special: AWH & Enhanced Sampling | AWH, enforced rotation, electrophysiology |
| [ch30](chapters/ch30-special-advanced.md) | Special: Advanced Methods | QM/MM, PLUMED, Colvars, NNP, FMM |
| [ch31](chapters/ch31-run-parameters.md) | Run Parameters & Programs | File types, program workflow |
| [ch32](chapters/ch32-analysis.md) | Analysis [+tutorial] | RMSD/RMSF/RDF/MSD/hbond/clustering, **analysis workflow** |
| [ch33](chapters/ch33-implementation.md) | Implementation Details | Virial, optimizations |
| [ch34](chapters/ch34-averages.md) | Averages & Fluctuations | Formulae, implementation |
| [ch35](chapters/ch35-bibliography.md) | Bibliography | Key references |
| [ch36](chapters/ch36-developer-guide.md) | Developer Guide | Contributing, coding standards |
| [ch37](chapters/ch37-doxygen.md) | Doxygen Documentation | API reference |
| [ch38](chapters/ch38-release-notes.md) | Release Notes | 2026.2 changes |
| [ch39](chapters/ch39-mdp-advanced.md) | Advanced MDP Parameters | Free energy, AWH, pull, expanded ensemble |
| [ch40](chapters/ch40-tutorials.md) | Tutorials & Resources | Training materials, references |

## Topic Index

- **Area per lipid** → ch16
- **AWH** → ch29, ch39
- **BAR** → ch21, ch32
- **Berendsen** → ch20
- **Biphasic systems** → ch16
- **Box types** → ch19
- **CGenFF / Ligand parametrization** → ch08
- **CHARMM** → ch8, ch26
- **Checkpoint (.cpt)** → ch6, ch27
- **Chinese terminology / 中文术语** → all 40 chapters
- **Chinese theory / 中文理论** → ch17-ch32, ch39 (comprehensive theory chapters)
- **中文关键词** → 参见各章中文术语对照表
- **CGenFF penalty scores** → ch08
- **CMake** → ch2
- **Colvars** → ch30
- **COM motion removal (membrane)** → ch16
- **Constraints (LINCS/SHAKE/SETTLE)** → ch20, ch9
- **CUDA** → ch2, ch11
- **Cutoffs** → ch23, ch9
- **Decoupling (vdW, Coulomb)** → ch21
- **Deuterium order parameters** → ch16
- **Domain decomposition** → ch22
- **Double precision** → ch18
- **Energy minimization** → ch21, ch9
- **Error diagnosis** → ch12
- **Ethanol hydration FE** → ch21
- **Expanded ensemble** → ch21, ch39
- **FAQs** → ch7
- **File formats** → ch27
- **Force field modification** → ch08, ch16
- **Force fields (AMBER/CHARMM/GROMOS/OPLS)** → ch8, ch25
- **Free energy (FEP/TI/BAR)** → ch21, ch25, ch28, ch39
- **GPU acceleration** → ch11, ch2
- **grompp** → ch13
- **HMR (mass repartitioning)** → ch25
- **Hydrogen bonds** → ch32
- **InflateGRO** → ch16
- **insert-molecules** → ch16
- **Installation** → ch2
- **Integrators (md/sd/bd/steep/cg)** → ch9, ch19
- **Ions** → ch5, ch16
- **Lambda schedule** → ch21
- **LINCS** → ch20
- **Lipid phase transition (Tm)** → ch16
- **LJ-PME** → ch25
- **MDP parameters** → ch9, ch39
- **mdrun** → ch10, ch11, ch13
- **Membranes** → ch16
- **Methane FE tutorial** → ch21
- **MPI** → ch2, ch22
- **MTS (Multiple Time Stepping)** → ch9, ch11
- **Neighbor list** → ch19, ch23
- **Neural Network Potentials** → ch30
- **Nose-Hoover** → ch20
- **NPT** → ch20
- **NVT** → ch20
- **OpenCL** → ch2, ch11
- **OpenMP** → ch22
- **Parrinello-Rahman** → ch20
- **pdb2gmx** → ch13
- **Performance tuning** → ch11
- **PLUMED** → ch30
- **PME** → ch25, ch9, ch19
- **PMF (potential of mean force)** → ch28
- **Protein-ligand complex** → ch08, ch05
- **Pull code** → ch28, ch39
- **QM/MM** → ch30
- **Reaction coordinate** → ch28
- **Replica exchange** → ch21
- **Reproducibility** → ch6, ch10
- **RMSD/RMSF** → ch32
- **Semi-isotropic coupling** → ch16, ch20
- **Soft-core potentials** → ch25, ch21
- **Solvation** → ch16
- **System preparation** → ch05 (includes full Lysozyme workflow)
- **Temperature coupling** → ch20, ch9
- **Thermodynamic Integration** → ch21
- **Timestep** → ch9, ch25
- **Topology (.top/.itp)** → ch26
- **Trajectory (.xtc/.trr)** → ch27
- **tune_pme** → ch11
- **Tutorials** → ch40
- **Umbrella sampling** → ch28 (includes full 4-step workflow)
- **Validation** → ch15
- **Velocity Verlet** → ch19
- **Virtual sites** → ch25 (includes CO₂ construction example)
- **V-rescale** → ch20
- **Water deletor** → ch16
- **WHAM** → ch28

## Supporting Files

- [glossary.md](glossary.md) — all key GROMACS terms (~130 terms, with tutorial + Chinese additions)
- [patterns.md](patterns.md) — all workflows and design patterns (20+ patterns, including tutorial workflows)
- [cheatsheet.md](cheatsheet.md) — quick reference: decision rules, tutorial commands, analysis commands, troubleshooting

---

## Scope & Limits

This skill covers the GROMACS 2026.2 manual content, enriched with:
- **Justin A. Lemkul's practical tutorials** (CC-BY 4.0) — workflow examples integrated into relevant chapters (ch05, ch08, ch16, ch21, ch25, ch28, ch32)
- **Chinese manual** (GROMACS 2019.6 中文译版) — comprehensive Chinese terminology, theoretical exposition, formulas, and concepts integrated into all 40 chapters

For hands-on implementation, combine with actual GROMACS command execution (Bash). For force field-specific details beyond what's covered, consult the force field's original publications. Tutorial content from mdtutorials.com is provided under CC-BY 4.0 by Justin A. Lemkul. For topics beyond this skill, check the GROMACS website (https://www.gromacs.org) or the GROMACS user forum (https://gromacs.bioexcel.eu).
