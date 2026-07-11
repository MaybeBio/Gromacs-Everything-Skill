# GROMACS Manual Glossary

## A
**AWH** — Accelerated Weight Histogram method; an adaptive biasing technique for free energy calculation along reaction coordinates (Ch 29)

## B
**BAR** — Bennett's Acceptance Ratio; method for calculating free energy differences between states (Ch 21)

**BD** — Brownian Dynamics; integrator where velocity = force/friction + thermal noise, used for implicit solvent (Ch 20)

**Berendsen thermostat** — Weak coupling thermostat that scales velocities toward target temperature; fast but does not produce correct canonical ensemble (Ch 20)

**Berendsen barostat** — Weak coupling pressure control; useful for equilibration, not recommended for production (Ch 20)

## C
**CHARMM** — Chemistry at Harvard Molecular Mechanics; force field family with extensive lipid and protein parameters (Ch 8)

**CG** — Conjugate Gradient energy minimization algorithm; more efficient than steepest descent near minima (Ch 21)

**Colvars** — Collective Variables module for free energy and enhanced sampling calculations (Ch 30)

**cpt file (.cpt)** — Binary checkpoint file containing full-precision coordinates, velocities, and simulation state for exact restarts (Ch 27)

**CUDA** — NVIDIA GPU programming platform; primary GPU acceleration path for GROMACS (Ch 11)

## D
**Domain decomposition** — Spatial parallelization scheme dividing the simulation box into domains assigned to MPI ranks (Ch 22)

## E
**edr file (.edr)** — Energy data file in binary XDR format; stores energy terms over time, read by `gmx energy` (Ch 27)

**Editconf** — `gmx editconf`; converts between structure formats, defines simulation boxes (Ch 16)

**Essential Dynamics** — PCA-based sampling method focusing on collective motions; also called PCA dynamics (Ch 21)

**Expanded Ensemble** — Method sampling multiple states (e.g. lambda values) in a single simulation (Ch 21)

## F
**FEP** — Free Energy Perturbation; computes free energy differences from ensemble averages (Ch 21)

**FFT** — Fast Fourier Transform library required for PME electrostatics; FFTW is recommended (Ch 2)

## G
**Genion** — `gmx genion`; replaces solvent molecules with ions to neutralize or set salt concentration (Ch 16)

**gmx** — GROMACS command-line wrapper; all tools accessed as `gmx <tool>` (Ch 13)

**Grompp** — `gmx grompp`; preprocessor combining .gro, .top, .mdp into .tpr run input file (Ch 13)

**gro file (.gro)** — GROMOS coordinate/velocity file; human-readable fixed-column format (Ch 27)

**GROMOS** — GROMOS force field family; historically tied to GROMACS, uses geometric combination rules (Ch 8)

## H
**HIP** — AMD GPU programming platform; GROMACS supports HIP for AMD GPU acceleration (Ch 11)

