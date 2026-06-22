---
name: gromacs-manual
description: "Knowledge base from \"GROMACS Documentation Release 2026.2\" by the GROMACS development team. Use when applying GROMACS for molecular dynamics simulations: system preparation, force field selection, .mdp parameter tuning, mdrun performance optimization, free energy calculations, enhanced sampling, trajectory analysis, and troubleshooting."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# GROMACS Documentation Release 2026.2
**Author**: GROMACS development team | **Pages**: ~953 | **Chapters**: 40 | **Generated**: 2026-06-22

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `PME`, `free energy`, `GPU performance`, `pull code`, or any indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch13`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index
- **Troubleshoot** — describe your error or problem; I match it to known issues and solutions

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
6. **Analysis** — `rmsd`, `rmsf`, `gyrate`, `hbond`, `energy`, `cluster`, `rdf`, `msd`

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
- **Alchemical (TI/FEP)**: For binding/solvation free energies. Use soft-core potentials (`sc-alpha=0.5`, `sc-power=1`, `sc-sigma=0.3`), 20+ lambda windows, analyze with BAR/MBAR.
- **Umbrella sampling (pull code)**: For PMF along 1-2 coordinates. Use harmonic restraints, WHAM analysis.
- **AWH**: Adaptive, automatic. Best for 1-3 CVs. No post-processing needed.
- **Expanded ensemble**: For sampling multiple lambda states in one simulation.
- **Replica exchange**: For overcoming energy barriers. Temperature or Hamiltonian exchange.

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

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-downloads.md) | Downloads | Source code, regression tests |
| [ch02](chapters/ch02-installation.md) | Installation Guide | CMake, compilers, FFTW, GPU backends |
| [ch03](chapters/ch03-known-issues.md) | Known Issues | ROCm, GCC POWER9, oneAPI, SVE |
| [ch04](chapters/ch04-getting-started.md) | Getting Started | Flow chart, file types, environment |
| [ch05](chapters/ch05-system-preparation.md) | System Preparation | Steps to consider, tips and tricks |
| [ch06](chapters/ch06-long-simulations.md) | Managing Long Simulations | Checkpoints, reproducibility, restarts |
| [ch07](chapters/ch07-faqs.md) | FAQs | Installation, prep, methodology, analysis |
| [ch08](chapters/ch08-force-fields-overview.md) | Force Fields in GROMACS | AMBER, CHARMM, GROMOS, OPLS |
| [ch09](chapters/ch09-mdp-options.md) | MDP Parameters | Integrators, dt, cutoffs, coupling, constraints |
| [ch10](chapters/ch10-mdrun-features.md) | Useful mdrun Features | Re-run, multi-sim, reproducibility |
| [ch11](chapters/ch11-performance-tuning.md) | Performance Tuning | GPU, MPI, domain decomposition, MTS |
| [ch12](chapters/ch12-common-errors.md) | Common Errors | pdb2gmx, grompp, mdrun diagnostics |
| [ch13](chapters/ch13-command-reference.md) | Command-Line Reference | All gmx tools with flags and usage |
| [ch14](chapters/ch14-terminology.md) | Terminology | MDP option glossary, GROMACS terms |
| [ch15](chapters/ch15-validation.md) | Validation | Experimental features, pending validation |
| [ch16](chapters/ch16-how-to-guides.md) | How-To Guides | Solvation, membranes, parameterization |
| [ch17](chapters/ch17-preface-introduction.md) | Preface & Introduction | Citation, free software, MD intro |
| [ch18](chapters/ch18-definitions-units.md) | Definitions & Units | Notation, MD units, precision |
| [ch19](chapters/ch19-algorithms-pbc.md) | Algorithms: PBC & MD | Box types, neighbor searching, integrators |
| [ch20](chapters/ch20-algorithms-constraints.md) | Algorithms: Constraints & Coupling | LINCS, SHAKE, thermostats, barostats |
| [ch21](chapters/ch21-algorithms-free-energy.md) | Algorithms: Free Energy | TI, FEP, BAR, replica exchange, EM |
| [ch22](chapters/ch22-algorithms-parallel.md) | Algorithms: Parallelization | Domain decomposition, GPU, load balancing |
| [ch23](chapters/ch23-interactions-nonbonded.md) | Non-bonded Interactions | LJ, Buckingham, Coulomb, PME, cutoffs |
| [ch24](chapters/ch24-interactions-bonded.md) | Bonded Interactions | Bonds, angles, dihedrals, restraints |
| [ch25](chapters/ch25-interactions-advanced.md) | Advanced Interactions | Polarization, soft-core, LJ-PME, virtual sites |
| [ch26](chapters/ch26-topologies.md) | Topologies | .top/.itp structure, force field files |
| [ch27](chapters/ch27-file-formats.md) | File Formats | .gro, .pdb, .tpr, .xtc, .edr, .cpt, .ndx |
| [ch28](chapters/ch28-special-free-energy.md) | Special: Free Energy & Pulling | PMF, umbrella sampling, pull code |
| [ch29](chapters/ch29-special-awh.md) | Special: AWH & Enhanced Sampling | AWH, enforced rotation, electrophysiology |
| [ch30](chapters/ch30-special-advanced.md) | Special: Advanced Methods | QM/MM, PLUMED, Colvars, NNP, FMM |
| [ch31](chapters/ch31-run-parameters.md) | Run Parameters & Programs | File types, program workflow |
| [ch32](chapters/ch32-analysis.md) | Analysis | RMSD, RMSF, RDF, MSD, hbond, clustering |
| [ch33](chapters/ch33-implementation.md) | Implementation Details | Virial, optimizations |
| [ch34](chapters/ch34-averages.md) | Averages & Fluctuations | Formulae, implementation |
| [ch35](chapters/ch35-bibliography.md) | Bibliography | Key references |
| [ch36](chapters/ch36-developer-guide.md) | Developer Guide | Contributing, coding standards |
| [ch37](chapters/ch37-doxygen.md) | Doxygen Documentation | API reference |
| [ch38](chapters/ch38-release-notes.md) | Release Notes | 2026.2 changes |
| [ch39](chapters/ch39-mdp-advanced.md) | Advanced MDP Parameters | Free energy, AWH, pull, expanded ensemble |
| [ch40](chapters/ch40-tutorials.md) | Tutorials & Resources | Training materials, references |

## Topic Index

- **AWH** → ch29, ch39
- **BAR** → ch21, ch32
- **Berendsen** → ch20
- **Box types** → ch19
- **CHARMM** → ch8, ch26
- **Checkpoint (.cpt)** → ch6, ch27
- **CMake** → ch2
- **Colvars** → ch30
- **Constraints (LINCS/SHAKE/SETTLE)** → ch20, ch9
- **CUDA** → ch2, ch11
- **Cutoffs** → ch23, ch9
- **Domain decomposition** → ch22
- **Double precision** → ch18
- **Energy minimization** → ch21, ch9
- **Error diagnosis** → ch12
- **Expanded ensemble** → ch21, ch39
- **FAQs** → ch7
- **File formats** → ch27
- **Force fields (AMBER/CHARMM/GROMOS/OPLS)** → ch8, ch25
- **Free energy (FEP/TI)** → ch21, ch25, ch28, ch39
- **GPU acceleration** → ch11, ch2
- **grompp** → ch13
- **Hydrogen bonds** → ch32
- **Installation** → ch2
- **Integrators (md/sd/bd/steep/cg)** → ch9
- **Ions** → ch5, ch16
- **LINCS** → ch20
- **LJ-PME** → ch25
- **MDP parameters** → ch9, ch39
- **mdrun** → ch10, ch11, ch13
- **Membranes** → ch16
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
- **Pull code** → ch28, ch39
- **QM/MM** → ch30
- **Replica exchange** → ch21
- **Reproducibility** → ch6, ch10
- **RMSD/RMSF** → ch32
- **Soft-core** → ch25
- **Solvation** → ch16
- **System preparation** → ch5
- **Temperature coupling** → ch20, ch9
- **Thermodynamic Integration** → ch21
- **Timestep** → ch9
- **Topology (.top/.itp)** → ch26
- **Trajectory (.xtc/.trr)** → ch27
- **tune_pme** → ch11
- **Tutorials** → ch40
- **Umbrella sampling** → ch28
- **Validation** → ch15
- **Velocity Verlet** → ch19
- **Virtual sites** → ch25
- **V-rescale** → ch20

## Supporting Files

- [glossary.md](glossary.md) — all key GROMACS terms with definitions (~100 terms)
- [patterns.md](patterns.md) — all workflows and design patterns (14 patterns)
- [cheatsheet.md](cheatsheet.md) — quick reference: decision rules, defaults, command patterns

---

## Scope & Limits

This skill covers the GROMACS 2026.2 manual content. For hands-on implementation, combine with actual GROMACS command execution (Bash). For force field-specific details beyond what's covered, consult the force field's original publications. For topics beyond this manual, check the GROMACS website (https://www.gromacs.org) or the GROMACS user mailing list.
