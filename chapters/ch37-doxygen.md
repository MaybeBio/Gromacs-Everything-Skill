# Chapter 37: Doxygen Documentation

## Core Idea
GROMACS provides Doxygen-generated API documentation for developers. The Doxygen docs cover the internal C++ class hierarchy, module structure, and API reference for all GROMACS components.

## Key Components Documented
- **MD modules**: Integrator, force calculation, constraints, coupling
- **NB (non-bonded)**: Neighbor search, pair interactions, PME
- **GPU backends**: CUDA, OpenCL, SYCL, HIP kernel interfaces
- **Analysis tools**: Trajectory analysis framework
- **File I/O**: Format readers/writers (.tpr, .xtc, .edr, etc.)
- **Utility libraries**: SIMD abstraction, random numbers, selection engine

## Accessing Doxygen Docs
- Online: https://manual.gromacs.org/doxygen/
- Build locally: `cmake -DGMX_BUILD_HELP=ON && make doxygen`
- Module overview: mainpage.h in source tree

## Key Takeaways
1. Doxygen docs are the primary API reference for GROMACS developers
2. Available online at manual.gromacs.org/doxygen
3. Covers all internal modules with class hierarchy and dependencies

## Connects To
- **Ch 36**: Developer guide and contribution workflow
