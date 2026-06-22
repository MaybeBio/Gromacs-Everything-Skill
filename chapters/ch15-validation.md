# Chapter 15: Validation

## Core Idea
GROMACS features are categorized by validation status: validated, experimental, or pending validation. Understanding these categories helps assess simulation reliability.

## Key Concepts
- **Validated features**: Tested extensively, suitable for production use
- **Experimental features**: Implemented but not fully validated — use with caution
- **Features with validation pending**: New features awaiting systematic testing

## Experimental Features (2026.2)
- Expanded ensemble checkpointing (known issue)
- MiMiC QM/MM coupling
- Neural Network Potentials (NNP)
- Fast Multipole Method (FMM)
- Multiple Time Stepping (MTS)

## Features with Validation Pending
- GPU FFT with vkFFT for some backends
- SYCL backend on some platforms
- New NNP architectures

## Key Takeaways
1. Check validation status before using features in production
2. Experimental features may have known issues — read release notes
3. Report validation results to help the community

## Connects To
- **Ch 3**: Known issues
- **Ch 38**: Release notes
