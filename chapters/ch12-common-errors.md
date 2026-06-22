# Chapter 12: Common Errors

## Core Idea
Most GROMACS errors occur during three stages: pdb2gpx (force field/topology issues), grompp (parameter/topology validation), and mdrun (runtime crashes). Each error has a specific diagnostic message and a systematic fix.

## pdb2gmx Errors

### Force Field Not Found
```
No force fields found (files with name 'forcefield.itp' in subdirectories ending on '.ff')
```
- **Cause**: GMXLIB not set or force field directory missing
- **Fix**: `export GMXLIB=/path/to/gromacs/share/top` or use absolute path

### Atom Naming Mismatch
```
Atom ... not found in residue ...
```
- **Cause**: PDB atom names don't match force field .rtp entries
- **Fix**: Edit PDB to use standard atom naming, or create custom .rtp entry

### Missing Residue
```
Residue 'XXX' not found in residue topology database
```
- **Cause**: Non-standard residue (ligand, modified amino acid)
- **Fix**: Create custom .rtp entry or use external parameterization (ACPYPE, CGenFF)

### Terminus Handling
- Use `-ter` flag for interactive terminus selection
- Common issue: N-terminal NH3+ vs NH2, C-terminal COO- vs COOH

## grompp Errors

### Topology Directive Order
```
The directives in the .top and .itp files have rules about the order in which they can appear
```
- **Cause**: Wrong order of `[moleculetype]`, `[atoms]`, `[bonds]` sections
- **Fix**: Follow correct topology directive order

### Missing Include File
```
#include file 'xxx.itp' not found
```
- **Cause**: Include file not in search path
- **Fix**: Check GMXLIB, add `-I` to include parameter

### Non-Zero Total Charge
```
System has non-zero total charge
```
- **Cause**: Unbalanced ions, missing counterions
- **Fix**: Use `gmx genion -neutral` or manually adjust ion count

### Inconsistent Atom Count
```
Number of atoms in topology is not equal to number of atoms in coordinate file
```
- **Cause**: Topology and structure files don't match
- **Fix**: Regenerate topology with current structure, or check for missing/wrong molecules

## mdrun Errors

### LINCS Warnings
```
Step N: Water molecule starting at atom M can not be settled
```
- **Cause**: Bad geometry, overlapping atoms, too large timestep
- **Fix**: Run EM first, reduce dt, check starting structure

### Neighbor List Overflow
```
Neighbor list overflow
```
- **Cause**: Atoms moved too far between neighbor list updates
- **Fix**: Reduce nstlist, increase rlist buffer, set `verlet-buffer-tolerance=-1`

### PME Load Imbalance
- **Cause**: Wrong number of PME ranks
- **Fix**: `gmx tune_pme -s topol.tpr`

### Segfault at Start
- **Cause**: Wrong GMX_SIMD for CPU, incompatible GPU driver, bad build
- **Fix**: Check `gmx mdrun -version`, recompile with correct SIMD

### Domain Decomposition Error
```
There is no domain decomposition for N nodes that is compatible with the given box
```
- **Cause**: Too many MPI ranks for system size
- **Fix**: Reduce MPI ranks, ensure ~500 atoms/core minimum

## Key Takeaways
1. pdb2gmx errors are usually naming/format issues in the input PDB
2. grompp errors are parameter validation — read the error message carefully
3. LINCS warnings at step 0 = bad starting geometry (run EM first)
4. Neighbor list overflow = atoms moving too fast (reduce dt or nstlist)
5. Always check .log file after mdrun for warnings
6. `GMXLIB` is the most common environment variable issue

## Connects To
- **Ch 4**: Getting started workflow
- **Ch 9**: MDP parameters
- **Ch 13**: Command reference
