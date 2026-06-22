# Chapter 36: Developer Guide

## Core Idea
The GROMACS developer guide covers coding standards, contribution workflow, and testing for developers contributing to the GROMACS source code. GROMACS is written in C++17 with extensive use of SIMD intrinsics and GPU programming.

## Key Concepts

### Contribution Workflow
1. Discuss changes on the GROMACS developer mailing list (gmx-developers)
2. Fork the GROMACS GitLab repository
3. Create a feature branch
4. Follow GROMACS coding standards (see Doxygen docs)
5. Submit merge request on GitLab
6. Pass CI (continuous integration) pipeline
7. Code review by core developers

### Coding Standards
- C++17 standard
- clang-format for code formatting
- Extensive use of templates and SIMD abstractions
- GPU code: CUDA, OpenCL, SYCL, HIP backends
- Comprehensive Doxygen documentation

### Testing
- Regression tests: `make check` (requires regressiontests download)
- Unit tests: Google Test framework
- Performance tests: `gmx nonbonded-benchmark`
- CI pipeline: automated testing on multiple platforms

### Build System
- CMake-based with extensive option system
- Separate build for each configuration (CPU, CUDA, HIP, etc.)

## Key Takeaways
1. GROMACS is C++17 with extensive SIMD/GPU abstractions
2. Contributions go through GitLab merge requests
3. All changes require passing CI and code review
4. Regression tests validate correctness across platforms

## Connects To
- **Ch 2**: Build system details
- **Ch 37**: Doxygen API documentation
