# Chapter 7: Frequently Asked Questions

## Core Idea
Common GROMACS questions organized by topic: installation, system preparation, simulation methodology, force fields, and analysis. This is the first place to look when encountering a problem.

## Installation FAQs
- **Q: FFTW not found?** Install FFTW3 development package or build with `-DGMX_BUILD_OWN_FFTW=ON`
- **Q: GPU not detected?** Check CUDA toolkit/driver version; verify `-DGMX_GPU=CUDA` was set; run `nvidia-smi`
- **Q: MPI vs thread-MPI?** Thread-MPI is built-in and sufficient for single node; real MPI needed for multi-node
- **Q: How to check if build is correct?** Run `make check` with regression tests downloaded

## System Preparation FAQs
- **Q: Which force field?** AMBER99SB-ILDN for proteins, CHARMM36 for protein-lipid, GROMOS54A7 for united-atom simplicity
- **Q: Which water model?** Must match force field. TIP3P for AMBER/CHARMM, SPC for GROMOS
- **Q: How many ions?** Neutralize first, then add physiological concentration (~0.15 M)
- **Q: PDB has missing residues?** Model them with external tools (Modeller, SWISS-MODEL) before pdb2gmx

## Simulation Methodology FAQs
- **Q: dt=1 fs or 2 fs?** 2 fs with `constraints=h-bonds` is standard. 1 fs only if you need flexible water.
- **Q: EM before MD?** Always. Steepest descent until Fmax < 1000 kJ/mol/nm, then optionally CG/L-BFGS.
- **Q: How long should equilibration be?** 100 ps NVT + 100-500 ps NPT, until density and temperature stabilize.
- **Q: Nose-Hoover or V-rescale?** V-rescale for equilibration, either for production. V-rescale is more robust.

## Force Field FAQs
- **Q: Can I mix force fields?** Generally no. Use consistent force field for all components.
- **Q: Ligand parameters?** Use tools like ACPYPE (Antechamber), CGenFF, or GAFF for small molecules.
- **Q: Non-standard residues?** Use `gmx pdb2gmx -ter` for termini; create custom .rtp entries for modified residues.

## Analysis FAQs
- **Q: RMSD calculation reference?** Use the starting (energy-minimized) structure or crystal structure.
- **Q: How to center trajectory?** `gmx trjconv -center -pbc mol -ur compact`
- **Q: Hydrogen bonds?** Use `gmx hbond` with donor-acceptor cutoff 0.35 nm and angle cutoff 30 degrees.
- **Q: Cluster analysis?** `gmx cluster -method gromos -cutoff 0.2` (0.2 nm for similar conformations).

## Key Takeaways
1. Force field + water model are inseparable — match them
2. Always EM before MD
3. 2 fs timestep with constraints=h-bonds is the production standard
4. V-rescale is safer than Nose-Hoover for equilibration
5. Check the .log file after every mdrun for warnings

## Connects To
- **Ch 2**: Installation details
- **Ch 5**: System preparation
- **Ch 9**: MDP parameters
- **Ch 12**: Common errors