## I
**itp file (.itp)** — Include topology file; contains molecular topology fragments (#include'd into .top files) (Ch 26)

## L
**L-BFGS** — Low-memory Broyden-Fletcher-Goldfarb-Shanno quasi-Newton minimizer; often faster than CG but not parallelized (Ch 21)

**Leap-frog** — Default GROMACS MD integrator; staggered update of velocities and positions, numerically stable (Ch 19)

**LINCS** — LINear Constraint Solver; default constraint algorithm for bonds, faster than SHAKE (Ch 20)

**LJ-PME** — Lennard-Jones PME; long-range dispersion correction using PME mesh (Ch 25)

**log file (.log)** — Text log from mdrun containing performance data, cycle counters, and warnings (Ch 27)

## M
**MDP file (.mdp)** — Molecular Dynamics Parameters file; contains all simulation parameters (integrator, dt, nsteps, temperature, pressure, cutoffs, etc.) (Ch 9)

**Mdrun** — `gmx mdrun`; the main MD simulation engine; performs integration and writes trajectories (Ch 13)

**MTS** — Multiple Time Stepping; evaluates PME less frequently than direct-space forces for performance (Ch 9)

**MPI** — Message Passing Interface; used for multi-node parallelization via domain decomposition (Ch 22)

## N
**ndx file (.ndx)** — Index file; defines atom groups for temperature coupling, analysis, restraints, etc. (Ch 27)

**Nose-Hoover** — Canonical thermostat producing correct NVT ensemble via extended system Hamiltonian (Ch 20)

## O
**OpenCL** — Open standard for heterogeneous computing; GROMACS supports OpenCL GPU backend (Ch 11)

**OpenMP** — Shared-memory threading; used for intra-node parallelization in GROMACS (Ch 22)

**OPLS** — Optimized Potentials for Liquid Simulations; force field family using geometric combination rules (Ch 8)

## P
**Parrinello-Rahman** — Isothermal-isobaric barostat producing correct NPT ensemble (Ch 20)

**pdb file (.pdb)** — Protein Data Bank coordinate format; standard input/output for many tools (Ch 27)

**pdb2gmx** — `gmx pdb2gmx`; converts PDB file to GROMACS topology (.top) and structure (.gro) (Ch 13)

**PME** — Particle-Mesh Ewald; efficient method for long-range electrostatics using FFT grid (Ch 25)

**PPPM** — Particle-Particle Particle-Mesh; alternative to PME, not recommended for new simulations (Ch 25)

**Pull code** — Applies external forces to defined groups; used for umbrella sampling, SMD, AFM pulling (Ch 28)

## Q
**QM/MM** — Quantum Mechanics/Molecular Mechanics hybrid simulation; GROMACS interfaces with CP2K or MiMiC (Ch 30)

## R
**Replica Exchange** — Parallel simulation method where replicas at different temperatures/hamiltonians exchange configurations (Ch 21)

**RMSD** — Root Mean Square Deviation; structural comparison metric (Ch 32)

**RMSF** — Root Mean Square Fluctuation; per-atom/ residue flexibility measure (Ch 32)

**RDF** — Radial Distribution Function; pair correlation g(r), used for structural analysis (Ch 32)

## S
**SETTLE** — Analytical constraint algorithm specific to rigid 3-atom water molecules; faster than iterative methods (Ch 20)

**SHAKE** — Iterative constraint algorithm; alternative to LINCS, resets bond lengths via Lagrange multipliers (Ch 20)

**Solvate** — `gmx solvate`; fills simulation box with solvent molecules (Ch 16)

**Steep** — Steepest Descent energy minimization; simple, robust, good for initial relaxation (Ch 21)

**SYCL** — Khronos standard for heterogeneous computing; supports Intel GPUs (Ch 11)

## T
**TI** — Thermodynamic Integration; free energy method integrating ∂H/∂λ along a lambda path (Ch 21)

**tng file (.tng)** — Trajectory Next Generation; compressed, portable trajectory format (Ch 27)

**top file (.top)** — System topology file; lists molecules, force field parameters, includes .itp files (Ch 26)

**tpr file (.tpr)** — Portable run input; binary file containing all simulation parameters and coordinates; sole input needed by mdrun (Ch 27)

**trjconv** — `gmx trjconv`; trajectory conversion and manipulation tool (Ch 13)

**trr file (.trr)** — Full-precision trajectory file with coordinates, velocities, and forces (Ch 27)

## V
**V-rescale** — Velocity rescaling thermostat with stochastic term; produces correct canonical ensemble (Ch 20) 

**Velocity Verlet** — Integrator alternative to leap-frog; produces full-step velocities, better for Nose-Hoover/PR coupling (Ch 19)

## X
**xtc file (.xtc)** — Compressed trajectory format using reduced precision; much smaller than .trr, suitable for analysis (Ch 27)

**xvg file (.xvg)** — Xmgrace plot data format; used for all GROMACS analysis tool output (Ch 27)

---

## Tutorial & Theory Additions

### A-B
**Area per lipid** — Key membrane simulation validation metric; DPPC experimental value ~62–64 Å² (ch16)

**BAR (Bennett Acceptance Ratio)** — Free energy analysis method using energy differences between neighboring lambda states; more efficient than TI, implemented via `gmx bar` (ch21)

**Berger lipids** — United-atom lipid parameters combining GROMOS atom types with OPLS partial charges; Ryckaert-Bellemans dihedral potentials with 1-4 LJ scaling factor 0.125 (ch16)

**CGenFF** — CHARMM General Force Field server; generates ligand topologies with penalty scores (<10: reliable; 10–50: validate; >50: reparametrize) (ch08)

**Decoupling (alchemical)** — Systematic removal of solute-solvent interactions via lambda scaling; discharge first (Coulomb → 0) then decouple vdW (LJ → 0) (ch21)

**Hydrogen bonds (GROMACS criteria)** — Donor-H distance ≤ 0.35 nm and donor-acceptor-H angle ≤ 30° (not donor-H-acceptor) (ch05)

### I-L
**InflateGRO** — Perl script for embedding proteins into lipid bilayers; inflates lipids (scale factor >1), deletes overlapping lipids, then iteratively shrinks (scale factor <1) until area per lipid converges (ch16)

**Insert-molecules (`gmx insert-molecules`)** — Adds molecules to simulation box with steric clash detection; used for non-water solvents where `solvate` is inappropriate (ch16)

**Lambda window** — A discrete value of the coupling parameter λ; 20+ windows (Δλ ≈ 0.05) needed for BAR convergence in free energy calculations (ch21)

**Ligand parametrization** — Process of generating topology/parameters for non-standard residues; requires external tools/servers specific to force field family (ch08)

### M-P
**Membrane packing** — InflateGRO methodology: orient → inflate → delete overlaps → EM → iteratively shrink to target area per lipid (ch16)

**Penalty score (CGenFF)** — Per-parameter quality metric; <10 = reliable for immediate use, 10–50 = validate before use, >50 = manual reparametrization required (ch08)

**Position restraints (ligand)** — Separate `posre_ligand.itp` file for restraining ligand during equilibration; controllable via independent `#ifdef POSRES_LIG` block (ch08)

**Pulling simulation** — SMD-like simulation where a harmonic restraint moves along a reaction coordinate at a defined rate; precursor to umbrella sampling windows (ch28)

### R-Z
**Reaction coordinate** — The degree of freedom along which PMF is calculated (e.g., COM distance between two groups); must be physically meaningful (ch28)

**Soft-core potentials** — Modified LJ/Coulomb functional form preventing singularities at intermediate λ values; parameters: sc-alpha=0.5, sc-power=1, sc-sigma=0.3 (ch25, ch21)

**Umbrella sampling** — Biased MD method: run multiple independent simulations with harmonic restraints at fixed positions along a reaction coordinate; WHAM reconstructs the unbiased PMF (ch28)

**Virtual sites** — Massless interaction centers constructed from real atoms; eliminate high-frequency bond vibrations → enable dt = 4–5 fs with constraints=all-bonds (ch25)

**Water deletor** — Custom Perl script for removing water molecules from the hydrophobic core of lipid bilayers after `gmx solvate` (ch16)

**WHAM (Weighted Histogram Analysis Method)** — Algorithm for reconstructing unbiased PMF from umbrella sampling histograms by deconvolving the known bias potential (ch28)

**蛙跳式算法 (Leap-frog)** — GROMACS默认MD积分器，对位置三阶精度，时间可逆 (ch17/ch19/ch20)

**速度重缩放 (V-rescale)** — 带有随机项的速度重缩放恒温器，产生正确的正则系综 (ch17/ch19/ch20)

**系综 (Ensemble)** — 统计力学术语，NVT(正则)/NPT(等温等压)/NVE(微正则)等 (ch17/ch19/ch20)
