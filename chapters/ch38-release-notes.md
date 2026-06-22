# Chapter 38: Release Notes

## Core Idea
GROMACS 2026.2 is a patch release focusing on bug fixes, platform compatibility improvements, and performance enhancements. Release notes are available online at http://manual.gromacs.org/current/release-notes/index.html.

## Key Changes in 2026.2
- Bug fixes for expanded ensemble checkpointing
- Platform support: GCC 12-14 on POWER9, oneAPI 2024.x compatibility
- Performance improvements for GPU-resident mode
- SVE optimization refinements for ARM platforms
- SYCL backend updates for Intel oneAPI
- CMake improvements for Windows and exotic configurations

## Release History
- **2026.2**: Bug fixes, platform compatibility (May 2026)
- **2026.1**: Previous patch release
- **2026**: Major release with new features

## Known Limitations
- See Ch 3 for known issues affecting 2026.2
- Expanded ensemble checkpoint issue
- SVE performance regression with LLVM 20
- NbnxmTest crash with oneAPI 2024.1

## Key Takeaways
1. 2026.2 is a bug-fix release — no major new features
2. Check known issues before upgrading
3. GPU-resident mode has improved performance
4. Online release notes have full changelog

## Connects To
- **Ch 3**: Known issues
- **Ch 15**: Validation status
